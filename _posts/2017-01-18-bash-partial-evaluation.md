---
layout: post
title: Partial Evaluation of Bash Scripts
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

Bash scripts are everywhere, they are used to run scripts, an easy way to execute other programs following some logical order. The portability of the bash shell means that they can be run anywhere - ranging from desktop OS (Linux, Mac and now Windows) to mobile devices (Android and iOS). This is not without problems though - using Bash means you open yourself a can of worms. All the variables are string-ly typed, all errors are ignored unless checked explicitly, undefined variables are treated as empty variables. Errors like these can be critical in a production pipeline and debugging these can be a nightmare.

### The Problem

I will take the example of how partial evaluation as a technique was useful in generating functionally correct and error free bash programs in context of a real problem we solved. BrowserStack provides real mobile devices (both Androids and iPhones) for access to the users on the cloud. These mobile phones run in the data centers connected to a host that is responsible for talking to these phones. In these scenarios, if you have to execute some commands inside the device, your easiest way out is to have a Bash script. Bash shell runs in all Android and iOS devices and is lightweight enough, so as not to slow down the phone from performing other tasks.

Whenever a user selects a device to run, the normal flow of execution is as follows:

1. The host needs to send some files and execute a couple of commands in the mobile phone. The easiest way out of that is to SCP the files and execute the bash commands inside the device via SSH. [^1]
2. After the commands run in the mobile phone, they come over the wire to the host machine. The host machine continues its execution depending on the response.
3. This in turn decides the next set of commands to be run in the mobile phone. This back and forth operation between them continues in order to complete the session setup.

This is a very basic form of remote-procedure-call, except using bash limits us to using primitive tools - like string matching and SSH as the protocol. But anybody who has used SSH a lot can recall random connection hang-ups, and consequent retries. This brings an inherent degree of unreliability to the setup. Every command that you execute in bash spawns a new process and the lack of first-class exceptions imply a cost incurred on running time and on the debuggability of the program. But moving away from bash wasn't a lucrative option either, because of the range multiple target CPU architectures of these mobile devices and performance requirements. Any one SSH connection error in the chain would mean either your session would fail or it would start slow (in case SSH retries succeed).

### Minimize SSH

What if, we could reduce the multiple points at which the script can break due to so many SSH calls? The primary motivation was if we reduce the SSH calls, we reduce our time spent in creating and destroying connections considering retry timeouts due to intermediate failures. So we wrote one mega shell script that has the meat of the program and all the required dependencies, deployed and run it in the mobile device in one go. This reduced a large amount of the time we spent in making/retrying SSH connections.

### Partial Evaluation

This large bash script had a lot of weird things. It would read configuration from JSON files over and over again using `jq`. This is important because every mobile device has their own manufacturer specific configuration. There were a lot of computations that would run on the fly over and over again, we were still troubled by bash variables becoming empty in case there was something wrong in the environment. We should be able to know if the variable environment was messed up or something could be computed ahead of time specifically for that session on that mobile device. This is where [partial evaluation](https://en.wikipedia.org/wiki/Partial_evaluation) of bash programs came in handy. A computer program $$prog$$ can be seen as a mapping of input data into output data:

$$prog: I_{static} \times I_{dynamic} \rightarrow O$$

where $$I_{static}$$ is the static data that can is known ahead of time before the bash script is run. Thus we can have a partial evaluator that can transform $$\langle prog, I_{static} \rangle$$ into a new program $$prog*: I_{dynamic} \rightarrow O$$. This new bash script $$prog*$$ can run faster and be error free because the partial evaluator can raise errors and optimize during the "compile" phase.

Thus we wrote a new bash script generator in Ruby, that generates a specialized script for that user session on that device. These factors are known ahead of time before program execution and can be baked into the generated bash script. We took the dynamic nature of Ruby to create variables inside the Ruby environment corresponding to the bash variables using `Kernel.const_set`.

{% highlight ruby %}
# validate the resolution
if (!vars[:resolution].nil?) && (vars[:resolution].include? 'x')
  Kernel.const_set("RESOLUTION", vars[:resolution])
end

# further bash code generator can rely on the `RESOLUTION` Ruby variable
{% endhighlight %}

This means any failure in evaluation of the script in the Ruby environment due to erroneous validation would raise exceptions (with real stack traces, yay!). This would pin point us to exactly which value became `nil` in that session with proper stack traces leading better development experience with faster debug times.

### Outcomes

We deployed both of these ideas combined on production. Our bash code generator generates all the program logic in bash after partial evaluation. Then the generated bash script is bundled up along with all the file dependencies on the fly as a tarball which is shipped to the mobile device for execution. The benefits we reaped out of it were:


* **Speed:** We removed the main bottleneck - the overhead of multiple SSH calls from the host machine to the mobile devices. This mean we can fail early in case of a failure, there would be no useless retries in the intermediate steps like before, saving us a lot of time. Indeed the numbers reflected this once this was deployed on production - the script execution time went down by 5 seconds.
* **Rate of errors:** The failure rates went down from around 3% to less than 1.5% everyday just because of these improvements and reducing the number of places at which a script could break.
* **Real Exceptions:** Errors with stack traces that actually told us when a value is `nil` or invalid value or if something else went wrong. This is a much better debugging experience than debugging bash where it just replaces variables as strings in the commands and never throws errors.

I am a big proponent of compile time errors rather than runtime corrupt execution and it was super interesting to get our bash scripts a step closer to that. Apart from this, faster session times makes everybody happy!

<blockquote class="twitter-tweet tw-align-center" data-lang="en"><p lang="en" dir="ltr">It&#39;s crazy that these days it&#39;s faster to fire up iOS in BrowserStack than it is to open the iOS Simulator. Plus BS provides Chrome DevTools</p>&mdash; Jay George (@_JayGeorge) <a href="https://twitter.com/_JayGeorge/status/817107605944418304?ref_src=twsrc%5Etfw">January 5, 2017</a></blockquote>
<script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

[^1]: Androids use the ADB shell and ADB push to send files, but these are functionally similar SSH/SCP so will refer them as the same term for simplicity reasons.
