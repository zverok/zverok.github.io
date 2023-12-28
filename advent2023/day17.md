---
layout: advent
title: "Advent of changelog: Day 17"
next: day18
next_title: "Day 18"
prev: day16
prev_title: "Day 16"
---

_(TBH, it was a really rough week for me, unrelated to the changelog. So my plans to work on it for 4-5 hours/day on the weekend weren't implemented; now is 20:00 Sunday evening, and I'll probably do something to move forward, yet not by any means close to what I was planning.)_

So, today's just moving forward with a standard library part. My rule for a changelog to follow what's in the `NEWS.md` file, which meant that the standard library section becomes smaller and smaller each year: with extraction of its parts to gems (split into _default_ ones and _bundled_ ones), the `NEWS.md` mostly stops tracking those gems changes, telling to

> See GitHub releases like [Logger](https://github.com/ruby/logger/releases) or changelog for details of the default gems or bundled gems.

...which is a) not really convenient (to check through all the standard gems you rely upon to see whether they've added some new features), and b) sometimes impossible (the standard gems not maintaining a detailed changelog), and I want at some point to tackle this problem too, and start re-adding a centralized list of the notable changes to the main language changelog... But probably not this year, and definitely not today.

Even if the current surface to cover is small, it is not without peculiarities and challenges. Say, a passing note in `NEWS.md`:

> * `ext/readline` is retired

...calls to:

* Investigate the related [discussion](https://bugs.ruby-lang.org/issues/19616) (and some [other](https://bugs.ruby-lang.org/issues/19866) [ones](https://github.com/ruby/reline/issues/546));
* Decide on amount of code and explanations necessary, and, like, write them;
* Notice that Reline (to which now Readline docs are redirecting) has a [significant lack of documentation](https://docs.ruby-lang.org/en/master/Reline.html), compared to the "old" [Readline](https://docs.ruby-lang.org/en/3.2/Readline.html) and make a TODO (probably post-release? we'll) see to try fix that;

Or, I investigate a new argument `chars:` to [Random.alphanumeric](https://docs.ruby-lang.org/en/master/Random/Formatter.html#method-i-alphanumeric), and is surprised it expects an _array_ of characters (and not a string of them... or, if it wants to be generic, not an `Enumerable`). I investigate [the change](https://github.com/ruby/ruby/pull/8312/files) and find out it was just following the underlying private API which powered `#alphanumeric`. I can't say I find the new API fully justified, but just thinking about raising the question about rarely needed API, a week before the release, with a proposal to complicate the (1-line) implementation— well, probably everybody can use their time better.

Or, investigating the `Socket#recv` and related methods return value change, I stumble upon the fact that change is implemented not in the class mentioned in the news (`BasicSocket`, not `Socket`), and that at least one of the methods has the [contradictory documentation](https://bugs.ruby-lang.org/issues/19012#note-8) now.

And I am still not sure what does "Name resolution such as `Socket.getaddrinfo`, `Socket.getnameinfo`, `Addrinfo.getaddrinfo`, etc. can now be interrupted." means (even after reading [the ticket](https://bugs.ruby-lang.org/issues/19965)), but I am not really sure it deserves much digging.

Anyway, with that, the whole yearly changelog, with _very, very_ rough explanations and examples is in place. Woohooo! **Commit: [1c77be7](https://github.com/rubyreferences/rubychanges/commit/1c77be7)**
