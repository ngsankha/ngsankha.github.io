---
layout: post
title: Webdriver over Websockets
---

JSONWire is a pure HTTP based protocol that uses discrete HTTP requests to automate the web browser or mobile devices. So for example, selecting an element would be one request, typing something would be another, clicking on a button be yet another request and so on. HTTP is a great protocal and can be used almost everywhere, but there are a number of shortcomings that we observe when we run a huge number of Webdriver based tests on the cloud at BrowserStack.

### The Problem

Every interaction with the web browser via Webdriver requires a HTTP request. It involves opening a connection to the server, sending the request, getting the response and then closing the request. Thus we incur a cost on opening and closing the TCP connection for every request that we make.
<div class="shaky-container"><pre class="shaky">

               |           |
         open*-+->         |
               |           |
               |  request  |
               | *-------> |
               | <-------* |
               |  response |
               |           |
        close<-+-*         |
client         |           |  server
               |           |
         open*-+->         |
               |           |
               |  request  |
               | *-------> |
               | <-------* |
               |  response |
               |           |
        close<-+-*         |
               |           |

</pre></div>

The situation improves when keep alive is enabled for HTTP requests. It allows the client to send subsequent requests on the same connection without the open and close steps in between. This saves a significant amount of time for each request. But sometimes the client needs to do some processing on its end, and if the keep alive timeout set by the server expires, it results in opening of a new connection over again.

Essentially, this manifests as increased duration for the test to run since each operation has extra latency involved in the process.

### Finding a way

Websockets are a relatively new thing, but it fits the requirements of the Webdriver protocol better. It allows full-duplex communication over a single TCP connection. Thus the opening and closing of the connection is done only at the start and end of the session. It also allows for bi-directional passing of messages between the client and server, which makes it suitable to push commands to the browser and get the results back in realtime, when the tests are running remotely over the internet.

<div class="shaky-container"><pre class="shaky">

                 |           |
 websocket open*-+->         |
                 |           |
                 | messages  |
      client     | *-------> |  server
                 | <-------* |
                 |           |
websocket close<-+-*         |
                 |           |

</pre></div>

To test this idea, I wrote a thin proxy that translates between the Websocket and HTTP protocol. The idea is that this proxy will be present on the machine that will actually run the browser that be automated via Webdriver. The Webdriver client libraries will connect to the proxy over Websockets to send the commands. The proxy will locally connect to the Selenium JAR and send in the requests as raw HTTP. Since these HTTP requests happen on the same machine, the latency is minimal. Of course, we would need a custom Webdriver library that can speak the Websockets protocol.

The source code for the same can be found in [ws-webdriver](https://github.com/sankha93/ws-webdriver) repository on GitHub.

### Benchmarks

To see how much difference would Websockets make over traditional HTTP I decided to benchmark a test suite. I forked [WebdriverIO](http://webdriver.io) client library and patched it to support the Websockets protocol for my proxy. The [fork](https://github.com/sankha93/webdriverio) can be found on my GitHub. Since the real difference shows when tests are run over the internet, I created a [ngrok](https://ngrok.com/) tunnel to my machine. Then I run the WebdriverIO test suite with the remote URL of the ngrok tunnel. There was a remarkable difference in the test running duration.

{% highlight text %}
Webdriver HTTP requests:         9 min 31.68 sec
Webdriver over Websockets:       5 min 55.63 sec
{% endhighlight %}

This chips off quite a bit of time from long running Webdriver tests and since Websockets is an open protocol we can write out own client libraries to send the Webdriver commands via it. Contributions are always welcome!
<script type="text/javascript" src="/js/shaky.dart.js">
