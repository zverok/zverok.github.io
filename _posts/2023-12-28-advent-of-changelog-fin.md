---
layout: post
title:  'The making of Ruby Changes: A boring advent'
date:   2023-12-28
description: "A diary of preparing Ruby 3.3's annotated changelog throughout the December of 2023."
categories: ruby philosophy
comments: true
image: "https://zverok.space/img/2023-12-28/days.png"
---

I have been writing in Ruby for almost 20 years; my first version was Ruby 1.6. I was always curious to observe how the language evolves, always trying to see what the changes might help me understand about my own usage of it, the community, and the ways of thinking in code. For me, understanding how it _changes_ helped me to understand how it _is_.

## The changelog

Five years ago, I started the [Ruby Changes](https://rubyreferences.github.io/rubychanges/) project, whose mission was providing the context of the new and adjusted features: why they were invented, what the design space was, and how _exactly_ it behaves. This activity was useful for my own deeper understanding, and, judging by the community reaction, many felt the same.

It also turned out that by working on the changelog, I was able to give back to Ruby, discovering small inconsistencies or lacking documentation about the change I was trying to describe. I told more about this in a "big picture of work on the changelog and participating in the Ruby evolution" series of articles a couple of years ago ("[What you can learn by merely writing a programming language changelog](https://zverok.space/blog/2022-01-06-changelog.html)", "[Following the programming language evolution, and taking it personally](https://zverok.space/blog/2022-01-13-it-evolves.html)", "[Programming language evolution: with all that, we are still flying](https://zverok.space/blog/2022-01-20-still-flying.html)").

Since then, every year, I covered a freshly released version and, sometimes, went deeper, like covering versions [2.4](https://rubyreferences.github.io/rubychanges/2.4.html) and [2.5](https://rubyreferences.github.io/rubychanges/2.5.html) that were released before the "Ruby Changes" project started or creating a "[Ruby Evolution](https://rubyreferences.github.io/rubychanges/evolution.html)" bird-eye view of the (by now) ten-year span of changes since 2013[^1].

[^1]: I know I am repeating this information from time to time, but I intend to share this article with a broader audience than just the Ruby community, so I decided to put down some context.

## The advent

This year, I decided on the experiment of "advent-style" work on changelog: the time-boxed everyday work, with parallel writing of the diary, explaining how the work is done and how I think about things related to it. At the very least, the idea helped to keep discipline: last year I was two months late, which is bad not only because by the time of the changelog publishing, those interested had already looked into everything themselves but also because the discovery of small inconsistencies in docs or behavior was made only after the release.

It was my hope that I could make "the diary of working on changelog" an interesting journey for the outside reader.

Well, now that everything is done and the changelog is [published](https://rubyreferences.github.io/rubychanges/3.3.html) on time, I have mixed feelings about the experiment.

The accidental journey just took some 55-65 working hours (24 days, with at least 2 hours on most working days, and usually more on weekends), during which I did:

* the changelog itself;
* a few tickets in the bug tracker, some [clarifying](https://bugs.ruby-lang.org/issues/20082), some [to be fixed before the release](https://bugs.ruby-lang.org/issues/20056), some uncovering [slight old bugs](https://bugs.ruby-lang.org/issues/20059) in the dark corners (there was also at least [one problem](https://bugs.ruby-lang.org/issues/20090) that I've missed despite adding the feature to the changelog—given, the corresponding feature were merged just a few hours before the final release);
* a few [documentation improvements](https://github.com/ruby/ruby/pulls?q=is%3Apr+author%3Azverok+is%3Aclosed);
* a small [PR](https://github.com/ruby/pathname/pull/33) to the standard library (unfortunately, not merged before the release—as well as a relatively big [core feature](https://github.com/ruby/ruby/pull/7444) I got accepted _in principle_ before my mobilization but didn't have time to finalize properly);
* a huge backlog of things that can be fixed in the old documentation for better understanding and consistency, and of ideas/proposals about the edge cases of the language behavior;
* ...and, well, the diary.

## The outcome

So, what about the diary?

I had an experiment with the ["advent" of code reading/rewriting](https://zverok.space/blog/2021-12-28-grok-shan-shui.html) two years ago; during that work, writing the diary was the activity that definitely helped. It documented the adventure and my thoughts/ideas on software writing and is, in fact, its primary outcome (while the rewritten code is rather supplementary material). It also helped to organize thoughts and was a welcome switch of activities after digging into the code and debugging.

The "advent of changelog" is a more specific product: I needed to write a text about (mostly) writing a text; there wasn't much "activity type" switching, and also I constantly needed to decide what to include in the diary, so it would be (hopefully) interesting to read, without just putting there "spoilers" of feature descriptions I just wrote.

In hindsight, all of that is not that exciting to do—and, most probably, not that exciting to read about! It also didn't help that this year's version has much more internal implementation changes (the main sections of the [official release note](https://www.ruby-lang.org/en/news/2023/12/25/ruby-3-3-0-released/) are all dedicated to that) than language syntax/API changes, which is what all of my changelog is about.

So the whole work seems a bit "out of focus" this year, as if everything I am covering is just notes on the margins of the "main changes of this version." Still, I maintain that observing how the language changes might be of interest—even if this year's changes aren't that big.

The part of the diary that concerns fixing the documentation wording and rendering might look patronizing, or else expose the community in a bad light, or emphasize me as an extremely valuable Keeper of Order. I meant nothing like that. There are just a few things I wanted to communicate with this diary:

* how the intention to see structures in everything can be tamed and used to do "the boring work" of just describing other people's work;
* what amount of small things can be uncovered and questioned by just creating such a structure;
* how many small things needs to be fixed every day of the language's life—even when all the big things are in place, thoroughly discussed, and well-tested.

This everyday work, which I take on myself only once a year, is just a fraction of what more experienced and involved core team members do all around the clock. Ruby's development process is highly informal, and somebody needs to handle "the boring stuff." And usually, somebody does. I just did a small bit.

## The diary

Here is a diary of this "boring work"—hopefully, not that boring still, with my "chaotic good" style of handling tasks and thinking about them (which I sometimes feel matches the language's development style in general).

There are 24 entries; each day is a link. The first half of them were already linked in the previous posts, but I am putting it all here for convenience:

* **[Day 1](/advent2023/day01.html)**, where I explain my usual process and start by looking into this year's `NEWS.md` file in Ruby repo. From first sight, it looks like this year's release has very few changes, and even less so to analyze and describe in detail, so it might not be the best year to showcase "what it takes."" (Spoiler alert: this was a false feeling!)
* **[Day 2](/advent2023/day02.html)**, where I go into more details about converting terse official `NEWS.md` into my wordy changelog sections, and a lot of uncertainties to think of in the process.
* **[Day 3](/advent2023/day03.html)**, where I do an initial pass through all the new/changed feature docs and uncover many small enhancement possibilities to put in my December TODO.
* **[Day 4](/advent2023/day04.html)**, where I start to look into the features that I need to understand better before explaining them, and, in particular, `Module#set_temporary_name`.
* **[Day 5](/advent2023/day05.html)**, where I look into a few more new features: a new `WeakKeyMap` class, and `Thread::Queue#freeze` method, and demonstrate how I reflect upon uncertainties in my own understanding and possible documentation problems I'd like to fix.
* **[Day 6](/advent2023/day06.html)** & **[day 7](/advent2023/day07.html)**, where I am diving head-first into enhancing docs for a new class `ObjectSpace::WeakKeyMap` and its older cousin `ObjectSpace::WeakMap`, and end with submitting a PR to Ruby core and finally writing some drafty texts for the changelog.
* **[Day 8](/advent2023/day08.html)**, where new features are landing in Ruby's `NEWS.md`, while I am working on [a PR](https://github.com/ruby/ruby/pull/9174) to fix small documentation problems in Ruby's docs.
* **[Day 9](/advent2023/day09.html)**, where I prepare one more documentation-fixing [PR](https://github.com/ruby/ruby/pull/9183) and notice some changes are still missing from `NEWS.md`.
* **[Day 10](/advent2023/day10.html)**, where I finally sumbit myself into writing a big part of the changelog, but also discover some small new inconsistencies to report, possible language improvements to pursue, and philosophical problems to ponder upon.
* **[Day 11](/advent2023/day11.html)**, where I try to proceed, but it entangled into some lack of understanding, old surprising behaviors, and historical discussions (and also meet my younger self accidentally).
* **[Day 12](/advent2023/day12.html)**, where I recompile Ruby and investigate the new (if not yet fully materialized) possibilities that would be opened with the introduction of `it` anonymous block argument next year.
* **[Day 13](/advent2023/day13.html)**, where I am almost done with the first, quick-and-rough part of the changelog.
* **[Day 14](/advent2023/day14.html)**, where I am summarizing what was done and writing this blog post.
* **[Day 15](/advent2023/day15.html)**, where I plan for the rest of the work and estimate the time it takes.
* **[Day 16](/advent2023/day16.html)**, where I get a first rendering of the changelog draft and explain my approach to structuring it into bigger sections (while also feeling uneasy about imposing the structure I invented).
* **[Day 17](/advent2023/day17.html)**, where I add the changes related to the standard library contents, and find a few more inconsistencies to poke and documentation problems to fix.
* **[Day 18](/advent2023/day18.html)** and **[day 19](/advent2023/day19.html)**, where I go over all of the small places in the changelog that created questions, needed improvement, led to unpleasant discoveries about documentation rendering—but in general, the changelog is kind of done by now.
* **[Day 20](/advent2023/day20.html)**, where I am taking on adjusting the official `NEWS.md` with the features that were added through the year but haven't been described there—and dutifully add them to the changelog, too.
* **[Day 21](/advent2023/day21.html)**, where I adjust the old changelog with forward links of "how it became in new versions."
* **[Day 22](/advent2023/day22.html)**, where I update the Ruby Evolution page (and explain how and why it is structured).
* **[Day 23](/advent2023/day23.html)**, where I add the `Set`'s changes to the changelog and explain why they were missing in `NEWS.md`
* **[Day 24-25](/advent2023/day24.html)**, where I wrap everything up, link to the important standard library changes, Ruby 3.3 is released, the changelog is published... and one last moment change is, of course, forgotten, and then merged right into the `master`, because that's how everything is done!

So, yeah... That's that.

Subscribe to my [Substack](https://zverok.substack.com/), or follow me on [Twitter](https://twitter.com/zverok).

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please read this too.** This is your weekly reminder that I am a living person from Ukraine, and a bit of useful related information._

  **One news item.** ["They brought him home and shot him": occupants killed a schoolboy in front of his family in Kherson region](https://eng.obozrevatel.com/section-news/news-they-brought-him-home-and-shot-him-occupants-killed-a-schoolboy-in-front-of-his-family-in-kherson-region-22-12-2023.html)

  **One piece of context.** [Merry Christmas! This is the stolen artwork from the Kherson museum, one of the masterpieces of 🇺🇦art "Carolers" by Mykola Pymonenko. In Ukraine, people are going from house to house singing about the birth of Jesus and wishing all the best for the next year.](https://twitter.com/ukr_arthistory/status/1738986964479541383)

  **One fundraiser.** This Christmas, as ever, the "Hospitalliers" medical battalion [can use some funds](https://www.hospitallers.life/needs-hospitallers) to save lives!

  **One plea.** Please [read and spread info](https://www.savedefenders.info/) about Azovstal and Mariupol defenders who are still in captivity.
</div>

