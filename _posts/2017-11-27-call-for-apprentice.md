---
layout: post
title:  Call for apprentice
date:   2017-11-27 14:36:00
categories: ruby mentorship
comments: true
---

**TL;DR**: I propose a **mentorship** for those ready to collaborate on a several **opensource** projects for **motivated** developers.

You will:
* work on one of projects, started or invented by me,
* following the plan we'll make together,
* through GitHub issues and PRs,
* with extensive and regular code reviews, advice and hints.

In a process, you'll have a practical experience of writing good code, structuring it, optimizing, testing and documenting, as well as an overall experience of real-world gem development participation — and something to add to your CV as a project you've created, maintained or participated in.

## Me

Commercial programming since 2003, Ruby programming since 2005. [Ruby Association Grant-2015](http://www.ruby.or.jp/en/news/20160406), several maintained [gems](http://github.com/zverok), one of core maintainer of [SciRuby](https://github.com/sciruby) organization, creator of [Molybdenum](https://github.com/molybdenum-99) project. Several years of mentoring experience, both offline and online, paid, volunteer and opensource, including Google Summer of Code. Several international meetups and conferences participation, including [RubyKaigi-2017](http://rubykaigi.org/2017/presentations/zverok.html).

In other words, I am a passionate and experienced developer and mentor, and I have a lot of opensource projects and ideas and not enough time to implement them all. The way of this proposal, I see a possibility of interesting things to be done and thrive.

## You

* Have some experience with Ruby and enjoy it;
* Have some (at least theoretical) understanding of testing with RSpec;
* Ready to learn by yourself and question everything you learn;
* Find yourself deeply interested in one of the projects listed above;
* Going to spend time on a project on a regular basis.

The proposal is **not for you** if:

* You know nothing about Ruby or programming, but "really want to learn";
* You already know your way of doing things, structuring code, writing tests (or avoid them), and just poking around;
* You don't give a damn about particular project, just want a free mentor;
* Will try to do something eventually, probably next month, or next year.

I believe that the only way to learn is being ready to learn, take a lot of responsibility on yourself and be excited but what you are about to do.

## Projects

### `yard-junk` -- documentation quality checker

**[yard-junk](https://github.com/zverok/yard-junk)** is already established and production-used solution for checking the quality of [YARD](https://github.com/lsegal/yard) documentation of Ruby projects. The current version only checks for errors is documentation (misused tags and such), but the next step would be to introduce Rubocop-style flexible quality checks, allowing to establish and maintain per-project preferred documentation style. The plan is briefly discussed in [this](https://github.com/zverok/yard-junk/issues/14) GitHub issue.

YARD is the de-facto standard of Ruby documentation (alongside with RDoc, but `yard` as a tool supports RDoc too, so `yard-junk` can be useful for checking virtually any Ruby project's docs). Establishing a tool for docs quality checking (and auto-fixing if possible) seems to be of general use.

While working on it, you'll need dive really deep into YARD tool internals, and create modular and pluggable architecture like Rubocop with its cops.

### `tlaw` -- generic HTTP API wrapper

**[tlaw](https://github.com/molybdenum-99/tlaw)** is a somewhat experimental library/DSL for creating fast, thin and reliable wrappers for HTTP APIs. Currently, it supports only GET APIs and lacks some deeper HTTP features (like headers passing or authorization), yet already proven to be a promising way to the "right" thickness balance. Current idea is to go through [public cloudsourced JSON API list](https://github.com/toddmotto/public-apis) and make a repository of usable wrappers for the most interesting of them, adding missing features to TLAW in process.

This task will require a lot of experimenting with API and architecture, and serious intent to write _less_ code, and rewrite a lot, and document a lot.

### Project idea: RUDE -- RUby Documentation Effort

The idea is pretty rough yet clean: parse docs from Ruby's official Repo (and from all tags, starting from 2.0) and format it in one informative static site (published with GitHub pages), auto-updated on new versions. Proposed differences from existing projects (say, ruby-doc.org):

* Ruby-version agnostic URLs (instead of having `.../2.4.2/Enumerable.html`, have `/Enumerable.html` and effectively render all version-dependent differences there);
* Better representation of "language core" docs (which are currently in `doc/*.rdoc` files of language repo);
* more compact, easily browsable, modern representation.

You'll do a lot of text parsing, preprocessing and formatting, probably hacking with RDoc internals, some basic UI design.

### Project idea: `hunspell` port to Ruby

[Hunspell](https://en.wikipedia.org/wiki/Hunspell) is currently the most popular open source spellchecking tool, having most of the actual dictionaries in its format. But [hunspell](https://github.com/hunspell/hunspell) itself is pretty complicated C++ software, that is hard to integrate and use from Ruby. I propose to create pure-Ruby Hunspell format dictionaries reading and application library, which can be easily integrated with other Ruby tools, like Markdown parsers (or even Ruby [parser](https://github.com/whitequark/parser), imagine you can spellcheck your Rake task descriptions?), Jekyll, CI tools and so on.

You'll need to be able to at least read C++ of hunspell's sources. And expect **a lot** of optimization practice.

### Project idea: `languagetool` port to Ruby

The same as above, but related to prominent [LanguageTool](https://languagetool.org/) style and grammar checker. This project is much more ambitious, as LanguagTool's rules format (deeply nested XML) and logic are much more complicated than simple Hunspell's dictionaries; besides that, not all of grammar checking logic is stored in data, some rules and services are represented by Java classes. Nevertheless, the task is bearable, and first useful results could be achieved rather quickly. The gain for the community (integrating style checks into checking markdown, docs, running in CI pipelines) seem to be even more than with spellchecker.

The task will include a reading of Java, obviously some linguistics stuff, a lot of optimizations and comprehensive text processing.

### `magic_cloud` -- drawing of word clouds

**[magic_cloud](https://github.com/zverok/magic_cloud)** is somewhat outdated, yet production-used pretty word cloud generator. While most of the today's visualizations are indeed done in a browser with D3.js or something like it, server-side generation still can be useful (for PDF reports, including in emails, visualizing simple scripting experiments, this kind of things). magic_cloud can be done faster, more configurable, with cleaner API. This task will include a lot of tinkering with image libraries (maybe porting to vips?) and optimization (calculating precise word positions in "pretty randomness" is _slooow_).

### `worldize` -- drawing of geographical data

**[worldize](https://github.com/zverok/worldize)** is an early prototype of geographical drawing library, but when it was published, it received some good amount of attention. There is a clear plan of development (different projections and map slices, generic API like `line(geo_point, geo_point)`, `marker(geo_point, text)` and stuff), but unfortunately, I never had enough time to continue. The reasons for "why somebody could need this" is the same as for the previous gem: static (PDF and images) rendering, pure-Ruby experimental scripts, and so on.

## How our interaction will look

* You apply to some of the projects (by dropping me a line at [zverok.offline@gmail.com](mailto:zverok.offline@gmail.com) or contacting me through [Gitter](https://gitter.im/zverok)), and we discuss the project, your vision of it and your expectation of mentorship for some time;
  * **Note**: for an already started project, you don't need to apply through email/IM, you can start immediately from GitHub PR!
* We start (or continue) to work on project, communicating mostly through GitHub issues and PRs, and (for bigger vision) through email or Gitter;
* I provide you with extensive PR reviews, discussion of further steps, new GitHub issues, coding advice and so on;
* Collaboration is stopped when you are bored or when the project is ready (for smaller projects).

## On project's authorship

* For already existing projects, the work continues in their GitHub repos, you are listed in "Authors" section after the very first non-trivial PR, and listed in it as close to top as your contribution allows (including "main author/maintainer"); I'll add you as a repo collaborator after the several PRs and established rhythm of collaboration, and never drop from collaborators after this (unless some serious misbehavior).
* For future project, I require them to be done in `molybdenum-99` GitHub organization; you'll be project's collaborator and listed as its primary author from the very beginning, I give credit as "initial idea and mentorship" (or whatever it would be in future);
  * The only exception is "ruby-docs" project, I suggest it be created in separate GitHub org;
* My initial "idea ownership" and authority for projects evolution's "big picture" is preserved **forever** (but nobody can stop you from forking project and make a turn in its development on your own if our visions will seriously diverge).

## Notice on conflict of interests

My current employer ([VerbIT.ai](https://verbit.ai/)) doesn't plan to use any of the above tools (though I hope to integrate `yard-junk` in our CI somewhere in future, but as an in-house volunteer effort). So, it is **not** a call to "do for free the work I am paid for".

## Interested?

Contact me at [zverok.offline@gmail.com](mailto:zverok.offline@gmail.com) or [Gitter](https://gitter.im/zverok)!
