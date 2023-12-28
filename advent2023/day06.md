---
layout: advent
title: "Advent of changelog: Day 6"
next: day07
next_title: "Day 7"
prev: day05
prev_title: "Day 5"
---

So, today would be the day for submitting enhanced docs for two classes: `WeakMap` and `WeakKeyMap`.

How would one do that?

1. Fork [Ruby's repo](https://github.com/ruby/ruby) and follow the standard clone-branch-pull request process.
2. The docs are in the [RDoc](https://ruby.github.io/rdoc/) format, in `*.c` and `*.rb` files (whatever implements the module in question); when I am not sure where to look for them, I just `grep` a phrase from docs in the folder, or, more reliably, a name of some documented method (can be seen [in class docs](https://docs.ruby-lang.org/en/master/ObjectSpace/WeakKeyMap.html) by a click on method docs). It is `weakmap.c` in our case.

The plan of things to do:

1. Fix "mechanical" problems (add method signatures in `WeakMap`);
2. Fix obviously erroneous wording (mostly in `WeakMap`);
3. Try to enhance method docs a bit, in both classes;
4. Try to enhance class docs in `WeakKeyMap`, adding some examples and a bit more explanations probably.
5. Maybe adjust `WeakMap`'s class docs a little, too (but not too much, it is an old class, and reviewing many paragraphs of new docs for it might be too much work).

The changes in C-defined method docs require to specify `call-seq:` directive if we want nicer method signature rendering than the default one. So we go from this:

```c
/* Retrieves a weakly referenced object with the given key */
static VALUE
wmap_aref(VALUE self, VALUE key)
```

To this:

```c
/*
 *  call-seq:
 *    map[key] -> value
 *
 *  Returns the value associated with the given +key+ if found.
 *
 *  If +key+ is not found, returns +nil+.
 */
static VALUE
wmap_aref(VALUE self, VALUE key)
{
```

(Mostly copying from `WeakKeyMap` docs for the same methods and adjusting them where necessary)

We can check the result by running RDoc command:

```bash
rdoc weakmap.c -o tmp/doc
```

> A couple of notes for others trying to render some Ruby docs:
>
> 1. `-o tmp/doc` is important, otherwise RDoc will use `./doc/` as a default output folder, and in Ruby repo it is not empty, and there would be a mess.
> 2. passing only one file (or, say, a pattern `*.c`) allows to receive results quickly. Yet the docs would be rendering with the "knowledge" only about classes from those files, so if you need to check the entire TOC structure, or cross-reference from your adjusted docs to other objects, you need to include more files or omit the files pattern altogether. Then you'll have docs from the entire repo, but it will take a lot of time.

At voila!

We can check at `tmp/doc/ObjectSpace/WeakMap.html` that we got from this:

![](/img/advent2023/image11.png)

To this:

![](/img/advent2023/image12.png)

The changes were trivial for methods like `#[]`/`#[]=`/`#delete` (as I've aknowledged, I've just copied `WeakKeyMap` docs), but not that trivial for other methods. Say, to document `#each` we should know whether it accepts no-block form (returning iterator), to know which signatures to put there (by convention, we need to [put both](https://docs.ruby-lang.org/en/master/Array.html#method-i-each) if it does).

Welp, it doesn't:

```ruby
m = ObjectSpace::WeakMap.new
k, v = 'foo', 'bar'
m[k] = v
m.each
# in `each': no block given (LocalJumpError)
```

I adjust docs a little, fighting the urge to expand them to the level of `Hash` or `Array`'s with nuances descriptions and examples: this class is still semi-internal, and I need a _reviewable_ PR, mostly concerned with fixing glitches and providing necessary docs for new features!

So, this part is done relatively quickly; and on the way I uncovered small bug in `WeakKeyMap` examples I copied: the code (probably adapted from `Hash` docs) referred to example object created as `m` by name `h` sometimes. Fixed, too!

This mostly covers points 1-3 from the TODO above. The changes still mostly about `WeakMap`, after some consideration I don't think `WeakKeyMap`'s methods require more explanations— Save for slightly confusing `#getkey`. So, I try to add an explanation and example:

![](/img/advent2023/image13.png)

With that, my timeslot allocated for changelog for today have ended. Tomorrow, I'll try to finish `WeakKeyMap`/`WeakMap` docs adjustments, create a PR, and work on the another: with small documentation fixes in other places I put in my TODO.
