---
layout: post
title: A Brainf*ck JIT
---

##### _This article was originally published on my [previous blog](https://thelimbeck.wordpress.com/2013/12/31/a-brainfck-jit/) in December 2013._

I recently came across an [article](http://blog.reverberate.org/2012/12/hello-jit-world-joy-of-simple-jits.html) by Josh Haberman on the web which demonstrates how to make simple fun JITs, with the help of LuaJIT. I am still a beginner in the field of compilers and JITs. So I tried to implement a [Brainf*ck](http://en.wikipedia.org/wiki/Brainfuck) JIT with the help of [GNU LibJIT](http://www.gnu.org/software/libjit/) library. It proved to be a really entertaining exercise. The results are up [here](https://github.com/sankha93/bf-jit) on Github.

The fun is in the part where we directly emit the machine instructions while reading source. The JIT doesn’t make any optimizations to the code, it just directly converts to the machine instructions.

[Wikipedia](http://en.wikipedia.org/wiki/Brainfuck#Commands) tells us what each of the BF instructions correspond to in C. This makes it easy to write the corresponding instructions for the JIT. The entire BF program is defined as single function that when once compiled will run. I’ll take up sections of the code from [`bf-jit.c`](https://github.com/sankha93/bf-jit/blob/5bd1fe1405eee8b804316b372173b0c6489fbf75/src/bf-jit.c#L47) and lookup what they do:

{% highlight c %}
case '>':
    tmp = jit_insn_add(function, ptr, uptr);
    jit_insn_store(function, ptr, tmp);
{% endhighlight %}

The `>` instruction increments the current pointer by an unit address value. The `uptr` value is defined to have a pointer of unit byte length. Once we do the add, we store the new value back to the `ptr` (since, `++ptr` is equivalent to `ptr = ptr + 1`).

{% highlight c %}
case '+':
    tmp = jit_insn_load_relative(function, ptr, 0, jit_type_ubyte);
    tmp = jit_insn_add(function, tmp, ubyte);
    tmp = jit_insn_convert(function, tmp, jit_type_ubyte, 0);
    jit_insn_store_relative(function, ptr, 0, tmp);
{% endhighlight %}

Similarly the `+` instruction loads the values from the address stored in `ptr`. We add one to the byte value and type cast it to ensure that it is still a byte value. We then store the result into the address contained in `ptr`.

{% highlight c %}
case '.':
    tmp = jit_insn_load_relative(function, ptr, 0, jit_type_ubyte);
    jit_insn_call_native(function, "putchar", putchar, putchar_sig, &tmp, 1, JIT_CALL_NOTHROW);
{% endhighlight %}

Here the `.` instruction prints the character pointed to by the `ptr`. Then we call a native C function, `putchar` that prints the character on the screen.

Making the loop constructs are the most interesting part. We defined a struct that has the labels to the start and end of the loop in the program. It also has a pointer to the parent loop, so that we can return to the parent loop.

{% highlight c %}
case '[':
    loop_start(function, &loop);
    tmp = jit_insn_load_relative(function, ptr, 0, jit_type_ubyte);
    jit_insn_branch_if_not(function, tmp, &loop->stop);
{% endhighlight %}

The condition the loop is the value pointed to by the `ptr`. We load that value and then use to decide whether we should branch on the loop, depending on whether it is `0` or not.

We then compile the JIT function and then call the function. The results of the benchmark by running the Mandelbrot code were really interesting:

{% highlight text %}
Implementation                      Time
----------------------------------------
Mandelbrot (C Unoptimized)       22.343s
Mandelbrot (C Optimized)          1.241s
Mandelbrot (BF Interpreter)       8.304s
Mandelbrot (BF JIT)               3.918s
{% endhighlight %}

The unoptimized JIT is 2x faster that the optimizing interpreter. Though we are nowhere close to the optimized C code, but the gap can be narrowed by making some optimizations to the generated code.

Future work can be done in constructing an IR of the source and making an optimization pass, for example `++++` can be converted to `+4` directly then, instead of `+1` instruction 4 times.

Certain references that really helped are:

* [http://blog.reverberate.org/2012/12/hello-jit-world-joy-of-simple-jits.html](http://blog.reverberate.org/2012/12/hello-jit-world-joy-of-simple-jits.html)
* [http://pwparchive.wordpress.com/2013/01/20/a-jit-compiler-for-brainfck/](http://pwparchive.wordpress.com/2013/01/20/a-jit-compiler-for-brainfck/)
* [http://www.gnu.org/software/dotgnu/libjit-doc/libjit.html](http://www.gnu.org/software/dotgnu/libjit-doc/libjit.html)
* [http://eli.thegreenplace.net/2013/10/17/getting-started-with-libjit-part-1/](http://eli.thegreenplace.net/2013/10/17/getting-started-with-libjit-part-1/)

---

### _Update:_ Optimizing JIT

I made a [few optimizations](https://github.com/sankha93/bf-jit/commit/26acc8b07bebf58e17b1674221dd8ebdb6f3049a) to fuse the multiple increment/decrement and pointer movement operations into a single one. The benchmarks from my JIT is now much closer to that of the optimized C version. The latest benchmark results are:

{% highlight text %}
Implementation                      Time
----------------------------------------
Mandelbrot (C Unoptimized)       22.343s
Mandelbrot (C Optimized)          1.241s
Mandelbrot (BF Interpreter)       8.304s
Mandelbrot (BF JIT)               1.609s
{% endhighlight %}

Maybe I should look at implementing [these optimizations](http://nayuki.eigenstate.org/page/optimizing-brainfuck-compiler) next.
