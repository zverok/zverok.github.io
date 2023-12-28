---
layout: advent
title: "Advent of changelog: Day 8"
next: day09
next_title: "Day 9"
prev: day07
prev_title: "Day 7"
---

Today I'll be working on more PRs with small documentation fixes, and on finalizing yesterday's `WeakKeyMap` docs PR, which in the meantime got reviewed.

But first, I need to handle all the new things in `NEWS.md` that happened since I processed it initially (because they also need documentation check, and I don't like to repeat the work).

So, today we are adding:
* `Fiber#kill` (which lists as its reason a bug so old [its number](https://bugs.ruby-lang.org/issues/595) is 3 digits, and it is written in Japanes originally! Thankfully, fresher parts of the discussion are in English, otherwise I would've had a lot of fun trying to understand the reasons). Its docs have some glitches, so it was a right call to postpone glitch-fixing PR till checking the new arrivals<br/>
  ![](/img/advent2023/image16.png)
* Removal of `Encoding#replicate` (there is a long entangled story behind this method and its removal, which I followed for some time— but thankfully, it is obscure enough, so I don't think it deserves a detailed changelog entry)
* `Range#overlap?` (very useful new core method, yay!), again, some small rendering glitches:<br/>
  ![](/img/advent2023/image17.png)
* Introduction of warnings about standalone `it` (in preparation to introduce it in Ruby 3.4 as a synonym for `_1`)—this one would be fun to describe accurately!

So, now it is time to create the PR(s) that will handle problems I noticed in new feature docs. After some considerations, I decide to do two PRs: one that fixes mechanical rendering glitches and bugs in examples code (easy to review and merge), and another one which improves wording, adds examples and so on. In the worst case, I want at least the one with plain visible bugs to be merged before the release!

So, once again in my `ruby` folder...
```bash
$ git checkout master
$ git pull upstream master
$ git checkout -b fix-documentation-bugs
```

And a lot of what seems to be just mechanical replacement... Mostly, the "glitches" are instances when RDoc's simple syntax for monospace code `+code+` doesn't work when there is complicated punctuation inside `+`s, and the wrapper should be replaced with `<tt>`. But, it is not always that mechanical!

Say, cleaning up `Fiber#kill`'s docs I need to decide whether references to other objects and methods should stay monospace, or maybe became links to them (which RDoc generates automatically for everything that looks like a known class... Which is not always convenient, because not every entry of the word `Array` or `Thread` or `Fiber` should be a link!)

Consequently, while cleaning up that `Range#overlap?` paragraph, I am trying to put a reference to `Float::INFINITY` from there... But for some reason it doesn't work— even on [RDoc's own site](https://ruby.github.io/rdoc/RDoc/MarkupReference.html#class-RDoc::MarkupReference-label-Links), demonstrating how it should:<br/>
![](/img/advent2023/image18.png)

Maybe should be fixed... Once... But somebody. Maybe me.

Soon though, the [first PR](https://github.com/ruby/ruby/pull/9174) is ready (I am also happy to see that some "glitches" I wrote done to my TODO were already fixed by somebody else; that's how it works: a huge community effort just happens all over the clock):

And a small **Changelog commit: [09e8cbb](https://github.com/rubyreferences/rubychanges/commit/09e8cbb)**.
