---
layout: advent
title: "Advent of changelog: Day 11"
next: day12
next_title: "Day 12"
prev: day10
prev_title: "Day 10"
---

Today, a Ruby 3.3 RC1 [have been released](https://www.ruby-lang.org/en/news/2023/12/11/ruby-3-3-0-rc1-released/), _more or less_ finalizing the shape of what would be in the final version. (It is not unseen to have some features merged after the RC, but it is less likely... Which, unfortunately, leaves a couple of things I cared about in a hanging state, but let's keep this for later.)

Today I am continuing to work on the drafty changelog description & addressing the PR comments for my documentation PRs.

Yesterday, I [asked about an inconsistency](https://bugs.ruby-lang.org/issues/20056) that I've apparently found in `Dir#chdir` behavior. Somewhat humblingly, it turned out there were no inconsistency (for some reason, I just checked the method _code_ and misread it instead of checking the behavior!), but the details of the behavior aren't included in docs, so I can fix that, too, while on it. ...but trying to do so, I think I've [uncovered another inconsistency](https://bugs.ruby-lang.org/issues/20056#note-3)?.. So I am postponing this documenting for now, and just continue with the changelog explanations.

Today was not very productive moving through the changelog, because the very next thing I met is a surprising behavior of `TracePoint` and [asked about that](https://bugs.ruby-lang.org/issues/20059).

And the next thing was diving into a fascinating multi-year-spanning [discussion](https://bugs.ruby-lang.org/issues/15973) of `lambda` method behavior, where I met my [4-years-younger self](https://bugs.ruby-lang.org/issues/15973#note-41), much more caustic than my current one became (I hope).

Anyway, two hours are gone, it is midnight, and I am just committing whatever I managed to write, stopping mid-sentence. **Commit: [be37de5](https://github.com/rubyreferences/rubychanges/commit/be37de5)**
