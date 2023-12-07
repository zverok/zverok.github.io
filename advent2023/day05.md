---
layout: advent
title: "Advent of changelog: Day 5"
next: day06
next_title: "Day 6"
prev: day04
prev_title: "Day 4"
---

Today I am spending time on investigating of other items from my TODO with code experiments and reading through their reasoning. At least, I'll just understand them a bit better (to use in the future), and at most, I might put some examples/explanations into the changelog draft.

So, I am starting to read through [WeekKeyMap](https://docs.ruby-lang.org/en/master/ObjectSpace/WeakKeyMap.html) justification [ticket](https://bugs.ruby-lang.org/issues/18498) and play with it and `WeekMap` to understand them better.

(Skip 40 minutes of reading the discussion, docs and playing with code... My ADHD is well-compensated, so I tend to just call it "my method of the problem studying", even if it doesn't move me much closer to writing a changelog entry.)

A few random notices:

* It turns out that despite similar names, `WeekKeyMap`'s and `WeekMap`'s APIs are quite different (the new class, say, doesn't have any iteration methods, as they are hard to implement correctly with weak references)—which breaks my assumption from Day 3 (as a reason to improvie `WeekMap`'s docs during changelog work): "most of the methods should've been similar!";
* There are internal differences, too: `WeekKeyMap` compares its keys by equality (akin to a regular `Hash`), while `WeekMap` compares them by identity;
* `WeekMap`'s docs ([here's 3.2 version](https://docs.ruby-lang.org/en/3.2/ObjectSpace/WeakMap.html)) have a lot of small problems: like, both `each` and `each_key` have a description "Iterates over keys and objects in a weakly referenced object", and `each_value` is explained as (emphasis mine) "Iterates over _values and objects_ in a weakly referenced object"; so, it seems that improving them is a good thing to do this year, anyway?
* `WeekKeyMap#getkey` has a confusing at first protocol "just return the key if it exists," but [deduplication](https://bugs.ruby-lang.org/issues/18498#Deduplication-sets) use case in the original discussion seems to clarify that.

Next, I investigate a `Thread::Queue#freeze` prohibition [discussion](https://bugs.ruby-lang.org/issues/17146), which is— quite lively, let's say; but the investigation is also fruitful, as by the end of the discussion there is a [clear explanation](https://bugs.ruby-lang.org/issues/17146#note-15) why freezing the `Queue` _should_ be prohibited (and not just fixed to forbid `#push` and `#pop`, which was the original ticket's scope).

And that would be it for today, no new commits, yet a few things became clearer!
