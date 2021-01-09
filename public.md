---
layout: page
title: Open source and other public activity
permalink: /public.html
---

<a class="github-button" href="https://github.com/zverok" data-size="large" aria-label="Follow @zverok on GitHub">Follow @zverok</a>

<a href="https://www.patreon.com/bePatron?u=4691811" data-patreon-widget-type="become-patron-button">Become a Patron!</a><script async src="https://c6.patreon.com/becomePatronButton.bundle.js"></script>

## Docs for Ruby

* [Ruby Reference](https://rubyreferences.github.io/rubyref/) is an _always actual_ current Ruby version reference, compiled from official docs and other stuff in a readable e-book format;
* [Ruby Changes](https://rubyreferences.github.io/rubychanges/) is full changelog of recent Ruby (2.4—3.0 at the moment), with full context and example for each and every change.

## Code

* **[Ruby lang contributions](https://github.com/zverok/my-ruby-contributions)** list of my contributions to Ruby language (proposed features and written documentation)
* **[Spylls](https://github.com/zverok/spylls)** -- Pure Python spell-checker, _"explanatory"_ full port of Hunspell;
* Experimenting on Ruby edges:
  * [time_calc](https://github.com/zverok/time_calc) -- Simple time arithmetics in a modern, readable, idiomatic, no-"magic" Ruby (previous approach to the problem: [time_math2](https://github.com/zverok/time_math2));
  * [sho](https://github.com/zverok/sho) -- "post-framework" views library.
  * [hm](https://github.com/zverok/hm) -- idiomatic nested hash transformations;
  * [delegates](https://github.com/zverok/delegates) -- `delegate :methods, to: :target`, extracted from ActiveSupport;
  * [fstrings](https://github.com/zverok/fstrings) -- Python-alike formatting strings;
  * [idempotent_enumerable](https://github.com/zverok/idempotent_enumerable) -- like `Enumerable`, but preserves type of initial collection;
* Development tools:
  * [the_schema_is](https://github.com/zverok/the_schema_is) -- Rails DSL for model annotation with DB schema, done right
  * [whatthegem](https://github.com/zverok/whatthegem) -- information about any Ruby gem in your terminal: general information, usage examples, popularity stats, changes and more (successor of [any_good](https://github.com/zverok/any_good) -- quick command-line evaluation of Ruby gem freshness/popularity)
  * [yard-junk](https://github.com/zverok/yard-junk) -- [YARD](https://github.com/lsegal/yard) plugin for checking for errors in docs;
  * [saharspec](https://github.com/zverok/saharspec) -- a set of RSpec addons for DRY-er specs;
  * [dokaz](http://github.com/zverok/dokaz) -- test code from inside of your Markdown documentation files;
* Graphic gems and other experiments:
  * [worldize](https://github.com/zverok/worldize) -- simple coloured countries stats drawing with RMagick;
  * [magic_cloud](http://github.com/zverok/magic_cloud) -- pretty Wordle-like wordcloud in pure Ruby;
  * [xkcdize](https://github.com/zverok/xkcdize) and [drosterize](https://github.com/zverok/drosterize) -- experiments on converting some "cool" image-processing tricks from Wolfram Language (Mathematica) to Ruby + RMagick;
* **[molybdenum-99](https://github.com/molybdenum-99)** project: developing a set of libraries to represent real-world data in Ruby:
  * [reality](https://github.com/molybdenum-99/reality) -- Most hi-level gem of the project, representing all world's data as Ruby entities;
  * [infoboxer](https://github.com/molybdenum-99/infoboxer) -- Wikipedia client and parser, targeting data extraction;
  * [mediawiktory](https://github.com/molybdenum-99/mediawiktory) -- low-level MediaWiki API client that just works;
  * [tlaw](https://github.com/molybdenum-99/tlaw) -- Pragmatic API wrapper framework;
  * [wheretz](https://github.com/zverok/wheretz) -- fast and precise _offline_ time zone by geo coordinates lookup;
  * [geo_coord](https://github.com/zverok/geo_coord) -- `Geo::Coord` class, abstracting `[latitude, longitude]` pair;
  * [tz_offset](https://github.com/molybdenum-99/tz_offset) -- simple abstraction of TimeZone offset;
  * [whatis](https://github.com/molybdenum-99/whatis) -- `WhatIs.this()`: simple entity resolution through Wikipedia;
  * [mormor](https://github.com/molybdenum-99/mormor) -- Morfologik dictionaries client in pure Ruby: POS tagging & spellcheck
* Other stuff:
  * [linkhum](https://github.com/zverok/linkhum) -- "links-in-text humanizer", linkify text-only input;
  * [did_you](https://github.com/zverok/did_you) -- easy-to-use wrapper for did_you_mean bundled Ruby gem (which dramatically changed behavior between versions)

<!--
## Mentoring

* [At mkdev.me](https://mkdev.me/en/mentors/zverok) as a paid mentor;
* [Let's Make Something Awesome](https://github.com/zverok/lmsa) is my personal list of projects I'd like to mentor;
* **Retired:** ~~[At SciRuby](http://sciruby.com/) as a maintainer of some projects and Google Summer of Code volunteer mentor.~~
-->

## Docs and texts

* [Blog](/blog/) is full of rants about Ruby, RSpec, my projects and development in general;
* [Good Value Object Conventions](https://github.com/zverok/good-value-object) is an RFC-alike document for Ruby Value Objects;
* [Participant](https://bugs.ruby-lang.org/users/710) of Ruby language development discussions (I'm that guy who invented `Kernel#then`, so sue me), and committer of documentation improvements to Ruby and its standard library.

***

**Appearances in [Ruby Weekly](https://rubyweekly.com/):** [#250](https://rubyweekly.com/issues/250) (xkcdize), [#287](https://rubyweekly.com/issues/287) (reality), [#300](https://rubyweekly.com/issues/300) (time_math2), [#301](https://rubyweekly.com/issues/301) (Geo::Coord), [#319](https://rubyweekly.com/issues/319) (blog post), [#322](https://rubyweekly.com/issues/322) (tz_offset), [#364](https://rubyweekly.com/issues/364) (yard-junk), [#366](https://rubyweekly.com/issues/366) (did_you), [#372](https://rubyweekly.com/issues/372) (blog post), [#375](https://rubyweekly.com/issues/375) (blog post), [#383](https://rubyweekly.com/issues/383) (any_good), [#384](https://rubyweekly.com/issues/384) (blog post), [#385](https://rubyweekly.com/issues/385) (blog post), [#387](https://rubyweekly.com/issues/387) (hm), [#393](https://rubyweekly.com/issues/393) (blog post), [#396](https://rubyweekly.com/issues/396) (Ruby core discussion), [#403](https://rubyweekly.com/issues/403) (sho), [#431](https://rubyweekly.com/issues/431) (Ruby Changes), [#432](https://rubyweekly.com/issues/432) (Ruby Reference), [#458](https://rubyweekly.com/issues/458) (time_calc), [#472](https://rubyweekly.com/issues/472) (Ruby Changes), [#473](https://rubyweekly.com/issues/473) (blog post), [#482](https://rubyweekly.com/issues/482) (Ruby Changes) [#483](https://rubyweekly.com/issues/483) (Ruby Reference and fstrings), [#502](https://rubyweekly.com/issues/502) (blog post), [#532](https://rubyweekly.com/issues/532) & [#533](https://rubyweekly.com/issues/533) (Ruby Changes).
