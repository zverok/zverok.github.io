---
layout: page
title: Rebuilding the spellchecker
permalink: /spellchecker.html
---

**This is the table of contents for the (finished in May 2021!) "Rebuilding the spellchecker" series, dedicated to explaining how the world's most popular spellchecker Hunspell works, via its Python port called [Spylls](https://github.com/zverok/spylls).**

* [Introduction](/blog/2021-01-05-spellchecker-1.html)
* ["Just look in the dictionary, they said!"](/blog/2021-01-09-spellchecker-2.html): Dictionary lookup, pt.1
* ["Compounds and solutions"](/blog/2021-01-14-spellchecker-3.html): Dictionary lookup, pt.2 (word compounding)
* [Introduction to suggest algorithm](/blog/2021-01-21-spellchecker-4.html)
* ["Hunspell and the order of edits"](/blog/2021-01-28-spellchecker-5.html): Edit-based suggest
* ["Well, akchualy..."](/blog/2021-02-10-spellchecker-6.html): Search for similar words suggest
* ["17 (ever so slightly) weird facts about most popular dictionary format"](/blog/2021-03-16-spellchecking-dictionaries.html)
* ["I forgot how to spellcheck"](/blog/2021-05-06-how-to-spellcheck.html): Summary of it all

---

This work is referenced by:
* [espells](https://github.com/Monkatraz/espells) is JS/TS port of Spylls;
* [spellbook](https://github.com/helix-editor/spellbook) is Hunspell-compatible Rust spellchecker, that lists the articles above as a resource for the early prototypes;
* [LESPELL – A Multi-Lingual Benchmark Corpus of Spelling Errors
to Develop Spellchecking Methods for Learner Language](https://aclanthology.org/2022.lrec-1.73.pdf) (Proceedings of the 13th Conference on Language Resources and Evaluation (LREC 2022), pages 697–706)
* [An exploratory investigation of functional variation in South Asian online Englishes](https://www.cambridge.org/core/journals/english-language-and-linguistics/article/an-exploratory-investigation-of-functional-variation-in-south-asian-online-englishes/9E6D173745FE63594CD4328BD71B34AD) (Cambridge University Press, English Language & Linguistics, Volume 28 Issue 2)
