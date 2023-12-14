---
layout: advent
title: "Advent of changelog: Day 13"
next: day14
next_title: "Day 14"
prev: day12
prev_title: "Day 12"
---

Today I just plan to move further. For that, I am starting back from the beginning of the currently ready part of the changelog and move forward, adjusting old texts and writing the new ones (this is a frequent writing technique to pick up a stead pace).

Not much to reason about today. By the end of my allotted timeslot (which is 1.5-2h on a workday) I managed to _roughly_ cover _almost_ all sections.

As always, there are digging and nuances and more digging and trying to produce artificial-yet-persuasive code examples... And I never know whether I am not outdoing things, making a "brief-yet-comprehensive" changelog into a huge list of trivia and opinions.

The curious example is `Warning[:performance]` category, which required for demonstration to explain the only currently existing performance warning: "too many different object shapes." It is an interesting and useful topic, and it also raises some questions about the border between the language and its implementation. Because "object shapes" is, from one point of view, is "just" CRuby's implementation detail, an internal optimization... And at the same time, "object-shapes-friendly" code should be written a bit differently, and a new peformance warning underlines that, and it affects _how we think_ in the language!

Anyway, only two sections (of the _currently_ available `NEWS`) are not covered yet: `Process.warmup` (as I've already said, it would probably be covered just by reusing the initial explanation) and a removal of `Encoding.replicate`—which, despite being a topic of [a long and interesting discussion](https://bugs.ruby-lang.org/issues/18949) seems negligible enough to just remove it from the changelog probably. Or otherwise, I risk to put a lot of effort into explaining the feature that wasn't necessary and wasn't used and was just removed. Though this decision to not put it into a changelog at all still feels a bit dirty: besides individual versions' changelogs, the "Ruby Changes" site also includes [Ruby Evolution](https://rubyreferences.github.io/rubychanges/evolution.html) bird-eye view of "everything significant that have happened," and removal of a method from a core class deserves at least a brief mention there?.. I don't know yet.

Anyway, here's a **commit: [bf77067](https://github.com/rubyreferences/rubychanges/commit/bf77067)**.
