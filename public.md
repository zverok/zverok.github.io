---
layout: default
---

# Public activity

## Code

* **[molybdenum-99](https://github.com/molybdenum-99)** project: developing a set of libraries to represent real-world data in Ruby:
  * [reality](https://github.com/molybdenum-99/reality) -- Most hi-level gem of the project, representing all world's data as Ruby entities;
  * [infoboxer](https://github.com/molybdenum-99/infoboxer) -- Wikipedia client and parser, targeting data extraction;
  * [mediawiktory](https://github.com/molybdenum-99/mediawiktory) -- low-level MediaWiki API client that just works;
  * [tlaw](https://github.com/molybdenum-99/tlaw) -- Pragmatic API wrapper framework;
  * [wheretz](https://github.com/zverok/wheretz) -- fast and precise _offline_ time zone by geo coordinates lookup;
  * [geo_coord](https://github.com/zverok/geo_coord) -- `Geo::Coord` class, abstracting `[latitude, longitude]` pair;
  * [tz_offset](https://github.com/molybdenum-99/tz_offset) -- simple abstraction of TimeZone offset;
  * [whatis](https://github.com/molybdenum-99/whatis) -- `WhatIs.this()`: simple entity resolution through Wikipedia;
* Experimenting on Ruby edges:
  * [hm](https://github.com/zverok/hm) -- idiomatic nested hash transformations;
  * [time_math2](https://github.com/zverok/time_math2) -- no-`core_ext` library for handling simple time arithmetics, like "round time to the day edge" or "add two months to current date" or "generate hour-by-hour sequence";
  * [idempotent_enumerable](https://github.com/zverok/idempotent_enumerable) -- like `Enumerable`, but preserves type of initial collection;
  * [`Object#enumerate`](https://github.com/zverok/object_enumerate) and [`Enumerator#generate`](https://github.com/zverok/enumerator_generate) -- proposals to extend Ruby enumeration API, demonstrated in a small gems.
* Development tools:
  * [yard-junk](https://github.com/zverok/yard-junk) -- [YARD](https://github.com/lsegal/yard) plugin for checking for errors in docs;
  * [saharspec](https://github.com/zverok/saharspec) -- a set of RSpec addons for DRY-er specs;
  * [dokaz](http://github.com/zverok/dokaz) -- test code from inside of your Markdown documentation files;
  * [any_good](https://github.com/zverok/any_good) -- quick command-line evaluation of Ruby gem freshness/popularity
* Graphic gems and other experiments:
  * [worldize](https://github.com/zverok/worldize) -- simple coloured countries stats drawing with RMagick;
  * [magic_cloud](http://github.com/zverok/magic_cloud) -- pretty Wordle-like wordcloud in pure Ruby;
  * [xkcdize](https://github.com/zverok/xkcdize) and [drosterize](https://github.com/zverok/drosterize) -- experiments on converting some "cool" image-processing tricks from Wolfram Language (Mathematica) to Ruby + RMagick;
* Other stuff:
  * [linkhum](https://github.com/zverok/linkhum) -- "links-in-text humanizer", linkify text-only input;
  * [did_you](https://github.com/zverok/did_you) -- easy-to-use wrapper for did_you_mean bundled Ruby gem (which dramatically changed behavior between versions)

## Mentoring

* [At mkdev.me](https://mkdev.me/en/mentors/zverok) as a paid mentor;
* [At SciRuby](http://sciruby.com/) as a maintainer of some projects and Google Summer of Code volunteer mentor;
* [Let's Make Something Awesome](https://github.com/zverok/lmsa) is my personal list of projects I'd like to mentor.

## Docs and texts

* [Blog](/blog/) is full of rants about Ruby, RSpec, my projects and development in general;
* [Ruby Reference](https://rubyreferences.github.io/rubyref/) is an _always actual_ current Ruby version reference, compiled from official docs and other stuff in a readable e-book format;
* [Good Value Object Conventions](https://github.com/zverok/good-value-object) is an RFC-alike document for Ruby Value Objects;
* [Participant](https://bugs.ruby-lang.org/users/710) of Ruby language development discussions (I'm that guy who invented `Kernel#then`, so sue me), and committer of documentation improvements to Ruby and its standard library.

Appearances in [Ruby Weekly](https://rubyweekly.com/): [#250](https://rubyweekly.com/issues/250) (xkcdize), [#287](https://rubyweekly.com/issues/287) (reality), [#300](https://rubyweekly.com/issues/300) (time_math2), [#301](https://rubyweekly.com/issues/301) (Geo::Coord), [#319](https://rubyweekly.com/issues/319) (blog post), [#322](https://rubyweekly.com/issues/322) (tz_offset), [#364](https://rubyweekly.com/issues/364) (yard-junk), [#366](https://rubyweekly.com/issues/366) (did_you), [#372](https://rubyweekly.com/issues/372) (blog post), [#375](https://rubyweekly.com/issues/375) (blog post), [#383](https://rubyweekly.com/issues/383) (any_good), [#384](https://rubyweekly.com/issues/384) (blog post), [#385](https://rubyweekly.com/issues/385) (blog post), [#387](https://rubyweekly.com/issues/387) (hm), [#393](https://rubyweekly.com/issues/393) (blog post), [#396](https://rubyweekly.com/issues/396) (Ruby core discussion), [#403](https://rubyweekly.com/issues/403) (sho).

## Talks and conferences

* **When the whole world is your database**: about [reality](https://github.com/molybdenum-99/reality), its reasons and implementation challenges;
  * RubyConf India 2018 ([video](https://www.youtube.com/watch?v=x9GePP3B0oE&t=1s&list=PLe872Yf6CJWGYKLny9jFs9mLv0Z94m8k4&index=26), [slides](https://docs.google.com/presentation/d/1I4mznHUBhVVDxWfO2DRzxP4wNhs9Mmtx09SizLqIbaE/edit?usp=sharing));
  * RubyConf Kenya 2017;
  * Meetups: RubyMeditation, Pivorak, Vilnius.rb
* **The curious case of Wikipedia parsing**: about [infoboxer](https://github.com/molybdenum-99/infoboxer) and challenges of parsing wikipedia and hi-level information extraction:
  * RubyKaigi 2017 ([video](https://www.youtube.com/watch?v=oqsX8kNq94I), [slides](https://docs.google.com/presentation/d/1r3xUjc9nXlwAOmgzCI26lELdNp8Mnsd5sfXa9JJwIME/edit?usp=sharing))
  * RubyConf Kenya 2017
  * Meetups: RubyMeditation
* **The tale of query languages**: about requirements to query languages in data-rich world, and whether GraphQL suits all of those requirements:
  * RubyC (Ukraine) 2018: ([video](https://www.youtube.com/watch?v=vLbcqtrh6Ys), [slides](https://docs.google.com/presentation/d/1u9K-GiLNQyQm5JwDQfcUE-JM43u_a_ATmEDk1ZlHwEM/edit?usp=sharing))
* **Ruby Rogues Podcast**: [Ruby Core Language Evolution: Moving towards functional with Victor Shepelev](https://devchat.tv/ruby-rogues/rr-367-ruby-core-language-evolution-moving-towards-functional-with-victor-shepelev)