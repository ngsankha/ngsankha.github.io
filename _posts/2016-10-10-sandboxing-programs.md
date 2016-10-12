---
layout: post
title: Some thoughts on sandboxing programs
---

Few years back I had written an [online judge](https://github.com/sankha93/codejudge) to host contests in college, with my admittedly beginner knowledge of a Linux system. I did a couple of iterations to [improve the judge](https://github.com/sankha93/judgev2) itself, but the core problem of sandboxing any arbitrary code (correctly) remains unsolved and an interesting one. Also, it turns out that even compiling a program is also not safe outside of sandbox either, there are [actual C programs](http://security.stackexchange.com/questions/138881/is-it-dangerous-to-compile-arbitrary-c) whose compilation can bring a the system to a halt - so the entire process from compiling the user submitted code to running the program on the test cases needs to happen in a sandbox. Recently, I have been thinking a lot about how to sandbox programs correctly (not just from the context of an online judge) given any capabilities that you would want to restrict on the program.

The following things are in the context of a Linux system and may not work on a OS X machine. Majority of these methods have a corresponding shell command, which can be triggered to set capabilities for the current session, but they also have a corresponding system call which can be directly called from a C program giving much more flexibility in setting capabilities - after which you can do a [`execve(2)`](https://linux.die.net/man/2/execve) to convert it into the required process with the restricted capabilities.

### A chroot jail

A `chroot` jail changes the apparent root directory of a process and it sub-processes. How this helps is that, the program cannot access files outside the chroot jailed directory and operations that access the parent directory etc. fail. This is helpful for blocking processes from accessing outside files that you have kept out of the jailed environment.

Even then, doing a chroot jail correctly is hard, because you can break out of the jail when running the process as `root` user, by chroot-ing into a directory and changing the current directory to something outside the jail (eg: `../../..`).

{% highlight C %}
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>

int main() {
    mkdir("myjail");
    chroot("myjail");
    chroot("../../../..");
    return system("/bin/bash");
}
{% endhighlight %}

The pitfalls are pretty well documented in the man-page of the [`chroot(2)`](https://linux.die.net/man/2/chroot) system call. A correct implementation of chroot jail changes the user ID of the process to something non-root:

{% highlight C %}
#include <unistd.h>
#include <sys/stat.h>
#include <sys/types.h>

int main() {
    mkdir("myjail");
    chdir("myjail");
    chroot(".");
    setuid(uid); // uid should not be zero for non-root user
    return system("/bin/bash");
}
{% endhighlight %}

This gives us some idea of how to prevent an arbitrary program from getting file access outside of its jail.

### Limit resource usage

The idea is a program should be allotted only limited amount of CPU time and RAM - anything that exceeds the set limits will be killed with a specific signal from the kernel itself. This is where the [`setrlimit(2)`](https://linux.die.net/man/2/setrlimit) call comes in. This also has a corresponding command line utlity called [`ulimit(1)`](https://linux.die.net/man/1/ulimit) that can be used to set different resource usage caps for a process. A look at the man-page of [`setrlimit(2)`](https://linux.die.net/man/2/setrlimit), you will see how `RLIMIT_CPU` and `RLIMIT_RSS` can be used to set the hard and soft limits for a process.

{% highlight C %}
#include <unistd.h>
#include <sys/time.h>
#include <sys/resource.h>

int main() {
    struct rlimit l;
    getrlimit(RLIMIT_CPU, &l);
    l.rlim_cur = 1;
    l.rlim_max = 1;
    setrlimit(RLIMIT_CPU, &l);
    return execve("./test", NULL, NULL); // process to be run with 1 sec CPU time
}
{% endhighlight %}

The `setrlimit(2)` system call allows for controlling of a number of parameters even the maximum number of processes, max file descriptor number, etc. on a child process.

### Restrict network access

If you have the network namespaces enabled in your kernel then you can use the `unshare` command to create a new network namespace which will have a different network interfaces from the host system and by default it would have no network - which means we effectively restricted entire network access of a program. It is as simple as running:

{% highlight bash %}
$ unshare -n ./myprocess
{% endhighlight %}

If we replace `./myprocess` by one of our above programs what we get is effectively a sandboxed program that has no network access and at the same time is bound to get a max CPU time of 1 second.

### seccomp - Secure Computing Mode

This is a simple sandboxing tool provided by the Linux kernel since version 2.6.12, when enabled on a process in the strict mode only allows a handful of system calls like `read()`, `write()`, `exit()`. A whole bunch of calls like `malloc()`, etc. becomes unavailable to the process when this is enabled.

A more recent extension called the Seccomp-BPF from kernel version 3.10 allows filtering of system calls via a filtering program called the [Berkeley Packet Filter (BPF)](https://en.wikipedia.org/wiki/Berkeley_Packet_Filter). It can be used to a allow or deny a family of arbitrary system calls on a target process. Brendan Gregg has an [awesome talk](http://www.brendangregg.com/blog/2016-03-05/linux-bpf-superpowers.html) about BPF programs and how they can be used to filter different system calls or for tracing purposes.

The [`seccomp(2)`](http://man7.org/linux/man-pages/man2/seccomp.2.html) man page describes a lot of the low level details. This is also used in the [Google Chrome](https://chromium.googlesource.com/chromium/src/+/master/docs/linux_sandboxing.md) and Gecko browser engines for sandboxing. Mozilla's [wiki page](https://wiki.mozilla.org/Security/Sandbox/Seccomp) help for a much easier understanding of creating a basic filter or how it is used in Gecko.

### Other ways

Virtualization has been a well known method to run an application process insided a VM with its own OS running on the host. While it is the best way to sandbox something, its probably the most heavyweight of all other approaches. You end up running an OS inside an OS with all hardware virtualized inside it which will be used by the sandboxed process.

Another recent development is that of containers, a lightweight way to isolate an application. This is all based on new kernel features like namespaces and cgroups. Namespace make it appear to a process that they have their own copy of the resource like the network or filesystem. Cgroups can set the bounds on the resources on a group of process like memory or CPU. Remember the `unshare` command showed above? There is a corresponding system call to create a new namespace called `unshare` as well. The system calls [`clone(2)`](http://man7.org/linux/man-pages/man2/clone.2.html), [`setns(2)`](http://man7.org/linux/man-pages/man2/setns.2.html) and [`unshare(2)`](http://man7.org/linux/man-pages/man2/unshare.2.html) forms the namespace API which allows a process to create a new namespace with their own view of the network, the mount points, etc. all mentioned in details in the [`namespace(7)`](http://man7.org/linux/man-pages/man7/namespaces.7.html) man page. What we did in the previous example is equivalent to calling:

{% highlight C %}
#include <unistd.h>
#include <sched.h>

int main() {
    unshare(CLONE_NEWNET); // creates a new network namespace
    return execve("./test", NULL, NULL); // run the process without network access
}
{% endhighlight %}

### Wrapping Up

Turns out sandboxing a userspace process is a non-trivial thing - so the sandbox that I had made for the judge was not really foolproof. Real world applications like Mozilla Firefox and Google Chrome implement multi-level sophisticated sandboxing based on seccomp BPF and user namespaces. I have experimented a bit with these system calls from basic C programs or shell programs for my own understanding, but a full end-to-end implementation needs to take into account a lot of specific details. This is something that I would like to work on in future and integrate it into the fully working judge _(and, I just added another item in an already long to-do list)_.
