---
layout: advent
title: "Advent of changelog: Day 10"
next: day11
next_title: "Day 11"
prev: day09
prev_title: "Day 9"
---

Today is Sunday, and I have (slightly) more times on my hands than usual. I want to use this time for a Big Catch-Up! For all the talks about the "Advent of the **Changelog**," the changelog itself doesn't have much meat yet on its bones: mostly what could've been done mechanically, like structure and links.

So, today I want to do as much of drafty explanations & examples as possible.

On this road, I try hard to avoid over-explaining things in the fear of making changelog sound like a too helpful AI assistant or a SEO-optimized blog. When you have a template of the change-dedicated section with many fields with good names like "Reasons" and "Notes", it is tempting to fill them all with a most detailed explanation of most trivial things.

As a short example: in Ruby 3.3 `Array#pack` and `String#unpack` raise `ArgumentError` when met with unknown directive (before, it was just a warning, and before that, a warning visible only in a _verbose_ mode!). Does this really need an _explanation_ and code examples (save for the trivial "it works")? I think not.

When I invented the Ruby Changes format, I was driven by irritation at many blog posts emerging each December/January to discuss "top new features of Ruby" in several pages of text, spending all those pages on the most wordy explanation of trivia, but never demonstrating even the basic attempt to use the feature (or look at its edge cases, or describe the reasoning behind it). So I described the main driving factors for the Ruby Changes as this:

![](/img/advent2023/image19.png)

...and I try to follow this to the best ability of my wordy, graphomaniac, ADHD-driven nature. I am trying to only detail the reasoning behind the feature when it is non-obvious and put as many examples as necessary to demonstrate the base usage an the edge cases/limitations. Frequently, though, I indulge myself to additional "Notes" section, when I think a more detailed story behind the feature or its current limitations deserves some attention.

So, in the next several hours, I did, like, 2/3 of the descriptions (still drafts!) and put it into a **commit [705bc6e](https://github.com/rubyreferences/rubychanges/commit/705bc6e)** (there is not that much text/code there, but writing many entries included re-reading of the relevant discussions, experimenting with code, and sometimes trying several ways of phrase things).

A few random notes on the road:

* I discovered a minor inconsistency in the new `Dir#chdir` behavior and [asked about it](https://bugs.ruby-lang.org/issues/20056) on the tracker;
* I discovered that a new `Range#reverse_each` specialization is only defined for `Integer` so far; but it is not an accidental omission, rather the simplest/most necessary thing to do. The further improvements were [discussed](https://bugs.ruby-lang.org/issues/18515#note-4), and I am putting it in my "probably maybe" TODO-list of things to follow/address after the 3.3 release;
* A skip `Process.warmup` explaining so far: the [original justification on the tracker](https://bugs.ruby-lang.org/issues/18885) and [the docs](https://docs.ruby-lang.org/en/master/Process.html#method-c-warmup) are pretty exhaustive as _explanations_, and the feature is impossible to _demonstrate_ with a short code snipped, so— I am always lost about what should be done in such cases (especially considering I am not an expert in VM internals and long-running processes optimization). The end result would probably be just a rephrasing (or maybe direct quoting) of the tracker's proposal;
* I allow myself an "advise"-style note for a new and interesting method `Module#set_temporary_name`. Maybe not a changelog content, but—

And just like that, the Sunday evening is gone.

