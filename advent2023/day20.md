---
layout: advent
title: "Advent of changelog: Day 20"
next: day21
next_title: "Day 21"
prev: day19
prev_title: "Day 19"
---

I am spending the 20th (the 20th! Well, duh, the Christmas is nigh) day checking through the previous versions of the changelog, adding cross-links back and forth between versions- And also thinking that I probably should formalize the format of the cross-links somehow, maybe just a human text "In <u>3.2</u> it became so and so" isn't informative enough. With the changelog, I always try to balance between formal enough so it would be easy to scan quickly and see the answers to most questions; and humane enough so it would be _readable_ as kind-of-curious-article[^1].

[^1]: Nobody asked me, but I think that's where the Wikipedia made a wrong decision, when they banned mere lists of facts and too-short-pages (as opposed to "a section in a bigger page"), so now many articles on a complicated well-known subjects are walls of text size of a moderate book you expected to read top to bottom to find some fact.

Then, as promised, I make some adjustments to the Ruby's NEWS file, as I planned: add changes to `Time.new` behavior and `NoMethodError` rendering, and fix various parsing errors discovered while [reading it on the docs.ruby-lang.org](https://docs.ruby-lang.org/en/master/NEWS_md.html), like code rendering glitches (RDoc's outlook at Markdown is slightly weird) and missing links to documentation of new features (due to the improper formatting).

[A small PR](https://github.com/ruby/ruby/pull/9308) is created relatively quickly (give or take several rerendering of the entire Ruby repo RDoc to see whether all the links from `NEWS` are now correct), but then I need to reflect "newly discovered" changes in _my_ rendering of the Changelog, which leads to, as usual, links to docs and discussions, which leads to discovering that some docs needs updating, which leads to [one more PR](https://github.com/ruby/ruby/pull/9309), which is how the work on the Changelog _usually_ goes.

The **commit: [a214752](https://github.com/rubyreferences/rubychanges/commit/a214752)**.
