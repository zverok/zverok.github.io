Another case to handle is if the misspelled word already has dashes in it (and is incorrect): it could be misspelled word with a dash—or several words joined by dashes, with some of them misspelled (like, "red-<i>grean</i>-blue distinction..."). For this case, and if no other dash-containing suggestion were found on steps 1-13, one additional attempt to find a suggestion is performed. For each part ("red", "grean", "blue"), if it is misspelled ("grean"), try to find all possible suggestions for it in isolation, and glue everything back together:

```python
print(*english.suggest("red-grean-blue"))
# red-gran-blue
# red-green-blue
# red-great-blue
# red-groan-blue
# red-glean-blue
# red-Reagan-blue
```

But only one misspelled part is allowed: so you'll have nothing for "red-grean-blui".

====

### Multiword suggestions are cumbersome

You might notice that the list of steps includes "split this into two words" twice: at (3) and (13). The idea is that "split this word into two" should be proposed only after the edits, which leave it as one word. Otherwise, we might overflow user with split suggestions, especially if there is a lot of short words in the dictionary, like english "at": if the step would be earlier, "split it" would be the first suggestion for many misspellings[^5]:

[^5]: For some languages, the problem of "splitting in short words" is so bad that their dictionaries turn the feature off completely with `NOSPLITSUG` directive.

```python
print(*english.suggest('buckat'))
# bucket
# buck at   <== this should be the last one
```

But sometimes, there are fixed expressions that are frequently misspelled, and we want to suggest them first: Hunspell's docs go-to example is "alot"→"a lot". In this case, such a phrase might be included in the dic-file as a _singular entry_ and, when found by suggest, deemed a final answer (no further search is performed).

```python
swedish = Dictionary.from_files('sv_SE')
print(*swedish.suggester.suggest('adhoc'))
# Suggestion[spaceword](ad hoc) -- the only suggestion!
```

There is one more step which might produce multi-word suggestions considered highly probable: step (2), common misspellings check. Actually, _that's how_ "alot"->"a lot" is implemented in the modern English dictionary (note that "a lot" is NOT the only suggestion).
```python
print(*english.suggester.suggestions('alot'))
# Suggestion[replchars](a lot)
# Suggestion[swapchar](alto)
# Suggestion[badcharkey](slot)
# Suggestion[extrachar](lot)
# ...
```

> Two side notes for the implementation of a decent spellchecker. First, "this word is actually two words with a space accidentally missed" is a very common misspelling—and suggesting it is frequently overlooked (everybody focuses on the distance between one misspelling and one suggestion).

> Second, word-by-word spellchecker by definition cannot handle another frequent case: one word is accidentally split into two (or a space misplaced between two words, like "bi gcat"). It can be solved by a context-aware spellchecker... which is far out of our scope today.

====

### Compound suggestions are found only after the regular ones

Hunspell's suggest makes a distinction between non-compound and compound words in a peculiar way. All edits generation steps are repeated twice: first, of all possible edits, choose correct non-compound words; second, of all possible edits, choose correct compound words _(if there were no non-compound suggestions found on steps 1-4)_.

I initially assumed that this approach is just a performance optimization: compound lookup (as we explained in the [corresponding chapter](2021-01-14-spellchecker-3.html)) is expensive: you need to split the word into parts in all possible ways and check each part. So, we might hope that the non-compound stage will bring enough good suggestions, and compounds wouldn't be necessary at all.

With this assumption, I committed my biggest sin against the "do exactly as Hunspell does" goal: Spylls performed the full lookup for each edit. I reasoned that optimization is not the goal; the explanation of the algorithm is. My results actually seemed to be _better_ sometimes: for example, "11thhour" wouldn't be corrected by Hunspell to "11th hour": "11th" is a compound word, and "hour" is not, so on the first stage, we split it into two words and check if they are both correct non-compounds (no), and then if they are both correct compounds (no).

But as it turned out (and made me refactor a significant part!), you should never digress from how it is done in Hunspell. The numbers of allowed non-compound and compound suggestions are very different (15 vs. 3), and additionally, aff-file can change the latter number with `MAXCPDSUGS` directive. Many real dictionaries actually do that or set it to `0`  to completely turn off the compound suggestions. And even those that leave it enabled seem to expect compound suggestions to come after non-compound ones (though I am not familiar enough with any of those languages to give a good example).

_All that was said above means that in the worst case, the 13 steps could be repeated up to 8 times: 4 word cases[^6], non-compound and compound edits for each._

[^6]: For a word like "McDonalds" (capital first letter and capital letters in the middle), four hypotheses would be checked: "McDonalds", "mcDonalds", "mcdonalds", "Mcdonalds".
