---
layout: post
title: Be cautious about Content-Length HTTP headers
---

The `Content-Length` HTTP header is used to inform the server or client of the size of the request or response body that is being transferred. Many times when writing our own HTTP clients/servers, we want to send a string in the body. To compute the `Content-Length` in such a case the easiest way to do it is to compute the string length, but that can lead to strange bugs.

Let's see what the [HTTP/1.1 RFC 2616](https://www.w3.org/Protocols/rfc2616/rfc2616-sec14.html) says about `Content-Length`:

> The Content-Length entity-header field indicates the size of the entity-body, in decimal number of OCTETs, sent to the recipient or, in the case of the HEAD method, the size of the entity-body that would have been sent had the request been a GET.

Thus it is the number of octets (8-bit bytes) that is to be sent in the entity body. Thus string lengths will horribly go wrong in case of UTF-8 strings, where more than a byte if required to represent the unicode characters. So strings in languages other than english or emojis will return wrong `Content-Length` causing our code to break, since it was reading less number of bytes.

For example in Node.js:

{% highlight javascript %}
'অ'.length             // returns 1, wrong
Buffer.byteLength('অ') // returns 3, correct
{% endhighlight %}

The official docs of Node.js for the HTTP module, shows [usage of string length](https://nodejs.org/api/http.html#http_response_writehead_statuscode_statusmessage_headers), but that is not really a safe thing to do. We should always use `Buffer.byteLength` or the corresponding methods in other languages to count the number of bytes.

A reduced test-case to demonstrate how it can lead to wrong data being received and errors happening is given below.

{% gist sankha93/2fb81c491a5006e45530 %}
