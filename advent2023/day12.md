---
layout: advent
title: "Advent of changelog: Day 12"
next: day13
next_title: "Day 13"
prev: day11
prev_title: "Day 11"
---

So, it is almost half of the advent, and even a draft version of the changelog is still not done... But I still need to adjust my current [Ruby PR](https://github.com/ruby/ruby/pull/9183) by addressing review comments and also adding a couple of forgotten fixes.

...which takes, like, half an hour with proper structure of the commits, checking everything renders OK and so on. I definitely should follow this kind of things through the year, not in the changelog-preparation rush!

Then I am getting back to changelog...

And starting to skip over things that will require additional in-depth thinking about the ways to explain and demonstrate them (like `Fiber#kill` or `:performance` warnings) and document an easy targets like `Range#overlap?`.

Then I try to check in which situations `it` will warn that it will be introduced as an anonymous block argument... And that requires to recompiling my copy of Ruby which was last compiled at the start of the changelog advent!

So in the meantime I might return to the skipped methods that require in-depth explanations.

`Fiber#kill` has especially long discussion around it. As I've already mentioned, the [original ticket](https://bugs.ruby-lang.org/issues/595) that led to its introduction is so old that it has 3-digit URL and discussion in Japanese; it spans 15 years (since 2008), and switches to English by 2014. But eventually, I think I am able to understand the problem and write very drafty description/code example before my Ruby is compiled and I am going back to `it`.

Fortunately, the [discussion](https://bugs.ruby-lang.org/issues/18980) about `it` (which is also quite long and has many voices) now, when the matter is settled, provides a [clear specification](https://bugs.ruby-lang.org/issues/18980#Specification) of the new behavior. Nevertheless, just for my own better understanding whether I "feel" the change, I first tried to _guess_ where the warnings (and future anonymous arguments) should be, then checked with my recently-built Ruby, and only then looked into the specifications.

I don't really want to brag, but all my guesses proved correct. This is actually not about me—and neither the attempt was done to prove how "smart" I am—but about whether the new behavior follows the _intuitions_ of "where it should mean an anonymous argument." It seems it does!

(Though I honestly still feel conflicted about the feature. I acknowledge `it` is, abstractly speaking, much more aesthetically pleasing than `_1`; yet I still have an uneasy feeling about the name that looks _nothing special_ means a _special_ thing in _some_ contexts. But as far as I understand, the pressure for "less ugly" solution was very high in the core team, and as nobody have proposed something that is both "nicely looking" and "semantically unambiguous"— Well, let's see how it will go.)

With that being said, BAMM, said the clock, and a small commit had marked the night: **[7b833cc](https://github.com/rubyreferences/rubychanges/commit/7b833cc)**.
