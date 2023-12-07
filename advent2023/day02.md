---
layout: advent
title: "Advent of changelog: Day 2"
next: day03
next_title: "Day 3"
prev: day01
prev_title: "Day 1"
---

The first steps after skeleton creation is pretty mechanical too: just move everything relevant from `NEWS.md` to the appropriate changelog structure (creating headers + half-empty sections for each entry to cover).

Two questions aren't that trivial, though:
* what exactly to cover?
* how to structure?

We'll get to the overall structure later, first we need to choose what to cover.

From the beginning of the changelog I made a decision to limit the changelog to the Ruby-the-language syntax, not Ruby-the-tool, i.e. only to cover changes in syntax, grammar, and core objects API. This is the area that interests me the most, and, consequently, the area I can hope to understand and explain well. The corresponding `NEWS.md` entries are mostly residing in the sections "Language changes" (empty this year, yay for everybody hating syntax changes!) and "Core classes updates." Some of the changes I feel necessary to cover, though, are in "Compatibility issues" section (when the behavior of the existing method changes).

In the previous years, there was also extensive "Stdlib updates" section, which I tried to cover to some extent. Through the last several version of Ruby, though, most of the standard library was extracted to standalone gems/repositories, with the development process and changes tracked in those repositories. I have a feeling the "annotated language changelog" should include at least some of the outstanding changes, but in previous years didn't have enough resource to follow all of them. We'll see, maybe this year (considering less core changes to track) I will try to catch up!

But for now, I just copy every relevant entry from `NEWS.md` and turn them into my Changelog format:

In `NEWS.md`:

```
* String

    ...
    * String#bytesplice now accepts new arguments index/length or range of the
      source string to be copied.  [[Feature #19314]]
```

In my `3.3.md`:

```markdown
### `String#bytesplice`: new arguments `index`/`length`

* **Reason:**
* **Discussion:** [Feature #19314]
* **Documentation:**
* **Code:**
* **Notes:**
```

Some highlights:
1. I keep the headers short so they would be informative in the changelog's table of contents (keep them just to method/class name when it is related to a new method or class).
2. I need to decide on grouping related entries together in one section, and it is not always trivial. Say, there are three new methods: `Dir.for_fd`, `Dir.fchdir`, and `Dir#chdir`, as three separate `NEWS.md` lines; all three are introduced and discussed in the same [tracker ticket](https://bugs.ruby-lang.org/issues/19347). For now, without looking too deep into the definition and reasoning, I decided that the first two methods (class methods accepting file descriptors) would make it to the same section, while the third one (instance method of the `Dir` object) would probably deserve its own. Other examples: change of behavior of `Queue` and `SizedQueue`'s `#freeze` method definitely should go together, but about changes in `Array#pack` and `String#unpack` I am not so sure yet: the change is the symmetric, but into which structural section of the changelog it will go?.. Sometimes in such cases I prefer to keep two separate sections, linking to each other.

For the reasons above (a lot of small decisions/analysis to make on the road, and the rephrasing of everything), the process is manual. I experimented with automated `NEWS.md` conversion when I first worked on the changelog six (wow) years ago, but I decided against it. Today it took just about half-an-hour, but in the years with more rich change lists, it might've took several hours on itself!

**Commit: [f250ef7](https://github.com/rubyreferences/rubychanges/commit/f250ef7).**
