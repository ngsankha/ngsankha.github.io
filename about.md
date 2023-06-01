---
layout:    page
title:     About
permalink: /
---

<img src="images/pp.jpeg" class="pp">

Hi! I'm Sankha. I am a Ph.D. student in [Computer Science](https://cs.umd.edu) at the [University of Maryland](https://umd.edu), advised by [Prof. Jeff Foster](http://www.cs.tufts.edu/~jfoster/) and [Prof. David Van Horn](http://www.cs.umd.edu/~dvanhorn/). I work with the folks at the [PLUM](https://plum-umd.github.io/) group.

I am interested in tools that help programmers build correct and efficient software. My research specifically focuses on program synthesis to facilitate automatic construction of functionally correct software from specifications like tests and types. For instance, my tool [RbSyn](https://github.com/ngsankha/rbsyn) demonstrates the approach by automatically synthesizing methods for Ruby on Rails apps.

<!-- synthesis, i.e., automation that writes code, and design foundational approaches to programming language abstractions, program analysis and synthesis. -->

Previously, I used to work at [BrowserStack](https://www.browserstack.com) where I helped build large scale testing infrastructure for apps and websites. I also used to contribute to [SpiderMonkey](https://spidermonkey.dev/) - Mozilla Firefox's JavaScript engine.

<i class="fa fa-star"></i>
**I am joining the [University of Kansas EECS Department](https://eecs.ku.edu/) as an Assistant Professor in Fall 2023. If you are interested in working with me as a PhD student, feel free to send me an email!**

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
[Preprint](https://arxiv.org/abs/2302.13145)
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

<i class="about-icon fa fa-envelope"></i> **Email:** [sankha@cs.umd.edu](mailto:sankha@cs.umd.edu)<br>
<i class="about-icon fa fa-twitter"></i> [@ngsankha](https://twitter.com/ngsankha) | <i class="about-icon fa fa-github"></i> [ngsankha](https://github.com/ngsankha)<br>
