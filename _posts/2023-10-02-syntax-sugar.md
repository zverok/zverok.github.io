---
layout: post
title:  "That useless Ruby syntax sugar that emerged in new versions"
date:   2023-10-02
description: "...and what might be interesting about it"
categories: ruby philosophy
comments: true
---

> Status update: since [my last blog post](https://zverok.space/blog/2023-05-05-ruby-types.html), I've finished my training and spent four months close to battlefront (last couple of months near Robotyne), trying to do my best there. Now, I am still in AFU but have recently moved to a more computer-related position, so I have some time to write in Ruby and about Ruby. And so I do.

**Ever since meeting it some 20 years ago, I love Ruby.**

Somebody might say that nobody should feel emotional about their _engineering tools_, but I don't know a law prohibiting the appreciation of means of expression. The means that make me think in new ways, solve problems with joy, and invent concepts that haven't existed previously.

**During my years with Ruby, I (mostly) enjoyed observing how the language changes.** And I am proud to [be a small part](https://zverok.space/ruby.html) of those changes.

I believe—and have expressed this belief many times—that this process is natural and inevitable for any means of expression. As the world changes, problems change, and our understanding of what and how we want to express changes, too. What once was esoteric or academic becomes an everyday task. The complicated ideas that needed a wordy explanation before become ubiquitous enough to be just briefly referenced.

I think I coined the term "Ruby Evolution"—or, at least, promoted it as something natural and positive. I have written about it in numerous blog posts ([1](https://zverok.space/blog/2022-01-06-changelog.html), [2](https://zverok.space/blog/2022-01-13-it-evolves.html), [3](https://zverok.space/blog/2022-01-20-still-flying.html), [4](https://zverok.space/blog/2022-06-11-ruby-evolution.html)), focused on it in my [RubyConf talk](https://zverok.space/talks/#language-as-a-tool-of-thought), and maintain [a site](https://rubyreferences.github.io/rubychanges/) documenting this evolution.

**In recent years, there have been quite a few additions to the language's syntax and standard library.**

Frequently, there was a vocal push-back in the community, ranging from honestly subjective ("I don't like it," "looks ugly to me"), through pretending-to-be-objective ("it hurts readability"), to something that looks like a measurable statement, only one that nobody knows how to measure ("it gives very _little_ gain to _significant_ drawbacks.") In several cases, the push-back went as far defining linter rules to prohibit a new feature completely.

Most of the time (though not _all_ of the time), discussing such feedback is futile: too frequently, it boils down to personal tastes and feelings, and, well— As my family proverb goes, "the crocodile doesn't eat herring. Why? Because he doesn't want to!"

What I feel as an interesting mind exercise, though, **is providing a thought framework of looking at the new syntaxes and language features to understand what are their causes, reasoning, and consequences.**

In other words, I want to propose to the reader my analysis of some of the most prominent (and, consequently, the most controversial) ones through the latest versions.

But first, a bit of generic considerations.

## How I think about the (programming) language

So, my approach to coding (I tend to call it "lucid code" since recently) states that my language should allow me to:

* **state the intention/meaning** of the code as clearly as possible, so the brief reading would already give a good overview of what's going on;
* is easy to **overview and follow**, so the reading would be a predictable experience;
* avoid stating the **obvious**, so the reader won't need to focus on unnecessary things, trying to understand whether they add some meaning or just here because the tool requires it to be written;
* use a **discoverable** dictionary, so it would be easy for the reader to investigate where the particular element is coming from and how it is related to other elements;
* if possible, be **insightful**, making something "click" for the reader in understanding the system the code is related to and making it easy to imagine _what else_ can be done here.

Note that all of those values are highly informal, high-level, and humane.

I am not talking about DRY or "removing all the boilerplate"—but rather about "not stating the obvious" (some things traditionally considered the "boilerplate" are actually adding non-obvious context for the reader; some repetitions are valuable for clarity).

I am not also talking about code "as short as possible"—and vice versa; some redundancy in statements and some spacing in formatting is frequently a quality of the lucid code. That's how we, humans, read and write.

So, it is all about the comfort of the reader—but the _comprehension-related_ comfort, allowing one to grasp large parts of the system quickly. That's why I avoid the term "readability," which has many conflicting usages but is frequently concerned about the ease of reading each particular line by itself—sometimes, even emphasizing that it should be "easily readable for a novice."

> For the record, "should be readable for a novice" is not a totally objectionable statement. Still, it might become when treated narrowly as "shouldn't contain any element other language/older versions of Ruby don't have." Which is just disrespectful for novices: being a novice mentor for many years, I can assure you that motivated junior programmers find joy in understanding new constructs when they make life easier!

The tremendous help to the reader and the writer is provided by following the **language intuitions** (another important phrase for me... I'll write [a book](https://rubyintuitions.substack.com/) called "Ruby Intuitions" once the war is over). I believe a well-designed, consistent language imposes a lot of implicit _intuitive_ knowledge about "how things are typically done, what this combination of elements might mean, what this particular name is related to," and alike. Acquiring those intuitions creates _fluency_ in reading and writing the code that follows them.

**For an attentive programmer, those intuitions might run ahead of reality**: "in other places of the language, _this_ is possible; oh, usually you don't make me repeat _this much_; this looks like a valid and meaningful code." That's where the new features frequently emerge!

So, new language features ideally follow existing language intuitions of "what should be possible/easy to write," pushing them further and producing _new_ intuitions on the way.

## Back to the details

Over the next couple of weeks, I'll publish small(ish! it is always the case with my writing) posts about some of the features that have caused controversy and fierce discussions recently. The structure of each post would be the same:

* feature explanation;
* why it was requested;
* how it was designed and why;
* how other languages solve the problem (if applicable);
* what are the consequences for _Ruby intuitions_ of the new syntax, and how does it make one think in the code; what are the quirks/problems that emerged with the feature.

The features I plan to cover:

* **[Numbered block parameters](/blog/2023-10-11-syntax-sugar1-numeric-block-args.html)** ([Ruby 2.7](https://rubyreferences.github.io/rubychanges/2.7.html#numbered-block-parameters));
* **Pattern matching: [part 1](/blog/2023-10-20-syntax-sugar2-pattern-matching.html), [part 2](http://zverok.space/blog/2023-10-27-syntax-sugar2-pattern-matching-cont.html), [part 3](https://zverok.space/blog/2023-11-03-syntax-sugar2-pattern-matching-fin.html)** (introduced in [Ruby 2.7](https://rubyreferences.github.io/rubychanges/evolution.html#pattern-matching), took shape over a few next versions);
* **[Hash/keyword arguments values omission](/blog/2023-11-10-syntax-sugar3-hash-values-omission.html)** ([Ruby 3.1](https://rubyreferences.github.io/rubychanges/3.1.html#values-in-hash-literals-and-keyword-arguments-can-be-omitted))
* **[Argument forwarding](/blog/2023-11-24-syntax-sugar4-argument-forwarding.html)** (introduced in [2.7](https://rubyreferences.github.io/rubychanges/2.7.html#keyword-argument-related-changes), adjusted in [3.0](https://rubyreferences.github.io/rubychanges/3.0.html#arguments-forwarding--supports-leading-arguments), [3.1](https://rubyreferences.github.io/rubychanges/3.1.html#anonymous-block-argument), [3.2](https://rubyreferences.github.io/rubychanges/3.2.html#anonymous-arguments-passing-improvements))
* "Endless" methods ([3.0](https://rubyreferences.github.io/rubychanges/3.0.html#endless-method-definition))

Feel free to ping me with suggestions about other features that [Ruby has gained](https://rubyreferences.github.io/rubychanges/evolution.html) over the course of its evolution if you want them covered!

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
