---
layout: post
title:  "Rebuilding the spellchecker, pt.5: Hunspell and the order of edits"
date:   2021-01-22
categories: python spellchecker
comments: true
---

**_This is the fifth part of the "Rebuilding the spellchecker" series, dedicated to explaining how the world's most popular spellchecker Hunspell works._**

Today, we talk about **edit-based suggestions.**

**Quick recap**: **[The first part](2021-01-05-spellchecker-1.html)** described what Hunspell is; and why I decided to rewrite it in Python. In **[the second](2021-01-09-spellchecker-2.html)** and **[the third](2021-01-14-spellchecker-3.html)** parts, we've talked about **lookup**: checking the word's correctness.

**[The fourth part](https://zverok.github.io/blog/2021-01-21-spellchecker-4.html)** introduced **suggest**, an algorithm for search for corrections. We described that Hunspell performs suggest in two main stages:

1. Generate edits and test them for the correctness
2. (If the first stage didn't produce good enough results) Search through the entire dictionary and find similar words

So, talking about the **edit-based suggestions search**: its full behavior is a curious example of emergent complexity: every single step and every single condition is quite simple, but the resulting behavior is quite sophisticated—and hard to dissect.

Let's dive in this beautiful mess!

> Before we proceed, let's try something new: in this chapter, I'll show some spylls code to play with and illustrate some points. So, if you want to follow, begin with:

  ```
  pip install spylls
  ```

> ...and load spylls into your REPL of choice:

  ```python
  from spylls.hunspell import Dictionary

  english = Dictionary.from_files('en_US')
  ```

> To simplify experimenting, a few dictionaries used in examples are distridbuted with spylls. Other dictionaries to play with can be downloaded from [Firefox](https://addons.mozilla.org/en-US/firefox/language-tools/) or [LibreOffice](https://extensions.libreoffice.org/?Tags%5B%5D=50) extension sites. For most examples, we'll use Swedish dictionary (it uses more of Hunspell features than English one and requires more thorough tuning):

  ```python
  swedish = Dictionary.from_files('sv_SE')
  ```

> Once the dictionary is loaded, you can use suggest:

  ```python
  # this will return suggestion generator
  print(english.suggest('spylls'))
  # <generator object Dictionary.suggest at 0x7f62e972d950>

  # ...and this will unpack it into the suggestions
  print(*english.suggest('spylls'))
  # spells spills

  # For illustrative purposes, there is also an
  # easy-to-use "internal" method, which produces
  # Suggestion objects, demonstrating the way it was found.
  print(*english.suggester.suggestions('spylls'))
  # Suggestion[badchar](spells)
  # Suggestion[badchar](spills)
  ```

## The order of edits

As we already mentioned, the "classic" edits for string comparison algorithms are insert, delete, replace, and swap adjacent letters. To contrast, Hunspell's carefully crafted set of edits is as follows:

1. Upcase the word (see also "Word case" sub-section below);
2. Replace common misspellings, like "f"→"ph" and vice versa, defined by `REP` table from aff-file;
3. Split the word in two parts in every position (with space or dash), to be tested as a _single dictionary entry_ (see also #13 below);
4. Replace related chars, like "a", "å", "ä", defined by `MAP` table from aff-file;
5. Swap every two adjacent letters,
  * oh, and for 4- and 5-letter words also try _two_ swaps: "ahev" → "have";
6. Swap two non-adjacent letters (up to distance 4);
7. Replace every letter with the adjacent on the keyboard "miraclw" → "miracle", with keyboard layout defined by `KEY` directive in aff-file;
  * and, at the same step, with the capitalized version of the character ("paris" → "Paris", but not vice versa), also considered as a possible keyboard-related error;
8. Remove every letter;
9. Insert every letter from the language's alphabet (defined by `TRY` directive in aff-file) into every position;
10. Move every letter forward and backward into all possible positions ("kiettn" => ...moving letter "e": "kitetn, kitten, kittne; ekittn");
11. Replace every letter with every other letter from the language's alphabet;
12. Find a duplicated _pair_ of letters and remove it: "chicicken" → "chicken";
13. Split the word in two in every position, to be tested as two separate words (see also #3 above).

Phew!

As ~~complicating matters is kinda Hunspell's shtick~~ there are many edge cases to handle, producing edits in some order is just the beginning!

### Suggestion ranking and limiting

Hunspell's edit suggestion generation process always produces edits in the same order. Instead of generating (some or all) suggestions and then sort them by some metric, all suggestions are generated in the order the user will see them.

So, _all_ suggestions to swap two letters that produce correct words will be returned earlier than _all_ suggestions to remove or insert a letter[^1], without any attempt to measure which is closer to the misspelled word or which is a more common word. The number of edit-based suggestions is hard-limited to 15. Even if there's a possibility to produce more, Hunspell does not attempt to balance a number of suggestions from each group: it just stops searching for more after the first 15, no matter how they were produced.

[^1]: Actually not "all"! Word case and compounding make this more complicated. See below.

There is also a hard cut-off by certain categories of suggestions: if (3) produces a correct word, no further edit-based and no ngram-based suggestions are tried; if any of (1-2) produce correct words, the rest of edit-based suggestions are produced too, but no ngram suggestions are produced.

Finally, there is a hard cut-off by time: 0.25 seconds for a whole suggestion algorithm, 0.1 seconds for each approach to edit-based algorithm (in each possible word case): once the time limit is reached, the suggestion generation stops[^2].

[^2]: Time limit is not implemented in Spylls currently: Python is obviously slower than C, so the original limits wouldn't do. And as "limit by the time" doesn't change the algorithm's principles, I skipped this part.

> Contrast this rigid ordering/limiting to aspell's approach: it provides several [suggestion modes](http://aspell.net/0.61/man-html/Notes-on-the-Different-Suggestion-Modes.html), allowing to trade time over the breadth or vice versa; and also has a lot of [fine-tuning options](http://aspell.net/0.50-doc/man-html/4_Customizing.html) to adjust the behavior.

### Word case

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

But when the misspelling is not in lowercase (and the target spelling is), the wonders begin. The word is transformed to all cases it _could_ have been if it was spelled correctly; then, for each of the variants, steps 1-13 run, then the final suggestions are capitalized back into the form the misspelling was[^3]. For example, let's misspell a kitten.

[^3]: Oh, unless it has a flag "Only this case allowed".

```python
print(*english.suggester.suggestions("Kittn"))
# Suggestion[badchar](Kitty)     |
# Suggestion[twowords](Kit tn)   | Three suggestions found from "Kittn": step 11 and step 13
# Suggestion[twowords](Kit-tn)   |
#
# Suggestion[forgotchar](Kitten) | Suggestion found from "kittn" (step 9), and then capitalized
```

Note that, again, suggestions are _not_ reordered: the user receives them in the order they were found.

### Aff-file content affects quality dramatically

One might notice that the description of steps refers to some of the **aff-file defined data**. It means that some of what is perceived as a suggest _algorithm_ is actually represented by _data_ and delegated to the dictionary's author. Some examples are below.

**`REP` directive** defines a table of common misspellings (parts of words and whole words) and how they might be fixed. If such a fix makes the word correct, the suggestion is considered "good," and no ngram-based suggestions would be tried.

```python
print(*english.suggest('caushun'))
# caution

# We might check how it was produced:
print(*english.suggester.suggestions('caushun'))
# Suggestion[replchars](caution)
#           ^^^^^^^^^^^

print(en.aff.REP)
# ...prints 90 patterns, including:
# RepPattern(pattern='shun', replacement='tion')

# But if we try to use the dictionary without this table...
en.aff.REP = []
print(*en.suggester.suggestions('caushun'))
# [Suggestion[ngram](Caucasus)] -- umm, no?..
```

The problem is, defining what the common misspellings for some language are is not trivial, so many dictionary authors just ignore the task, or make a very small list, missing the opportunity to cover the most important mistakes.

> There is another way for Hunspell to populate this list: the dic-file might contain the data tag `ph:` with alternative spelling for the word: `which ph:wich`. It would mean that suggest should try to replace any occurrence of the substring "wich" with "which" and see whether it produces a good word. Of all LibreOffice and Firefox dictionaries I've checked, this feature is used in only two dictionaries: Hungarian and Slovak (with the latter using it for only one word).

**`MAP` directive specifies related characters.** For languages using diacritics, if the `MAP` table is not provided or not complete, "obvious" misspellings would be missing or provided further the list (found by other steps):

```python
print(*swedish.suggest('bokstaver'))
# bokstavera
# bokstaven
# bokstäver   <== This one is "obviously" right
# ...

print(swedish.aff.MAP)
# [['ﬁ', 'fi'], ['ﬂ', 'fl']]

# But if we specify that 'a' and 'ä' are related:
swedish.aff.MAP.append(['a', 'ä'])

print(*swedish.suggest('bokstaver'))
# bokstäver   <== Voila!
# bokstavera
# bokstaven
# ...
```

**`KEY` directive specifies the keyboard layout.** If the `KEY` directive is not provided, Hunspell uses the default one:

```python
print(*swedish.suggest('lppen'))
# lappen
# loppen
# luppen
# läppen
# appen
# lipen
# löpen
# öppen   <== "ö" is near the "l" on Swedish keyboard, so this is the "obvious"
# ...

# But if we tell it the right keyboard layout...
swedish.aff.KEY = 'qwertyuiopå|asdfghjklöä|zxcvbnm'

print(*swedish.suggest('lppen'))
# öppen    <== Tada!
# lappen
# loppen
# ...
```

**`TRY` directive specifies the target language alphabet**. If it is absent, Hunspell will skip _all_ steps like "try replacing wrong letter with every other letter" or "try inserting missing letter":

```ruby
russian = Dictionary.from_files('ru_RU')
# "кошка" is "female cat", "коша" is informal name/misspelling for it:
print("кошка" in russian.suggest('коша'))
# False

russian.aff.TRY = 'абвгдеёжзийклмнопрстуфхцчшщъыьэюя'
print("кошка" in russian.suggest('коша'))
# True
```

> **Note:** In fact, our "fix" by providing a sorted alphabet is too naive. Hunspell's documentation advises sorting letters in `TRY` in the order of frequency (they'll be actually _tried_ in the order they are specified).

There is one more way the absence/wrong formatting of `TRY` directive might affect suggest, as we'll see in the following section.

### Multiword suggestions are cumbersome

You might notice that the list of steps includes "split this into two words" twice: at (3) and (13). The idea is that "split this word into two" should be proposed only after the edits, which leave it as one word... But sometimes, there are fixed expressions that are frequently misspelled, and we want to suggest them first (Hunspell's docs usual example is "alot"->"a lot"). In this case, such a phrase might be included in the dic-file as a singular entry and, when found by suggest, deemed a final answer.

```python
print(*swedish.suggester.suggest('adhoc'))
# Suggestion[spaceword](ad hoc) -- the only and ultimate suggestion!

# BTW, split into several word suggestions are totally prohibited in Swedish dictionary:
print(swedish.aff.NOSPLITSUGS)
# True
# "bra katt" is "good cat", but it will not be suggested...
print(*swedish.suggest('brakatt'))
# brakat bakratt brakteat braskat braka Bratt
```

Multi-word suggestion might also come out of `REP` table of frequent misspellings (step 4), and _that's how_ "alot"->"a lot" is actually implemented in the modern English dictionary:
```python
print(*english.suggester.suggestions('alot'))
# Suggestion[replchars](a lot)
# Suggestion[badcharkey](slot)
# Suggestion[extrachar](lot)
# ...
```

> Two side notes for the implementation of a decent spellchecker. First, "this word is actually two words with a space accidentally missed" is one of the most widespread and useful suggestions—and frequently overlooked one (everybody focuses on the distance between one misspelling and one suggestion).

> Second, word-by-word spellchecker by definition cannot handle another frequent case: one word is accidentally split into two (or a space misplaced between two words, like "bi gcat"). It can be solved by a context-aware spellchecker... which is far out of our scope today.

Also, there are dashes. If "foobar" appears to be two valid words, should we suggest only "foo bar", or also "foo-bar" (as in "coca-cola" or "Microsoft-Apple lawsuit")? Hunspell's guess is both:
```python
print(*english.suggest('MicrosoftApple'))
# Microsoft Apple
# Microsoft-Apple
# ...
```
...but actually it depends on the dictionary's alphabet (specified by `TRY` directive): the dash version is suggested if the alphabet is Latinic (more precisely: if it contains Latin "a"), or if it explicitly includes a dash!

```ruby
# "гуси-лебеди" — literally, "geese-swans" (like, geese AND swans, not some mutants)
# is a frequently mentioned group of birds in Russian folklore
print(*russian.suggest('гусилебеди'))
# гуси лебеди  <== without dash in TRY, it is the only multi-word suggestion
# лебединый

# Let's fix it!
russian.aff.TRY = '-абвгдеёжзийклмнопрстуфхцчшщъьэюя'

print(*russian.suggest('гусилебеди'))
# гуси лебеди
# гуси-лебеди  <== Now we are talking!
# лебединый
```

### Compound suggestions

Hunspell's suggest makes a distinction between non-compound and compound words in a peculiar way. All edits generation steps are repeated twice: first, of all possible edits, choose correct non-compound words; second, of all possible edits, choose correct compound words _(if there were no non-compound suggestions found on steps 1-4)_.

I initially assumed that this approach is just a performance optimization: compound lookup (as we explained in the [corresponding chapter](2021-01-14-spellchecker-3.html)) is expensive: you need to split the word into parts in all possible ways and check each part. So, we might hope that the non-compound stage will bring enough good suggestions, and compounds wouldn't be necessary at all.

With this assumption, I committed my biggest sin against the "do exactly as Hunspell does" goal: Spylls performed the full lookup for each edit. I reasoned that optimization is not the goal; explanation of the algorithm is. My results actually seemed to be _better_ sometimes: for example, "11thhour" wouldn't be corrected by Hunspell to "11th hour": "11th" is a compound word, and "hour" is not, so on the first stage, we split it into two words and check if they are both correct non-compounds (no), and then if they are both correct compounds (no).

But as it turned out (and made me refactor a significant part!), you should not digress from how it is done in Hunspell. A number of allowed non-compound and compound suggestions is very different (15 vs. 3), and additionally, aff-file can change the latter number with `MAXCPDSUGS` directive. Many real dictionaries actually do that or set it to `0`  to completely turn off the compound suggestions, including today's test subject, Swedish. And even those that leave it enabled seem to expect compound suggestions to come after non-compound ones (though I am not familiar enough with any of those languages to give a good example).

_All that was said above means that in the worst case, the 13 steps could be repeated up to 8 times: 4 word cases[^4], non-compound and compound edits for each._

[^4]: For a word like "McDonalds" (capital first letter and capital letters in the middle), four hypotheses would be checked: "McDonalds", "mcDonalds", "mcdonalds", "Mcdonalds".

Too often, though, this enumeration of guesses would be in vain: none of them would produce a good suggestion, and in order to find the answer, we would need to do a **full dictionary scan, with an ngram-based similarity check:**

```python
print(*english.suggester.suggestions("londun"))
# [Suggestion[ngram](London)]
#             ^^^^^
```

**Ngram-base suggestion would be the theme of the next chapter.** Follow me [on Twitter](https://twitter.com/zverok) or [subscribe to my mailing list](/subscribe.html) if you don't want to miss the follow-up!

PS: Let me know your opinion on code examples: are they making the reading easier and more fun, or hard and tiresome? Should we have them in the next series?
