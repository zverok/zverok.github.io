---
layout: advent
title: "Advent of changelog: Day 23"
next: day24
next_title: "Day 24"
prev: day22
prev_title: "Day 22"
---

So, not much time left to finalize everything. And I mostly dedicate myself to fill the missing final parts: like adding a link to the new version to the main page and site's History, writing this year's intro (keeping to nag people about this small nasty genocidal war waged by Russia that still somehow bothers me for some reason!), improving consistency of the similar headers in different years, and so on.

While doing so, I from time to time still check whether NEWS.md file have received significant updates, and what I see?..

![](/img/advent2023/image22.png)

Well... The problem here is that `Set` is a library with a somewhat uniquely weird state: it is both a "default gem" (one that is developed in a separate repo, but distributed with Ruby), but also, [since Ruby 3.2](https://rubyreferences.github.io/rubychanges/3.2.html#set-became-a-built-in-class), it is a "builtin" collection, or, rather, pretends to be one. Unlike `Array` or `Hash`, it is not some constant defined by the core language, and rather loaded with `autoload` mechanism:

```ruby
p Object.const_source_location('Set')
#=> ["<internal:prelude>", 24] -- that's where the autoload statement is defined
Set # mention the constant, invoking the autloading
p Object.const_source_location('Set')
#=> ["/home/zverok/.rbenv/versions/3.2.2/lib/ruby/3.2.0/set.rb", 222] -- the real location
```

In the context of the changelog, though, it means that `Set` _looks_ like a core collection, but doesn't "deserve" mention in `NEWS.md`, because gems have separate maintainers and [changelogs](https://github.com/ruby/set/blob/master/CHANGELOG.md) (though, it is interesting to notice that it doesn't mention the 1.1.0 version which Ruby 3.3.0 apparently would use?..).

Anyway, the bottom line is while I don't maintain `NEWS.md`, I wouldn't dare to mention `Set`s API improvements there, but I can and do include it into my annotated changelog. 🤷

With this new entry added, and the housekeeping stuff been taken care of, I reluctantly switch back to unfinished business with better docs/descriptions for `Fiber.kill` and `Process.warmup`.

Whereupon I am stuck with [something that looks like a bug](https://bugs.ruby-lang.org/issues/20082) (or my misunderstanding of the intended usage). Anyway, it is close to midnight, and I decide to wrap it up.

**Commit [0a694a9](https://github.com/rubyreferences/rubychanges/commit/0a694a9)**
