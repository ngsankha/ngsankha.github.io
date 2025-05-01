---
layout:    page
title:     About
permalink: /
---

<img src="images/pp.jpeg" class="pp">

**Assistant Professor, [The University of Kansas](https://ku.edu)**

<i class="about-icon fa-solid fa-location-dot"></i> **[Electrical Engineering and Computer Science Department](https://eecs.ku.edu/)**<br>
<i class="fa fa-fw"></i> 2034 Eaton Hall

<i class="about-icon fa-solid fa-location-dot"></i> **[Institute for Information Sciences](https://i2s-research.ku.edu/)**<br>
<i class="fa fa-fw"></i> 137 Nichols Hall

<i class="about-icon fa-solid fa-envelope"></i> [sankha@ku.edu](mailto:sankha@ku.edu) or [me@sankhs.com](mailto:me@sankhs.com)<br>
<i class="about-icon fa-brands fa-x-twitter"></i> [@ngsankha](https://twitter.com/ngsankha) /
<i class="about-icon fa-brands fa-bluesky"></i> [@sankhs.com](https://bsky.app/profile/sankhs.com) /
<i class="about-icon fa-brands fa-github"></i> [ngsankha](https://github.com/ngsankha)<br><br>

<!-- --- -->

I am an Assistant Professor in the [EECS Department](https://eecs.ku.edu) at [The University of Kansas](https://ku.edu). I am interested in practical tools that help programmers build correct and efficient software. My research specifically uses programming language abstractions to design program analysis and synthesis techniques that facilitate automatic construction of functionally correct software. I direct the [KU Programming Systems Group](https://ku-progsys.github.io/).

I got my PhD from the [University of Maryland](https://cs.umd.edu), advised by [Prof. Jeff Foster](http://www.cs.tufts.edu/~jfoster/) and [Prof. David Van Horn](http://www.cs.umd.edu/~dvanhorn/). My background includes significant industry experience, from contributing to the [SpiderMonkey](https://spidermonkey.dev/) JavaScript engine at Mozilla Firefox to building cloud-scale testing infrastructure for apps and websites at [BrowserStack](https://www.browserstack.com). I also gained valuable insights while working on the Hack programming language at [Meta](https://meta.com).

> <i class="about-icon fa-solid fa-star"></i> **I am hiring Ph.D. students for our group!** [Here are some reasons](https://ku-progsys.github.io/why-ku/) why our group and KU might be a great fit. Please [send me an email](mailto:sankha@ku.edu) and apply to the [graduate program in EECS](https://eecs.ku.edu/graduate-programs) at KU if you want to join us. If you are already at KU, shoot me an email to set up a meeting.

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

[**Program Synthesis with Lightweight Abstractions**](/static/phd-thesis.pdf).<br>
_Sankha Narayan Guria_.<br>
PhD Dissertation.<br>
<span class="pubs-subtext">
[DRUM](https://drum.lib.umd.edu/items/608c0922-9e90-4064-84dd-60e500ec9c6a)
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

**EECS 700: Introduction to Program Synthesis**<br>
[Fall 2024](/eecs700/) / Fall 2023

**EECS 662: Programming Languages**<br>
[Spring 2025](https://ku-progsys.github.io/eecs662/) / Spring 2024
