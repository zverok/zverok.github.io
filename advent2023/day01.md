---
layout: advent
title: "Advent of changelog: Day 1"
next: day02
next_title: "Day 2"
---

Every year, it starts with updating my local copy of Ruby and reading `NEWS.md`, which is an official registry of notable changes. It is created manually rather than being pulled from commits, so in rare cases, it misses an entry for a notable change, so in the end, we'll need to glance briefly through last years commit, of which there are, at a moment of writing (Dec 1, 17:45 Kyiv time) a nice round number:

```bash
$ git rev-list --count --since="2022-12-25" HEAD
6000
```

(Not that I am planning to read all of them, just quickly skim over to see whether something important have happened. And maybe only a part of the list.) But for now, let's look into NEWS.md. [At the moment of writing it is](https://github.com/ruby/ruby/blob/e005c517325f3559acd2c75d368a58b50ebbb0f9/NEWS.md), to be honest, disappointingly small. (Compare [to 3.2's version](https://github.com/ruby/ruby/blob/master/doc/NEWS/NEWS-3.2.0.md).)

So maybe I've chose a wrong year to show how I work with it!

Or, maybe, it will grow larger through this month: it is not unseen to have features that were waiting for decision through the year to be merged in December. ([One of mine](https://github.com/ruby/ruby/pull/7444) is in that state right now, but it's on me: I didn't have time to address review comments in March, and returned to finalizing it in the late November, so it might not make it to 3.3.)

> Ruby's development process is structured in a way that is unusual for other programming languages: while alpha versions and release candidates are released before the version cut, there is no official features freeze. I saw features introduced or changed as late as December 24 afternoon. So, from some point of view, a version released on Dec 25 is rather "very stable release candidate" with a feature freeze, and the first or the second patch version might be seen as a "release."

So far, the list of changes looks mostly like a "maintenance" release, with clarifying some earlier introduced names, deprecating some old stuff, enhancing wrong parameters handling and so on. This still will require some effort to properly describe in "my" version of the changelog (even "mundane" features require, at least, investigating what was the justification for a change, and looking for short, persuading examples).

But to get involved, I usually look for some "interesting" features first: either introducing something that I can imagine affecting a lot of my code (preferably in a good way!), or enhancing very core functionality, or, at least, "curious to understand and explain."

There are some in this year's `NEWS`, even though much less than usual. At least `Module#set_temporary_name` is peculiar (I vaguely remember a loooong tracker discussion about it), and new warnings category "performance" is interesting to explore (also remember noticing _some_ discussions, but forgot most of the details).

Oh, and the changelog seems to never mention the new Prism parser which (as I believe, at the moment, at least) is to be introduced in this version? That's a track to investigate!

After that preliminary check, I am finally doing
```bash
$ cd ~/projects/rubychanges
$ git checkout -b ruby-3.3
$ subl _src/3.3.md
```

...and filling the file with the first skeleton of the future contents.

Commit: [4349369] (Nothing interesting there yet.)

BTW, while copying the header from 3.2's changelog, I (like any good programmer with mild ADHD) started to look through it to see what was declared there as possible changes in the next versions, and find at least one change that seems to be notable enough to make it into `NEWS`, but isn't there: the change of `Data.with` behavior (discussed in "Notes" section of the [yesteryear's entry](https://rubyreferences.github.io/rubychanges/3.2.html#data-new-immutable-value-object-class)). More on it later!
