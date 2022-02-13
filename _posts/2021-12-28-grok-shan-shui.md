---
layout: post
title:  "Grok {Shan, Shui}*: Advent of understanding the generative art"
date:   2021-12-28
categories: js programming porting generative-art
comments: true
description: I spent 24 days digging into the code of {Shan, Shui}* Chinese painting generator and lived to tell the story.
---

**I spent 24 days digging into the code of [{Shan, Shui}\*](https://github.com/LingDong-/shan-shui-inf) Chinese painting generator and lived to tell the story.**

> **This article is written for my [Substack](https://zverok.substack.com/), you can subscribe to it by email now!**

While a lot of fellow developers spent this year's advent on solving this year's [Advent of Code](https://adventofcode.com/) puzzles, I decided to have my own little "advent."

A few weeks before December, I've stumbled upon an awesome [generative art project](https://github.com/LingDong-/shan-shui-inf) by developer and artist [Lingong Huang](https://github.com/LingDong-). It generates infinite (and surprisingly varying) scrolls of Chinese paintings like this:

![](/img/advent2021/image00.png)

It struck me with its beauty—as well as with the fact that I don't have the slightest idea of how it might work. After half-an-hour looking into the [code](https://github.com/LingDong-/shan-shui-inf/blob/master/index.html), I knew I wanted to have some level of understanding of how those things are done in general—and this one, in particular—enough to spend some hours on it. So, I thought, let it be my "advent"—and that happened.

## Trying to understand

My usual way to understand how others' code works is to translate it into another language. I did [quite](https://github.com/zverok/xkcdize) [a](https://github.com/zverok/drosterize) [few](https://github.com/zverok/magic_cloud) [projects](https://zverok.github.io/blog/2020-05-16-ruby-as-apl.html) [of](https://github.com/molybdenum-99/mormor) [this kind](https://zverok.github.io/spellchecker.html). And deciding on what language to translate it to, I made a weird decision: to just rewrite (some parts of the) `{Shan, Shui}*` in the same JavaScript—but using modern features of the latest versions and semi-functional style of chained computations I am fond of.

This is not to say the original author's code was "bad," and I made it "good"—nothing of the kind. I just experimented with how things _might_ be expressed so they would be closer to my reading/writing habits—just as _some_ goal making me understand them.

The result of the project is the diary below. By the end of 24 advent days, I haven't seen/read/rewritten all parts of the algorithm—that would require trice this amount of time, probably. But I went through all the layers of the picture and got _some_ understanding on how things are done there—from a singular brushstroke to generating a whole mountain with a small village and a forest. And I didn't lose my awe, and I am trying to pass it to others.

For the impatient: you can probably just go through the [day 1 intro](/advent2021/day01.html) and then go immediately to [day 24](/advent2021/day24.html) with the overview of everything I've understood on the road. But I hope that reading through the whole diary might also be of some fun. There are small philosophical bits here and there, too, discussing how the code _might_ be written, and it can be either insightful or repulsing, or both.

> Note that JS is not my mother tongue. While using modern features for better expression of the meaning, I probably still went far for the modern preferred JS _style_: say, I dropped a lot of `var`s everywhere (never bothering with `let`/`const`, though I probably should've) and didn't use any linter/autoformatter. So it might be a somewhat painful reading or a JS developer by trade.
>
> And English is not my first language either. Usually, I use at least Grammarly to make sure the texts are decent... But with the amount of work this project brought, I can only promise the diary was spellchecked. Have mercy.

So, here goes!

## The diary of grokking!

* **[Day 01: Advent of `Grok {Shan, Shui}*`](/advent2021/day01.html)**—Intro to the project and the way I am planning (planned) to do it.
* **[Day 02: Overview and starting to change the code](/advent2021/day02.html)**—Analysing the whole, and starting to read & rewrite one of the tree-generating functions, `tree01`...
* **[Day 03: Fighting the tree](/advent2021/day03.html)**—...and continue...
* **[Day 04: Making sense of the tree](/advent2021/day04.html)**—...and continue; also look into `poly` and `blob` low-level shapes-generating functions.
* **[Day 05: Moar treees!](/advent2021/day05.html)**—Finalize `tree01` and look into another tree, `tree02`,
* **[Day 06: Even moar treees!](/advent2021/day06.html)**—...and `tree03`.
* **[Day 07: Build me a house](/advent2021/day07.html)**—With trees being (somewhat) understood, starting to look into how architectural elements are made, starting with `arch02`; which turned out to be a tough nut to crack. A lot of simple arithmetic, but, like, a lot of it! So I started with the `box` function for drawing parallelepipeds...
* **[Day 08: What's in a box?](/advent2021/day08.html)**—...and spent on it...
* **[Day 09: Lines and boxes](/advent2021/day09.html)**—...good...
* **[Day 10: (there are no good puns with the word "stroke")](/advent2021/day10.html)**—...few days, finishing with other small(ish) functions.
* **[Day 11: Back to home](/advent2021/day11.html)**—And with that, considering the `arch02` logic more or less understood.
* **[Day 12: Seeing the forest behind the trees](/advent2021/day12.html)**—Starting anew, from top-to-bottom now, I started to look into `chunkloader` function that created big parts of the landscape.
* **[Day 13: Plan for the mountains](/advent2021/day13.html)**—...and into the `mountplanner` function that decides which landscape feature would be where...
* **[Day 14: Mountain-planning, continued](/advent2021/day14.html)**—..and looked into how it decides on a list of big objects.
* **[Day 15: Between the landscape and the tree](/advent2021/day15.html)**—Then, I started with the last big endeavor: the thing between chunk and a singular tree: a `mountain` with all the contours, trees, and houses.
* **[Day 16: Deeper into the mountain](/advent2021/day16.html)**—...and...
* **[Day 17: Let's Vegetate!](/advent2021/day17.html)**—...continued...
* **[Day 18: Generation details](/advent2021/day18.html)**—...to do...
* **[Day 19: Wrapping up the mountain](/advent2021/day19.html)**—...so...
* **[Day 20: REALLY wrapping up the mountain](/advent2021/day20.html)**—...for a while.
* **[Day 21: Now what?](/advent2021/day21.html)**—Then, I reviewed the big picture: planning and rendering "chunks"
* **[Day 22: Where were we?..](/advent2021/day22.html)**—...and how it is made truly infinite.
* **[Day 23: Housekeeping](/advent2021/day23.html)**—The technical one, explaining how I'll organize the project/diary to wrap it up.
* **[Day 24: Putting it all together](/advent2021/day24.html)**—The most important one, summarizing all of the findings and outcomes of the investigation!

## PS

You know what? I have a Substack now! And _a lot_ of plans for writings for the upcoming year.

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
