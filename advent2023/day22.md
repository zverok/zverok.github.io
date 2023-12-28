---
layout: advent
title: "Advent of changelog: Day 22"
next: day23
next_title: "Day 23"
prev: day21
prev_title: "Day 21"
---

So, today is the day for updating the generic [Evolution](https://rubyreferences.github.io/rubychanges/evolution.html) page of the changelog.

To prepare a reliable list of all changes that needs to go there, I have an old script so dirty and naive that it haven't even commited to repo, just had lived in the (`.gitignore`d) `tmp/` folder. But I might commit it as well, alongside with a sacramental comment:
```ruby
# Used to prepare sources for evolution.md
# Obvously should be ported to Rake task and prettified but :shrug:
```

The logic of the script is relatively simple: Just go through all `_src/<version>.md` files, and collect every change (= section) converted into a one-liner of this structure:

```markdown
* [3.3](3.3.md#dirfor_fd-and-dirfchdir) Core classes and modules: Filesystem and IO: `Dir.for_fd` and `Dir.fchdir` ([Dir.for_fd](https://docs.ruby-lang.org/en/master/Dir.html#method-c-for_fd), [Dir.fchdir](https://docs.ruby-lang.org/en/master/Dir.html#method-c-fchdir))
```
I.e., a link to the source version's change + a description, made of all the nested sections, just in case. The description would be edited; the main thing is the link stays, as an unique "id" of the line with a change; so that the next time I run the script, it will only output only those links _not_ already in `evolution.md`. (So I can rerun it even on old versions, if I cover or update them.)

As not all of the changes from all the files deserve mentioning, I leave all the not-that-significant ones [there, just hidden](https://github.com/rubyreferences/rubychanges/blob/master/_src/evolution.md?plain=1#L67-L71) by `<!-- -->` HTML comments. This way, `evolution.md` serves as both human-readable _and_ machine readable index.

I think it is pretty witty invention! _Though in general I have a suspicion for human-and-machine-readable Markdown based formats. If it wouldn't be for my own hobby project (that should be convenient for editing just by a few contributors) I would probably choose a more formal/robust format._

But after this initial automation, the rest is still manual. Automation is necessary to not miss something accidentally, but the structuring/phrasing is for the human to decide. Thankfully, "Evolution"'s format is deliberately succinct, so in most cases all the "description" is just a new class/method name + link to the docs and changelog entry. Sometimes I add a phrase to explain, or, for really interesting changes, even a short code example.

So the only slightly non-trivial task is what to consider unimportant enough to not go to the "Evolution" at all. When in doubt (like with `IO.read('| command')` deprecation, or new `bytesplice` arguments), I ask myself whether it would be possible for some future library, app, or tutorial author to ask a question "since which version is it so?.."— and most of the time I leave it there. The rare exceptions are the changes of the behavior that are either very obscure, or (and that's the most frequent case) the changes towards the "how it actually should be, intuitively," so it doesn't make sense to mention the change without a long explanation "why it wasn't this way from the beginning" (like `Array#pack` raising on unknown arguments).

This is very hand-wavy, vague, and vulnerable to counter-arguments (what if somebody _really_ wants to know when the `Array#pack` started to raise?), but well, the whole "Ruby Changes" project is that way. I try to make it useful and generic, but it is still rooted in _my_ perspective of looking into the language as, well, language. And the "Evolution" page has an intention to, first and foremost, to, well, show how the _evolution_ of the language's representation means developed.

**Commit: [f47effb](https://github.com/rubyreferences/rubychanges/commit/f47effb)**
