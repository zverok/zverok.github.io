---
layout: advent
title: "Advent of changelog: Day 7"
next: day08
next_title: "Day 8"
prev: day06
prev_title: "Day 6"
---

A week of ADHD-driven running around changelog and other new Ruby release-related tasks!

The plan for today:
* finalize adjusting `WeakMap`/`WeakKeyMap` docs and create a PR;
* put what was discovered/understood into the corresponding changelog entries (that's the main driver of this adventure, after all!)
* publish a blog post/email summarizing the first seven days and linking to this rough diary;
* (maybe) also handle `Module#set_temporary_name` docs/changelog entry, while my memory of the investigation is still fresh.

Also, before we start, there is some new information that to some extent changes the scope of my work:

* today, a development meeting discussion was [published](https://github.com/ruby/dev-meeting-log/blob/master/2023/DevMeeting-2023-11-30.md), leading to adjustments in some outstanding tickets on the tracker; some of the messages are related to those "new syntaxes" I described in the recent series:
  * **anonymous block parameters**: after a lot of discussions and pressure, Matz finally agreed to have something more nicely-looking than `_1`: [the name `it`](https://bugs.ruby-lang.org/issues/18980) would be warned in corresponding context in Ruby 3.3, and introduced as an alternative name for anonymous block parameter in 3.4 (it would be done in a gentle way, so, say, RSpec's API wouldn't be broken);
  * **endless methods**: the core team is not persuaded that the "quirks" with parsing order are deserving to be fixed, [the ticket](https://bugs.ruby-lang.org/issues/19392) is closed; I am still not giving up hope and will try to reiterate on the problem, but only after 3.3 release;
* A lot of new things [have happened](https://github.com/ruby/ruby/commits/master/NEWS.md) in `NEWS.md` since I initiated the changelog, so I'll need to add more entries into my files— but not today.<br/>
  ![](/img/advent2023/image14.png)

So... Here we go at `WeakKeyMap` docs:

![](/img/advent2023/image15.png)

> As I explained [yesterday](day06.html), to check if references to other docs—here, `Object#eql?`—rendered correctly, I need to include more files in `rdoc` command, namely
> ```bash
> rdoc weakmap.c object.c -o tmp/doc
> ```

And finally... [Here we go](https://github.com/ruby/ruby/pull/9160).

A half-an-hour later, a small commit to the changelog is ready, too: **[c716b8e](https://github.com/rubyreferences/rubychanges/commit/c716b8e)**. The texts are very rough for now, but at least, I now can put a mental checkbox that some features alreadys have drafty coverage. Once they _all_ would have at least that, I would be able to move to reorganizing and polishing.

After that, I am switching to house-keeping for my "self-publishing": move rough `day**.md` files into a separate folder, add some navigatin to them, and write a blog post covering this first week.

> I would probably be more efficient working on changelog in silence, as I do every year... Or maybe not, because commitment to everyday diary makes me work on it _every single day_ instead of depending on the mood, physical and mental state. It is nice to have something organizing when no external factors push you to work on a hobby project (last year, I finally made myself finish it by _February_!)
