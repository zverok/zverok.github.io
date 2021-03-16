---
layout: post
title:  "What I learned while recreating the most popular spellchecker in Python"
date:   2021-01-10
categories: python spellchecker
comments: true
---

> **Abstract:** I rewrote a [Hunspell](https://github.com/hunspell/hunspell) spellchecker in Python: not to replace it, but in order to understand better how it works, why it needs to be so complicated; and to explain it to others. In this article, I am explaining the reasons to port, how it was implemented, and also share some generic thoughts and findings about Hunspell.

## What's hunspell?

> **Note:** The information on Hunspell's origins and history is mostly my guesses, following partial and incomplete sources everywhere.

Hunspell (<small>[Wikpedia article](https://en.wikipedia.org/wiki/Hunspell)</small>), initially **Hun**garian spellchecker, emerged as an alternative for previxously existing aspell/ispell/myspell somewhere in 2002 (I guess?), in a need of supporting languages with complicated suffixing/prefixing rules and word compounding. Hunspell design seemingly have proved itself to be flexible enough to support most of the word languages, and in n a few years became the most used spellchecker in the world. You have most probably used it even if you never heard of it before today: Hunspell is the default spellchecking engine in Chrome and Firefox, Libre/OpenOffice, Adobe products and MacOS (not an exhaustive list). Dictionaries in Hunspell format exist for almost all of actively used languages for which the concept of word-by-word spellchecking makes sense.

## Why port it to another language?

First time I had an idea to port Hunspell several years ago. At that time, I wanted to port it to _Ruby_ (my primary language), and I assumed it is a "weekend-long task", especially if I want only port "check if the word is correct" part (how hard could it be, read dictionary and find the word in it?). The goal was to make a small Ruby tool to check spelling in source code comments, and I wanted it to be a pure Ruby for easy usage in CI environments.

After the start of my investigation of Hunspell's formats and logic, the illusion of triviality dispelled quickly.

> My favorite inner joke about the project is that it can be thoroughly documented in "falsehoods programmers believe about <spellchecking>" style. Like "plain list of words is enough for spellchecking", "ok, then just words and possible affixes", "ok, then just words and affixes and rules of affix applications"; or, "text case is not important for spellchecking", "ok, it is, but just to distinguish the words at the beginning of the phrase", "ok, but at least there shouldn't be a case change in the middle of the word"; and so forth. The complexity grows quickly once you start adding more languages support.

The more I tried to understand what would be the "minimal" way to reuse Hunspell's dictionaries, the more I was drawn into the idea of understand the software in all of its details and dark corners (and the more I understood that almost no part of Hunspell's core algorithms could be extracted and reused independently). Eventually I became attracted to the idea of "explanatory port": to rewrite the algorithms in some high-level language (which would allow to bypass the accidental complexity of memory allocation and freeing, working with encodings in byte-by-byte fashion, etc.), in the most clear way I'll be capable of, and with detailed comments explaining all the ins and outs.

Through the next several years, I approached the task a few times (in Ruby, Rust, and Python), and eventually settled down on the Python as the implementation language, for it being almost a "lingua franca" of high-level languages. I still miss my Ruby, but I wanted the implementation which would represent the understandin of Hunspell's algorithms, not my understanding of how the software should be written.

**The resulting library can be seen [on GitHub](https://github.com/zverok/spylls), and its [documentation](https://spylls.readthedocs.io/en/latest/) on class and method levels, as well as inline in the code, tries to give as full explanation of all the details and complexities as I found possible.**

## Implementation, and why is it so complex?

There are two main part for word-by-word spellchecker algorithms:

1. Check if some word is correct ("lookup")
2. Propose the correction for incorrect words ("suggest")

(Hunspell-the-software includes a lot of other algorithms, like parsing text out from HTML/TeX/Markdown, splitting it into separate words (tokenizing), and other "wrapping" parts necessary for the pipeline of spellchecking to work in real life, but we'll not stop at those here: they are implemented in a lot of libraries and packages, and have only tangential relation to spellchecking itself.)

When coming from English-only spellchecking, the developers tend to perceive the first, "lookup" part as trivial (so that the famous [Peter Norvig's article](https://norvig.com/spell-correct.html) starts from the assumption that only the correction—"suggest"—part deserves some explanations). But that's not quite so.

### Lookup

The first and most straightforward idea about lookup would be: we'll just take the dictionary (presumably, the flat list of all correct words) and look for our candidate word in this list: if it is there, it is correct. End of story.

With this assumption, and the knowledge that there are dictionaries for all known languages in simple text format, prepared for Hunspell, one might expect the use of those dictionaries would be something close to one-liner. But, downloading any dictionary, one then finds each line has the following format (from `en_US.dic`):

```
ache/DSG
```

With `ache` being a _stem_, and `D, S, G` attached _flags_. The meaning of flags is explained in `en_US.aff` (named "affix file"; every Hunspell dictionary is distributed as a pair of `.dic` and `.aff` files), in this particula case coding the fact that the stem `ache` might have suffixes `-s`, `-ing` (which should remove `-e` from stem, when applied) and `-d`; so this line of the dictionary, in combination with settings from affix file, encodes words "ache", "aches", "aching", and "ached".

This technique, called "affix compression", allows to optimize dictionary size (on disk and in memory) at the price of increased search time and complexity: it is not "(binary) search through the flat list", but "check if the word has one of the known suffixes, try to get the stem without this suffix, now check through the dictionary if the stem exists and has the flag for this suffix". Words also can have prefixes; suffixes and prefixes might specify they are incompatible with each other; moreover, suffixes and prefixes might have they own flags marking "can have a suffix/prefix", allowing, as of today, maximum depth of 2 suffixes and 2 prefixes (which might be not enough for some languages -- for example, Finnish developers maintain their own library [voikko](https://voikko.puimula.org/) for this reason).

Some kind of dictionary compression is necessary for languages with a high rates of word flexion: if in English one stem might have 2-3 possible suffixes, in Ukrainian it will easy be dozens, and if all possible word forms would be stored in a linear list, it will take unreasonable amounts of memory (or completely different format needs to be choosen: for example, Java spell checker [morfologik](https://github.com/morfologik/morfologik-stemming) uses binary format based on finite state automata to achieve reasonable memory footprint with much higher lookup speed).

> Random fact: In existing dictionaries, "suffixes" and "prefixes" specified for some language may, or may not correspond to grammatical suffixes/prefixes of the language. Most frequently, they are _not_: the splitting into stems and affixes is deduced automatically from flat word lists, so it is up to probabilities whether "common endings" deduced by script have any grammatical meaning. Actually, Hunspell's format has a rich [sub-language to specify grammatical information](https://manpages.debian.org/experimental/libhunspell-dev/hunspell.5.en.html#Optional_data_fields), but of all LibreOffice and Firefox dictionaries I've checked, only a few (Latvian, Slovak, Galician, Breton) made a use of this feature.

And once you tackle the affixes problem, it is all uphill from there!

The next not-so-obvious thing is that a lot of languages has _word compounding_: basically, two or more stems could be joined together to produce a valid word. Defining which stems could be joined and which could not, which ones could be only at the beginning or only at the end of the compound word, what happens with suffixes and prefixes of compound words -- this all takes a lot of settings in affix file, and a lot of processing during the lookup. Imagine that any "foobar" can actually be "foo+bar", "fo+obar", "f+oobar", and so on, and also "'fo' with suffix 'o' + 'tor' with prefix 'ba' (which chops off 'to' before applying)"; or, for that matter, it also might be "foo+obar" with separate setting "simplify tripling letters"—imagine all this on word lookup, and simplicity and performance go out of the window.

There are ton of edge cases piled on top of it, like word casing: both "Kitten" and "kitten" are correct, but "paris" is not; and in some languages there are _suffixes_ that require the rest of the word to become capitalized; or, there are also word-breaking rules ("foo-bar" might be need to looked up as the whole word "foo-bar", or "foo" and "bar" separately... but only unless there is a setting in affix-file to prohibit such breaking) and ... well, a whole lot of other stuff.

Does it all _should_ be this complex? It seems to be grown organically by different language users' requests, and to solve real users' problems. But let's postpone the theoretical discussion of the overall design till closer to the end of this series.

For now, I urge you to not take my bare word for the description, and check on the [Lookup](https://spylls.readthedocs.io/en/latest/hunspell/algo_lookup.html) module description in the `spylls` reimplementation (including the inline docs available when unfolding the methods' source code), which provides the detailed overview of the complexity.

In the next series:

* Explanations on the Suggesting algorithm;
* Deeper dive into dictionary/affix files and how they are looking in the wild;
* The philosophical overview of the big picture of word-by-word spellchecker problem, and Hunspell's approach to it.

Stay tuned.

### Suggest

## On the unreducible complexity

## Conclusions: the problem with Hunspell

## Links
