---
layout: advent
title: "Advent of changelog: Day 24-25"
prev: day23
prev_title: "Day 23"
---

## Evening of Dec 24

That's a Christmas evening already[^1], so I focus on the final touches.

[^1]: I am in a different city than my family, and I am not very religious, but there is a warm glow in Christmas anyway, and also this year's Christmas has a symbolic value for our country: that's the first year we celebrate it together with the rest of the Europe/US, not on Jan 7 "old calendar" imposed by Russian Orthodox Church. Anyway, Ruby release is tomorrow, and I don't have anything else to do today, but I have prepared some nice dinner for myself... And something to work on!

I copy the list of the latest versions of the "default" and "bundled" gems into a changelog (yes, the versions updated [till the last evening before the release](https://github.com/ruby/ruby/commit/86893b28f7ac7cc522d628577564d49a8f558d70), and it is [not even](https://github.com/ruby/net-imap/pull/254) always some insignificant cosmetic last fixes.)

Updating library versions and inserting links to their repos make me think a few things. Like, the fact that many important libraries are extracted to their own repos and have their own release cycle greatly helps with maintenance burden of the main Ruby code; yet, it is very hard to observe what changed and whether it is significant. Some libraries make a lot of change between their versions associated with Ruby's releases (like [json](https://github.com/flori/json/blob/v2.7.1/CHANGES.md)), some kinda have a "version" yet nothing more serious in it than changing the CI settings or exposing the `VERSION` constant (like [syslog](https://github.com/ruby/syslog/releases)). Some have `CHANGES.md`, some have `History.rdoc`, some use GitHub releases to describe changes (and then release notes are frequently auto-generate from the list of commits, mixing serious changes and small documentation adjustments), and some (like [typeprof](https://github.com/ruby/typeprof)) have neither. Finally, the libraries size is different, and their frequency of usage in the community is different, so probably, providing meaningful change descriptions for `csv` is somewhat more valuable than doing so for something like [tsort](https://github.com/ruby/tsort/releases)!

So, if (not today, but maybe in the upcoming year) I want to somehow broaden the scope of the Changelog[^2] but stay reasonable on the effort necessary and size of the text... I need to somehow make choices of libraries and changes, and organize a process to gather it... Anyway, for now, I am just putting a few links associated with corresponding library versions to their changelog/release notes, when the library itself and the changes in it seems to be of interest (I don't check every one of the 70 libraries, only those that I _suspect_ to have some changes of general interest).

[^2]: Which narrowed down due to "gemification" of the libraries. In [older versions](https://rubyreferences.github.io/rubychanges/2.5.html#standard-library), following the `NEWS.md`, the Changelog provided more info on the standard library.

Next, some really final stuff:
* Replace the `[Feature #19572]` text fragments with a link to the issue (yeah... using SublimeText's search-and-replace, like an animal! Every year I do promise myself that next year I'll just auto-convert them, but as it is a thing that doesn't bother until the last minute...);
* Fill the "Status" of the upcoming version;
* Check everything and fix some leftover typos;
* The rest should be done after the release itself, tomorrow...

..Ugh, save for unfinished business with `Fiber#kill` and `Process.warmup`. Which, honestly, I kinda give up on. The [killing fibers across the threads](https://bugs.ruby-lang.org/issues/20082) ticket is still unanswered (so I don't try to provide the code demonstrating that); and as for `Process.warmup`— for now, I leave the short description that I have. Maybe I'll rethink it later.

## Morning of Dec 25

Ruby 3.3 is [released](https://www.ruby-lang.org/en/news/2023/12/25/ruby-3-3-0-released/).

Unfortunately, [docs.ruby-lang.org](https://docs.ruby-lang.org/en/) don't have 3.3's link yet, so I need to publish my changelog linking to `master/` docs, and set a reminder to check in a couple of days and replace the links.

**Commit: [a4f5aa3](https://github.com/rubyreferences/rubychanges/commit/a4f5aa3)**

**And just like that, [we are online](https://rubyreferences.github.io/rubychanges/3.3.html)!**

Ugh, except for the [change that was merged](https://github.com/ruby/ruby/commit/a44a59dbaf1b5c930c9f476077b0be9b1653ad9c) in the night before Christmas (and not a tiny one), so I push an update to already published changelog right into `master` with **commit: [95077d1](https://github.com/rubyreferences/rubychanges/commit/95077d1)**.

And this concludes the "advent" of this year—save for a wrapping-up blog-post which I am planning to publish in a few days, to not over-saturate the internets with my content.

Honestly, I am not sure whether this diary was a fun reading. At least, it helped me to organize myself and observe how I do things—and I still hope that might be at least of _some_ interest to others. And if not... Well, at least I hope the [end result](https://rubyreferences.github.io/rubychanges/3.3.html) would be considered worthy of your attention.
