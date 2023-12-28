---
layout: advent
title: "Advent of changelog: Day 14"
next: day15
next_title: "Day 15"
prev: day13
prev_title: "Day 13"
---

So, that marks two weeks of working on the changelog. This means something between 25 and 35 hours (~1.5-2 hours on a workday, more on the weekends) of time.

Today, I am finishing the first go through all of the changelog sections (there's not much left), and reiterating on "where do we stand" on the further plans.

I write a very, very dumb and simple explanation from the leftover `Process.warmup` explanation (though reading through the discussion gives me some ideas on what might be useful to explain here), and drop `Encoding#replicate` section altogether. With that, the "very first draft" of everything is in place. There is still a lot of TODOs there (mostly calling for missing/better code examples), but it is important to put a mental check mark on the "first pass finished" box.

So, what was done during these two weeks?

* an amount of changelog texts no more than my regular blogpost (and much rougher);
* a lot of code studying, ticket reading, and head scratching;
* three documentation PRs:
  * a [small cosmetic one](https://github.com/ruby/ruby/pull/9174) just handling the rendering glitches;
  * a bigger (yet still pretty small) [one](https://github.com/ruby/ruby/pull/9183) with adjustment of phrasing and examples for several new methods;
  * and [one more](https://github.com/ruby/ruby/pull/9160) specific to `WeakMap` and `WeakkeyMap`;
* A couple of tickets:
  * [One](https://bugs.ruby-lang.org/issues/20059) with a weird small (but very old) bug/inconsistency, for later discussions;
  * And [another one](https://bugs.ruby-lang.org/issues/20056) about equally small inconsistency in the newly-introduced `Dir` methods, that was promptly fixed by Jeremy Evans (and, truth be told, probably would've been fixed anyway);
* A growing list of TODOs to _eventually_ follow upon (most probably not before the release), related to small better docs and (mostly) small core API changes for more consistency;
* ...definitely not enough sleep.

So, here is today's **commit: [997f248](https://github.com/rubyreferences/rubychanges/commit/997f248)**, and now I'll try to publish the last seven days of the log + a really small blog post on my site.

**_To be continued..._**
