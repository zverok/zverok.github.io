---
layout: post
title:  'Advent of Ruby Changelog: Week 1'
date:   2023-12-07
description: "A diary of preparing Ruby 3.3's annotated changelog."
categories: ruby philosophy
comments: true
image:
---

Every year since 2018, I write the annotated [Ruby Changelog](https://rubyreferences.github.io/rubychanges/).

The work on the changelog, for me, includes:

* Writing (and investigating) a detailed explanation of each feature's reasons, quirks, and details of usage;
* Trying this feature on the most recent Ruby versions to better understand the details (sometimes, finding the bugs);
* Checking the feature docs to see if it is covered properly and fixing them, if necessary.

The process of preparation for the new version also includes:

* Checking docs inconsistencies and lacks gathered through the year, and fixing them.
* Checking the status of feature requests gathered through the year, and understanding where we are standing on the Ruby Evolution.

TBH, most of this activity (keeping an eye on docs, tracking new features and discussions) could be more efficiently done as a constant activity, and a couple of years ago, I planned to switch to this mode, but then you-know-what happened, and only now I've found some time for my Ruby activities, so *\*shrugs noncommittally\**.

I already covered what I am doing and thinking when working on the changelog from the bird-eye view in a series of articles: three in Jan 2022 ("[What you can learn by merely writing a programming language changelog](https://zverok.space/blog/2022-01-06-changelog.html)", "[Following the programming language evolution, and taking it personally](https://zverok.space/blog/2022-01-13-it-evolves.html)", "[Programming language evolution: with all that, we are still flying](https://zverok.space/blog/2022-01-20-still-flying.html)") and a Feb 2023 addendum "[Participating in programming language's evolution during interesting times](https://zverok.space/blog/2023-02-07-changelog2023.html)".

This year, though, I want to cover it in a day-by-day diary (which would also incentivize me to do it in day-by-day small chunks, not leave for the last moment and then be criminally late after the release).

_At the moment of writing this paragraph, I am not sure whether I'll publish the diary as a whole in the end or in several weekly installations (or there wouldn't be a thing to publish at all). We'll see._

...And at the moment of writing _this_ paragraph (a week later), I decided to, indeed, publish parts of the diary every week. It also turned out that there are a lot of things to say about the process—so many, in fact, that while each day's entry is quite short, seven of them are longer than a decent blog post (or Substack email size limit).

So, I'll try to do a weekly round-up post with links to more detailed entries for those who are interested.

Here are diary entries for the first week:

* **[Day 1](/advent2023/day01.html)**, where I explain my usual process and start by looking into this year's `NEWS.md` file in Ruby repo. From first sight, it looks like this year’s release has very few changes, and even less so to analyze and describe in detail, so it might not be the best year to showcase “what it takes.” (Spoiler alert: this was a false feeling!)
* **[Day 2](/advent2023/day02.html)**, where I go into more details about converting terse official `NEWS.md` into my wordy changelog sections, and a lot of uncertainties to think of in the process.
* **[Day 3](/advent2023/day03.html)**, where I do an initial pass through all the new/changed feature docs and uncover many small enhancement possibilities to put in my December TODO.
* **[Day 4](/advent2023/day04.html)**, where I start to look into the features that I need to understand better before explaining them, and, in particular, `Module#set_temporary_name`.
* **[Day 5](/advent2023/day05.html)**, where I look into a few more new features: a new `WeakKeyMap` class, and `Thread::Queue#freeze` method, and demonstrate how I reflect upon uncertainties in my own understanding and possible documentation problems I'd like to fix.
* **[Day 6](/advent2023/day06.html)** & **[day 7](/advent2023/day07.html)**, where I am diving head-first into enhancing docs for a new class `ObjectSpace::WeakKeyMap` and its older cousin `ObjectSpace::WeakMap`, and end with submitting a PR to Ruby core and finally writing some drafty texts for the changelog.

**_To be continued..._**

Subscribe to my [Substack](https://zverok.substack.com/), or follow me on [Twitter](https://twitter.com/zverok).

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please read this too.** This is your weekly mid-text reminder that I am a living person from Ukraine, and a bit of useful related information._

  **One news item.** The body of a 15-year-old Ukrainian girl, who had been missing for 1,5 years following the Russian invasion, was [discovered in Belarus](https://twitter.com/United24media/status/1732369569447047396).

  **One piece of context.** December 6 was the Day of the Armed Forces of Ukraine. There were many excellent materials published for the day, but I ask you to read this [short graphic story/conversation](https://twitter.com/polosunya/status/1732388614737178698) with a wonderful woman from my home city Kharkiv, an LGBT activist and combat medic since summer 2022.

  **One fundraiser.** Please help [this international fundraiser](https://twitter.com/Teoyaomiquu/status/1731738225033510991) held by Ukrainian combat veteran for desperately needed drones.

  **One plea.** Please [read this](https://twitter.com/anightsostill/status/1732506580191506449), and consider helping us by contacting your representative.
</div>
