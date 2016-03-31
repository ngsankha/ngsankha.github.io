---
layout: post
title: Malloc and RSS memory usage
---

We run a service in production that routes network data through it, it's very similar to a proxy but with its own quirks. The service is written in Node.js, and we noticed a very strange problem - if some user downloads very large amount of data through it, the memory would shoot up and the process would crash sometimes.

The first intuition was that we have a memory leak in our code, and after making improvements from our end, the situation didn't improve much. I decided to write a bare minimal testcase that emulates our code. This sample downloads a huge file (in this case the Ubuntu ISO image file) whenever you shoot a HTTP request to `localhost:3001` and also prints the [RSS memory usage](https://en.wikipedia.org/wiki/Resident_set_size) of process every second. We write the chunk as soon as we receive it, this code shouldn't have any memory leaks as there are no dangling references anywhere.

{% highlight javascript %}
var http = require('http');
const HUGE_DOWNLOAD_URL = 'http://releases.ubuntu.com/14.04.4/ubuntu-14.04.4-desktop-amd64.iso';

setInterval(() => console.log(process.memoryUsage().rss / 1000000), 1000);

var server = http.createServer((req, res) => {
  http.get(HUGE_DOWNLOAD_URL, (downloadRes) => {
    downloadRes.on('data', (chunk) => res.write(chunk));
    downloadRes.on('end', res.end);
  });
});

server.listen(3001);
{% endhighlight %}

But as expected (really?) the RSS memory of the process grew as it kept on downloading the file. There can be times where the input stream is producing data at a faster rate than we can write to the output stream, in which case the `res.write` would queue the chunks in user memory. But such chunks would be cleared when the download is complete and the GC has run already. I even tried running Node with `--expose-gc` flag and using `global.gc()` to explicitly call the garbage collector, but still the RSS memory was high.

Ok, time to dig deeper.

Here as we can see `chunk` is the part of the data the flows into our application as we are downloading the file. That is the part which can actually grow our memory. This `chunk` is a Node `Buffer` object, it is a chunk of raw memory buffer. I decided to check the implementation of Node `Buffer` and saw [`node_buffer.cc`](https://github.com/nodejs/node/blob/bb28770aa1fb541b5a7cc0745b17ca4881bfade6/src/node_buffer.cc) calls `BUFFER_MALLOC` to [allocate](https://github.com/nodejs/node/blob/bb28770aa1fb541b5a7cc0745b17ca4881bfade6/src/node_buffer.cc#L227) [memory](https://github.com/nodejs/node/blob/bb28770aa1fb541b5a7cc0745b17ca4881bfade6/src/node_buffer.cc#L273). `BUFFER_MALLOC` is [defined](https://github.com/nodejs/node/blob/bb28770aa1fb541b5a7cc0745b17ca4881bfade6/src/node_buffer.cc#L51) as call to `malloc` or `calloc` as the case maybe. The [`free`](https://github.com/nodejs/node/blob/bb28770aa1fb541b5a7cc0745b17ca4881bfade6/src/node_buffer.cc#L117) function was also being called to free up the data blocks that was reserved by `malloc`. So this wasn't a memory leak on Node.js' side as well.

A bit more intensive googling threw me to [results](http://stackoverflow.com/questions/2215259/will-malloc-implementations-return-free-ed-memory-back-to-the-system) that state calling `malloc` and then `free` doesn't really return the memory back to the system, it keeps it reserved for the process - and only returns it back to the system only in certain circumstances. So repeatedly `malloc`-ing could be a source of the problem. Since in this example we create a lot of chunks from the file download, we are actually creating a lot of `Buffer`s.

With this in mind, I delved into Google black magic only to dig up some [issue](https://github.com/nodejs/node-v0.x-archive/issues/4217) from the old Node archive repository, with a comment from Ben Noordhuis. Ben [states explicitly](https://github.com/nodejs/node-v0.x-archive/issues/4217#issuecomment-9926227) buffering up a lot of data can explain a growing RSS memory. He even made a nice little [sample C file](https://gist.github.com/bnoordhuis/3983624) to demonstrate the malloc implementations are very reluctant at returning free memory to the system. This shouldn't just affect Node.js, but also any other program that uses `malloc` to allocate memory.

Thus the root cause was that even though memory was freed, the Linux had kept the memory reserved for the process, causing an increasing RSS. As it turns out RSS is a very bad metric for measuring memory leaks of a program and should be avoided if possible. As for our problem, we just ended up piping the input stream to the output stream - that helped in getting the memory growth under control.

{% highlight javascript %}
var server = http.createServer((req, res) => {
  http.get(HUGE_DOWNLOAD_URL, (downloadRes) => downloadRes.pipe(res));
});
{% endhighlight %}

