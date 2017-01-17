---
layout: post
title: Specializing Bash Programs
---

BrowserStack being an infrastructure heavy company, we need to execute a lot of commands on our systems to get things done. Shell scripts are the easiest way to get this done until at one point you realize that bash is not really a language that you would want to debug your issues in.

Let us imagine a simple scenario:

1. There is a machine A connected to machine B.
2. Machine A needs to send some files and execute a couple of commands in Machine B and the most common way to do that is SCP-ing the files and executing bash commands over SSH. [^1]
3. Some processing happens on machine A depending on the results of commands executed in machine B.
4. Results of the above step decide what will be the next set of commands that need to be executed in machine B.
5. Repeat from step 2, until no more commands are remaining to be executed.

SSH and bash are both great tools. But, anybody who has used SSH a lot will recall random connection hang-ups, thus requiring retries. This brings an inherent degree of unreliability to our setup. In bash, everything you do (mostly) spawns a process and there are no first-class exceptions in the language. Both of these means there is a cost incurred on running time and on debuggability of the program. We didn't want to move away from bash because of the restrictions of the environment we were running in and all we wanted to do was execute other commands on the system anyway.

It was because of this that we saw around higher running time of our scripts and a daily failure rate of around 3%. A large part of the higher running time was because of the repeated back and forth of SSH connection making and retries in-case that failed. [^2] A large chunk of the failures were due to some intermediate SSH connection failing altogether (even after retries) causing all subsequent commands to fail. We thus concluded that the SSH daemon was hanging up in between causing the failures and our best shot would be to minimize the number of SSH calls that we do. This entails moving all of the computation done in machine A in step 3 to machine B along with all the necessary dependant files that are required for them to succeed.

### Idea #1

The first idea was to reduce the multiple points at which the script could break due to so many SSH calls. The primary motivation was if we reduce the SSH calls we reduce our time spent in creating and destroying connections with a higher turnaround time considering retries due to intermediate failures. So we wrote a mega shell script that gets most of the work done once deployed inside machine B at one go. We had to send all the dependency files that the script requires to execute along with it as well. We bundled it all in a compressed tarball and deployed it in the machine B where it would be extracted and run directly inside. This reduced a large amount of the time we spent in making/retrying SSH connections.

### Idea #2

This bit is my personal favourite. Few years ago, I had worked on a project that involved [program specialization in JavaScript](http://link.springer.com/chapter/10.1007/978-3-662-46823-4_26), and I saw this as an opportunity to use the same ideas here. If we put in a compile step that generates the bash script as an output we would be able to do so much more. The core idea is this - in the script there are some static arguments that remain the same through out the runtime, and some dynamic that change during the runtime. If we bake those constants directly in the script we will be saving some runtime costs by preventing them from being recomputed again. So a function on repeated execution would need to execute `jq` everytime to read a JSON file.

Thus we wrote a code generator in Ruby that generates the specialized target bash script reading a special config file tailored towards that specific session. Another area of improvement we got was proper exceptions with stack traces for free (something that I regularly miss in bash). Then whenever we ready the config file we create constants in the Ruby interpreter enviroment with `Kernel.const_set`. Thus if any value went missing the constant would not exist in the Ruby runtime itself and our script would automatically fail to generate the bash script. This would pin point us to exactly which value became `nil` in that session with proper stack traces leading better development experience with faster debug times.

### Outcomes

We deployed both of these ideas combined on production. Our bash code generator generates all the program logic in bash after proper pre-processing. Then the generated huge bash script is bundled up along with all the file dependencies on the fly and the tarball is sent to the machine B for doing the actual processing. The benfits we reaped out of it were:

* **Speed**: The primary motivation to do this. We removed the main bottleneck - the overhead of many SSH calls from machine A to B, getting things done in just 2 calls. This mean we can fail early in case of a failure, there would be no useless retries in the intermediate steps like before, saving us a lot of time. Indeed the numbers reflected this once this was deployed on production - the script execution time went down by 5 seconds.
* **Rate of errors**: The failure rates went down from around 3% to less than 1.5% everyday just because of these improvements and reducing the number of places at which a script could break.
* **Real Exceptions**: Errors with stack traces that actually told us when a value is `nil` or has something else went wrong. This is a much better debugging experience than debugging bash where it just replaces variables as strings in the commands and never throws errors.

<!-- Easter Egg: Users actually noticed things becoming faster because of this work and tweeted about it: https://twitter.com/_JayGeorge/status/817107605944418304 -->

I am a big proponent of compile time errors rather than runtime exceptions and it was super interesting to get us a step closer to that for our bash scripts which are much more error prone. Apart from this, faster session times makes everybody happy!

[^1]: Another obvious way is to use some form of RPC, but we cannot do that due to some other limitations of the program execution environment.
[^2]: SSH control sockets were an option to prevent that - but they also had high failure rates.
