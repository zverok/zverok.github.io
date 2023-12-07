---
layout: advent
title: "Advent of changelog: Day 3"
next: day04
next_title: "Day 4"
prev: day02
prev_title: "Day 2"
---

The next thing I do is adding the links to the doc for each change. It is good to do as early as possible, for two reasons: first, looking into the docs gives me additional understanding for the changes; second, frequently at this point I notice that the new change is missing the docs, or there are some typos, or other things that could be improved, so I want to have time to make a PR with the enhancment.

Thankfully, the docs for the current master [are rendered](https://docs.ruby-lang.org/en/master/) at docs.ruby-lang.org in (almost) real time sync with the language changes, so it is easy to browse without looking into the code comments that are the source of those docs.

A few things I have uncovered:

* Both `Array#upack` and `String#pack` delegate most of the docs to a [separate document](https://docs.ruby-lang.org/en/master/packed_data_rdoc.html), so it would be reasonable to squish packing logic change into one changelog section;
* This document has no mention for handling unknown directives, so, that's a potential area for improvement, put in my TODO;
* `Dir.for_fd` docs have a rendering glitch (goes to my TODO)<br/>
  ![](/img/advent2023/image00.png)
* `Dir.fchdir` docs have a slightly weird wording (maybe it is me misunderstanding it, maybe can be improved)<br/>
  ![](/img/advent2023/image01.png)
* ...and a same rendering glitch:<br/>
  ![](/img/advent2023/image02.png)
* `Dir#chdir` probably has a typo in the example (this should be `Dir.pwd`):<br/>
  ![](/img/advent2023/image03.png)
* Also I am not sure whether it returns `nil` (as the docs state), should check when will try to write the code examples;
* Also, `Dir#chdir` is a nice and natural OO idea... And I am thinking about Ruby's standard library `Pathname` (which is OO wrapper around paths, files, and directories): Does it has this method? If not, should it be added? It does not, and I am putting into a separate TODO an idea to propose or implement it.
* `MatchData#named_captures`: a rendering glitch again (the first paragraph is erroneously parsed as part of the code block, and, consequently, the documentation renderer can't highlight the code as Ruby)<br/>
  ![](/img/advent2023/image08.png)
* `Module#set_temporary_name`: I think the example is somewhat confusing (and maybe have a typo: is the result in that line would be as example shows), and I leave a note to myself to check it later:<br/>
  ![](/img/advent2023/image04.png)
* I also find myself wondering about what are possible acceptable "temporary" names (the docs say it shouldn't be empty and shouldn't look like a constant name, but are there other limitations?), and about the consequences of the name assignment. All of that would be clarified on the example-writing phase.
* Oh, and maybe the "why would anyone needs that" should be clearer in docs, but I don't have an answer myself yet, so it is to "deducing the reasons" phase.
* `ObjectSpace::WeakKeyMap` docs are pretty spartan for a completely new class (especially considering non-trivial logic of "what can be garbage collected and what can't"), so— to TODO it goes!
* Oh, and also I tried to go to [ObjectSpace](https://docs.ruby-lang.org/en/master/ObjectSpace.html) page to navigate to nested `WeekKeyMap` from there, but there is no list of nested classes; I vaguely remember it was previously? (Or, I might be wrong, and it was some another document rendering site. But even in this case, maybe we should have them there?..) So I put that in "optional TODO" to check at my free time, maybe even after finalizing the changelog.
* `ObjectSpace::WeakMap#delete`'s docs are even more spartan (with the "default name" of the argument of a method defined in C) and very brief description. To be honest, most of this class docs look this way: it once was considered an (almost) internal class, but later turned out to be practically useful, so now it would be good to have it adhering to public classes documentation standards... Maybe it is what I should do together with `WeekKeyMap`'s docs improvements (most of the methods should've been similar)!<br/>
  ![](/img/advent2023/image05.png)
* `Proc#dup` and `#clone` doesn't have dedicated documentation and it is OK, their behavior should adhere to generic `Object`'s methods (and actually the change in this version is about better adhering!)
  * Though, while putting reference to those, I remembered a stale [discussion](https://bugs.ruby-lang.org/issues/19304) about documenting `Object` and `Kernel` methods (confusingly, those two similar methods are documented as `Object#dup` and `Kernel#clone`), and open the discussion's page to circle back on it.
* `Process::Status#&`  and `#>>` docs have deprecation notices that a) vaguely suggest to "use other methods" and b) don't have any special rendering: I need to check is there the convention for rendering deprecation notices (and if there isn't, maybe propose some?.. you never know where it is OK to stop your perfectionism)<br/>
  ![](/img/advent2023/image06.png)
* `Queue` and `SizedQueue` are mentioned in the NEWS without `Thread::` prefix (I need to check how it was done in the previous years, and maybe adjust NEWS)
* Their `#freeze` methods docs just tell matter-of-factly tells that it raises the exception (and even shows the code to proves that), but never explains why one can't freeze a queue
* `Range#reverse_each` never mentions any details of the behavior with semi-open ranges, so might be something to add there (but I'll think about it a bit more, maybe it is an unnecessary level of details)
* `Refinement#target` has very spartan docs; has slightly off declaration of the return value (should be `module` or `class_or_module`); also, the method `#refined_class` it intends to replace doesn't have deprecation notice:<br/>
  ![](/img/advent2023/image07.png)
* `String#bytesplice` can use some examples, probably (though it is a minor perfectionism), and also my title of the section describing the change was wrong (just saying that there are new index/length arguments, while there are new arguments for the _replacement_ string)! Corrected.
* `TracePoint` doesn't mention new `:rescue` event at all (unless I am missing something)
* `Kernel#lambda`, one of the very core methods, has very, very, very old and short docs... And once its behavior slightly changes in this version, maybe this is the time to update them?..
* `Kernel#open`, `IO.read` and other methods' docs doesn't include a notice about deprecation introduced.

Phew.

Of course, neither of the "problems" above are critical (and some might be non-existent at all), but it is things that definitely can be made better— So, it is something I am willing to do, and working on the changelog helps me finding area that need some attention.

Considering that it is better to submit PRs to the Ruby core as soon as possible (and not few days before the release— or after it), I'll probably dedicate the next couple of days to clarifying the behaviors I'd like to see documented better, and preparing that PR. This clarifying will also allow me to gain enough understanding for new features to write proper changelog entries.

**Commit: [47bb485](https://github.com/rubyreferences/rubychanges/commit/47bb485)** (only includes new links, but the result of today's work is also a list of TODOs in my private notes).
