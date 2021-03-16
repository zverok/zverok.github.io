---
layout: post
title:  "The confusing landscape of spellchecking dictionaries"
date:   2021-02-27
categories: python spellchecker
comments: true
---

People tend not to make a distinction between

http://xahlee.info/comp/aspell_vocabulary.html

Software versus data

While writing the series, I've shown that this data is hard to use without full knowledge of the algorithms. Now I want to study another part of the equation: no matter how sophisticated spellchecker is, it is useless without good dictionaries.

And good dictionaries is something that quite hard to create, maintain and reuse.

For obvious reasons, we'll talk about Hunspell's dictionaries, but lot of the considerations apply to any multilingual software with open dictionaries.

## So, where are the dictionaries?

Let's put it in the common user story formula:

**As a _spellchecker user_, I want _the good dictionary for my language_, so I _will have relevant error detection and correctioin suggestions for my spelling_.**

So, where'd you take them?.. The answer depends on **what software are you using the dictionaries with**. Some of the common answers (and their discussion) below.

**[LibreOffice Extensions](https://extensions.libreoffice.org/?Tags%5B0%5D=50) site.** That's also the usual first suggestion for generic Hunspell's usage (LibreOffice `.oxt`- extension is actually just a ZIP archive, which can be unpacked into a usual two-files structure).
  ? https://extensions.libreoffice.org/en/extensions/show/ukrainian-spelling-dictionary-and-thesaurus
  ! https://extensions.openoffice.org/en/project/ukrainian-dictionary

**[Mozilla Extensions](https://addons.mozilla.org/en-US/firefox/language-tools/) site**. Same, or not same. (Ru, Se)

Mosly on dictionary maintainers and not a primary source

Good secondary: wooorm/dictionaries


**Linux packages repository**. Deb rpm


**Google Chrome** and **MacOS**

Mozilla + https://github.com/mozilla/gecko-dev/tree/master/extensions/spellcheck/locales/en-US/hunspell/dictionary-sources
Chrome(ium) + https://chromium.googlesource.com/chromium/deps/hunspell_dictionaries/+/refs/heads/master

Dictionary sources — defunct sites, amateur linguist forums, and whatnot

## Is it so hard to maintain a dictionary?



word list
affixes/word forms
  heterogenousness

two-files, different formats for each spellchecker
how to test

settings

testing and suggestion

where?
how?

## Is it ever possible to maintain a dictionary?

the correct word and the incorrect word
algorithms creeping in dictionaries
(+numtext)
probabilities, probability converged to zero


========================================================

People tend to not make a distinction between a spellchecking software and its dictionary. If somebody writes "I googled this and that", and their editor complain "googled" is not an existing word, many [would complain](http://xahlee.info/comp/aspell_vocabulary.html) about "bad spellchecking".

As Hunspell is that "spellchecking" behind lion's share of modern software, the interesting question to ask is where the dictionaries are coming from, how are they updated and maintained?..

The answer is **it depends**!

Three major, open and accessible repositories of Hunspell spellchecking dictionaries are maintained by: [LibreOffice](https://extensions.libreoffice.org/?Tags%5B0%5D=50), its close cousin [OpenOffice](https://extensions.openoffice.org/en/search?f%5B0%5D=field_project_tags%3A157) (confusingly, extensions are compatible, but repositories are independent), and [Mozilla](https://addons.mozilla.org/en-US/firefox/language-tools/).

All three sources share the same traits:
* Own format
* Uploaded by (and might be seriously outdated)
* There are several versions
* Authority proof
* No common versioning system

> LibreOffice versions also, as far as I can guess, reused for `deb` (Ubuntu and other "user" Linuxes packaging). On the other hand, `rpm` (RedHat) packages seem to use quite outdated versions.

Hunspell dictionaries are also used in Chromium/Google Chrome. It maintains its own copy of the dictionaries: https://chromium.googlesource.com/chromium/deps/hunspell_dictionaries/+/refs/heads/master — some of them quite outdated; but there is a `_delta` files maintained. (Though, Google is switching Chrome to its own proprietary engine).

How MacOS sources and maitains its Hunspell dictionaries, I do not have a way of knowing.

Finally, perhaps the most orderly, accessible, and regularly updated list of the dictionaries is maintained by [@wooorm](https://twitter.com/wooorm) (Titus Wormer) in [GitHub repo](https://github.com/wooorm/dictionaries). (Titus is also an author of [nspell](https://github.com/wooorm/nspell)—Hunspell's port to JS!)

### The maintainers

## What it takes to maintain a (Hunspell) dictionary

As we already described, Hunspell's dictionaries aren't just lists of words. The words are split into _stems_ and _affixes_ and stored in separate files (.dic and .aff), with stems marked with _flags_ saying which affixes they might have. Aff-file also can store various settings (you can consult [official docs]() for overview, or [spylls' `Aff` docs](https://spylls.readthedocs.io/en/latest/hunspell/data_aff.html#aff) for explanations for how they are used).

The easiest—though not that easy by itself—way of creating a dictionary is to take a list of all valid language words from somewhere, prepare a list of most common affixes, and then use Hunspell's utility `munch` to convert the dictionary into stems-with-flags.

Some dictionary authors stop at this point. The dictionary prepared this way would probably work well for **identifying the misspelling** (save for languages with word compounding, which require some additional settings in aff-file).

### Those "not very important" settings

But most of the useful **suggestion** functionality requires more preparations; and there is absolutely no way for dictionary author to guess it, or automatically check "what else I should add": only reading and rereading through all the monstrous docs, and experimenting with lots of words and lots of directives will give the decent suggestion.

Say, if the `TRY` directive is omitted (list of all alphabet letters), half of the suggestion algorithms wouldn't work: "try inserting letter, try replacing one letter with another". That's how it is in one of the two Russian dictionaries.

If the `REP` directive omitted (list of common misspellings), the most typical spelling errors of the language wouldn't be corrected. That's how it is in both Russian dictionary: they'll never guess that "жаловаца" is "жаловаться" ("to complain", with "ться"→"ца" being quite common pronunciation-based misspelling).

If the `MAP` directive is omitted,

If the `KEY` directive is omitted,

And so on and so forth

### The limitation of the word forms




### The questionability of the "correct" word

