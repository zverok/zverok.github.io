---
layout: post
title:  'Advent of Ruby Changelog: Week 2'
date:   2023-12-14
description: "A diary of preparing Ruby 3.3's annotated changelog."
categories: ruby philosophy
comments: true
image:
---

**Every year since 2018, I write the annotated [Ruby Changelog](https://rubyreferences.github.io/rubychanges/). This year I want to cover it in a day-by-day diary to make it a bit more visible—and to incentivize for the regular work.**

Here is **Week 2** of it!

Quoting from the 14th day (which is today):

> [...] Two weeks of working on the changelog. This means something between 25 and 35 hours (~1.5-2 hours on a workday, more on the weekends) of time.
>
> So, what was done during these two weeks?
>
> * an amount of changelog texts no more than my regular blogpost (and much rougher);
> * a lot of code studying, ticket reading, and head scratching;
> * three documentation PRs:
>   * a [small cosmetic one](https://github.com/ruby/ruby/pull/9174) just handling the rendering glitches;
>   * a bigger (yet still pretty small) [one](https://github.com/ruby/ruby/pull/9183) with adjustment of phrasing and examples for several new methods;
>   * and [one more](https://github.com/ruby/ruby/pull/9160) specific to `WeakMap` and `WeakkeyMap`;
> * A couple of tickets:
>   * [One](https://bugs.ruby-lang.org/issues/20059) with a weird small (but very old) bug/inconsistency, for later discussions;
>   * And [another one](https://bugs.ruby-lang.org/issues/20056) about equally small inconsistency in the newly-introduced `Dir` methods, that was promptly fixed by Jeremy Evans (and, truth be told, probably would've been fixed anyway);
> * A growing list of TODOs to _eventually_ follow upon (most probably not before the release), related to small better docs and (mostly) small core API changes for more consistency;
> * ...definitely not enough sleep.

---

**So, here goes** (each day is a link):

* **[Day 8](/advent2023/day08.html)**, where new features are landing in Ruby's `NEWS.md`, while I am working on [a PR](https://github.com/ruby/ruby/pull/9174) to fix small documentation problems in Ruby's docs;
* **[Day 9](/advent2023/day09.html)**, where I prepare one more documentation-fixing [PR](https://github.com/ruby/ruby/pull/9183) and notice some changes are still missing from `NEWS.md`;
* **[Day 10](/advent2023/day10.html)**, where I dive head-first into finally writing a big part of the changelog, but also discover some small new inconsistencies to report, possible language improvements to pursue, and philosophical problems to ponder on;
* **[Day 11](/advent2023/day11.html)**, where I try to proceed, but it entangled into some lack of understanding, old surprising behaviors, and historical discussions (and also meet my younger self accidentally);
* **[Day 12](/advent2023/day12.html)**, where I recompile Ruby and investigate the new (if not yet fully materialized) possibilities that would be opened with the introduction of `it` anonymous block argument next year;
* **[Day 13](/advent2023/day13.html)**, where I am almost done with the first, quick-and-rough part of the changelog;
* **[Day 14](/advent2023/day14.html)**, where I am summarizing what was done and writing this blog post.

**_To be continued..._**

Subscribe to my [Substack](https://zverok.substack.com/), or follow me on [Twitter](https://twitter.com/zverok).

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please read this too.** This is your weekly mid-text reminder that I am a living person from Ukraine, and a bit of useful related information._

  **One news item.** Like the last winter, Russia started to target a critical infrastructure of Ukraine. This week, there were several multi-missile attacks ([listen to one of them here](https://twitter.com/saintjavelin/status/1734879233174856083) to imagine what it is to live in Kyiv right now—not even mentioning the cities that are closer to the front lines).

  **One piece of context.** It is probably a well-known fact, but worth reminding nevertheless, due to the season: a famous "Carol of the Bells" melody [was written](https://en.wikipedia.org/wiki/Carol_of_the_Bells) by Ukrainian composer [Mykola Leontovich](https://en.wikipedia.org/wiki/Mykola_Leontovych). What happened to him later? [Take a guess](https://en.wikipedia.org/wiki/Mykola_Leontovych#Death).

  **One fundraiser.** Ada Wordsworth runs a [charity](https://twitter.com/kharpproject) helping my home region of Kharkiv and does a lot of valuable humanitarian work. Currently, she [gathers funds through PayPal](https://twitter.com/padochka/status/1734194518306472306) to provide the necessary equipment to her friend, who just joined the Ukrainian army. Oh, and also read her [awesome article](https://www.telegraph.co.uk/books/non-fiction/best-books-ukraine-russia-2023/) for The Telegraph about the best English-language books of 2023 written by Ukrainians and about Ukraine.

  **One plea.** This week, I ask Rubyists from the US to **[call your representative](https://twitter.com/prorizna/status/1734642999420731673)**. It is vital.
</div>
