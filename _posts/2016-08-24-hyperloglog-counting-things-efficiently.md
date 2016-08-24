---
layout: post
title: "HyperLogLog: Counting things efficiently"
---

<script src="https://cdn.mathjax.org/mathjax/latest/MathJax.js?config=TeX-AMS-MML_HTMLorMML" type="text/javascript"></script>

Today at work I came across an interesting problem - how to count the number of items in a set efficiently. To understand why this is an important thing let me first describe a problem. Say for example you have an application that is that is getting a lot of search queries. At any point in time you would like to know what is number of unique search queries that were performed till now. Since this is a live production application the number of actual searches could be in the range of millions to billions.

The traditional way to tackle this is to keep the data in a set and count the number of elements in the set. But we have a problem if we keep so many keys in a set in memory - we will be using memory in the order of elements that we have stored in our set. That is wasting a whole lot of memory.

Today while trying to figure out what techniques are used to tackle situations like this I came across the algorithm called HyperLogLog proposed in the [paper](http://algo.inria.fr/flajolet/Publications/FlFuGaMe07.pdf) by Flajolet et. al. It is a randomized algorithm that can be used to approximate the count of distinct elements but within a small limited amount of memory.

### How does it work?

Let say we have a stream of numbers/data flowing in. We use a hash function to hash it to `k` bits. Now we can infer from that bit representation that:

* A number has 50% chance it will start with `1`
* A number has 25% chance it will start with `01`
* A number has 12.5% chance that it will start with `001`

This means if you see a number begin with `001` it means there is a higher chance that there will be $$2^3 = 8$$ unique elements.

This is what is known as the bit-pattern observable. Flajolet describes it as:

> Bit-pattern observables: these are based on certain patterns of bits occurring at the beginning of the (binary) S-values. For instance, observing in the stream S at the beginning of a string a bit-pattern $$O^{\rho-1}1$$ is more or less a likely indication that the cardinality n of S is at least $$2^\rho$$.

This [article](https://research.neustar.biz/2012/10/25/sketch-of-the-day-hyperloglog-cornerstone-of-a-big-data-infrastructure/) gives a very nice example for this intuition. Let's say you are flipping a coin and the longest run of heads that you got was 2 - I would gather that you were not flipping the coin for a long time. Now if you say that your longest run of heads was 10 - I would say that you were probably flipping the coin for a long time. Thus by keeping a very small piece of information (the longest run of heads) we can get some understanding of what has happened in the stream.

As you can see the estimates we get by this technique are very way off from the possible actual number. To improve this, HyperLogLog stores many estimators and uses the mean. The way it does is creating a fixed number of buckets from the hash values.

Let's say we have an input number as `0010000010101` and we decided to use $$m = 3$$ bits for the buckets, then we would have $$n = 2^3 = 8$$ buckets. Taking the right most $$m$$ bits we have $$101_2 = 5$$, thus this entry would go to the 5th bucket. With the remaining bits `0010000010` we take the longest run of 0 which is $$R = 5$$ and place it in the 5th bucket. Thus, to compute the number of distinct values in the set we can do:

$$
DV_{LL} = c \times n \times 2^\bar{R}
$$

where $$\bar{R}$$ is the average of the value $$R$$ in all the buckets. But this actually gives us the estimator for LogLog algorithm.

But if we notice carefully, if we have a outlier in any of the buckets the average would be affected badly. Thus HyperLogLog changed this to use the harmonic mean of the values of each bucket. So we have the distinct value estimator as:

$$
DV_{HLL} = c \times n \times \frac{n}{\sum_{i=1}^{n} 2^{R_i}}
$$

where $$R_i$$ is the longest run of zeros in the $$i^{th}$$ bucket.

### Implementations

Redis [supports this algorithm](http://antirez.com/news/75) as part of one of its core data structure. It uses 64 bit values for its hashes - using only 14 bits for bucketing, giving us 16384 buckets. That leaves us with 50 bits - so the longest run of 0 will fit in a 6 bit register (approximately $$log_2 50$$) thus making the Redis HyperLogLog take up only 12KB of memory. The standard error of HyperLogLog is $$1.04\sqrt{n}$$ where $$n$$ is the number of buckets - giving us 0.81% error . Since it uses 64 bit output function we can practically count the cardinality of any set.

There are other implementations of the same as a library in almost every language.

### Conclusion

I found the HyperLogLog algorithm super interesting and useful for calculating the approximate cardinality of very huge sets. There are a number of practical gotchas to implementing it well - keeping in things like bias correction for a variety of cardinality ranges. Google has a [paper](http://static.googleusercontent.com/media/research.google.com/en//pubs/archive/40671.pdf) on implementing it in practice, Redis also takes an interesting approach to [bias correction](http://antirez.com/news/75).
