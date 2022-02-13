---
layout: page
title: Projects
permalink: /projects/
---

<div class="callout">A list of public/open-source things I am working on, with a weak attempt of most important-first ordering.</div>

* Gimme TOC
{:toc}

## Ruby {#ruby}

I am an active contributor to Ruby programming language. My main interests are making the language more naturally expressive, and more friendly for the newcomers.

* [List of my contributions to the language itself](/ruby.html): documentation, and some features mostly on idea level, like `Kernel#then`, beginless ranges, `Enumerator#produce` etc.
* [Ruby Changes](https://rubyreferences.github.io/rubychanges/) is full changelog of recent Ruby (2.4—3.1 at the moment), with full context and example for each and every change.
* [Ruby Reference](https://rubyreferences.github.io/rubyref/) is an attempt to generate full Ruby reference from official docs and other stuff in a readable e-book-alike format; this year, there is a work planned to make it an official version at `ruby-lang.org`.

## Common world knowledge API {#wikipedia_ql}

Since 2016, I am experimenting with possibilities to make common open knowledge easily accessible from scripting languages.

Current ongoing effort is focusing on a **Python library [WikipediaQL](https://github.com/zverok/wikipedia_ql)**.

Before that, I developed a set of Ruby libraries, also mostly based on Wikipedia/Wikidata (with some sprinkles of OpenStreetMap and other services). The whole effort was named **[molybdenum-99](https://github.com/molybdenum-99)** (as a pun on Wolfram language which was the big inspiration), and peeked in [reality](https://github.com/molybdenum-99/reality)—the most hi-level gem of the project, representing all world's data as Ruby entities. Beneath it, multiple libraries were developed:

* [infoboxer](https://github.com/molybdenum-99/infoboxer): Wikipedia client and parser, targeting data extraction;
* [mediawiktory](https://github.com/molybdenum-99/mediawiktory): low-level MediaWiki API client that just works;
* [tlaw](https://github.com/molybdenum-99/tlaw): pragmatic API wrapper framework;
* [wheretz](https://github.com/zverok/wheretz): fast and precise _offline_ time zone by geo coordinates lookup;
* [geo_coord](https://github.com/zverok/geo_coord): `Geo::Coord` class, abstracting `[latitude, longitude]` pair;
* [tz_offset](https://github.com/molybdenum-99/tz_offset): simple abstraction of time zone offset;
* [whatis](https://github.com/molybdenum-99/whatis): `WhatIs.this()`: simple entity resolution through Wikipedia;
* [mormor](https://github.com/molybdenum-99/mormor): Morfologik dictionaries client in pure Ruby: POS tagging & spellcheck

## Rebuilding the spellchecker {#spylls}

In 2021, I built **[Spylls](https://github.com/zverok/spylls)**: pure Python spell-checker, _"explanatory"_ full port of the most popular open source spellchecker [Hunspell](https://hunspell.github.io/).

I documented the journey to do this and my discoveries in [a series of posts](/spellchecker.html).

## Ruby libraries

* Experimenting on Ruby edges:
  * [time_calc](https://github.com/zverok/time_calc) -- Simple time arithmetic in a modern, readable, idiomatic, no-"magic" Ruby (previous approach to the problem: [time_math2](https://github.com/zverok/time_math2));
  * [sho](https://github.com/zverok/sho) -- "post-framework" views library.
  * [hm](https://github.com/zverok/hm) -- idiomatic nested hash transformations;
  * [delegates](https://github.com/zverok/delegates) -- `delegate :methods, to: :target`, extracted from ActiveSupport;
  * [fstrings](https://github.com/zverok/fstrings) -- Python-alike formatting strings;
* Development tools:
  * [the_schema_is](https://github.com/zverok/the_schema_is) -- Rails DSL for model annotation with DB schema, done right
  * [whatthegem](https://github.com/zverok/whatthegem) -- information about any Ruby gem in your terminal: general information, usage examples, popularity stats, changes and more (successor of [any_good](https://github.com/zverok/any_good) -- quick command-line evaluation of Ruby gem freshness/popularity)
  * [yard-junk](https://github.com/zverok/yard-junk) -- [YARD](https://github.com/lsegal/yard) plugin for checking for errors in docs;
  * [saharspec](https://github.com/zverok/saharspec) -- a set of RSpec addons for DRY-er specs;
  * [dokaz](http://github.com/zverok/dokaz) -- test code from inside of your Markdown documentation files;
* Other stuff:
  * [linkhum](https://github.com/zverok/linkhum) -- "links-in-text humanizer", linkify text-only input;
  * [did_you](https://github.com/zverok/did_you) -- easy-to-use wrapper for did_you_mean bundled Ruby gem (which dramatically changed behavior between versions)

## Fun and experiments

* [Grok {Shan, Shui}\*: Advent of understanding the generative art](/blog/2021-12-28-grok-shan-shui.html)
* [Game of Life in one Ruby statement... inspired by APL](/blog/2020-05-16-ruby-as-apl.html)
* [worldize](https://github.com/zverok/worldize) -- simple colored countries stats drawing with RMagick;
* [magic_cloud](http://github.com/zverok/magic_cloud) -- pretty wordcloud in pure Ruby, done a long time ago;
* [xkcdize](https://github.com/zverok/xkcdize) and [drosterize](https://github.com/zverok/drosterize) -- experiments on converting some "cool" image-processing tricks from Wolfram Language (Mathematica) to Ruby + RMagick;
