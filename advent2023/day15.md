---
layout: advent
title: "Advent of changelog: Day 15"
next: day16
next_title: "Day 16"
prev: day14
prev_title: "Day 14"
---

Today is Friday, and I am exhausted from the day's work and everything else going around (today I woke up to the information that my homecity Kharkiv, where my family lives, was bombed overnight... But it is not even _news_, it is just _information_).

So, I'll just make a plan for the future changelog-related work that needs to be handled the next ten days (Ruby release is Christmas morning), in a rough order in which it should be done:

* Try to follow up about a change I [proposed last January](https://bugs.ruby-lang.org/issues/19324) (as a follow-up of 3.2 changelog) and that was agreed _in principle_, but then there is still no implementation for it;
* Try to provide a patch for `Pathname#chdir`: the idea that grew out of the new `Dir#chdir` method and is probably not too late to follow upon (worst case, it would be in the next version of the gem);
* Structure the changelog by topic: interestingly, for me it is a more productive way to move after the first draft of the texts themselves (instead of just continue polishing texts and leaving the structuring to the very end);
* Add the missing standard library-related section (again, in a drafty state);
* Go through previous versions and see which parts of this year's changes should be linked as "follow-ups" to it (this sometimes uncovers forgotten changes, or a better way to discuss something);
* See if something missing in the `NEWS.md` file (there are a few small things I've already noticed) and submit a PR to fix it;
* General polishing and finalization of the texts and examples—usually after structuring everything and giving myself a feeling of "almost done" changelog it is much easier, both psychologically and logically (it is easier to observe a common tune of changes and explanations and fix those that are dissonate);
* Update `evolution.md`'s [general list of changes](https://rubyreferences.github.io/rubychanges/evolution.html) in the language with notable ones from this version;
* (Not very thorough) check through the Ruby repo and the standard library about some changes that might've happened this year but went under the radar;
* Gather all the leftover TODOs (proposals of new features and old documentation improvements) into some sensible schedule to follow after the release;
* If, after all that, I'll still have my allotted "advent" hours (which I'll doubt), I'll try to improve a few things in the changelog site itself, I have a separate long-standing list of TODOs for that (which, actually, would've be better to have as a list of GitHub issues and rely on the power of the open source community... So maybe I should do that).

So... Yeah. I assume some 20-30+ more hours of work.

> Random fun fact: when in 2018 I invented the format, my first estimation was that the whole thing should be taking _one or two evenings_. And honestly, till I started to track the work with this diary, it was my assumption that it takes, like, maybe 2 working days of pure working time, and the fact that I was so late with the changelog last two years is just my laziness. (And note that previous two versions had much more significant changes that required detailed explanations than 3.3 has!)
