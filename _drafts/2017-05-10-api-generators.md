---
layout: post
title:  "API wrapper generators, and when they are good"
date:   2017-05-10 12:20
categories: ruby gems
comments: true
---

**The theme:** New library for MediaWiki (Wikipedia, Wiktionary and others) API access is released,
named [MediaWiktory](https://github.com/molybdenum-99/mediawiktory) (and no, I can't stand a temptation
of stupidly witty naming). It utilises some unusual approaches to API wrapping:

* the wrapper is fully _generated_ (alongside with docs), not hand-crafted;
* the wrapper provides Arel-style chainability of queries, like `api.query.titles('Argentina').prop(:revisions).format(:json)...`
* as there are tons of MediaWiki installations with ton of different features and versions, the gem
  does not tries to fit all the situations, yet instead provides _wrapper generator_ able to create
  several independent wrappers for different situations.

## Context

[MediaWiki](https://www.mediawiki.org/wiki/MediaWiki) is a prominent software powering Wikipedia,
its "sister projects" (Wiktionary, Wikivoyage, Wikidata...), as well as a _lot_ of common good and
private knowledge bases. Data extraction from those knowledge bases is fun. And a lot of work, also.

MediaWiki provides

## Lessons learned
