---
layout: advent
title: "Advent of changelog: Day 9"
next: day10
next_title: "Day 10"
prev: day08
prev_title: "Day 8"
---

So, today is "documentation adjustment PR" day.

To be completely honest, I still feel somewhat uneasy and almost arrogant doing this: to change existing docs into the form that would better explain some things, one needs to believe they know how to explain things better!

While I do have some experience of studying Ruby, thinking about Ruby, and teaching Ruby (as a personal mentor, courses teacher, and older colleague), I can only hope my vague feelings that "this would be easier to understand" are leading to some improvements. The good thing is there is a community of maintainers & reviewers for the documentation PRs; the bad thing is it is frequently hard to argue about why something should be explained in a certain way, or how to demonstrate the behavior with examples.

Unlike code, there is only "I believe this explanation is better"-level arguments (and there is no separation of "how it works," provable by tests, and "how is it written," which can be compromised to the common ground). We'll see how it works.

Anyway— Here we go with a PR: https://github.com/ruby/ruby/pull/9183

After all, I've chosen from my TODO only items strictly related to the new features, and only improvements that (in my eyes!) would affect the feature understanding/adoption. So there is still a backlog of things like "Looks like `Kernel#lambda` docs are outdated in general" or "Think if `String#bytesplice` needs more examples." There is a lot of work ahead more closely related to the changelog itself, and I wanted to make sure at least those improvements will have chance to make it before the release.

As an aside note, while experimenting with `Module#set_temporary_name` I noticed one more changes that happened but is missing from the `NEWS.md`: a change in `NoMethodError` rendering:

```ruby
class Foo; end; Foo.new.bar
# Ruby 3.2: undefined method `bar' for #<Foo:0x00007f33082a85b8> (NoMethodError)
# Ruby 3.3: undefined method `bar' for an instance of Foo (NoMethodError)
```

I remember a [lively discussion](https://bugs.ruby-lang.org/issues/18285), and the outcome probably deserves a `NEWS`/changelog entry!
