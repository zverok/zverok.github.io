---
layout: advent
title: "Advent of changelog: Day 21"
next: day22
next_title: "Day 22"
prev: day20
prev_title: "Day 20"
---

Today is just a _regular_ day of working on the changelog, when I finally have nothing philosophical to say.

* [Two](https://github.com/ruby/ruby/pull/9308) [PRs](https://github.com/ruby/ruby/pull/9309) from yesterday got merged.
* I added a lot of "follow-up" references throughout the past years changelogs, going as far as Ruby 2.4 (which was the first version to introduce `Warning` module, so a new warnings category added by 3.3 can be considered a "follow-up" for the concept);
* Caught a few small old typos and broken links in past versions and fixed them;
* Added a reference to [FastRuby's blogpost](https://www.fastruby.io/blog/ruby-3-3.html) on new features in 3.3, which focuses on the truly significant internal changes on this version, which my _language_ changelog fails to cover;
* I reassessed the remaining work, and now plan to go at "Ruby Evolution" [generic page](https://rubyreferences.github.io/rubychanges/evolution.html) next: it is the only part of the changelog that misses any mention of 3.3, and, preparing for the worst case scenario (= I wouldn't have enough time to wrap-up everything properly)[^1], I need to make sure _most_ is done and ready to release. Also, trying to place the significant changes in the "evolution" frameworks sometimes allows to see clearer how they should be described and structured.

[^1]: As I hear air raid sirens right when writing this phrase, I am reminded "I would be busy" is by far not a _worst case scenario_ in this time and place.

**The commit: [ed1054b](https://github.com/rubyreferences/rubychanges/commit/ed1054b)**
