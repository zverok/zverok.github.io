---
layout: post
title:  "On GitHub Copilot and (Ruby's) Rubocop"
date:   2021-12-07
categories: ruby programming philosophy rant
comments: true
description: Or, How AI Could Help Coding
---

Amid GitHub Copilot kerfuffle _(I am as concerned about the usefulness and ethical implications of this glorified fuzzy copy-paste as the next guy)_, I started to think about a related question:

**Can AI be really useful in code writing? If so, how?..**

I mean here the AI-as-we-know it, a.k.a. statistically trained "models", not some genius "general intelligence". My take on modern AI is that it encapsulates "experience" (as opposed to knowledge/analysis). As stated in [one of my spellchecker articles](https://zverok.space/blog/2021-05-06-how-to-spellcheck.html):

> The solution to every intellectual problem lies between two opposites: “with understanding” and “with experience”. Most of the time, we use a mix of both, though one can imagine “pure understanding, no experience” and “pure experience, no understanding” ways of solving.

> When implemented in software, understanding-based (analytical) solutions require a developer to understand the domain fully and reflect it in the code; while experience-based solutions can be created by fitting a few heuristics matching most important cases and then iteratively changing these heuristics (or adding new ones) when new cases are uncovered. This iterative fitting can be performed by the developer or another algorithm—the latter we now call “machine learning” (a criminal oversimplification, but it is enough for the current context).

> _There is one common confusion related to using the word “model” in ML context—which isn’t anything near “models” of the problem domain in the analytical sense. I’d prefer we name those “experiences”; it would be much more appropriate (“download those experiences of image recognition trained on that dataset, or gather custom experiences from a different dataset by running this training script”), don’t you think?_

**So, where "automated experience" can be used in code writing?**

In Ruby, we have a tool called [Rubocop](https://rubocop.org). It is usually brushed off as linter/autoformatter (even by intro on its own site!) that "every language has today", but it is, in essence, quite an interesting tool.

In the core, Rubocop is a foundation for building "cops" (I know, not the most fashionable word in 2021!): independent small(ish) classes that have powerful DSL for analyzing Ruby AST, spotting some patterns, reporting them, and advise on alternatives.

Of course, it can be, and is, used for linting/formatting. But there are also a whole "department" of [`Style/*` cops](https://docs.rubocop.org/rubocop/1.18/cops_style.html) there, as well as derivative code checkers for different contexts and usages (like [rubocop-rspec](https://docs.rubocop.org/rubocop-rspec/) to check the style of tests written with particular framework).

There are two interesting things to notice about the currently existing vast amount of cops:

* First, many of them (especially `Style/` ones) are highly configurable, so instead of enforcing standard style, they serve "establish OUR style" purpose.
* And second, a lot of them are actually serving spreading knowledge/teaching purpose: like, "you know that in Ruby 2.4+ you can write _this_ shorter? You know that _this_ approach is highly ineffective?" Etc., etc.

Rubocop-the-foundation also frequently (though less frequently than deserves) is used to create custom cops for some codebases, check against established patterns, internal frameworks approach, preferred ways of expressing things, and so on.

In my day, I wrote a lot of cops (some made way into Rubocop's core, some were used in private codebases, some I inventively [applied to unrelated to code style tasks](https://github.com/zverok/the_schema_is)).

It is relatively easy, but, TBH, not _too_ easy. You need to become familiar with [node matching patterns](https://github.com/rubocop/rubocop/blob/v1.23.0/lib/rubocop/cop/style/hash_conversion.rb#L44), and to construct them, you need to investigate AST structure the [parser](https://github.com/whitequark/parser) produces, and after that, you'll [fight with a `corrector`](https://github.com/rubocop/rubocop/blob/v1.23.0/lib/rubocop/cop/style/hash_conversion.rb#L73) to replace the code, and so on.

It is harder than, say, writing a regexp which will find "bad" code and replace it with "good" (though [some may argue](https://twitter.com/junpenglao/status/1410855165515677699) that writing regexp is not that easy either).

**If writing custom code queries/suggesters would be easier**, I'd maybe do it a lot more.

> There are, of course, other approaches to define code matching/replacing tools, like [SemGrep](https://semgrep.dev/).

Just last few years, I've recognized that a _lot_ of what I do as a mentor/principal dev/reviewer is vaguely repetitive suggestions:

* "did you know it can be written this way?",
* "...maybe you've missed, but we recently decided not to use _this_ form of query objects, it can be done simpler",
* "oh, this imperative code can be nicely expressed with lesser-known [Enumerable](https://ruby-doc.org/core-3.0.3/Enumerable.html) method" and so on and so forth.

But I am not writing cops for each of the above suggestions: the situations, if repetitive, are too vague to be generalized in a hard heuristic; and repeating too randomly to decide "oh, I suggested it ten times just this week, should be automated" (best case, I document it).

And here we go: **"we have too much too vague heuristics to write by hand" seems like a textbook case for ML!**

Obviously, this imaginary "AI Rubocop" should be trained on a supervised subset of "code we'd like to follow" (in _any_ codebase, there is code that "looks as we like" and "looks like a pile of legacy", and if you train any model on top of it ALL... garbage in, garbage out).

We can also imagine "community-driven" models... But again: there are many "good" styles out there, and they change with time. The code to train on should be small(ish) subsets; it is NOT the case of "the more we have input, the smarter we get!"

And, to reiterate: it is not about "writing code instead of you"; it is rather "looking at your code, and applying _experience_ of others (consensually gathered and properly filtered) to give a feedback". It is experience sharing, not experience abusing.

Welp... For every supervised-learning approach, there would be "unsupervised learning" disruptors. But (even on the ML approach with 100% consensual and ethically clear sources), it is hard for me to imagine what it might give. Because—

**What is the "success function" of "good code"** the unsupervised ML might teach itself to write? We seem not to be able to agree between us humans for the last fifty years or so! Until then, the only result unsupervised ML is capable of is guesswork.

And we are not impressed.
