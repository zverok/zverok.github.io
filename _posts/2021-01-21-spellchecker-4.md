---
layout: post
title:  "Rebuilding the spellchecker, pt.4: Introduction to suggest algorithm"
date:   2021-01-21
categories: python spellchecker
comments: true
---

**_This is the fourth part of the "Rebuilding the spellchecker" series, dedicated to explaining how the world's most popular spellchecker Hunspell works._**

Today's topic is **suggest**!

**Quick recap**:

1. In the **[first part](2021-01-05-spellchecker-1.html)**, I've described what Hunspell is; and why I decided to rewrite it in Python. It is an **explanatory rewrite** dedicated to uncovering the knowledge behind the Hunspell by "translating" it into a high-level language, with a lot of comments.
2. In the **[second part](2021-01-09-spellchecker-2.html)**, I've covered the basics of the **lookup** (word correctness check through the dictionary) algorithm, including _affix compression_.
3. In the **[third part](2021-01-14-spellchecker-3.html)**, the rest of the lookup is explained: compounding, word breaking, and text case.

And now, we'll switch to the juiciest part of the spellchecking problem: guessing the corrections for the misspelled word, called _suggest_ in Hunspell. This post only draws the big picture of suggestion algorithms in general and the Hunspell's particular flavor. Even more ~~nasty~~ amazingly curious details would be covered in the next issue (or, rather, issues).

## The problem with suggest

The question "how the suggest works?" was what drew me initially to the project. The lookup part seemed trivial. And even if, as I understood later, it is not that trivial, the lookup is still a task with a _known answer_. The word is either correct or not; the spellchecker, however it is implemented and however it stores its data, should just say whether it is correct. All the complexity of lookup implementation is only a set of optimizations, because it is hard or impossible to just store a list of "all correct words".

**But suggest is a different beast altogether.** There are many ways to misspell a word, due to mis<i>typing</i>, genuine error, or OCR glitch; and going back from the misspelled word to the correct one is no easy task. Frequently, only the text's author can say for sure what is right: was "throught" meant to be "through", "thought", or maybe "throughout"?.. What about "restraunt": "restraint" or "restaurant"? Ideally, there should be exactly one guess (then we can even apply auto-correct to the user's text), but that's rarely the case.

Even when the human can guess "what word was misspelled here", it is not always obvious what is an algorithmic way to deduce the correct word from the misspelled one, such that its results _felt correct_ for the human. Moreover, the algorithm found for one case or set of cases may produce an irrelevant result in others, and it is hard to find the objective measure of whether your suggester is "good".

So, while lookup approaches vary only by their performance, the smallest tweaks in the suggestion algorithm might produce dramatically different results.

## How it can be done

The famous article by Peter Norvig "[How to Write a Spelling Corrector](https://norvig.com/spell-correct.html)" describes the possible algorithm in these steps:

* generate multiple "edits" of the word (insert one letter, remove one letter, swap two adjacent letters, etc.)
* from all edits, select the words that are present in the dictionary;
* rank them by word's commonness (using a source dictionary with weights, or a big source text which is summarized to "word → how often it is used");
* take the first one as a singular good suggestion.

The entire algorithm implementation in Python takes less lines than most of the core methods of Spylls.

> Note that Norvig's article is an awesome, concise, and friendly explanation of the basic _idea_ of how spellchecking _might_ work, intended to create the intuition about the process. But it is by no means enough to build a good spellchecker. Unfortunately, quite a few libraries exist that claim to be production-ready spellchecking solution implementing "the famous Norvig's algorithm". They ignore both "The full details of an industrial-strength spell corrector are quite complex..." at the very beginning of the article and a large section "Future Work" in the end. In real life, the results are typically less than satisfying. Much less.

Some of the modern approaches to spellchecking still take this road: for example, [SymSpell](https://github.com/wolfgarbe/SymSpell) algorithm (claiming to be "1 million times faster") is at its core just a brilliant idea for a novel storage format for a flat word list, that allows optimizing the calculation of edit distance significantly.

Most of the "industrial-strength spell correctors" (using Norvig's definition), though, are multi-stage. They produce possible corrections with several different algorithms and, most frequently, return several suggestions, not relying on the algorithm's ability to guess the very best one.

For example, [Aspell](http://aspell.net/), one of the Hunspell's "uncles"[^1]  (still considered by some to have better suggestion quality _for English_), has quite [succinct description](http://aspell.net/man-html/Aspell-Suggestion-Strategy.html) of its suggestion strategy, and even exposes command-line options for the user to control [some parameters](http://aspell.net/0.50-doc/man-html/4_Customizing.html#suggestion) of this strategy.

[^1]: Aspell is older than Hunspell, but it is not its direct ancestor. There was once an old [Ispell](https://en.wikipedia.org/wiki/Ispell), then [Aspell](https://en.wikipedia.org/wiki/GNU_Aspell) and [MySpell](https://en.wikipedia.org/wiki/MySpell) were created independently to replace it, then Hunspell superseded MySpell (and also Aspell took some features from MySpell too, namely affix compression). It's complicated. "Uncle" would be the most appropriate family relation.

Hunspell's approach is much more complicated, not to say "cumbersome". From what I can guess—I didn't dive deep into history and reasoning behind all the decisions—it grew organically with Hunspell's popularity, resulting from a multitude of cases and requirements from users of a variety of languages. There is no single "complex algorithm" that can be extracted and explained on the whiteboard, but rather a sequence of simpler algorithms. They are guided by a ton of settings that can be present in aff-files and kept together by lots of tests.

## How Hunspell does it

Hunspell does the search for a correction in the following stages:

1. Generate a list of edits and check their correctness with the lookup, but
  * there are many more of them than the classic insert-delete-swap-replace; in fact, more than dozen, depending on the particular language meta-information provided by aff-file;
  * there is no ranking/reordering of edits (neither by word popularity nor by closeness to the original word); the order of their calculation _is_ the order they will be returned: it is assumed that Hunspell's code already applies edits in the highest-probability-first order.
2. If there were no results on the edit stage, or they weren't considered very good (more on this later), the search through the entire dictionary is performed:
  * the similarity of the misspelled word and each dictionary stem is calculated with rough and quick formula;
  * for top-100 similar stems, all of their affix forms are produced, and similarity to them is calculated with another rough and quick formula;
  * for top-200 of similar affixed forms, a very complicated and precise formula is used to choose only the best suggestions.
3. There _might_ be an optional third stage: metaphone (pronunciation) based suggestions... Although, it depends on the existence of the metaphone encoding data in dictionary's aff-file, and there is a _very_ small number of such dictionaries in the wild (namely, one). We'll touch on this curious topic briefly in the future.
4. Finally, some post-processing is performed on the suggestion, like converting it to the same character case as an initial misspelling (_unless_ it is a prohibited case for this word!) or replacing some characters with "output conversion" rules.

> For the impatient: we'll cover the details of the implementation of each stage in the future posts, but you can begin reading the docs and the code right now, starting from the [`algo.suggest`](https://spylls.readthedocs.io/en/latest/hunspell/algo_suggest.html) module.

## Quality estimation

Is Hunspell's suggestion algorithm good? And _how_ good is it?

Those questions are open ones—and even the way they can be answered is unclear. Intuitively, Hunspell's suggestions are quite decent—otherwise, it wouldn't be the most widespread spellchecker, after all. A fair amount of "unhappy customers" can be easily found, too, in [hunspell's repo issues](https://github.com/hunspell/hunspell/issues). At the same time, one should distinguish between different reasons for the sub-par suggestion quality. It might be due to the algorithm itself, or due to the source data quality: the literal absence of the desired suggestion in the dictionary, or lack of aff-file settings that could've guided Hunspell to finding it.

Hunspell's development process, to the best of my knowledge, doesn't use any realistic text corpora to evaluate suggestion algorithm—only feature-by-feature synthetic tests.

> In contrast, Aspell's site [provides an evaluation dataset](http://aspell.net/test/cur/) for English, including comparison with Hunspell (Aspell wins, by a large margin). Hunspell's repo actually [contains](https://github.com/hunspell/hunspell/tree/master/tests/suggestiontest) something similar: script to evaluate Hunspell vs. Aspell based on Wikipedia's [List of common misspellings](https://en.wikipedia.org/wiki/Wikipedia:Lists_of_common_misspellings) (Hunspell wins), but mostly for informational purposes: the results are neither promoted nor used as a reference point for further development.

The current Hunspell's development consensus "what's the best suggestion algorithm" is maintained by a multitude of synthetic [test dictionaries](https://github.com/hunspell/hunspell/tree/master/tests), validating that one of the suggestion features, or set of them, works (and frequently indirectly validating other features). This situation is both a blessing and a curse: synthetic tests provide stable enough environment to refactor Hunspell (or to rewrite it in a different language, IYKWIM); on the other hand, there is no direct way to test the _quality_—the tests only confirm that _features work in expected order_. So, there is no way to prove that some big redesign, or some alternative spellchecker passes the quality check at least _as good as Hunspell_ and improves over this baseline.

> There is, for example, a curious [evaluation table](https://github.com/bakwc/JamSpell#benchmarks) provided by a modern ML-based spellchecker JamSpell. According to it, JamSpell is awesome—while Hunspell is a mere 0.03% better than dummy ("fix nothing") spellchecker... Which doesn't ring true, somehow!

My initial assumption for the Spylls project was that understanding the current implementation in full would be a precondition for public experimentation to improve it significantly. Or—as I dreamed—we'll be able to mix-and-match approaches of several spellcheckers (at least Hunspell and Aspell, considering, say, [the popular article](https://battlepenguin.com/tech/aspell-and-hunspell-a-tale-of-two-spell-checkers/) demonstrating the cases where the latter beats the former). What I uncovered, though, makes me suspect that relying on feature-by-feature tests and strict ordering of simple algorithms makes Hunspell too rigid for a breakthrough quality improvement... But more on this later.

For now—however we estimate the quality, _practically, it works_. **In the [next part](2021-01-28-spellchecker-5.html),** we'll look closely at all the hoops Hunspell jumps through in order to provide meaningful **edit-based suggestions**.  Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](/subscribe.html) if you don't want to miss the follow-up!

PS: Huge thanks to [@squadette](https://twitter.com/squadette), my faithful editor. Without his precious help, the text would be even more convoluted!
