---
layout: post
title:  "I forgot how to spellcheck"
date:   2021-05-06
categories: python spellchecker
comments: true
---

**I spent a year building a spellchecker, and all I got is some grumbling to share.**

When I [started rebuilding](https://zverok.space/blog/2021-01-05-spellchecker-1.html) the [world's most popular spellchecker](http://hunspell.github.io/), I had several goals. But the main one was: understand spellchecking better, share this understanding with others, and, ideally, push spellchecking development a little bit.

I spent most of 2020 building and documenting [Spylls](https://github.com/zverok/spylls). Then, I am spending my 2021 on [the series of articles](https://zverok.space/spellchecker.html) about the details of Hunspell's algorithms.

After all those months, I can proudly state that now I understand less about how the spellchecking should be done "right" in 2021 than I've understood at the beginning of the project. But I am almost sure it shouldn't be done the way Hunspell does it.

In this sense, my project has ultimately failed. All I am left with is some insights on spellchecking problem, and I am sharing them now as a tribute to the fun time spent on Spylls.

But first —

## How it started / how it's going

Here are some assumptions about Hunspell, and spellchecking in general I had at the beginning of the project:

1. Spellchecking is easy: take input text word-by-word, and look these words up in a pre-defined dictionary.
2. Spelling correction/suggestion is: apply a bunch of heuristics to guess the correct versions of the word and then score those guesses in most-probable-first order.
3. There are lots of dictionaries out there in Hunspell format, and it is a precious public asset. Reimplementation of the format reader in such a way that it would be easy to reproduce would allow other software authors to reuse dictionaries for different purposes.
4. The suggestions guessing algorithm has clearly defined boundaries. It can be extracted and then reused with a different format of dictionaries. Or, vice versa, other algorithms might be thrown in, for example, covering some cases [where Hunspell is known to be weak](https://battlepenguin.com/tech/aspell-and-hunspell-a-tale-of-two-spell-checkers/).

(To be honest, my dream was an open repository of dictionaries and algorithms from many spellcheckers like Hunspell, Aspell, Morfologik, Voikko... I hoped it would be possible, even if hard, to reuse and mix their formats and parts of their algorithms.)

As the faithful reader of the series would already know, most of those assumptions proved to be incorrect or, at least, too naive. Hunspell's dictionary format is [quite peculiar](/blog/2021-03-16-spellchecking-dictionaries.html) and contains a lot of data that makes sense only in the context of Hunpell. Even dictionary lookup [requires exact reimplementation](/blog/2021-01-14-spellchecker-3.html#lookup-takeout) of Hunspell's approach. As for the suggestion, it is [implemented by a sequence of trivial algorithms](/blog/2021-02-10-spellchecker-6.html#suggest-takeout), tightly coupled with dictionary data and providing useful results only when used in the exact order Hunspell does.

All of it makes the idea of reusing parts of Hunspell's data and code in other algorithms much less appealing.

But more important, and on a more general level, it turned out —

## It doesn't work this way

The problem with industrial-strength spellchecking is a typical Pareto: it is easy to implement and demonstrate the detection of small typos in common words, like "demonstrate"-"deomnstrate". But anyone with experience of using the spellchecker can remember occurrences when it missed important errors or suggested some "obviously unrelated" corrections. Some of us regularly yell at their spellchecker, asking rhetorically, "why don't you ...?"

To list just a few things that can go wrong (and frequently do) in word-by-word, one-dictionary-fits-all spellchecking:

* "Whether the word is a misspelling" is a probability, not a "yes-no" answer. Is the name "Aurthur" misspelling? (No, it is a rare yet existing form.) But what is more valuable: to consider it correct, or to miss the misspelling of "Arthur"? [Here](https://bugzilla.mozilla.org/show_bug.cgi?id=301712#c11), for example, is quite curious, if old, discussion of this problem in Firefox's bug tracker. Another example: in maritime-related text, "abeam" ("at a right angle to the centerline or keel of a vessel or aircraft") is probably correct, but in most other contexts, it could be a simple misspelling of "a beam"; same goes for any professional or subcultural jargon.
* Text topic, style, genre, and a decade of writing all affect "what can be correct", and also it changes quickly: a day comes, and in many (but not all!) texts, not only a lowercase "google" should be considered correct, but also "googled", "googles", and "ungoogleable" should.
* Surrounding words (which word-by-word spellchecker, such as Hunspell, ignores) are important to catch the misspellings that produce existing word, like infamous "you're/your", "their/they're/there", or pairs like "run/gun", "gate/date" etc. Considering the surrounding words is also the only way to spellcheck common phrases like "et cetera", or to catch common accidental split misspelling ("ac cidental").
* Author's personal traits: for a proficient language user, there would be one type of common errors (mostly mechanical typos). For a learner, there would be different ones (naive misspellings). For a foreigner coming from a somewhat similar language, there would still be a third class of errors (writing English words the way French ones are written or misusing entire stems). Factoring this in to provide the most relevant suggestion can be invaluable.
* Finally, the nature of mistakes and the best ways to correct them depends on the structure of the language being corrected. Some of the assumptions that are true for European languages (that swapping two letters in the word produces close enough word, that searching words by stem similarity is a reasonable optimization of dictionary lookup) might be absolutely irrelevant for languages with a different structure.

Even this short list breaks all the assumptions that Hunspell (and similar programs) are built upon.

It seems that a better spellchecker would be:

* context-aware, considering the place of the word in the text, text structure and intention, and author's traits;
* probabilities-based, with probabilities significantly adjusted by the context;
* naturally and constantly updatable with new words, new contexts, new ways to misspell.

Sounds like machine learning to you? You are not alone. Let's talk about machine learning!

## Analytical versus statistical

The solution to every intellectual problem lies between two opposites: "with understanding" and "with experience". Most of the time, we use a mix of both, though one can imagine "pure understanding, no experience" and "pure experience, no understanding" ways of solving.

When implemented in software, understanding-based (analytical) solutions require a developer to understand the domain fully and reflect it in the code; while experience-based solutions can be created by fitting a few heuristics matching most important cases and then iteratively changing these heuristics (or adding new ones) when new cases are uncovered. This iterative fitting can be performed by the developer or by another algorithm—the latter we now call "machine learning" (that's a criminal oversimplification, but it is enough for the current context).

> There is one common confusion related to using the word "model" in ML context—which isn't anything near "models" of the problem domain in the analytical sense. I'd really prefer we name those "experiences"; it would be much more appropriate ("download those _experiences_ of image recognition trained on that dataset, or _gather_ custom experiences from a different dataset by running this training script"), don't you think?

Experience works better than understanding in two base cases (which can be seen as one and the same): i) either the full understanding is too complex/unclear for human to model it in code (image recognition), ii) or there is no underlying "analytical model" in the target domain, just an infinite list of edge cases (say, bank scoring).

The problem of spellchecking seems to be asking for an experience-based solution. It is incredibly hard to factor in all the contexts, author personalities, language changes, topic-specific dictionaries analytically.

> The interesting thing about Hunspell is, that at the first sight, its terminology and data structures ("affixes", "stems", "compounding", "morphology") imply that it is analytical. But looking closer at its bunch of heuristics invoked in an arbitrary order and at how real dictionaries reuse features to cover edge cases, it becomes clear that Hunspell is rather "experience-based". The difference from the ML approach is that this experience is gathered and coded by Hunspell and dictionary developers manually through the years of its existence. [One of the previous articles](/blog/2021-02-10-spellchecker-6.html#n-gram-similarity-beautifully-ad-hoc), dedicated to n-gram-based suggest, illustrates this with pragmatic examples.

So, should we look for an ML-based spellchecker? I am aware of at least two of them: [JamSpell](https://github.com/bakwc/JamSpell) and (just recently emerged) [NeuSpell](https://github.com/neuspell/neuspell).

But, before we all get excited and abandon "legacy" approaches forever, there's one important question to ask:

## Where's the data?

There is one important thing about Hunspell dictionaries: there are a lot of them, and they are easily accessible. You can find one for almost any natural language Hunspell's algorithm is able to support. They are small, easy to redistribute, and easy to alter on redistribution (both [Firefox](https://searchfox.org/mozilla-central/source/extensions/spellcheck/locales/en-US/hunspell/dictionary-sources/5-mozilla-added) and [Chromium](https://github.com/chromium/chromium/tree/master/third_party/hunspell/google) have their custom additions to the common dictionaries).

Moreover, it is relatively easy to participate in dictionary development: the format, while having its peculiarities, can still be understood by a layperson. Adding a missing stem or a wordform to your language's dictionary is frequently as easy as making a one-line GitHub PR or an issue describing the omission, which the author will handle.

But when we are talking about ML _models_, the situation is different. Both JamSpell and NeuSpell provide a very limited number of models and recommend training your own when in need of the better/different one. But the spellchecker is frequently an important but small part of the software. Allegedly, only a tiny fraction of software authors will bother with finding/training models for all languages their software can potentially support. This will lead to a lack of spellchecking for underrepresented languages.

Another problem with spellchecking "models" is that they are black boxes, and therefore much harder to tweak, enhance and investigate.

(There are a lot of problems with black boxes, and societal one is not the last between them. Here is a [curious issue](https://github.com/hunspell/hunspell/issues/697) in Hunspell's repository suspecting that somebody "gamed" Hunspell's models to suggest "rioting" and "looting" as a correction of "voting". While easily dismissable in Hunspell's simplistic dictionaries case, such suspicions seem inevitable for black-box models. Software **is** political.)

## So what?

I actually don't know! (As the title of this article suggests.)

Nevertheless, I can **imagine some kind of open, centralized, online storage of language information**. Yes, we have Wiktionary (and [some](http://jortho.sourceforge.net/) [spellcheckers](https://github.com/nifgraup/hunspell-is#readme) even use it as their primary data source), but to this day, it lacks the context necessary for, well, context-aware spellchecking.

Of course, maintaining such storage will require overcoming a lot of challenges related to the constant gathering of new user information: spam, privacy, moderation, classification, to name a few. Of course, it is easier for corporations to gather this kind of data through their search engine input fields, online services, mobile keyboard software (I was surprised when [my favorite Android keyboard](https://en.wikipedia.org/wiki/Microsoft_SwiftKey) was bought by Microsoft, but it actually makes a lot of sense if you think about it).

I assume that building this imaginary "linguistic storage" would be a lot like building the map of the world. The first steps are easy and obvious: import existing border/road information from public sources for map; import official language dictionaries for spellchecker. But what makes the project precious for everyday use is all the small, non-obvious, ever-changing things that users need once they are done with navigating the city center or writing a formal letter in a neutral style.

Only a large, open, and diverse community can pull off such a task. In the mapping domain, OpenStreetMap managed to gather such a community. And my dream is, there once would be common linguistic information (not only spellchecking-related) with the same levels of obvious usefulness and community support.

---

For me, though, the story ends here. Despite all the frustration and complexities, "[Rebuilding the spellchecker](/spellchecker.html)" project was fun while it lasted. I learned a lot about spellcheckers, Huspell, Python (which is not my native language), algorithms, and whatnot. More than a year from the project's start, I feel I exhausted my resources for digging this topic.

I still hope the end result (both [software](https://github.com/zverok/spylls) and [writeup](/spellchecker.html)) would be useful or at least curious for some.

I'll continue to write about adjoining topics: open data, handling cultural differences in software, reading and understanding code. Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](https://zverok.substack.com/) if you are interested in those topics.

PS: "Rebuilding the spellchecker" article series were conceived as an idea for one long post-mortem article. In the end, it spanned eight installments through 5 months. I would never finish this work (maybe will never move further than the first draft of the first article) without the precious advice and razor-sharp editing of my friend and colleague Alexey Makhotkin. BTW, he now writes "[Minimal modeling](https://minimalmodeling.substack.com/)" substack: a series of takes on DB modeling approaches, challenging common truths and dark corner cases of it. Follow it; it is brilliant.
