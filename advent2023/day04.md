---
layout: advent
title: "Advent of changelog: Day 4"
next: day05
next_title: "Day 5"
prev: day03
prev_title: "Day 3"
---

So, my next direction would be moving towards better understanding of the more complicated features and their use cases from my TODO (`ObjectSpace::WeakKeyMap`, `Module#set_temporary_name`, etc.), I want to have a PR (or several) with docs enhancements soon.

But before that, I am rechecking `NEWS.md`: whether it was changed through last several days, or whether I have missed something on the first go. No new changes yet, phew!

But I notice that I did, in fact, miss a thing: new Warnings category (mentioned under "Command line options", that's why I didn't notice it when moving everything to my structure—even though noticed and mentioned on "Day 1").

So, I add it to `3.3.md` and check the docs. A new category mentioned there, but I make a TODO item to check the overall list of categories and examples of related behavior: it seems outdated (no part of pattern-matching is experimental anymore):

![](/img/advent2023/image09.png)

Also, it seems like generic module docs are outdated in general and have some rendering glitches. Also, maybe they deserve mentioning about how to see the warnings (new "performance" warnings category would only be seen with `-W:performance` command line flag).

![](/img/advent2023/image10.png)

OK, now to particular "interesting" feature discussions! Never looking for an easy path, I started with `Module#set_temporary_name`. I remembered the discussion was long, and I was at some point lost in it, so I need to refresh my understanding of the reasons.

The discussion spans several tickets actually, [#19450](https://bugs.ruby-lang.org/issues/19450) → [#19520](https://bugs.ruby-lang.org/issues/19520) → [#19521](https://bugs.ruby-lang.org/issues/19521).

After a lot of reading (the discussion was really lively!), and some experimenting, I think I understand the case for the new feature and how to describe it in changelog and docs to expose this case.

It took a lot of time though, so I am finishing the day with a lot of "TODOs" and a small commit to the changelog (with a draft of the feature initial explanation): **[7ae4182](https://github.com/rubyreferences/rubychanges/commit/7ae4182)**.
