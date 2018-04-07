---
layout: post
title:  The Missing Ruby Reference
date:   2018-04-06
categories: ruby rants
comments: true
---

Imagine your friend, thinking about learning programming, or switching to new languages, asks you:

> OK, I'll give your beloved Ruby a try, point me to the language reference, I want to taste it. No, please, not "your first steps" schoolbook, and not a link to 5-year-old Amazon book, just the normal language reference.

**What would you do?**

Let's see.

What official site suggests? [Documentation](https://www.ruby-lang.org/en/documentation/) section is full of helpful resources (though, I find a bit funny that first recommended manual in "Getting Started" section is "Why's Poignant Guide", which is the piece of art we all adore, but by no means proper language introduction).

There is, though, one small problem with those links: there is **no up-to-date, full, free, official language reference** between them!

The options are:

* [The first version of Programming Ruby](http://www.ruby-doc.org/docs/ProgrammingRuby/), dedicated to Ruby **1.6**;
* [Ruby User’s Guide](http://www.rubyist.net/~slagell/ruby/), written by Matz initially; this book is pretty short though helpful; at least it is updated for, as far as I can guess, Ruby 2.1;
* [Collaborative Wikibook](https://en.wikibooks.org/wiki/Ruby_Programming), which, as any Wiki, is somewhat helpful yet in constant "in construction" state;
* [Ruby Core Reference](http://ruby-doc.org/core-2.5.0/), which, well, is The Reference... Yet its navigation is, let's say, not _that_ friendly, especially for those just coming to the language.

Let's see what _other_ languages have:

* [Python](https://docs.python.org/3/)
* [PHP](http://php.net/manual/en/)
* Elixir: [language](https://elixir-lang.org/getting-started/introduction.html), [standard library](https://hexdocs.pm/elixir/Kernel.html)
* [Crystal](https://crystal-lang.org/docs/)
* [Rust](https://doc.rust-lang.org/book/second-edition/)

Enough?..

So, **what about Ruby?** Back in 2000th, when I switched to Ruby, I used Programming Ruby from Pragmatic Programmers as my main reference. The book still _may_ be suggested as an answer, but it is outdated (the last version is unfortunately dedicated to Ruby 1.9 & 2.0), and, all in all, it is non-free PDF or paper book. Which is _not optimal_, to say the least, for the main language reference.

So, without further ado, I am now presenting my **answer to the reference question**:

**[The Ruby Reference](https://rubyreferences.github.io/rubyref/)**

The idea is that reference should be:

* **full**: cover all aspects of the language, core classes, and standard library;
* **continuous**: provide unambiguous reading order from the simplest concepts to advanced development techniques;
* **actual**: correspond to the latest version of Ruby;
* **accessible**: easy to find, navigate and read from any device.

In my effort, those goals are achieved by technique borrowed from Dr. Frankenstein. The reference is "compiled" from:

* RDoc documentation from source repo;
* Some pages of ruby-lang.org website;
* Some custom content is written to fill the gaps (did you know there is no _official_ reference of how Ruby's comments should look like?)
* A huge config to cut only useful pieces and sew them together into a good-looking book, that can be read top-to-bottom, or easily navigated through.

This way, on each new version of Ruby, the reference will be easy to update (as the most of the content is taken from Ruby or its site, and the rest is just small additions). And it can be said to be _almost_ official.

This is the first (draft) release of the reference. **I am looking for feedback and contributions:**

* "Custom content", written by some Ukrainian guy, can use a lot of editing, probably.
* Importing scripts could be improved (not all formatting is converted RDoc=>Markdown properly).
* The overall book structure should be reviewed and stabilized (in the future, it is desirable that URLs will stay stable, even on new language versions).
* And, probably, the whole idea needs some discussion and validation.

That's it!