---
layout: post
title:  '“Useless Ruby sugar”: Pattern matching (Pt. 1)'
date:   2023-10-20
description: "They said “...just switch to the language that already has it, would you?”"
categories: ruby philosophy
comments: true
image:  https://zverok.space/img/2023-10-20/pm.png
---

> This is a part of a blog post series about "useless" (or: controversial) syntax elements that emerged in recent Ruby version. The goal of the series is not to defend (or criticize) the features, but to share a "thought framework" for analysis of their reasons, design, and effect the new syntax has on a code that uses it. See also [intro post](https://zverok.space/blog/2023-10-02-syntax-sugar.html), and the previos text that covered [numbered block arguments](https://zverok.space/blog/2023-10-11-syntax-sugar1-numeric-block-args.html).

Our today's theme is **pattern matching**.

## What

Pattern matching [emerged in Ruby 2.7](https://rubyreferences.github.io/rubychanges/2.7.html#pattern-matching) as an experimental feature and went through several improvements and scope expansions in [3.0](https://rubyreferences.github.io/rubychanges/3.0.html#pattern-matching), [3.1](https://rubyreferences.github.io/rubychanges/3.1.html#pattern-matching), and [3.2](https://rubyreferences.github.io/rubychanges/3.2.html#pattern-matching).

It is a syntax that allows to **match** nested data structures declaratively and, at the same time, **bind** some of its parts to local variables. Say,

```ruby
case point
in [Integer => x, Integer => y]
  # `point` was a pair of integer coordinates,
  # they are now put in `x` and `y` local variables
in Point[x, y]
  # ..it was a two-field structure of class Point,
  # its fields are now in `x` and `y`
in lat:, lng:
  # it was a hash with :lat and :lng keys,
  # corresponding values are put in `lat` and `lng`
in {coord: {latitude:, longitude:}}
  # it was a nested hash format, `latitude` and `longitude`
  # are taken from the middle of it

# ...and so on
```

There are several language constructs in which patterns can be used:
* `case ... in` for branching, as shown above;
* standalone `value => pattern` to match and raise if the structure doesn't correspond to a pattern (validate and bind known structure);
* and `value in pattern` for the boolean check.

There is an elaborate yet pretty natural syntax for patterns, which we'll see in a few paragraphs.

**The importance of this feature and its effect on the Ruby code** is a source of severe disagreements. Some put it in the "mere syntax sugar" bin (which is how it ended up as a part of this series... which I already regret a bit because the feature is a huge thing to discuss!). At the opposite extremum, there are people who believe that pattern matching is a separate paradigm, and "if you want a language with one, you just switch to that language."

At the time of writing, pattern matching definitely got some adoption, and for all I can say, it didn't (yet?) change Ruby's style significantly. All in all, if you focus only on the "check the structure" aspect, it is easy to argue that we've got just a witty short syntax for a bunch of `is_a?` and `==`. But all the variety that "match-and-bind" brings to a code style requires more layered assessment.

You might do yours by reading further!

## Why

There are many ways to talk about pattern matching's virtues, but the main intuition that triggers the craving for "something like that" is **symmetry**.

In any modern high-level language, it is very easy to declaratively build a nested data structure of arbitrary depth, width, and complexity. You just spell it "as it is," and embed variables, constants, and calculations as you please:

```ruby
{
  events: [
    {
      timestamp: Time.now,
      kind: 'created',
      tags: [*DEFAULT_TAGS, 'created'],
      # ...and so on ...
    }
  ]
}
```

We take this way of building data structures for granted already, and any language/API that still requires an imperative way to pronounce it step by step ("create an array, put this into first element, then put this in second...") looks retrograde.

But what about the opposite operation? What if one wants to take a big (or not so big, yet nested) data structure and get data from it in a way that will look just as declarative? "We expect this structure, and we need to work those parts of it."

There are many possible answers ([lenses](https://theswiftdev.com/lenses-and-prisms-in-swift/) are quite cool!), but the usual process of the evolution of programming languages, via ideas blending and migration from academic to everyday seems to have established the common sympathy for a **structural pattern matching**: some variety of a declarative match-and-bind syntax, preferably looking symmetrically to a data structures construction.

Newer languages like Rust arrived with the construct from the start, and older high-level ones started to introduce it all over the industry, from Python to C#.

A couple of decades before, the same happened with the idea of regular expressions: once seen as esoteric or, in the best case, specialized library tool, they eventually become ubiquitous.

The "can we match other data structures declaratively" is logically the next step—and Ruby is not an exception[^1]. Especially considering that **some amount of deconstructing/structural checks is present in the language** already—and that seemed to be fortunate, as matching generic data structures requires much deeper integration with the language than regexps do.

[^1]: I believe that would Ruby be designed in today's landscape, when "what is typical structural matching" is established, it would have that in the core of the language immediately—like back at the times it was born, it took a great effort to incorporate then-bleeding-edge-of-mainstream concepts like class methods, iterators, and such.

## How (it was)

Ruby does have a **structural deconstruction** already—unfortunately, only for arrays.

```ruby
a, b = [1, 2]
p(a:, b:) #=> {:a=>1, :b=>2}

head, *tail = [1, 2, 3, 4]
p(head:, tail:) #=> {:head=>1, :tail=>[2, 3, 4]}
```

> For those who missed the recent developments and is confused by `p(a:, b:)` syntax: I am using [keyword argument values omission](https://rubyreferences.github.io/rubychanges/3.1.html#values-in-hash-literals-and-keyword-arguments-can-be-omitted)—a feature introduced in Ruby 3.1. Basically, `p(a:, b:)` is exactly the same as `p(a: a, b: b)`, and it is super-helpful for debugging, among other things. Oh, and there definitely would be a post later in the series discussing reasons and consequences of this "useless sugar"!

It is more powerful than many of us are initially aware, allowing nested structures unpacking:

```ruby
(top_left, *top), *middle, (*bottom, bottom_right) =
  [[1, 2, 3], [4, 5, 6], [7, 8, 9], [10, 11, 12]]

p(top_left:, top:, middle:, bottom:, bottom_right:)
#=> {:top_left=>1, :top=>[2, 3],
#    :middle=>[[4, 5, 6], [7, 8, 9]],
#    :bottom=>[10, 11], :bottom_right=>12}
```

It also works in implicit assignment to method and block arguments:

```ruby
# hash iteration produces pair of [key, value],
# with_index wraps it into another pair [element, index]
# We can unpack it back in the proc:
{a: 1, b: 2}.each.with_index { |(key, val), idx| p(key:, val:, idx:) }
#=> {:key=>:a, :val=>1, :idx=>0}
#=> {:key=>:b, :val=>2, :idx=>1}

def smart_method((head, *middle, (left_tail, right_tail)))
  p(head:, middle:, left_tail:, right_tail:)
end

data = [1, 2, 3, 4, [5, 6]]

# Note I just pass one value, not *data, and the second
# parentheses in methods declaration take care of unpacking the array
smart_method(data)
#=> {:head=>1, :middle=>[2, 3, 4], :left_tail=>5, :right_tail=>6}
```

One might've dreamt that this worked for hash decomposition and an arbitrary mix of arrays and hashes:

```ruby
a:, b: [left, right] = {a: 1, b: [2, 3]}
# Could it put 1 in `a:`, and 2, 3 inside `b:` in `left` and `right`?..
```

...but alas, this is a syntax error!

It should be noted that this syntax only provides **binding** (destructuring), not **matching** (checking the shape). If you pass data that doesn't match the intended shape, there would be no error most of the time[^2], just a bunch of `nil`s/empty arrays in the parts that weren't matched:

[^2]: Except for attempt to pass not enough argument on `method(*args)` call— but even that works only for a limited case of one layer of positional arguments.

```ruby
(top_left, *top), *middle, (*bottom, bottom_right) = 1
#=> {:top_left=>1, :top=>[], :middle=>[], :bottom=>[], :bottom_right=>nil}
```

---

The **matching** in classic Ruby is implemented with the "case equality operator" `===`. By default, it is the same as equality, but a lot of classes redefine it, so you have this:

```ruby
"b" === "b"             #=> true, simple equality
String === "b"          #=> true, Class redefines `#===` to match objects of this class
("a"..."z") === "b"     #=> true, Range redefines it to match objects inside range
/\w/ === "b"            #=> true
# and even
proc { _1.length < 3 } === "b" #=> true
```

The best-known usage of the operator is implicit invocation inside `case` branching (hence the operator name). In the following code `===` is implicitly called for each branch, passing the `argument` there, until one returns `true`:

```ruby
case argument
when nil
  # handle it one way...
when 1..10
  # it is a number between 1 and 10, handle it another way
when /user:.*/
  # it is a string matching this regexp, handle it the third way
when User
  # ...and so on...
end
```

(Obviously, in most `case`s the list of options is not that motley; it is more usual to see homogeneous branches like "it is `nil`, or one of those classes"; "it is definitely a string, but branch by regexp," and so on.)

_Despite_ the operator name, it is useful in a few other core constructs, such as generalized `grep`:

```ruby
# #grep calls `#===` inside, so besides the classic Unix grep usage...
lines.grep(/^user:/)
# ...this works too:
numbers.grep(0..20)
results.grep(Success)

# as well as some predicate methods:
lines.any?(/^user:/)
numbers.all?(0..20)
results.none?(Success)
```

The power of `===` stops here, though: there is no way to **recursively match** nested structures, nor is there a way to **assign (bind) some variables** based on a successful match.

`case` has an "escape hatch" form for when patterns aren't powerful enough: without object to match, it just works like a tree of `if`/`elsif`, but looking more regularly and underlining the branches are homogeneous:

```ruby
case
when args.size == 2 && args.first.is_a?(String)
  # do something
when args.size == 1 && (arg = args.first).is_a?(Number) # can even assign vars on the way!
  # do something else, `arg` is assigned here
```

Several codebases I worked with encouraged this style of delegating by branches—of course, with reasonable complexity of checks. Others (including the default Rubocop style) consider it a "stylistic error" and misuse of `case`. In any case, it doesn't help much with comparison expressiveness, just unifies branches a bit.

To those who handle vaguely structured data a lot, there was always a feeling—an _intuition_, if you want!—that there should be a more expressive and Ruby-idiomatic way to describe the expectations. And so, in Ruby, the quest for a pattern matching frequently was seen as taking the "case equality" power further[^3].

[^3]: Sometimes, I wonder what turn the history might've had if the discussion focused on "better unpacking" instead and tried to get closer to the goal from this side— But we'll never know, I guess.

_Random fun fact: my Ruby blog was once started because I wanted a place to [share my experiments/thoughts](https://zverok.space/blog/2015-07-18-matchish.html) on `===`-based pattern matching implementation in a library. The conclusion of that early article was: "All in all, powerful pattern matching needs to be a core language feature."_

In the wake of the Ruby 3.0 approach, the pressure in the community, "we should finally introduce it," grew higher. Say, Ruby Conf 2017 had a talk that showed the [working prototype](https://rubykaigi.org/2017/presentations/yotii23.html) of `%p()` syntax for patterns. At that time, Matz said about such prototypes:

> If we were going to add pattern matching in Ruby, we should add it with better syntax. [...] The problem is that I have no idea for an excellent syntax for the pattern matching right now.

I, too, [returned to the topic](https://zverok.space/blog/2018-06-26-pattern-matching.html) at that time, trying to reason about how that "excellent syntax" might look. A curious reader might enjoy comparing the ideas in that post with the form the pattern matching syntax has taken in Ruby eventually.

## How (it arrived)

**Finally, in 2.7 (Christmas 2019), the [new feature](https://rubyreferences.github.io/rubychanges/2.7.html#pattern-matching) arrived,** to some extent as a surprise: it emerged not long before the final release and was documented only by a conference slides link in the language changelog. But it was what Matz finally accepted (and, if I understand correctly, to some extent designed).

It turned out to be integrated with the `case` statement, but not via some new powerful object supporting `===`, as many expected. The chosen solution was to use a new `in` keyword/operator[^4]:

[^4]: `in` was a reserved keyword in older versions of Ruby, supporting the syntax `for el in array` (which is almost never used), so it was safe to reuse: `in` as a local name was always invalid, and there was no conflict with `for ... in ...` for the parser, even if somebody _was_ using it.

```ruby
case args
in [Integer, Integer]
  # match a pair of integer
in [String => first, String => last]
  # match a pair of strings, put them into local vars `first`, `last`
in [foo, bar]
  # match a pair of any values, put them into local vars `foo`, `bar`
# ...and so on...
```

The highlight of the solution is that the syntax of the patterns themselves turned out to look incredibly well-aligned with the language user's habits (or, again, _intuitions_!):

* you have an array of values `[1, 2]`, you match it with an array of patterns `[pattern, pattern]`,
* same with a hash: `{x: 1, y: 2}` is matched with just `{x: pattern, y: pattern}`,
* you want to put some part into a variable? There is a common construct seen in Ruby's error handling (`rescue ErrorType => error_var`), so just use that: `pattern => variable`;
* but if putting in the variable without any additional checks is all you want, there is a straightforward way to do that: just write `[x, y]` to mean "put in `x` and `y`", or `{x:, y:}` for a hash.
* a few other things needed learning but had easy mnemonics: `pattern | pattern` to match with several different ones, `^x` as "variable pinning" (a way to specify "use this variable for comparison, not to put value into it").

This clarity came with a price that is not that low: **the patterns syntax is fully isolated from the rest of the language**. You can't put a pattern into a variable or constant or pass it to a method: there is no such thing as a "pattern object." It is just a special syntax that works after `in`.

To be fair, **that was probably the only way to introduce a proper pattern matching** into a language that expressive this late into its life cycle: all of the "natural" syntax of the patterns above (`[Integer, Integer]` and so on) were _already_ valid syntax constructs in Ruby with different meanings.

The feature was cautiously marked "experimental" in Ruby 2.7, yet the basis laid then had proven itself reasonable.

In the next versions, pattern matching received some polishing and cleaning up. The most notable change was the establishing of two types of one-line pattern matching statements:

```ruby
# standalone `in` just returns `false` the pattern doesn't match, useful in `if`:
if point in [x, y]
  # it was a 2-element array, and we checked and deconstructed it
  # ...
end

# standalone `=>` is "declarative" match, stating it _should_ match or raising an error
kwargs => db: {user:, password:, **}, logger:
# => here, `user`, `password`, and `logger` are assigned if the structure was right
# ...otherwise NoMatchingPattern is raised
```

In 2.7, there was initially only `in` form, and it was raising an error when not matched. There was a [turbulent discussion](https://rubyreferences.github.io/rubychanges/3.0.html#one-line-pattern-matching-with-) about this and also about the order of arguments (in many languages, it is `values, to, assign = pattern`), which eventually led to establishing the pair of operators we have now.

One of the results of this discussion (and comparing with other languages) is emphasizing the one-line statements are also working in the simplest form of "match the whole statement into one variable." This is mostly useless for `in`, but made `=>` into a novel form of assignment, dubbed "rightward assignment":

```ruby
1 => x
# just assigns 1 to x. Weird, huh?..

# But this one might be appealing!
File.read('data.txt')
  .split("\n")
  .map { |ln| some_processing(ln) }
  .select { |ln| some_filter(ln) }
  .and_so_on => result

# here, it is easier to see where the `result` have came
# from if it is assigned with `=>` at the end of the calculation
pass_further(result)

# And it is also easier to update if the result
# became more complicated:
File.read('data.txt')
  .some_complex_processing
  .and_so_on => users: [admin, *rest], transactions:
```

A lot of other things [have happened](https://rubyreferences.github.io/rubychanges/evolution.html#pattern-matching) to pattern matching through the recent versions: the introduction of "find patterns," more powerful pinning, adding support for deconstruction to several core classes, and so on.

A big feature, after all. It required some significant compromises, and brought some interesting and far-reaching consequences, which, I believe, haven't fully materialized yet!

**The feature is so big, that we are currently at half of its supposed discussion.** See you next week for the rest of it. As with [the numbered parameters](https://zverok.space/blog/2023-10-11-syntax-sugar1-numeric-block-args.html), it would contain the feature _grounded critique and analysis_:

* Irks and Quirks;
* Consequences;
* How others do it;
* Taking it further;
* Conclusions.

You can subscribe to my [Substack](https://zverok.substack.com/) to not miss the next version, or follow me on [Twitter](https://twitter.com/zverok).

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
