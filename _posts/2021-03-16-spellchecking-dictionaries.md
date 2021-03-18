---
layout: post
title:  "17 (ever so slightly) weird facts about the most popular dictionary format"
date:   2021-03-16
categories: python spellchecker
comments: true
---

**Hunspell is the most used spellchecker in the world.** It is built-in Mozilla Firefox, Google Chrome, Libre/OpenOffice, MacOS, Adobe products, and whatnot.

Thus, **Hunspell dictionaries are the most common open dictionaries format**; they are available for almost a hundred of world languages. The temptation to reuse those dictionaries for text processing is high: they are _somewhat_ suitable not only for spellchecking but also for determining the canonical (base) form of words, canonical capitalization, stemming, etc.

**But the dictionary format of Hunspell has a lot of peculiarities.** Depending on your mindset, you might find the facts below curious, fascinating, ridiculous, or just plain boring. From my perspective, they are, first of all, didactic: they demonstrate the blossoming complexity of organically evolved software that solves the complicated task.

> This text is one of the results of ["Rebuilding the spellchecker"](/spellchecker.html) effort, striving to explain how the world's most popular spellchecker Hunspell works via its Python port called [Spylls](https://github.com/zverok/spylls). Six-part series already covered all the algorithms in Hunspell, and now we switch to broader topics.

### 1. The dictionary is stored in two text files

A dictionary for each language consists of two human-readable text files (more on that below).  Some dictionaries are developed in the open, and you could find those files in Github or some other source repository.  It's relatively easy to find and fix the issues in such dictionaries.

However, dictionaries are usually packaged for distribution as software-specific packages: .xpi for Mozilla, .odt of LibreOffice, .deb for Debian/Ubuntu.  Those are ZIP files with a different file extension and some additional meta-information inside.

If your dictionary is only available as a package and you want to make some changes, you would have to unpack them or find the original source.

### 2. The first file contains the list of words, but it can't be interpreted without the other

The `<languagename>.dic` file, or dictionary file, has a simple format: one word per line. The line might contain just a word or, more commonly, a word with flags (separated by `/`).

```
speleological
speleologist/MS
speleology/M
spell/JSMDRZG
spellbind/ZGRS
spellbinder/M
spellbound
```

Here, for example, "speleological" has no flags; "spell" has flags J, S, M, D, ... Only the second file (`<languagename>.aff`) can tell the meaning of those flags. (In this particular case, it tells that for the base form "spell", several suffixes can be used, effectively declaring that words "spelled", "spells", "spelling", ... are correct).

Some dictionaries attach even more information to the word (see 16).

### 3. The other file is called the "affix file", but it contains much more than just affixes

The second file of the dictionary has the `.aff` extension and is usually called "affix file". But in modern Hunspell, it not only stores affixes (suffixes and prefixes) for base forms declared in dictionary files, but a whole lot of _directives_. Here is the beginning of `en_US.aff`:

```
SET UTF-8
TRY esianrtolcdugmphbyfvkwzESIANRTOLCDUGMPHBYFVKWZ'
ICONV 1
ICONV ’ '
NOSUGGEST !

COMPOUNDMIN 1
ONLYINCOMPOUND c
```

It sets encoding (for both `.aff` and `.dic` files); describes language's alphabet (sorted by letter frequency); instructs to convert `’` (typographic apostrophe) to `'` before checking; sets flags to mark some words as not-to-be-suggested (`!`), and words that can only be parts of other words (`c`), and sets a minimal size of the word for word compounding (1 character instead of the default 3: English dictionary uses "compounding" feature to check the spelling of ordinals like "111th").

And it is just the first 7 lines of a 200-something-line file, for the language with one of the simplest settings.

### 4. There is no specification for the comments format

A lot of existing dictionaries have comments in their `.aff`- and `.dic`-files. Comments seem to begin with the `#` character (as it is common for many other scripting and config file formats).

However, Hunspell's code doesn't have special logics for handling `#`. While reading affix-file, it just ignores lines that do not start with the known directive; and the rest of every line after the expected directive's content. This allows a comments _agreement_ to exist between dictionary authors without being explicitly implemented.

Hunspell-compatible software should not treat `#` specially: for example, `en_GB` dictionary uses it as a _flag_.

Oh, and the `.dic`-file ignores no lines, so if the dictionary's author included "comments" in there (and they do!), those would be interpreted just like the other words.

For example, checking the spelling with `an_ES` dictionary, one of the suggestions would be what the dictionary author meant to be just a comment:

```
$ echo "verbosirregulars" | hunspell -d an_ES
verbos irregulars, verbos-irregulars, # VERBOS IRREGULARS
                                      ^^^^^^^^^^^^^^^^^^^
```

### 5. There are 54 directives Hunspell understands

Some of them define global dictionary settings (for example, `SET`—encoding); some provide language-specific data for internal Hunspell's algorithms (like `REP`—table of typical misspellings); some tune those algorithms (`MAXNGRAMSUGS`—limit for n-gram-based suggestions) or turn them off completely (`COMPLEXPREFIXES`—whether some words might have double prefix); some just assign meaning to _flags_, that then can be attached to words (like `SFX`: a word with this flag can have that suffix; or `KEEPCASE`: a word with this flag should always be in the exact case provided by dictionary).

None of the directives are mandatory.

### 6. A lot of directives have unique syntax

Many directives have a simple format of `DIRECTIVE_NAME <value>` (with a value known to always be numeric, or always be a valid flag, or always be just a string). Still, many have value logic unique for this particular directive. For example, `KEY` directive has format `qwertyuiop|asdfghjkl|zxcvbnm`. It defines the keyboard layout of the target language, used to analyze if this character is mistyped because it is close to another on the keyboard; pipe characters separate keyboard rows.

Many directives are multi-line tables, with a format like:

```
REP 90
REP a ei
REP ei a
REP a ey
REP ey a
REP ai ie
...85 more rows...
```
(This reads: there are 90 rows for the directive `REP`—typical misspellings—below.) How many values are in every row is defined by the directive itself: for `REP`, it is two, for `MAP`—one, and so on. (And remember: everything besides the expected number of values is silently ignored, as it could be a "comment").

Interpretation of the row also depends on the directive: it can be just strings, Regexp-like patterns, flags, or something unique; e.g. `MAP` directive (characters that should be considered similar) in each row has a simple string of characters, but it also supports character groups with parentheses, like in German dictionary: "ß(ss)".

### 7. Some directives define data tables used by Hunspell algorithms

Suffixes and prefixes are defined in tables such as:

```
SFX D Y 4
SFX D   0     d          e
SFX D   y     ied        [^aeiou]y
SFX D   0     ed         [^ey]
SFX D   0     ed         [aeiou]y
```

It reads: _Words with flag `D` can have this suffix. They also can have prefixes (`Y` in the header line). First data row: If the word ends in `e`, to apply this suffix, we need to strip empty string (`0`) and add `d`. Second data row: If the word ends with `y`, but not one of `aeiou` before it, we need to strip `y` and apply `ed`. [and so on]_

Another example is the usage of `COMPOUNDPATTERN` directive: it is described as a way of defining rules to build compound words with regexp-like rules (`a?b*cd`: "valid compound is: 0 or 1 words with the flag `a`, then any number of words with flag `b`, then one word with flag `c` and one word with flag `d`"), but it turned out that basically the only usage of this approach is to define an algorithm of ordinal indicators for numbers (e.g., "2051st" is correct, but "2051nd" is not). _Curious fact: László Németh, Hunspell's original author, also develops [Soros language](https://numbertext.github.io/) for describing rules of spelling out numbers in words, in any human language._

And there's a `PHONE` directive that describes the "phonetical encoding" rules as data (which other phonetical encoding libraries put in code). It allows to make phonetical encoding multi-lingual ... but only hypothetically, as only the English implementation of this table exists.

### 8. Some dictionaries would rely on the most peculiar directives to work properly

`ICONV` directive is used for preprocessing the text before spellchecking: usually, just dealing with extra Unicode (turning `’` into `'`, converting characters with diacritics into a canonical form, and so on).

But in the Dutch dictionary, one of `ICONV` rules is key for handling the peculiarity of the Dutch grammar: digraph `ij` should be considered a single character, so, for example, "ijsberg" and "IJsberg" is correct, while "Ijsberg" is not. To handle it without introducing the knowledge into the Hunspell's _code_, the dictionary uses the clever trick: `ICONV` preprocessing rule converts `ij` to `ĳ` (single Unicode character), and all dictionary words include this character instead of separate `i` and `j` (so, there is no way to capitalize them separately).

Therefore, any software that uses the Hunspell Dutch dictionary should handle `ICONV` correctly.

### 9. Some directives support mind-blowing edge cases ... that nobody seems to need

There are half a dozen directives that have special forms that are never used in any dictionary.

For example, `COMPOUNDPATTERN <pattern1> <pattern2>` directive is used when checking the spelling of compound words to specify "this combination of word part ending, and next part beginning is disallowed" (therefore, the word containing it shouldn't be considered correct compound). It has a third optional parameter "...so it is replaced with _this_": e.g. `COMPOUNDPATTERN oo ba blah` specifies that, having "foo" and "bar" in a dictionary, we shouldn't consider "foobar" a correct compound word, but with "ooba" replaced with "blah", "fblahr" is a correct compound of "foo" and "bar". This complicates compound word correctness check significantly. No existing dictionary uses this feature.

### 10. Flags are the main way of tagging the words for further processing, and they can assign a lot of meanings

This "simple" entry from `sv_SE.dic`:

```
afrika/AcYZ
```

Describes (when interpreted with all the relevant directives from `sv_SE.aff`) that the word "afrika":
* can be only used in compounds and not as a separate word (`ONLYINCOMPOUND Z`);
* can only be at the end of a compound word (`COMPOUNDEND Y`);
* it forces the whole compound to be capitalized (`FORCEUCASE c`);
* ...and can have suffix `-s` (`SFX A` for producing genitives).

And that's not even half of what flags can tell about the word.

### 11. Affixes may have flags of their own, significantly changing the behavior of some algorithms

Again, from the Swedish dictionary:

```
SFX  t  N  8
SFX  t  la  el/WXZ  la
SFX  t  ma  0/WXZ   ma
SFX  t  me  0/WXZ   me
# ...
```

`/WXZ` are three flags saying:
* we can use this suffix only inside compound words (`ONLYINCOMPOUND Z` again);
* and only attached to their beginning (`COMPOUNDBEGIN X`);
* ...and this suffix is allowed inside the compound (`COMPOUNDPERMITFLAG W`; suffixes without this flag can only appear at the end of the compound word).

All in all, it allows, for example, to deconstruct the word `"tavelförsäljare"` (seller of paintings): "tavel+försäljare": the first word has this suffix "el", base form is "tavla", "tavla" and "försäljare" are in the dictionary, the suffix is allowed, the word is correct.

### 12. Several flags can be combined into the numeric alias

If a group of flags is repeated many times (think English verbs: most of them can have suffixes `-s`, `-d`/`-ed`, and `-ing`), the format introduces flag group aliases. They are defined by `AF` directive, with a table like this:

```
AF 100
AF ABC
AF BC
AF DCE
# .... 97 other rows...
```
It means that the group of flags `ABC` now has alias `1` (because they are in the row 1), group `BC` has alias `2`, and so on. Words could now be marked with aliases instead of flags: `foo/3`.

### 13. Most of the entries in the `.dic` are real words. But some are not

An entry like `spell/JSMDRZG` typically declares the existing word "spell" plus flags describing how it can be used and combined. But one shouldn't rely on the assumption that taking all dictionary entries and ignoring the flags will give a list of base word forms of some language. The entry might have a flag declared by a directive like `NEEDSAFFIX` (this form is only valid as a base for attaching affixes) or `ONLYINCOMPOUND` (this form is valid only when joined with some other word).

### 14. ...sometimes, they explicitly specify non-existing words

The words in the dictionary marked with a flag specified by the `FORBIDDENWORD` directive are forbidden. This feature is used to specify exceptions: words that are logically possible (to produce as a compound, or from some base form + suffixes) but are actually incorrect.

### 15. Frequently, `.dic` entries are base forms of the word. But just as frequently, they are not

If we consider that the whole idea of "affixes" allows us to describe base form and its derivatives, it is tempting to use Hunspell's dictionaries to normalize words to their base forms. But not all dictionary entries are base forms, so the ability to normalize is not guaranteed.

The reason is that Hunspell only supports up to two prefixes and up to two suffixes, and specifying proper rules for them might be a tedious task. So, many dictionaries just include inflected forms instead of describing them with affix rules. For example, `en_US` dictionary doesn't specify `mis-` as a prefix and has entries like "misread" or "misremember". It also contains words like "knew", "brought", and even "potatoes".

### 16. Dictionary can encode grammatic information, but rarely does

In the Slovak dictionary, we can find:

```
kat/C po:noun is:masculine
```
Meaning: "Kat _(executioner)_: part of speech noun, masculine grammatical gender".

Oh, and affixes can have, too:

```
SFX C 0 a [^céior] is:genitive
```

Meaning that, for example, "kata" ("executioner" with suffix "a") is a word in a genitive case.

Only five dictionaries contain such information: Breton, Galician, Icelandic, Lithuanian, and Slovak. My guess is languages with fewer users have their dictionaries prepared by professional linguists, while more common languages dictionaries are authored by IT people.

### 17. Many dictionaries encode special cases with generic rules

While authors of Hunspell made a huge effort to introduce the _generic_ way of describing language rules, many dictionaries reuse them to handle very special cases, like singular misspelling or a rare combination of words.

For example (see #10), the Swedish dictionary uses the flag "this word, when compounded with another, requires it to be capitalized", for exactly 13 words: "afrika", "europa", "finland" and similar, to force words like "Sydafrika" ("South Afrika") to be capitalized. It has a side effect of suggesting any other compound words like "operaafrika" (_any_ possible beginning of compound, when combined with "-afrika") to be capitalized! This is probably not a big deal for the users of this dictionary.

Another example is a huge list of `CHECKCOMPOUNDPATTERN` in Dutch dictionary, having entries like "hart long" ("heart lung") to prohibit this particular sequence of words from being joined in the compound: it should be "hart-longmachine", not "hartlongmachine", though both words can be compounded with other words when they do not follow each other.

## With all this, let's attempt to lift off!

The next installment of the series will cover how all of those details are used in existing dictionaries, mapping the **landscape of the spellchecking dictionaries in 2021**.

Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](/subscribe.html) if you don't want to miss the follow-up!
