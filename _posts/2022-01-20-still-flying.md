---
layout: post
title:  "Programming language evolution: with all that, we are still flying"
date:   2022-01-20
categories: ruby development philosophy
comments: true
description: Final part of adventures in understanding, explaining, and challenging the facts that should be obvious.
---

**Final part of adventures in understanding, explaining, and challenging the facts that should be obvious.**<br/> ([Part 1](/blog/2022-01-06-changelog.html) → [Part 2](/blog/2022-01-13-it-evolves.html) → **Part 3**)

> **This article is written for my [Substack](https://zverok.substack.com/), you can subscribe to it by email now!**

_In the previous two parts, I described how working on Ruby's changelog made me imagine I understand language's logic and intentions behind it. Then, that fantasy brought me to participate more insistently in language development. And then, that participation made me suffer when several aspects of Ruby 2.7 evolution hasn't developed the way I expected—and in one case, an important feature was reverted a couple of months before the final release._

I was devastated. Probably it all wouldn't bear that much significance for me, would I consider the programming language as just a bag of convenience features thrown together: one feature less, one feature more, one feature doesn't look like others, who cares?

**What always amazed me in Ruby is the feeling of a very small core of informal rules—let's say _intuitions_—that everything else followed.** You could've uncovered behaviors without explicitly looking for them in the docs, just by assuming that what's intuitively right would work. And a lot of my work in the Ruby community was dedicated to those intuitions: sharing them with others in my roles of a mentor and senior/principal developer; documenting them; and—yes—trying to push them further, to make small parts of the language as short and clear as they _intuitively should've been_.

> Spoiler alert: this year, a small(ish) book of mine, called—you guessed!—"Ruby Intuitions" is in development. It is still in the early stages, but I really hope I can lift it off.

That's why I was sad about what happened. It didn't feel like a rejection of "just some feature I liked" (I had plenty of such rejections and was totally OK with them), but rather that my imaginary alignment with the intentions of Ruby core developers was broken. Either my understanding of the aforementioned intuitions was loose, or—and I imagined that being the case—language development ceased to be coherent; now it is "every core member for themselves," and I don't have a place in this carnival, and don't want to.

I proudly proclaimed I was done with Ruby and moving forward. But it couldn't happen immediately due to several factors—and eventually, it didn't happen at all. Which turned out to be a good thing.

### Making peace

The decision to "be done with Ruby" came in an awkward moment: I was in the final stages of preparation for talking at [RubyConf Nashville](https://rubyconf.org/2019/program.html#session-871)—my first time at _The_ RubyConf, first time in the U.S. (and, as it turned out later, my last talk on the in-person conference).

Even worse: the talk I worked on summarized my views on Ruby's evolution and future, and the new "method reference operator" was one of the central points in it! To add insult to injury, the reverting discussion started three weeks before the conference and ended (with reverting confirmed) one week before it.

Maintaining a talk plan/slides (and my sanity) during those weeks was not an easy task: I needed to ensure my ideas were still relevant regardless of the recent development. I also needed to maintain a light and constructive tone of the talk, which I mostly did—though I couldn't keep myself from pasting the random photo I made from the window of Georgia-Nashville Greyhound:

![](/img/2022-01-20-still-flying/falls.png)

_[The talk](https://www.youtube.com/watch?v=iMBqqjkbvl4) flopped. Not even in a catastrophic way, rather it was "Nobody cared enough to even tweet about it" offensive way. But that's not the point, because—_

**At the conference, I had several important conversations, but most importantly—I had an opportunity to reconcile with the latest events in Ruby evolution.** For that, I just needed to talk to Yukihiro Matsumoto—Matz—openly (and I did!), and listen to his [talk](https://www.youtube.com/watch?v=2g9R7PUCEXo) and [Q&A](https://www.youtube.com/watch?v=vqpNmaEDn-o) (and I did!).

That brought me two insights into the unexpected turns in development.

For one, the **method reference operator's reversal** was initiated not by one of the core team members due to gate-keeping of the language becoming "too functional" (that's how it [looked in the tracker](https://bugs.ruby-lang.org/issues/16275)), but by Matz himself—he never liked how the feature _looked_ ("looked like Braille to me!").

In the same way, **pattern-matching** wasn't just a one-person project merged in an inexplicable rush: it was discussed _and redesigned_ by Matz to match the rest of the language.

Those two simple discoveries made me rethink my disenchantment. Not of mindless fanboyism "oh, if Matz said so, it is good!" I just understood that **the internal consistency/coherence of the language is still deeply cared about; and yes—I trust Matz's _intuitions_ a lot. Even if I am still, to the day, unhappy about not having method references—I hope one day we'll find a way to express them better; maybe, with `.:` syntax, it would just become the hated feature nobody uses.**

### ...and moving forward

Soon, I was able to check the new understanding of the coherence still being maintained. It happened during work on [that year's changelog](https://rubyreferences.github.io/rubychanges/2.7.html) (we started from changelogs, you remember?).

As usual, working on the changelog gave me mixed feelings. I was happy to describe [several](https://rubyreferences.github.io/rubychanges/2.7.html#beginless-range) [features](https://rubyreferences.github.io/rubychanges/2.7.html#better-methodinspect) [that](https://rubyreferences.github.io/rubychanges/2.7.html#comparableclamp-with-range) [grew](https://rubyreferences.github.io/rubychanges/2.7.html#enumeratorproduce) of my proposals. I was curious and energetic to understand better some features and the reason for their changes. I was mildly irritated with new/changed features lacking documentation updates (but just sighed and got to work).

But I was totally stunned to find out that the two new big features—numbered block arguments and pattern-matching—got no docs at all! So, eventually, I had to document them, too, suppressing my inner resistance to their design. _Pattern-matching docs were submitted even later than the release date—till the end, I couldn't believe nobody else works on documentation and didn't want to duplicate efforts... but alas, it was back to me again._

And it turned out to be an ultimately good thing for me.

Because, while trying to document the pattern matching, I caught myself doing this: I looked into the initial description of the feature on the tracker; then, I wrote draft docs with assumptions of what examples will make sense and how they should work; then, I started to really run the examples to check my assumptions.

And you know what? Most of the time (save for minor things), my first guess of "how it should be possible to express" and "what it should produce" were completely right. The drafty code (and approximate output) that I just guess-imagined was the real code (and the real output) that worked.

It didn't mean I am a genius with extrasensory abilities. **It just meant that at the end of the day, pattern-matching is integrating naturally with language and its intuitions.**

And that's how working on that year's changelog/docs gave me back peace of mind and appreciation for the language I am fond of.

_And I needed that peace of mind when, after a month of back and forth and describing all the details, the docs were [finally merged](https://github.com/ruby/ruby/pull/2786). I proudly proclaimed the work done, and the very first Reddit comment was: "These docs are pretty horrible."_

But I was mostly undisturbed. I knew what I do and why it should be done and felt useful and ready for whatever happens to the language next. Next year, "whatever" promptly started to happen.

## You know nothing

**The one where a routine task of describing and documenting Ruby 3.0 poses a challenge and requires a level-up (December 2020).**

I haven't participated much in the Ruby development/discussions during the preparation of the Big Three-Point-Oh due to a number of reasons (the residual offense was one of them, but not the main—it was 2020, after all!). I still followed the development from a distance and assumed I had a pretty clear idea of how much work there would be in December—when preparing the changelog and updating docs for new features. I was (habitually) wrong in those assumptions.

There was a bunch of small(ish) nice things that lacked their respective docs (like new [method definition syntax](https://github.com/ruby/ruby/pull/3997)), but also **there were two new concurrency primitives I needed to wrap my head around: [Ractors](https://docs.ruby-lang.org/en/3.0/doc/ractor_md.html) (Ruby's take on actors/isolated threads) and [non-blocking Fibers](https://bugs.ruby-lang.org/issues/16786) (Ruby's take on `async`)**.

Now, to put a final point in the demolition of whatever reputation I had, I should acknowledge that I don't possess a deep knowledge of concurrency/parallelism. By day, I am a fairly traditional web developer of Rails-based, mostly business-related code. In my OSS maintainer suite, I am interested in open data, development tools, and small idiomatic libraries. Neither of those typically required to write a lot of concurrent/parallel code on a daily basis. So... I have a solid understanding of the bases, but it is not as nuanced as I'd liked to have—especially that year.

**To describe new features in the changelog, I need to take them close to my heart. To write some code that demonstrates feature's usage—and demonstrates meaningfully, not only "how it works", but also "why would you use it." I need to be sure I really _get it_ before explaining.**

If only I had enough documentation for those features to rely upon! To be fair, neither was _completely_ undescribed, but descriptions provided by respective authors were by means of [design](https://docs.ruby-lang.org/en/3.0/doc/ractor_md.html) [documents](https://docs.ruby-lang.org/en/3.0/doc/fiber_md.html#label-Scheduler) in `doc/` folder, that weren't integrated tightly with the corresponding modules. To make it more challenging, the non-blocking Fibers is a unique feature relying on the user to implement some interface, so you can't "just" run it to experiment (changelog—in the end—[explains it all in details](https://rubyreferences.github.io/rubychanges/3.0.html#non-blocking-fiber-and-scheduler)).

Of course, I took the challenge of fully understanding and documenting both: "whatever it takes," right? Even if it takes to become quickly and intimately familiar with ruby core's approach to an area of CS I lacked a clear knowledge about. I managed to, in the end, document both: [Ractor](https://docs.ruby-lang.org/en/3.0/Ractor.html), and [Fiber::SchedulerInterface](https://docs.ruby-lang.org/en/3.0/Fiber/SchedulerInterface.html), and participated in a lot of design/edge case discussions on the road.

> I would never pull it up without the enormous help of two people: ruby-core member [Marc-André Lafortune](https://github.com/marcandre) who worked with me side-by-side on documentation merging and feature discussion, and my long-time friend and mentor Alexey Makhotkin, who was the first reader and ruthless editor of the first attempt to document things I am yet to understand (he writes [an awesome Substack on database modeling](https://minimalmodeling.substack.com/), go check it).

Cutting the already long story, there was an important lesson for me in that year's adventures (besides becoming a bit more versed in concurrency primitives). I can say even that I gathered some deeper understanding into a (part of) the nature of the Ruby development. If I try to formulate it in one phrase, it would be: **there is no higher authority of the small things**. Or: there is no "them" who will take care of small-scale consistency and per-method docs. There is only "us." At least that's how it is in the Ruby community, and despite sounding at times like a grumpy old man, I see a great justice and wholesomeness in how things are.

![](/img/2022-01-20-still-flying/wait-it-is.png)

_Even if you are slightly jealous about the authors of "real features" who gets praised after the release, you should carry on._

## With all this, we are still flying

**The one where the author doesn't give up, works on the changelog for Ruby 3.1, and looks into the future (Dec 2021-Jan 2022)**

> “I am a leaf on the wind. Watch how I soar.”—Hoban “Wash” Washburne

My friends—those unlucky enough to know what Ruby is—could testify I've promised to stop caring about Ruby for many years now. Some of the community, too, witnessed my grumpy ramblings about the current state of affairs and doomsaying of the near future.

**It is not that I was unhappy with the language itself. Rather, I believed (and to some extent, still do) that the language is underused and underappreciated for the things it fits best—teaching, experimenting, prototyping, playing—and overused for things it wasn't designated (large apps development).**

This situation seems to distort the language's reputation. The qualities that shouldn't be central for expressive dynamic language (performance and robustness) are widely discussed, criticized, defended, improved, and criticized again. At the same time, the qualities that are important to me—small and coherent core, leaning on several explicit intuitions, phrase-level clarity, and pliability—are brushed off as secondary for "the real work."

When I started to submerge into Ruby's development process, its visible informality and "homemadeness" were initially perceived as just another sign of nearing doom. "Is it just me, or does nobody care anymore?" was my thinking.

> "Just compare the process some similar language has," I might've cried, "take Python: they have feature-freeze several months before the release!" And when they introduced pattern-matching—just a year after Ruby—it was the [first and explicit goal](https://mail.python.org/archives/list/python-committers@python.org/thread/SQC2FTLFV5A7DV7RCEAR2I2IKJKGK7W3/) to have it widely discussed beforehand and extensively documented on release.

After a few years, with not a small amount of personal struggles behind, I understood this initial impression was wrong. That's why I started to write this big three-part text about the changelog—not really about the changelog, after all.

The text is turned out to be rather about the changing language. And its way of keeping balance, or—keeping in the air.

The balance between those who love the language for its expressiveness and constantly want to take it further—and those using it for building large apps and looking for performance and robustness. The balance between coherence and humanity. The balance between keeping pace and maintaining altitude.

**And I now believe the highly informal, decentralized (almost aggressively so) development process is instrumental in keeping true to _this_ language. It might bring frustration and sudden turns and might seem "broken" for those striving for more controlled environments—but that's the process that is keeping us airborne.**

The language is still here and still evolving. And I am still here and still proud of being part of this evolution. The [3.1's changelog](https://rubyreferences.github.io/rubychanges/3.1.html) was published the day I've started the first part of this big text, and I [became a Ruby committer](https://twitter.com/zverok/status/1482997291975860229) the day I am finishing the last part. After all, I am happy to be a small wind under the wing of this aircraft.

_But I need to keep my balance too. After last month and a half almost fully dedicated to Ruby's release, the changelog, and this text, I'll be switching back for my [investigations of Wikipedia](https://zverok.substack.com/p/wikipediaql-1) and open data next week. Stay tuned._

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
