---
layout: post
title:  "Rebuilding the spellchecker: Hunspell and the order of edits"
date:   2021-01-28
categories: python spellchecker
comments: true
---

**_This is the fifth part of the ["Rebuilding the spellchecker"](/spellchecker.html) series, dedicated to explaining how the world's most popular spellchecker Hunspell works._**

Today, we talk about **edit-based suggestions.**

**Quick recap**: **[The first part](2021-01-05-spellchecker-1.html)** described what Hunspell is; and why I decided to [rewrite it in Python](https://github.com/zverok/spylls). In **[the second](2021-01-09-spellchecker-2.html)** and **[the third](2021-01-14-spellchecker-3.html)** parts, we've talked about **lookup**: checking the word's correctness.

**[The fourth part](https://zverok.space/blog/2021-01-21-spellchecker-4.html)** introduced **suggest**, an algorithm for search for corrections. We described that Hunspell performs suggest in two main stages:

1. Generate edits and test them for the correctness
2. (If the first stage didn't produce good enough results) Search through the entire dictionary and find similar words

So, speaking about the **edit-based suggestions search**: its full behavior is a curious example of emergent complexity. Every single step and every single condition is quite simple, but the resulting behavior is quite sophisticated—and hard to dissect.

Let's dive into this beautiful mess!

> Before we proceed, let's try something new: in this chapter, I'll show some spylls code to play with and illustrate some points. So, if you want to follow, begin with:

  ```
  pip install spylls
  ```

> ...and load spylls into your REPL of choice:

  ```python
  from spylls.hunspell import Dictionary

  english = Dictionary.from_files('en_US')
  ```

> To simplify experimenting, a few dictionaries used in examples are distributed with spylls. Other dictionaries to play with can be downloaded from [Firefox](https://addons.mozilla.org/en-US/firefox/language-tools/) or [LibreOffice](https://extensions.libreoffice.org/?Tags%5B%5D=50) extension sites.

> Once the dictionary is loaded, you can use suggest:

  ```python
  # this will return suggestion generator
  print(english.suggest('spylls'))
  # <generator object Dictionary.suggest at 0x7f62e972d950>

  # ...and this will unpack it into suggestions
  print(*english.suggest('spylls'))
  # spells spills

  # For illustrative purposes, there is also an
  # easy-to-use internal method, that produces
  # Suggestion objects with an indicator of the
  # method it was found with.
  print(*english.suggester.suggestions('spylls'))
  # Suggestion[badchar](spells)
  # Suggestion[badchar](spills)
  ```

## The order of edits

As we already mentioned, the "classic" edits for string comparison algorithms are insert, delete, replace, and swap adjacent letters. To contrast, Hunspell's carefully crafted set of edits is as follows:

1. Change the word to the uppercase (see also "Word case" sub-section below);
2. Replace common misspellings, like "f"→"ph" and vice versa, defined by `REP` table from aff-file;
3. Split the word in two parts in every position (with space or dash), to be tested as a _single dictionary entry_, like "ad hoc" (see also #13 below);
4. Replace related chars, like "a", "å", "ä", defined by `MAP` table from aff-file;
5. Swap every two adjacent letters,
  * oh, and for 4- and 5-letter words also try _two_ swaps: "ahev" → "have";
6. Swap two non-adjacent letters (up to distance 4);
7. Replace every letter with the adjacent on the keyboard, e.g. "miraclw" → "miracle". The keyboard layout is defined by `KEY` directive in aff-file;
  * and, on the same step, with the capitalized version of the character ("paris" → "Paris", but not vice versa), also considered as a possible keyboard-related error;
8. Remove every letter in turn;
9. Insert every letter from the language's alphabet (defined by `TRY` directive in aff-file) into every position;
10. Move every letter forward and backward into all possible positions;
11. Replace every letter with every other letter from the language's alphabet;
12. Find a duplicated _pair_ of letters and remove it: "chicicken" → "chicken";
13. Split the word in two in every position, to be tested as two separate words (see also #3 above).

Phew!

As ~~complicating matters is kinda Hunspell's shtick~~ there are many edge cases to handle, producing edits in some order is just the beginning!

### Suggestion ranking and limiting is hard-coded

Classical spellchecker algorithm papers typically describe the process of ranking of suggestion after they are generated: by edit's probability, by similarity to the original word, or by word's popularity. In Hunspell, the ranking is _implicit_: the order of steps is already considered the best possible order.

So, all suggestions to remove a letter that produce correct words (step 8) will be returned before all suggestions to insert a letter (step 9)[^1]:

```python
print(*english.suggester.suggestions('haz'))
# Suggestion[extrachar](ha)       # all "extra character removed" suggestions
# Suggestion[forgotchar](haze)    # then, all "forgotten character inserted"
# Suggestion[forgotchar](hazy)
# Suggestion[badchar](has)        # then, all "bad character replaced"
# Suggestion[badchar](hat)
# Suggestion[badchar](had)
# <...skip, there are lots...>
# Suggestion[twowords](ha z)      # finally, all "should be split in two words"
# Suggestion[twowords](ha-z)
```

[^1]: Actually not "all"! Word case and compounding make steps sequence repeat several times. See below.

The number of edit-based suggestions is hard-limited to 15. Even if there's a possibility to produce more, Hunspell does not attempt to balance the number of suggestions from each group: it just stops searching for more after the first 15, no matter how they were produced.

There is also a hard cut-off by certain categories of suggestions: if (3) produces a correct word, no further edit-based and no ngram-based suggestions are tried; if any of (1-2) produce correct words, the rest of edit-based suggestions are produced too, but no ngram suggestions are produced.

Finally, there is a hard cut-off by time: 0.25 seconds for a whole suggestion algorithm, 0.1 seconds for each approach to edit-based algorithm (in each possible word case): once the time limit is reached, the suggestion generation stops. (Spylls doesn't try to reimplement this, as Python is obviously slower than C.)

> Contrast this rigid ordering/limiting to aspell's approach: it provides several [suggestion modes](http://aspell.net/0.61/man-html/Notes-on-the-Different-Suggestion-Modes.html), allowing to trade time for breadth or vice versa; and also has a lot of [fine-tuning options](http://aspell.net/0.50-doc/man-html/4_Customizing.html) to adjust this behavior.

### Non-lowercase words increase the number of searches

What if the misspelling is not in the same case as the desired correction is in the dictionary? The answer (as one who already has some gist of this series might imagine) is complicated.

Lowercase word that should be capitalized will be fixed relatively easy, on step (7):

```python
print(*english.suggester.suggestions("london"))
# [Suggestion[badcharkey](London)]
```

The word that should've been uppercased will be fixed even easier, on step (1):

```python
print(*english.suggester.suggestions("nasa"))
# Suggestion[uppercase](NASA)
# Suggestion[forgotchar](nasal)
```

But when the misspelling is not in the lowercase (and the target spelling is), the wonders begin. The word is transformed to all cases it _could_ have been if it was spelled correctly; then, for each of the variants, steps 1-13 run, then the final suggestions are capitalized back into the original form of the misspelling[^2]. For example, let's misspell a kitten.

[^2]: Oh, unless it has a flag "Only this case allowed".

```python
print(*english.suggester.suggestions("Kittn"))
# Suggestion[badchar](Kitty)     |
# Suggestion[twowords](Kit tn)   | Three edit suggestions from "Kittn"
# Suggestion[twowords](Kit-tn)   |
#
# Suggestion[forgotchar](Kitten) | Edit suggestion from "kittn",
#                                | capitalized after it was found
```

Note that, again, suggestions are _not_ reordered: the user receives them in the order they were found.

### Aff-file contents affects quality dramatically

One might notice that descriptions of some steps refer to the **data defined in aff-file**. It means that some of what is perceived as a suggest _algorithm_ is actually represented by _data_ and is delegated to the dictionary's maintainer.

Some of the directives are hard to populate—for example, `REP`, the list of commonly misspelled letter groups ("f/ph", or "tion/shun"). Many dictionaries omit it or make a very small list, missing the opportunity to cover the most important mistakes. Other directives (like `KEY` or `TRY`) are quite trivial and yet are not included in some dictionaries, leading to a sharp drop in suggestion quality.

We'll discuss dictionaries and their quality in one of the future chapters. For now, just one telling example.

To perform steps like "insert a missing letter" or "replace a letter with another," Hunspell needs to know _the language's alphabet_. Being truly multi-lingual, it can't assume regular Latinic `a-z` as a default. So, there is a **`TRY` directive**, which should specify this alphabet. If it is absent,  several of the important steps would be missing[^3].

```ruby
russian = Dictionary.from_files('ru_RU')
# "кошка" is "female cat", "коша" is informal name/misspelling for it:
print("кошка" in russian.suggest('коша')) # "к" is missing
# False
print("кошка" in russian.suggest('тошка')) # "к" is replaced with "т"
# False

# Let's fix it!
russian.aff.TRY = 'абвгдеёжзийклмнопрстуфхцчшщъыьэюя'

print("кошка" in russian.suggest('коша'))
# True
print("кошка" in russian.suggest('тошка'))
# True
```

[^3]: The following example uses one of the two Russian dictionaries in the [Firefox addons](https://addons.mozilla.org/en-US/firefox/addon/russian-spellchecking-dic-3703/) repository, which is included in Spylls distribution; [another one](https://addons.mozilla.org/en-US/firefox/addon/russian-hunspell-dictionary/) defines the `TRY` directive. We included one without `TRY` to demonstrate how it affects the spelling. Note, though, that it is not an artificial example: dictionaries are [listed](https://addons.mozilla.org/en-US/firefox/language-tools/) beside each other, and for an innocent user, it is hard to guess which is better.

> **Note:** In fact, using the alphabetic order of letters to fix the issue is too naive. Hunspell's documentation advises sorting letters in `TRY` in the order of frequency (they'll be actually _tried_ in the order they are specified). Also, we forgot to include `-` (dash) into the alphabet—which will lead to omitting of all "(good word)-(another good word)" suggestions.

### Multiword suggestions are cumbersome

One of the most common spelling mistakes is **accidentally joining two words**: so, every good spellchecker should try to cover this case. Unfortunately, it is frequently overlooked in spellchecking libraries and algorithm explanations (everybody focuses on the distance between one misspelling and one suggestion)[^4].

[^4]: Another frequent misspelling: one word is accidentally split into two (or a space misplaced between two words, like "bi gcat"). Word-by-word spellchecker by definition cannot handle this case—it can be solved by a context-aware spellchecker... which is far out of our scope today.

Usually, we want split suggestions at the end of the suggestion list (thus, the last step). Otherwise, short words will overwhelm the user with possibilities:

```python
[*en.suggester.suggestions('Itall')]
# Suggestion[extrachar](Tall)
# Suggestion[extrachar](Ital)
# Suggestion[badchar](Stall)
# Suggestion[badchar](Italy)
# Suggestion[twowords](I tall)
# Suggestion[twowords](It all)
# ...
```

Sometimes, though, we might want to ensure that some frequently misspelled fixed expressions will have a priority on suggestion. Hunspell has two ways to do so: either include the whole expression in the dic-file as a single entry (step 3, then it would be the only one):

```python
swedish = Dictionary.from_files('sv_SE')
print(*swedish.suggester.suggest('adhoc'))
# Suggestion[spaceword](ad hoc) -- the only suggestion!
```

...or, include the frequent misspelling in the `REP` table (step 2, note that in this case "a lot" is NOT the only suggestion)

```python
print(*english.suggester.suggestions('alot'))
# Suggestion[replchars](a lot)
# Suggestion[swapchar](alto)
# Suggestion[badcharkey](slot)
# Suggestion[extrachar](lot)
# ...
```

## So many edits, wow!

All in all, the whole edit suggestion generation process produces typically **hundreds, if not thousands, of word forms to check**: 13 steps (and some of them, like "insert every letter from the alphabet in every position," give a lot of options), repeated several times for different word cases; and then the whole process is repeated twice to generate non-compound and compound suggestions separately[^5]. Each candidate is checked with the lookup algorithm, which, as we have already seen, is non-trivial by itself.

> You can follow all the quirks and dark corners through docs and code of the [Suggest](https://spylls.readthedocs.io/en/latest/hunspell/algo_suggest.html#spylls.hunspell.algo.suggest.Suggest.__call__)!

[^5]: Which is an important implementation detail but only marginally interesting to explain.

**Too often, though, this enumeration of guesses would be in vain: none of them would produce a good suggestion, and in order to find the answer, we would need to do a full dictionary scan, with an ngram-based similarity check:**

```python
print(*english.suggester.suggestions("londun"))
# Suggestion[ngram](London)
#            ^^^^^
```

**Ngram-base suggestion would be the [theme of the next chapter](2021-02-10-spellchecker-6.html).** Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](https://zverok.substack.com/) if you don't want to miss the follow-up!

PS: Let me know your opinion on code examples: are they making the reading easier and more fun, or hard and tiresome? Should we have them in the next series?

> PPS: As usual, this post would be four times more cumbersome (and would have a much more boring title) without the precious help of [@squadette](https://twitter.com/squadette), my faithful editor.
