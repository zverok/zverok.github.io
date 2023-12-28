---
layout: advent
title: "Advent of changelog: Day 19"
next: day20
next_title: "Day 20"
prev: day18
prev_title: "Day 18"
---

Today, I am just continuing the regular paint-the-fence work of enhancing the formatting, structure, and examples throughout the file.

It turned out not that much left to handle, in principle, though there are still questions, uncertainties, and challenges, like:

* It is hard to add a _persuasive_ example for `String#bytesplice` with the new protocol (copy part of one string into the other), because this specific change is most useful for implementing low-level stuff like code editors and network protocols; so I just add a _demonstrative_ ones, with simple strings (with Ukrainian text, just to demonstrate Unicode, you know).
* I add base example/demo for `Fiber#kill`, but plan to do more, demonstrating its edge cases and limitations; I always feel kinda weird documenting fibers: this is a piece of well-designed machinery, easy to understand and explain and able to achieve complex things— which I happen to never use in my code (which is mostly application-level/business-level, usually, while fibers are a powerful _underlying_ abstraction for enumerators and async code).
* I try to show some code examples for `Process::Status` deprecated methods... and decide against it, eventually: it seems that concerned parties (rare, by my estimation) wold anyway understand better what the deprecation means and how to check it;
* I add a bit more details about Prism parser and notice that its docs are rendered [quite badly](https://docs.ruby-lang.org/en/master/Prism.html) by RDoc on the Ruby Core docs site. (On [its own site](https://ruby.github.io/prism/rb/Prism.html) at least method docs are informative, while module docs are stil taken from some auto-generated files.) It is frequently problematic with big and complicated libraries, especially when they are integrated into the main Ruby's RDoc. I put into my "long TODO of Ruby docs" to look into it some time in the upcoming months.
* I start to link the standard library names to the corresponding repos... And think that some time in the future I'd better automate that. Probably there should be a machine-readable data about current versions, repo links and default/bundled/non-gem status of each. And, once I think about it, I understand that Jan Lelis' awesome [stdgems.org](https://stdgems.org/) probably should have a machine-readable version... And indeed, [it does](https://stdgems.org/stdgems.json)! So I probably should somehow incorporate that into Ruby Changes build process (though not this week, probably!)
* Which also makes me think about the fact it would be cool to have a machine-readable changelog for each and every Ruby library (and, for that matter, for Ruby itself).
* Which reminds me of a long-lasting [discussion/plan](https://github.com/rubyreferences/rubychanges/issues/46) about making Ruby Changes itself into some formal DB/YAML, which would probably open some interesting possibilities. Though, I am not sure how/when this avenue of thought will be followed.

Anyway, I finally fill "Highlights" section (again, confirming that this version's language/API changes are smaller than some previous versions), and with that— it is _mostly_ ready? **Commit: [7f65d0f](https://github.com/rubyreferences/rubychanges/commit/7f65d0f)**

There are still a few `TODO`s here and there, and I should think about some better description at least for `Process.warmup`, but overall—

Let's say even if for some unexpected reasons I wouldn't be able to work on it anymore, it can be published already, with a couple of small fixes.

So if I _will_ have time to continue this advent, I plan to spend a next few days on general housekeeping: both for the changelog's site improvements and on Ruby's docs/NEWS.
