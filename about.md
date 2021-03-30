---
layout:    page
title:     About
permalink: /
---

<img src="images/pp.jpeg" class="pp">

Hi! I'm Sankha. I am a Ph.D. student in [Computer Science](https://cs.umd.edu) at the [University of Maryland](https://umd.edu), advised by [Prof. Jeff Foster](http://www.cs.tufts.edu/~jfoster/) and [Prof. David Van Horn](http://www.cs.umd.edu/~dvanhorn/). I work with the folks at the [PLUM](https://plum-umd.github.io/) group.

I am interested in programming languages and formal methods with a focus on foundational, yet practical, techniques that facilitate understanding programs, improve software reliability, and in general, help in building functionally correct software. Currently, I work on building expressive type systems and verification tools for Ruby programs and scaling them to work on large Ruby on Rails web applications.

In the past, I have worked at [BrowserStack](https://www.browserstack.com) where I helped build infrastructure to allow developers to test apps and websites at large scale. I also used to contribute to SpiderMonkey - Mozilla Firefox's JavaScript engine.

---
### News

<ul class="posts">
{% for post in site.posts limit: 3 %}
{% if post.news %}
<li>{{post.content | markdownify | remove: "<p>" | remove: "</p>"}}<span>{{ post.date | date: '%B %d, %Y' }}</span></li>
{% else %}
<li>New post: <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a><span>{{ post.date | date: '%B %d, %Y' }}</span></li>
{% endif %}
{% endfor %}
</ul>

[[More ...](/blog/)]

---
### Publications

[**RbSyn: Type- and Effect-Guided Program Synthesis**](https://arxiv.org/abs/2102.13183).<br>
_Sankha Narayan Guria_, Jeffrey S. Foster and David Van Horn.<br>
[PLDI 2021](https://pldi21.sigplan.org/).<br>
<span class="pubs-subtext">[Preprint](https://arxiv.org/abs/2102.13183) / [Source Code](https://github.com/ngsankha/rbsyn)</span>

[**Type-Level Computations for Ruby Libraries**](https://dl.acm.org/citation.cfm?id=3314630).<br>
Milod Kazerounian, _Sankha Narayan Guria_, Niki Vazou, Jeffrey S. Foster and David Van Horn.<br>
[PLDI 2019](https://pldi19.sigplan.org/).<br>
<span class="pubs-subtext">[Paper (PDF)](/static/comptypes-pldi19.pdf) / [Video](https://www.youtube.com/watch?v=cmK7TzvhEds) / [Extended Version](https://arxiv.org/abs/1904.03521) / [Source Code](https://github.com/tupl-tufts/rdl)</span>

[**Transparent Object Proxies for JavaScript**](http://dx.doi.org/10.4230/LIPIcs.ECOOP.2015.149).<br>
Matthias Keil, _Sankha Narayan Guria_, Andreas Schlegel, Manuel Geffken and Peter Thiemann.<br>
[ECOOP 2015](https://2015.ecoop.org/).<br>
<span class="pubs-subtext">[Paper (PDF)](/static/tproxy-ecoop15.pdf) / [Video](https://www.youtube.com/watch?v=TOjKhi_VZBQ) / [Project Homepage](http://proglang.informatik.uni-freiburg.de/proxy/) / [Artifact](http://dx.doi.org/10.4230/DARTS.1.1.2) / [Source Code](https://github.com/ngsankha/js-tproxy)</span>

---

<i class="about-icon fa fa-envelope"></i> **Email:** [sankha@cs.umd.edu](mailto:sankha@cs.umd.edu)<br>
<i class="about-icon fa fa-twitter"></i> [@ngsankha](https://twitter.com/ngsankha) | <i class="about-icon fa fa-github"></i> [ngsankha](https://github.com/ngsankha)<br>
