---
layout: post
title:  "Rebuilding the spellchecker, pt.2: Just look in the dictionary, they said!"
date:   2021-01-09
categories: python spellchecker
comments: true
---

**_This is the second part of the "Rebuilding the spellchecker" series, dedicated to the explanation of how the world's most popular spellchecker Hunspell works._**

**Quick recap:** [In the first part](2021-01-05-spellchecker-1.html), I've described what Hunspell is; and why I decided to rewrite it in Python. It is an **explanatory rewrite** dedicated to uncovering the knowledge behind the Hunspell by "translating" it into a high-level language, with a lot of comments.

Now, let's dive into how the stuff really works!

There are two main parts of word-by-word spellchecker algorithms:

1. Check if a word is correct: **"lookup"** part
2. Propose the correction for incorrect words: **"suggest"** part

> Hunspell also implements several other algorithms to be useful as standalone software. It can extract plain text from numerous formats, like HTML or TeX, split it into words (tokenize), correctly handling punctuation—but at the end of the day, a word-by-word correctness check is applied. I excused myself from implementing "wrapper" algorithms: text extraction and tokenization are thoroughly investigated topics, and there are numerous libraries in any language solving it with decent speed and quality.

Hunspell works on a word-by-word basis (no context is taken into account). Each word is just **looked up** in the **dictionary** loaded from the plaintext  `<langname>.dic` file in Hunspell-specific format. If it is not considered correct by dictionary lookup (which, as we'll see soon, is more complex than "is it present in the dictionary"), several algorithms of **suggest** are applied sequentially, trying to find correct words similar to the given one.

## Hunspell's lookup algorithm, or, Just look in the dictionary, they said!

When coming from English-only spellchecking, the developers tend to perceive the "lookup" part as trivial (e.g., the famous [Peter Norvig's article](https://norvig.com/spell-correct.html) starts from the assumption that only the correction—"suggest"—part deserves some explanations). But that's not quite so.

The first and most straightforward idea[^1] for the lookup would be: we'll just take the dictionary (presumably, the flat list of all correct words) and look for our candidate word in this list: if it is there, it is correct. End of story.

[^1]: [This blog entry](https://prog21.dadgum.com/29.html) circa 2008, recently [seen on HackerNews](https://news.ycombinator.com/item?id=25296900), even uses the spellchecking as an example of how much we've progressed with our tools, making the spellchecker implementation as trivial as hashmap lookup.

Now, Hunspell's dictionaries exist for many languages and have a plaintext format, which, at the first sight is quite close to a plain word list—so, probably it would be easy to reuse them? Let's take a look into the [`en_US.dic`](https://github.com/LibreOffice/dictionaries/blob/master/en/en_US.dic) in the LibreOffice dictionary repository. You'll see a list of words in the following format:

```
...
acetyl
acetylene/M
ache/DSMG
achene/MS
achievable/U
achieve/BLZGDRS
achievement/SM
...
```

The line `ache/DSMG` specifies the _stem_ `ache`, having `D`, `S`, `M`, `G` _flags_ associated with it. The meaning of flags is defined by [`en_US.aff`](https://github.com/LibreOffice/dictionaries/blob/master/en/en_US.aff) (called "affix file", or just aff-file; every Hunspell dictionary is distributed as a pair of `.dic` and `.aff` files).

### Affix compression

In this particular case, all four flags are associated with word _suffixes_. Here's the definition of `D` suffix:

```
SFX D Y 4                    # Suffix header: suffix (SFX), with flag D, combinable with prefixes (Y), 4 entries:
SFX D   0  d    e            # * if the stem ends is "e", strip nothing (0), and add "d"
SFX D   y  ied  [^aeiou]y    # * if the stem ends with "y", preceded by non-vowel, strip "y" and add "ied"
SFX D   0  ed   [^ey]        # * if the stem ends with not "e", and not "y", strip nothing, add "ed"
SFX D   0  ed   [aeiou]y     # * "y" with preceding vowel: strip nothing, add "ed"
```

For our `ache` stem, this definition says that form `ached` exists. In the similar fashion, `S` flag defines that word `aches` exists, `G` flag defines `aching`, `M` flag defines `ache's`. So, the `ache/DSMG` line in `.dic` file specifies 5 correct words: "ache", "ached", "aches", "aching", "ache's".

> Note: the fact that flags are similar to the suffixes they define (`S` flag defines suffix `-s` and so on) is just a convention. It is rather a handy mnemonics that creators of `en_US` dictionary used, and there could be any other symbol.

In the similar fashion, word prefixes might be defined (specified with flags in dic-file, and described with `PFX` directive in aff-file). Say, this definition:

```
advantage/GMEDS
```

...defines these forms: _advantage, advantage's, advantaging, advantaged, advantages, disadvantage's, disadvantaging, disadvantaged, disadvantages, disadvantage_ (all combinations of four suffixes and a prefix "dis-").

This technique of "packing" dictionaries is called **affix compression**, and its primary goal is to optimize dictionary size: on disk and in memory. It becomes extremely important for languages with rich inflection. For example, in English one stem might produce no more than ten forms, but in Ukrainian it could easily be dozens or even hundreds, so the amount of possible correct forms quickly grows to tens of millions. "Just a flat list of all known words" _might_ become impractical (not on today's top MacBook, probably, but still), and that's where the affix compression comes in handy.

> This is not the only possible approach to make dictionary storage more effective: for example, [morfologik](https://github.com/morfologik/morfologik-stemming) (the default internal spellchecker of the most widely used open-source proofreading software [LanguageTool](https://languagetool.org)) codes _all_ possible forms in binary files, using finite-state automata. And this approach is very efficient by speed and memory, but very hard for humans to edit and review, and thus, to keep dictionary up-to-date.

Another benefit of splitting words into stems and affixes: when Hunspell's user wants to add a new word to their personal dictionary, Hunspell allows to just specify "it is inflected the same way as (some other word)", thus sparing the user of teaching the dictionary "'monad' is a word, and 'monads' too, as well as 'monad's'...".

Note, though, that "suffixes" and "prefixes" specified in some language's dictionary not necessarily correspond to _grammatical_ suffixes/prefixes of the language. The splitting into stems and affixes is deduced automatically by dictionary authoring tools from flat word lists, so it is up to probabilities whether "common endings" deduced have any grammatical meaning. Actually, Hunspell's format has a rich [sub-language to specify grammatical information](https://manpages.debian.org/experimental/libhunspell-dev/hunspell.5.en.html#Optional_data_fields), but of all LibreOffice and Firefox dictionaries I've checked, only a few (Latvian, Slovak, Galician, Breton) made use of this feature.

> One important factor to mention is Hunspell's limitation for the amount of suffixes/prefixes a word may have. Currently, the software understands no more than 2 suffixes and 2 prefixes in a word, which is lower than the common number of grammatical suffixes a lot of languages allow. Most of the dictionaries solve this by "linearizing" the word list: Ukrainian word "громадянство" (citizenship) grammatically consists of the stem "громад-" and suffixes "-ян-", "-ств-", "-о" (the latter is called an ending in grammatically correct terms), but Hunspell's Ukrainian dictionary includes the full word "громадянство". For other languages, the suffix number limitation makes Hunspell totally unusable, so to spellcheck the Finnish, you need to install [Voikko](https://voikko.puimula.org/) spellchecker.

**Affix compression comes with a price** paid in lookup algorithm complexity and performance. Instead of just a quick lookup through a hashtable or other lookup-optimized structure, we now have to:

1. Check if the whole word is in the list of stems. If yes, it is correct,
  * ...unless it has a flag corresponding to the aff-file directive "this stem _requires_ prefixes or suffixes".
2. If no, check if the word has some of the known suffixes; and if so, whether the stem without one of those suffixes is in the stem list, _and_ has a flag corresponding to this suffix.
3. If no, check if one more suffix can be found in the stem (then we'll have a stem and two suffixes, and need to check the compatibility of their flags).
4. Repeat with prefixes (up to two), and with all possible suffix-prefix combination (taking into account whether suffix and prefix both have "cross-product" allowed).
5. Consider that suffixes and prefixes can have flags of their own, specifying "suffixes with this flag might be attached after me", or "if used, this suffix requires that at least one other affix would be present" and ... many other things.
6. And there are funny cases like `CIRCUMFIX` flag: if the suffix has it, this means that this suffix is only allowed in words having a prefix with the same flag.

> It is _still_ a simplified description. To follow the algorithm in full, you can read Spylls docs starting from [`Lookup.affix_forms`](https://spylls.readthedocs.io/en/latest/hunspell/algo_lookup.html#spylls.hunspell.algo.lookup.Lookup.affix_forms), follow the links to methods it invokes, and read inline comments under "Show code".

Performance-wise, for each word correctness check, there could be many lookups through known suffixes and prefixes lists, and quite a few dictionary lookups (as consecutive chopping off of suffixes and prefixes produces new stems we need to check).

And once you tackle the affixes problem, it is all uphill from there!

**Stay tuned for the next installment about the Hunspell lookup, where [we cover](2021-01-14-spellchecker-3.html) word compounding problem, and some other important edge cases of the word correctness check.** Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](/subscribe.html) if you don't want to miss the follow-up!
