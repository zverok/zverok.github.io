---
layout: post
title:  "Thou Shalt Not Complicate, Pt.1: Three Kinds of Programmers"
date:   2015-09-22 22:21:30
categories: ruby rants
comments: true
---

_**Preface**: "Thou Shalt Not Complicate" is planned as a series of
strong rants on programming in general and Ruby programming in
particular. The main thought is dead simple (and already emphasized in
series title): complexity is our enemy. All we do, all we try to do
while developing software—is bearing with complexity introduced by
real-life. Yet some of our approaches, techniques and libraries introduce
unwanted complexity by themselves, which is bad. As simple is that._

**Statement 1**: I'm deeply convinced there's only three kinds of good
programmers:

* a) Programmers-engineers: entire "program" is a mechanizm for them, or
  even blueprint of mechanizm;
* b) Programmers-mathematicians: entire program is a formula, or set of
  formulae, proof of something;
* c) Programmers-writers: entire program is a text.

(There's also numerous kinds of bad programmers: programmers-managers,
programmers-moneymakers, programmers-labourers and so on. Let's not talk
about them. Ever.)

What it means to belong to some of kinds? How you write code. What you
consider good and bad code. How development of overall architecture works
in your head. What kind of problems you are best at.

Very schematically, it can be said that "engineers" tend to see perfect
hi-level structures, embrace usage of patterns and sometimes neglect
details. "Mathematicians" are irreplaceable around algorithmically-complex
problems yet can sacrifice almost everything in the name of purity, and
"writers" are all about "pretty code" and "code smells" and stuff, while
sometimes neglecting the huge picture.

It's cosidered useful to know your own type: this way you can develop
its strengths better and compensate its weaknesses wiser.

Here I'm expected to say each good team should have all of types, and
distribute tasks between them etc etc. Yet it is almost never the case.
Because —

**Statement 2**: Your "ideal" programming language depends on your type.

This is tricky one, yet obvious if you think. All of those discussions
with good developers speculating on "how could possibly anyone write
big amount of code in dynamically typed language?" or "how can anybody
enjoy Java?" or what is right OO (and how nobody groks it) or how the FP
it The Answer to Everything...

As much as about Statement 1, I'm convinced that language (or rather
language + tools + infrastructre + community) imposes some mindset,
and your hapiness and productivity as a developer is deeply correlated
with correspondence of your mindset to mindset of language and ecosystem
you work with.

There can be many speculations thought out about which language represents
which mindset, but one thing I know for sure —
 
**Statement 3**: Ruby is for (c) kind, for programmers-writers.

Fortunately, I should not defend this statement, as it is defended much
more strongly and clearly than I could by language's creator. In
"Beautiful Code" book, Yukihiro Matsumoto's chapter is called
["Treating Code As an Essay"](https://books.google.com.ua/books?id=gJrmszNHQV4C&pg=PA477#v=onepage&q&f=false)
and expresses exactly the thought about Ruby as a language: it designed
the way allowing to rather _write_ in it than _develop_ or _calculate_.
It is either for you, or not for you.

> Still, for both essays and computer code, it's always important to look
  at how each one is written. Even if the idea itself is good, it will be
  difficult to transmit to the desired audience if it is difficult to
  understand. The style in which they are written is just as important as
  their purpose. Both essays and lines of code are meant—before all
  else—to be read and understood by human beings.

Unlike Matz, I don't think that "code is meant to be read" (and therefore
be The Text in a first place) is such a strong imperative. Some languages,
and good languages it is, are meant rather to do the "blueprints"—large
and complex whole pieces, which you investigate back and forth until
grasp. Others are mindblowing yet concise formulations of state-of-the-art
algorithms.

But Ruby, definitely, is for writing texts. Either you feel it, or you've
choosen the wrong language

> **Note**: Of course, all those "texts" are having pragmatic purpose.
  We are writing something working, extendable and supportable, not
  just expressing ourselves.
  In some sense, we—Ruby programmers—are all writing the same text.
  Therefore, our [style guides](https://github.com/bbatsov/ruby-style-guide)
  and code conventions are a good thing. All in all, there's no point in
  trying to mix 5 parts of _Finnegan's Wake_ with 1 part of _Stranger in
  a Strange Land_ and hope for reasonable outcome.

**The Big Conclusion**: One and the only sign of good Ruby code, good
Ruby library, good Ruby architecture is this:

**On every level you have clean, readable, terse, ingenious text.**

Somebody can argue about usability of methods of 140 lines length because
they provide "the great overview". But in Ruby, in good Ruby, in our Ruby,
each line or statement is an extended sentence for thoutful reading, and
each method is a paragraph of overall text. And it concerns all levels:
ideally, even the top level of your software is just a few paragraphs
saying "it does so-and-so", with deeply "explanations" in components and
libraries.

> **P.S. The Strong Side Note**: By some strange irony, Rails—that very thing
made Ruby popular—is not sharing the same "writer" mindset. It tends to
hide expressiveness and variability under the hood, providing instead
multitude of patterns, conventions, best (and the only) practices and
other typical signs of "engineering" tools. That is the reason why two
different trends exist nowadays: some Rails-ists drifting towards other
languages and approaches, while still prasing rails (those are engineers),
and others rather inclined to reinvent approaches to ORM, routing, views
in more Ruby-ish way than Rails was.
