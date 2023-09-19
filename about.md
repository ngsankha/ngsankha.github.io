---
layout:    page
title:     About
permalink: /
---

<img src="images/pp.jpeg" class="pp">

**Assistant Professor, [The University of Kansas](https://ku.edu)**

<i class="about-icon fa fa-map-pin"></i> **[Department of Electrical Engineering and Computer Science](https://eecs.ku.edu/)**<br>
<i class="fa fa-fw"></i> 2034 Eaton Hall

<i class="about-icon fa fa-map-pin"></i> **[Institute for Information Sciences](https://i2s-research.ku.edu/)**<br>
<i class="fa fa-fw"></i> 137 Nichols Hall

<i class="about-icon fa fa-envelope"></i> [sankha@ku.edu](mailto:sankha@ku.edu) or [me@sankhs.com](mailto:me@sankhs.com)<br>
<i class="about-icon fa fa-twitter"></i> [@ngsankha](https://twitter.com/ngsankha) | <i class="about-icon fa fa-github"></i> [ngsankha](https://github.com/ngsankha)<br>

---

I am an Assistant Professor in the [EECS Department](https://eecs.ku.edu) at [The University of Kansas](https://ku.edu). I am interested in practical tools that help programmers build correct and efficient software. My research specifically uses programming language abstractions to design program analysis and synthesis techniques that facilitate automatic construction of functionally correct software.

I got my PhD from the [University of Maryland](https://cs.umd.edu), advised by [Prof. Jeff Foster](http://www.cs.tufts.edu/~jfoster/) and [Prof. David Van Horn](http://www.cs.umd.edu/~dvanhorn/). I have some industry experience working at [Meta](https://meta.com) on the Hack programming language and at [BrowserStack](https://www.browserstack.com) where I helped build cloud-scale testing infrastructure for apps and websites. Previously, I have contributed to [SpiderMonkey](https://spidermonkey.dev/) - Mozilla Firefox's JavaScript engine.

<i class="about-icon fa fa-star"></i>
**I am looking for students interested in programming languages research. Please send me an email if you want to work with me.**

---
### News

<ul class="posts">
{% for post in site.posts limit: 5 %}
{% if post.news %}
<li>{{post.content | markdownify | remove: "<p>" | remove: "</p>"}}<span>{{ post.date | date: '%B %d, %Y' }}</span></li>
{% else %}
<li>New post: <a href="{{ post.url | prepend: site.baseurl }}">{{ post.title }}</a><span>{{ post.date | date: '%B %d, %Y' }}</span></li>
{% endif %}
{% endfor %}
</ul>

[[More ...](/news/)]

---
### Publications

[**Absynthe: Abstract Interpretation-Guided Synthesis**](/static/absynthe-pldi23.pdf).<br>
_Sankha Narayan Guria_, Jeffrey S. Foster and David Van Horn.<br>
[PLDI 2023](https://pldi23.sigplan.org/).<br>
<span class="pubs-subtext">
[ACM](https://dl.acm.org/doi/10.1145/3591285) /
[Preprint](https://arxiv.org/abs/2302.13145) /
[Source Code](https://github.com/ngsankha/absynthe)
</span>

[**ANOSY: Approximated Knowledge Synthesis with Refinement Types for Declassification**](/static/anosy-pldi22.pdf).<br>
_Sankha Narayan Guria_, Niki Vazou, Marco Guarnieri and James Parker.<br>
[PLDI 2022](https://pldi22.sigplan.org/).<br>
<span class="pubs-subtext">
[ACM](https://dl.acm.org/doi/abs/10.1145/3519939.3523725) /
[Preprint](https://arxiv.org/abs/2203.12069) /
[Source Code](https://github.com/ngsankha/anosy) /
[Talk](https://www.youtube.com/watch?v=Xwo3rTcl0Lo)
</span>

[**RbSyn: Type- and Effect-Guided Program Synthesis**](/static/rbsyn-pldi21.pdf).<br>
_Sankha Narayan Guria_, Jeffrey S. Foster and David Van Horn.<br>
[PLDI 2021](https://pldi21.sigplan.org/).<br>
<span class="pubs-subtext">
[ACM](https://dl.acm.org/doi/abs/10.1145/3453483.3454048) /
[Extended Version](https://arxiv.org/abs/2102.13183) /
[Source Code](https://github.com/ngsankha/rbsyn) /
[Talk](https://www.pldi21.org/poster_pldi.124.html)
</span>

[**Type-Level Computations for Ruby Libraries**](/static/comptypes-pldi19.pdf).<br>
Milod Kazerounian, _Sankha Narayan Guria_, Niki Vazou, Jeffrey S. Foster and David Van Horn.<br>
[PLDI 2019](https://pldi19.sigplan.org/).<br>
<span class="pubs-subtext">
[ACM](https://dl.acm.org/doi/10.1145/3314221.3314630) /
[Video](https://www.youtube.com/watch?v=cmK7TzvhEds) /
[Extended Version](https://arxiv.org/abs/1904.03521) /
[Source Code](https://github.com/tupl-tufts/rdl)
</span>

[**Transparent Object Proxies for JavaScript**](/static/tproxy-ecoop15.pdf).<br>
Matthias Keil, _Sankha Narayan Guria_, Andreas Schlegel, Manuel Geffken and Peter Thiemann.<br>
[ECOOP 2015](https://2015.ecoop.org/).<br>
<span class="pubs-subtext">
[LIPICS](http://dx.doi.org/10.4230/LIPIcs.ECOOP.2015.149) /
[Video](https://www.youtube.com/watch?v=TOjKhi_VZBQ) /
[Project Homepage](http://proglang.informatik.uni-freiburg.de/proxy/) /
[Artifact](http://dx.doi.org/10.4230/DARTS.1.1.2) /
[Source Code](https://github.com/ngsankha/js-tproxy)
</span>

---
### Teaching

**Introduction to Program Synthesis**<br>
[Fall 2023](/eecs700)
