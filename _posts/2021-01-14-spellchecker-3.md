---
layout: post
title:  "Rebuilding the spellchecker, pt.3: Lookup—compounds and solutions"
date:   2021-01-14
categories: python spellchecker
comments: true
---

**_This is the third part of the ["Rebuilding the spellchecker"](/spellchecker.html) series, dedicated to the explanation of how the world's most popular spellchecker Hunspell works._**

**Quick recap**:

1. In the **[first part](2021-01-05-spellchecker-1.html)**, I've described what Hunspell is; and why I decided to rewrite it in Python. It is an **explanatory rewrite** dedicated to uncovering the knowledge behind the Hunspell by "translating" it into a high-level language, with a lot of comments.
2. In the **[second part](2021-01-09-spellchecker-2.html)** I've covered the basics of the **lookup** (word correctness check through the dictionary) algorithm, including _affix compression_.

This part is a carry-over of **lookup algorithm explanation**, dedicated to **word compounding** and some less complicated but nonetheless important concerns: word case and word-breaking. To understand this part, reading [the previous one](2021-01-09-spellchecker-2.html) is strongly suggested. At very least you should remember that there are _stems_ with _flags_, specified by `.dic`-file, with the meaning of flags defined in `.aff`-file.

## Word compounding

Many languages, like German, Dutch, or Norvegian, have _word compounding_: two stems can be joined together, producing the new word. To check the spelling of the word in the language with compounding, the spellchecker needs to break it into all possible parts, and check if there exists a combination of parts such that all parts would be correct words that are allowed inside compound words.

Hunspell has two independent mechanisms to specify the compounding logic of a language in aff-file: **per-stem flags**, and **regexp-like rules**. Sometimes both mechanisms are used in the same dictionary.

### Per-stem flag checks

There is a generic `COMPOUNDFLAG` directive to specify a flag, which, when attached to a stem, means "this stem can be anywhere in a compound" (examples from LibreOffice's Norvegian dictionary):

```
# In nb_NO.aff:
...
# Directive defines: any word with "z" flag is allowed to be in a compound
COMPOUNDFLAG z

# In nb_NO.dic:
...
fritt/CEGKVz
...
røyk/AEGKVWz
```

Both `fritt` ("free") and `røyk` ("smoke") have `z` flag, which means they could be in any place in compound word, and thus, "røykfritt" ("smoke-free" = "non-smoking") is a valid one—and "frittrøyk" too[^1].

[^1]: The second one can't be found in Google—this compound form is _grammatically correct_, but doesn't make sense.

There are also more precise `COMPOUNDBEGIN`/`COMPOUNDMIDDLE`/`COMPOUNDEND` directives, setting the flags for stems which can be only at a certain place in compounds. Flags designated by those directives could be freely mixed: a compound can consist of a part marked with generic `COMPOUNDFLAG`, and another part marked with `COMPOUNDEND`.

To check the compound word for correctness, Hunspell needs to chop off the beginning of the word, of every possible length, and check if it is a valid stem which is allowed at the beginning of the compound. If so, the algorithm recursively chops the next parts, till the whole word is split into compound parts (or no suitable parts found).

Note that depending on the word's length, and on how many dictionary words are allowed to be in compounds, the loop can take quite some time: the process we described in the [previous part](2021-01-09-spellchecker-2.html) (affix-based search of the correct form) can be repeated dozens of times for various "part candidates".

> Let [`Lookup.compound_by_flags`](https://spylls.readthedocs.io/en/latest/hunspell/algo_lookup.html#spylls.hunspell.algo.lookup.Lookup.compounds_by_flags) in Spylls documentation be your guide!

### Defining compounds as regexp-like rules

There is another way to specify compounding logic. It is implemented by `COMPOUNDRULE` directive, with statements like `A*B?C` (meaning, "correct compound consists of any number of words with the flag `A`, then one or zero words with the flag `B`, then a mandatory word with the flag `C`"). The most common use of it is specifying suffixes in numerals. For example, in the `en_US` dictionary:

```
# en_US.aff
COMPOUNDRULE 2     # we have 2 compound rules listed below
COMPOUNDRULE n*1t  # rule 1: any number (*) of "n"-marked stems, then "1"-marked stem, then "t"-marked stem
COMPOUNDRULE n*mp  # rule 2: any number (*) of "n"-marked stems, then "m"-marked stem, then "p"-marked stem

# en_US.dic
# ...defines numbers as "stems" for this rule:
0/nm
1/n1
2/nm
3/nm
4/nm
5/nm
6/nm
7/nm
8/nm
9/nm
# ...and numerical suffixes as stems, with different flags, too!
0th/pt
1st/p
1th/tc
2nd/p
2th/tc
3rd/p
3th/tc
4th/pt
5th/pt
6th/pt
7th/pt
8th/pt
9th/pt
```

This leads to Hunspell being able to say that "1201st" is correct (the rule `n*mp` matched: "1" and "2" with "n" flags, "0" with "m", and "1st" with "p"), and "1211th" is correct (another rule in action: `n*1t`), but "1211st" is not.

Handling the word correctness check in a presence of the `COMPOUNDRULE` requires to again recursively split word into possible parts—but this time already found parts should be checked for a partial match against known rules.

> [Lookup.compound_by_rules](https://spylls.readthedocs.io/en/latest/hunspell/algo_lookup.html#spylls.hunspell.algo.lookup.Lookup.compounds_by_rules) implements this in a complicated, yet concise way.

### But wait, there is more!

~~To make things more complicated~~ To match the complexity of real life, both algorithms of compound words checking need to consider:

* Numeric **limitations**: some dictionaries might limit the minimum size of a part of the compound, or the maximum number of parts.
* **Affixes:** By default, any prefix is allowed at the beginning of the compound word, and any suffix is allowed at the end; and yet, some affixes might have flags saying "it should never be in any compound", and some others might have flags saying "it is allowed _in the middle_ of the compound" (e.g. prefix to non-first or suffix to a non-last part).
* **Several rules** that, being present in aff-file, reject some compound words with seemingly correct parts as incorrect: for example, if the letter at the boundary of the compound is tripled (`fall+lucka`); if some parts of the compound are repeated (`dubb+bon+bon`); if the non-first part of the compound is capitalized, or "this regexp-like pattern is prohibited at the boundary of the compound parts", and so on.
* Some of those settings might lead to a whole **new word checking loop** in the middle of compound checking: for example, `CHECKCOMPOUNDREP` setting tells the algorithm: use the `REP`-table specified in aff-file (typical misspelled sequences of letters, like "f=>ph", usually used on suggest) to check if some part of the compound, with replacement applied, is the valid word. If yes, then it is an incorrect compound! E.g. "badabum" split into parts "ba", "da", "bum", but then, if we apply the replacement "u=>oo", turns out "daboom" is a correct non-compound word... Then we should consider "badabum" a misspelling, and the "ba daboom" is most probably what was misspelled.

> Are you thrilled? Then follow the [`Lookup.compound_forms`](https://spylls.readthedocs.io/en/latest/hunspell/algo_lookup.html#spylls.hunspell.algo.lookup.Lookup.compound_forms) docs to uncover even more dirty details.

## ...and other complications

Affix check and (de)compounding are the main parts of the algorithm, yet there is more! Just a brief overview to give you some taste:

* Words case: "kitten" can be spelled "Kitten" or "KITTEN", but "Paris" can't be spelled "paris"
  * ...but, the word might have a flag defined as `KEEPCASE` in aff-file, meaning it should be ONLY in the exact case as in the dictionary;
  * ...and there are complications with the German language: "SS" can be downcased as "ss" or "ß" ("sharp s"), and both should be checked through the dictionary, and also, when the word is uppercased, it is allowed to have "ß": "STRAßE";
  * ...and in Turkic languages casing rules for "i" are different: "i=>İ" and "I=>ı";
  * ...and the ending part of the compound might have a flag saying "this compound should be titlecased": in Swedish dictionary, there are special words like "afrika", which are allowed only at the end of compounds, and require the whole compound to be in titlecase: "Sydafrika" (South Afrika);
* Word breaking: "foo-bar" should be checked as the whole word, and also as two separate words "foo" and "bar"
  * ...unless aff-file redefines this, by prohibiting word-breaking, or changing by which patterns words should be broken.
* Some words might be present in the dictionary with a flag defined as `FORBIDDENWORD`: it is used to disallow words that are logically possible (allowed stem with allowed suffix), but this specific combination is incorrect in the language.
* There might be an `ICONV` ("input conversions") directive defined in aff-file, saying which chars convert before the spellchecking: for example, replacement of several kinds of typographic apostrophes with simple `'` to simplify the dictionary, or unpacking the ligatures (`ﬁ` → `fi`).
  * But this feature can be used not only for handling fancy typography: for example, the Dutch dictionary uses it for enforcing proper case of "ij": In Dutch, it is considered a single entity and both letters should always have the same case. It is achieved by `ICONV`-ing to ligatures: `ij`→`ĳ` and `IJ`→`Ĳ` (but `Ij` wouldn't be converted, and wouldn't be found in a dictionary, as all dictionary words also contain ligatures).
* An `IGNORE` directive, defined in aff-file, says which characters to drop before spellchecking (in Arabic and Semitic languages, where vowels may be present but should be ignored).

That's mostly the size of it!

> To go for the full ride, start reading the Spylls docs from the [`Lookup.__call__`](https://spylls.readthedocs.io/en/latest/hunspell/algo_lookup.html#spylls.hunspell.algo.lookup.Lookup.__call__) method. You won't be disappointed!

## Lookup: takeout

To reiterate on everything said above: There are good and useful dictionaries for spellchecking of many languages, freely available in Hunspell's format, and one might be tempted to reuse them in own code. But the process of going from the not-that-complicated input format to a full reliable spellchecking includes at least:

* _Reading of aff-files (consisting of multiple directive "types", with reading logic depending on particular directive) and dic-files (words with flags)—we'll talk about this interesting task in later installments[^2];_
* Affix analysis: either on-the-fly (how Hunspell and Spylls do), or once: "unpack" the list of stems with flags into words with affixes;
* Compounding analysis—unless you just want to omit the support for languages with compounding (this apparently _can't_ be solved with a pre-generated list of "all correct compound words");
* Handling of complications with word breaking, text case, special characters, and whatnot.

[^2]: I admit that starting with the proper format description would be _logically correct_, but I wanted to tell the juiciest stuff first. There would be a post on file formats and how to read them. For the impatient, Spylls docs cover [aff](https://spylls.readthedocs.io/en/latest/hunspell/data_aff.html) and [dic](https://spylls.readthedocs.io/en/latest/hunspell/data_aff.html) files quite comprehensively.

Some of those tasks are directly related to how Hunspell is built and could've been done differently. But mostly, this chapter tries to show that **"check if the word spelled correctly", especially in languages other than English, should be seen as a not-that-trivial task**, to say the least.

And we haven't even started on **corrections suggestion yet, which we'll gladly do [in the next part](2021-01-21-spellchecker-4.html)**.  Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](https://zverok.substack.com/) if you don't want to miss the follow-up!

PS: Huge thanks to [@squadette](https://twitter.com/squadette), my faithful editor. Without his precious help, the text would be even more convoluted!

