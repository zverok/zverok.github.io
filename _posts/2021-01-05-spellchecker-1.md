---
layout: post
title:  "What I learned while recreating the most popular spellchecker. Part 1"
date:   2021-01-05
categories: python spellchecker
comments: true
---

## Part 1. How I decided to write a spellchecker and almost died trying

A few years ago I had a fun idea for a "weekend project": pure-Ruby spellchecker. Ruby is my language of choice, and no-dependencies spellchecker seemed a small useful tool for the CI environment: for example, to check comments/docs spelling without installing any third-party software. I actually _could've_ pulled out the project in its limited scope (only English, only spot misspelled words without fixing, limited dictionary) with just a flat list of known words, but that's not what happened.

Back then, I decided to make a moderately generic tool, at least able to work with multiple languages. Fortunately (or so I believed!), there were many already existing and freely available spellchecking dictionaries distributed as LibreOffice and Firefox extensions. All of those dictionaries are in the format defined by the **[Hunspell](http://hunspell.github.io/)** tool/library—which is an open-source library that is used for spellchecking in Libre/OpenOffice, Mozilla products (Firefox, Thunderbird), but also Google Chrome/Chromium, macOS, several Adobe products, and so on.

The dictionaries looked like easy to reuse text files with some ("insignificant" as it seemed) metadata, and the whole "use Hunspell dictionaries from pure Ruby spellchecker" project _still_ felt like a "weekend-long" one, for the first few weekends. Only gradually the underwater complexity of the multilanguage word-by-word spellchecking uncovered. Eventually, I was distracted from the project and abandoned it, but I still had the fascination with the seemingly-simple, actually-mind-blowingly-complicated Hunspell, the software everybody used daily and hardly ever notice.

The idea to dig deeper into it, to _understand_ it and _explain_, grew on me and bothered me for quite some time. And what is a better way to understand something, if not to retell it in your own words? After several lazy and not very far-progressed attempts to write something Hunspell-alike (twice in Ruby, once in Rust, once in Python), eventually, in February 2020, the task I settled down to solve is: "explanatory rewrite" of the Hunspell into high-level language with a lot of comments. I achieved this goal by December 2020, with the first release of the [Spylls](https://github.com/zverok/spylls) project: **the port of Hunspell's core algorithms into modern, well-documented, well-structured Python**.

And now I want to share some insights of what I uncovered on the road: about spellchecking in general and Hunspell in particular.

In the ongoing article series, I'll cover these topics:

* What is Hunspell, why is it significant, and why try to "explain" it (current article)
* Base spellchecking concepts: lookup and suggest, as seen by Hunspell
* How lookup (checking if the word is correct) works, and why it could be much more complicated than "just look in the list of the known words"
* How suggest (proposed fix for the incorrect word) works, and how hard it is to estimate its quality
* A closer look into Hunspell's dictionary format. It is the most widespread open dictionary format in the world, and we'll see what linguistic and algorithmic information it _potentially_ can carry, and what part of it is actually used in existing dictionaries
* Some details on Spylls implementation process and results
* Closing thoughts on the big picture of word-by-word spellchecker problem, and Hunspell's approach to it

### What is Hunspell?

> **Note:** The information on Hunspell's origins and history is mostly my guesses, following partial and incomplete sources everywhere.

Hunspell (<small>[Wikpedia article](https://en.wikipedia.org/wiki/Hunspell)</small>), initially **Hun**garian spellchecker, emerged as an alternative for previously existing aspell/ispell/myspell somewhere in 2002 (I guess?). It was created by László Németh, in a need of supporting languages with complicated suffixing/prefixing rules and word compounding (such as Hungarian). Hunspell's design seemingly proved itself to be flexible enough to support most of the world's languages, and in a few years, it became the most used spellchecker in the world. You have most probably used it even if you've never heard the name before today: Hunspell is the default spellchecking engine in Chrome and Firefox, Libre/OpenOffice, Adobe products, and macOS (not an exhaustive list). Dictionaries in Hunspell format exist for almost all actively used languages for which the concept of word-by-word spellchecking makes sense[^1].

[^1]: Word-by-word spellchecking makes less sense for hieroghliphic languages like Chinese and Japanese; it is also problematic for languages where words aren't separated by whitespaces, like Lao or Thai.

Currently, Hunspell is maintained [on GitHub](https://github.com/hunspell/hunspell) (repo has only around 1k stars, will you believe it?). It seems that maintenance is not that easy if you'll weight the number of open issues and PRs, and the latest commits timeline: at the time of writing it (Jan 2021), the last commit to master was of May 2020, and the last release was 1.7 on Dec 2018. Hunspell's codebase is mostly "old-school" C++. It is being slowly modernized and it has very few comments; there are thousands of two-branch `if`s to handle non-Unicode and Unicode text separately. There is also an attempt to rewrite Hunspell from scratch in a modern C++, which at some point was developed under the `hunspell` GitHub organization. Now it is independent and called [nuspell](https://github.com/nuspell/nuspell) (and, while not yet supporting all of the Hunspell features, already "achieved" version 4.2.0).

Obviously, there are open-source spellcheckers other than Hunspell. GNU aspell (that at one point was superseded by Hunspell, but still holds its ground in English suggestion quality), to name one of the older ones; but also there are novel approaches, like [SymSpell](https://github.com/wolfgarbe/SymSpell), claiming to be "1 million times faster" or ML-based [JamSpell](https://github.com/bakwc/JamSpell), claiming to be much more accurate.

And yet, what makes Hunspell stand out is its coverage of the world's languages. It is not ideal, but the amount of dictionaries ready to use immediately, and amount of _experience_ of dealing with typical problems and corner cases, coded into the codebase, is hard to beat.

### Why rewrite it?

As I've already stated above, the goal of the Spylls project was to create an _explanatory_ rewrite: E.g., the "retelling" of how Hunspell works in a way that is easy to follow and to play with.

The necessity of this approach came to me from three facts:

1. Hunspell is used almost everywhere and is taken for granted;
2. It is much more complicated than one might naively expect;
3. This complexity—and years of human work that was spent growing the project—is notoriously hard to follow through the Hunspell's codebase and grasp in full.

In other words, I wanted to **make the knowledge behind Hunspell more open**.

The way I have chosen was not, of course, the only one possible. I could've just read through the original code and write a series of articles (or, rather, a book?) on how it works. I could've thoroughly commented and republished the original source code. But I felt that _reimplementing_ is the only way of understanding what's and why's of the algorithms (at least for somebody not being a Hunspell's core developer); and that implementation in a high-level language will allow focusing on words and language-related algorithms, not memory management or fighting with Unicode.

> Note that there are also a few "pragmatic" ports of Hunspell into other languages (in order to use it in environments where C++ dependency is undesireable), namely [WeCantSpell.Hunspell](https://github.com/aarondandy/WeCantSpell.Hunspell) in C# and [nspell](https://github.com/wooorm/nspell) in JS (very incomplete); and aforementioned [nuspell](https://github.com/nuspell/nuspell) can also be considered a "port" (from legacy C++ to a modern one).

### Why Python?

My language of choice is Ruby. It was also the first language that I've tried to port Hunspell into. I'd be happy to proceed with Ruby if my goal has been just a "pragmatic" library. And yet, when I decided that my goal is to make the knowledge of Hunspell's algorithms accessible to a wide audience, I understood that Ruby is not the best choice: language reputation (slightly esoteric and mostly-for-web) would make my project lest noticeable; and my preferred coding style (mix of OO and functional, with lots of small immutable domain objects and fluent chains of iterators), while allowing me to be very effective, would make the code less accessible to other languages users.

What I needed was a high-level language, with as low boilerplate as possible; as mainstream as possible; as easy to experiment with and prototype as possible. Without diving into too much argument here, Python and modern JavaScript seemed to be the most suitable options, and, to be honest, Python was just closer to my soul. So, here we are!

The code style is mostly imperative (as it corresponds to how Hunspell is structured), with large-ish, but clearly structured methods, and a small number of classes/objects (mostly they are either "whole algorithm as a class" or almost-passive "structs" -- or, in Python, dataclasses). I tried to limit myself in the usage of complex Python-specific features (like functools or itertools), but have a decent use of "list comprehensions" (as they are quite readable and Pythonic) and generators (lazy lists). Overall, I wanted the code to be good Python, but not too smart. Whether I succeeded, is up to you to decide.

Currently, [Spylls](https://github.com/zverok/spylls) has **≈1.5k lines of library code** in 14 files. It conforms (with [some reservations](https://spylls.readthedocs.io/en/latest/#completeness)) to all Hunspell's integrational tests. Those tests look like a set of files each, consisting of "test dictionary + what words should be considered good, what words should be considered bad, what should be suggested instead of the bad words", and there are **127 of such sets to pass**. There are **2 thousand comment lines** in the code, explaining thoroughly every detail of the algorithm and rendered at the [Spylls documentation site](https://spylls.readthedocs.io/en/latest/hunspell.html); note that besides docstrings at the beginning of each class and method, there are also inline comments in code—that's why the documentation site uses custom theme with inline "Show code" feature.

***

With this being said, I am wrapping up the introductory post.

**In the next series: An introduction to Hunspell's "lookup" and "suggest" concepts; and deeper dive into the lookup.**
