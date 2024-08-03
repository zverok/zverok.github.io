---
layout: post
title:  'How it became like this? Ruby Range class'
date:   2024-07-26
description: "Understanding the core class design and usage via its evolution"
categories: ruby evolution
comments: true
image: https://zverok.space/img/2024-07-26/cover.png
---

**Understanding the core class design and usage via its evolution**

Years ago, my studies into the [Ruby Evolution](https://rubyreferences.github.io/rubychanges/) started with the persuasion that mastering the programming language to express one’s intentions clearly and efficiently may grow significantly by understanding how it evolved and what intentions were put behind its various elements.

Moving back through the history of a change of some element of the language exposes a thinking and consensus process that led to API design. It allows one to internalize its functioning, as opposed to just memorizing the cheatsheets.

To illustrate that, let’s look into one of Ruby’s core classes: [Range](https://docs.ruby-lang.org/en/master/Range.html).

## What’s a Range?

```ruby
range = (1..5)
range.cover?(3) #=> true
range.each { puts _1 } # Prints 1, 2, 3, 4, 5
```

Range is a type/data structure that is defined by two range boundaries (**beginning** and **end**) and designates the space of values before them.

It is somewhat less ubiquitous in programming languages than array/list or dictionary/map, but still pretty widespread. Two main meanings of the range are:

* a _discrete sequence_ of values between its boundaries and
* a _continuous space_ of values between its boundaries.

The main usages are:

* iteration (kinds of `for` loops over the specified sequence);
* testing values for being inside some interval;
* collections slicing.

Not all programming languages that have ranges and range-like objects provide both kinds (discrete and continuous); not all of them use ranges for all three cases listed above.

For example, in Python, there is a [range](https://docs.python.org/3/library/stdtypes.html#range) type for iteration and `obj in range` testing, and a separate [slice](https://docs.python.org/3/library/functions.html#slice) type for collection slicing (that doesn’t have any further semantics other than knowing its start, stop, and step), while in C#, the class called [Range](https://learn.microsoft.com/ru-ru/dotnet/api/system.range?view=net-8.0) has _only_ this functionality (slicing collections). At the same time, Rust, Kotlin, and Scala use their ranges for all listed cases, and Zig, as far as I can understand, has a range-like syntax for [slicing](https://ziglang.org/documentation/master/#Slices), [iteration](https://ziglang.org/documentation/master/#for), and [matching](https://ziglang.org/documentation/master/#switch), but this syntax doesn’t produce a value and can’t be used outside of those constructs.

So...

**What has been happening with Range through Ruby’s history? And, indirectly, another question: is there a big design/change space there? Turns out there is some!**

> Some milestones in Ruby’s history, to put it in perspective:
>
> * Ruby 3.0 (2020) is the current major version, with each 3.x released once a year being significantly advanced over the previous one, the current one is 3.3 (with 3.4 coming December’24);
> * Ruby 2.0 was released in 2013, and it introduced that “new release every year”, and lived through it till 2.7;
> * Ruby 1.9 (since 2007) was a big preparatory branch before 2.0, with each of 1.9.1, 1.9.2, and 1.9.3 introducing many changes;
> * Ruby 1.8 (since 2003), again, introduced many changes in each "patch" release (the last notable one was 1.8.7); it is probably the version that was first to gain a lot of notoriety due to Rails (first version released in 2004) and widely popular “Pickaxe” (Programming Ruby from Pragmatic Programmers, 2nd edition) book;
> * Ruby 1.6 (since 2000) was probably the first version known to English-language programmers, the first edition of Pickaxe was dedicated to it and eventually was donated to Ruby community as an [online reference](https://ruby-doc.com/docs/ProgrammingRuby/) to the language;
> * I can’t say much about versions before that, but Ruby went from v. 1.0 in 1996 (the first public release) to 1.4 in 1999, through very active development.

But let’s get back to our questions.

## Is this value in a range? And what does it mean?

Given a range from `b` (beginning) to `e` (end), and value `v` of the compatible type, how to check that value “belongs” to the range, and what’s the semantics of this “belonging”?

In Ruby, Range has two methods to answer the question:

* [#include?](https://docs.ruby-lang.org/en/3.3/Range.html#method-i-include-3F) (also aliased as `#member?`) to check if `v` is a part of the sequence from `b` to `e`, and
* [#cover?](https://docs.ruby-lang.org/en/3.3/Range.html#method-i-cover-3F) to check if `v` is in the continuous value space between `b` and `e`.

To demonstrate the difference on strings:
```ruby
('a'..'e').include?('b') #=> true, it is a part of sequence
('a'..'e').cover?('b') #=> true too

('a'..'e').include?('bat')
#=> false, sequence from 'a' to 'e' doesn't include value 'bat'
('a'..'e').cover?('bat') #=> true
```

There is a third method, [#===](https://docs.ruby-lang.org/en/3.3/Range.html#method-i-3D-3D-3D) (three equal signs), which is rarely used explicitly, but implicitly invoked in pattern-like matching contexts[^1]:

[^1]: Ruby has had structural pattern-matching [since Ruby 2.7](https://rubyreferences.github.io/rubychanges/evolution.html#pattern-matching), but here we are talking about a simpler and more widely used feature, mostly implemented by various objects having redefined `#===` method.

```ruby
case year
when 2000..2005 # implicitly calls (2000..2005) === year
  # ...
end

# implicitly calls `0..18 === item` for each item of the collection,
# returns those matched; there is also grep_v
collection.grep(0..18)

# implicitly calls `0.8..1.0 === item` for each item of the collection,
# returns if any of them returned true; there are also all? and none?
collection.any?(0.8..1.0)
```

In **Ruby 2.6**, I [persuaded](https://bugs.ruby-lang.org/issues/14575) the core team to change the implementation of `#===` for _generic ranges_ to always use `#cover?` instead of `#include?`, so, for example, this code started to work:

```ruby
require 'date'
case DateTime.now
when Date.new(2024, 6, 1)...Date.new(2004, 9, 1)
  puts "still summer!"
end
```

The range of dates obviously _covers_ the time between them but doesn’t _include_ it in the sequence from the beginning to the end. My winning argument in making this change was that it **always worked this way for numbers**, creating a surprising inconsistency:
```ruby
(1..5) === 2.3             #=> true
('1.8'..'1.9') === '1.8.7' #=> false, even though comparison-wise it is in between
```

This inconsistency was probably rarely noticed before: the most widespread ranges are still numeric ones (many other languages have them as the _only_ kind of ranges), and if somebody has tried it with other values and received a “somewhat weird” result, they probably just decided “that’s how it is.”[^2]

[^2]: At the time of the change, the string behavior was left untouched (so the example above still returned `false` in Ruby 2.6), but it was [fixed in 2.7](https://rubyreferences.github.io/rubychanges/2.7.html#-for-string). The reason for caution was breaking the old code; but once _other_ ranges started to check values continuously, the string’s inconsistency was sticking out like a bug.

Though, before that, people have noticed that using `Time` in `case` statements might be convenient, and it just didn’t work (as time is not a discrete type, there is no “sequence” between two time points, so `#===` was trying to invoke `#include?`, which tried to produce a sequence and raised an error).

So, in **Ruby 2.3**, it was solved by introducing the (internal, not exposed to the user code) concept of “linear” objects. It was [hardcoded](https://github.com/ruby/ruby/blob/v2_3_0/range.c#L323-L335)[^3] to be real numbers and core class `Time` (but not standard library’s `Date` or `DateTime`). For such “linear” objects, the behavior of `#include?` was made like that of the `#cover?` (comparison with range ends).

[^3]: Ruby typically uses duck typing (presence of some method) to check whether some object corresponds to some “interface”, but this is impossible for the concept of “linear” objects: their basic definition is “for every two values of the type, the order is defined,” which is represented by method `<=>` returning -1, 0, or 1, but the _presence_ of this method does not tell anything useful (almost every object has it, it just returns `nil` for not comparable values).

But this discrepancy does not always exist in the language!

Long before that, when **Ruby 1.9.1** introduced the `#cover?` method, its intention was probably to have a name clearly representing the concept of the element being between the boundaries of the range. That version also changed the implementation of `#include?` (to mean “sequence inclusion” for everything other than numbers), but `#===` stayed to be implemented via `#include?`!

Because before that, when **Ruby 1.8** introduced `#include?`, there were two methods: `#member?` to check for sequence inclusion and `#include?` itself, to check whether some value is between the range’s boundaries (and `#===` worked through it).

Interestingly, the git history of Ruby also can show us the doubts around `#member?`/`#include?` behavior:

* In Ruby 1.8.0, when the pair was introduced, `#member?` was consistently checking sequence inclusion _even for numbers_ and `#include?` checked covering;
* Very soon, in 1.8.2, they were changed to have the same implementation (only covering);
* And then in 1.9.1, the method—now having both names `#member?` and `#include?`—was sophisticated to check covering for numbers and ASCII-only strings and delegate to generic collection `#include?` otherwise (back to checking sequence);
* ...which slowly migrated to the situation we had by 2.6.

But, getting back to `#===` and covering problem, before Ruby 1.9.1 and after Ruby 2.6/2.7, there was the same behavior:
```ruby
("1.8".."1.9") === '1.8.7' #=> true
['1.6.1', '1.8.1', '1.8.7', '1.9.1'].grep("1.8".."1.9")
#=> ['1.8.1', '1.8.7']
```

...while the versions between them had this weird(ish) discrepancy! Also, only in the short span of Ruby 1.8.0-1.8.2, and never since, `#member?` worked as “checking it is in a sequence” for numbers:
```ruby
(1..5).member?(2) #=> true
(1..5).member?(1.3) #=> false on Ruby 1.8.0-2, true ever since!
```
...but probably the case for checking “this number is a part of the specified sequence of integer numbers” is too esoteric to be requested to have a method that supports it.

Finally, **before Ruby 1.8** (and since the very early versions of Ruby), there was only `Range#===` (to use both implicitly in `case`-like situations, and explicitly, when checking values), and its only behavior was like modern `#cover?`.

> **How others do it:** In Rust and Kotlin, there is only one contains method ([Rust](https://doc.rust-lang.org/std/ops/struct.Range.html#method.contains), [Kotlin](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin.ranges/-closed-range/contains.html)), which behaves like Ruby’s `cover?`; in Python and Scala, only integer ranges are allowed, and, consequently, only integer values are included in ranges. Rust, Kotlin, and Zig (with its range-like, but not value-producing syntax) allow ranges in `case`-like constructs, while Python and Scala do not.

**So, that’s it about the Range inclusion turbulent story. But there are other stories!**

<div class="one-ukrainian-thing" markdown="1">
  <h3>A postcard from 🇺🇦</h3>

  _**Please stop here for a moment.** This is your regular mid-text reminder that I am a living person from Ukraine, with the Russian invasion still ongoing. Please read it._

  **One news item.** On July 24, Russian ballistic missiles [hit my home city, Kharkiv](https://x.com/NatalieZubar/status/1815988487780208791), destroying the office and cars of a humanitarian demining fund. (Russians immediately claimed it was a “lair of foreign mercenaries.”)

  **One piece of context.** [97% of Russian missiles, drones, bombs hit civilian infrastructure, destroying "private & commercial buildings, hotels, schools, churches, hospitals, & numerous infrastructure facilities" in Ukraine. Only 3% strike military targets.](https://x.com/GicAriana/status/1809064164377117146)

  **One fundraiser.** [The PayPal fundraiser](https://x.com/sternenko/status/1816403956299428172) from prominent and competent Ukrainian volunteer for drone-fighting drones. Russian (and Russian-Iranian) drones are a huge threat to our cities and to our frontlines, and there is a new perspective development in the industry that might change that.
</div>

## What values can be range boundaries?

...And, by extension, what values and types ranges can support, in general?

In Ruby, the Range can be made of boundaries of any type if they are _comparable_: namely, if `begin <=> end` returns `0`, `1`, or `-1`[^4].

```ruby
# Valid ranges
(1..5)
('a'..'b')
(Time.parse('13:30')..Time.parse('14:30'))

# Invalid ranges
(1..'3')
# bad value for range (ArgumentError)
((2 + 3i)..(2 + 4i))
# bad value for range (ArgumentError) -- complex numbers aren't linear
```

[^4]: [Object#<=>](https://docs.ruby-lang.org/en/3.3/Object.html#method-i-3C-3D-3E) is the main generic comparison method that is used in ranges, sorting, and similar situations; types with a defined order of values redefine it. When the linear order of two values is undefined (string and number or two complex numbers), it just returns `nil`. [Comparable](https://docs.ruby-lang.org/en/3.3/Comparable.html) is a convenient mixin that can be included in class that defines this method and will provide all other comparison methods based on it (`==`, `<`, `>`,`<=`,`>=`), and they would behave reasonably (say, `==` implemented via Comparable returns `false` for incomparable objects, while `>` and `<` raise `ArgumentError`).

Order of boundaries is not enforced, though: range like `(0..-5)` is valid. One of the reasons is probably using it for array slicing:
```ruby
ary = [1, 2, 3, 4, 5]
ary[-1] #=> 5 -- "the last item"
ary[2..-1] #=> [3, 4, 5] -- from third to the last one
```

In **Ruby 2.6**, the “endless” ranges were introduced:
```ruby
r = (1..) # from 1 to infinity
r.end #=> nil
# Explicit nil works too:
r == (1..nil) #=> true
```

Initially, they were meant just as a small syntax sugar for array slicing “till the last item”—as writing `ary[2..]` seemed nicer than mathematically awkward `ary[2..-1]`  or too wordy `ary[2...ary.length]`.

At that point, “it is mostly for array slicing” was a counter-argument against symmetrical ranges without beginning. But at **Ruby 2.7**, I [managed to find](https://bugs.ruby-lang.org/issues/14799) persuasive enough arguments for them to be introduced, emphasizing usage as a pattern and as a constant in DSL:

```ruby
case release_date
when ..1.year.ago
  puts "ancient"
when 1.year.ago..3.months.ago
  puts "old"
when 3.months.ago..Date.today
  puts "recent"
when Date.today..
  puts "upcoming"
end

# Celsius degrees
WORK_RANGES = {
  ..-10 => :off,
  -10..0 => :energy_saving,
  0..20 => :main,
  20..35 => :cooling,
  35.. => :off
}
```
...enforced by [proposing](https://bugs.ruby-lang.org/issues/14784) the usage of ranges in the [Comparable#clamp](https://docs.ruby-lang.org/en/3.3/Comparable.html#method-i-clamp) method (limit the value)

```ruby
# #clamp before Ruby 2.7: two separate values for boundaries:
-2.clamp(0, 100) #=> 0
20.clamp(0, 100) #=> 20
101.clamp(0, 100) #=> 100

# #clamp since Ruby 2.7: can use range
-2.clamp(0..100) #=> 0
# ...which allows to use one-sided ranges when necessary:
-2.clamp(0..) #=> 0
10000.clamp(..100) #=> 100
```

The existence of endless and beginless ranges raised a question of the possibility of a range without either boundary. It is made possible, though there is no specialized literal for it:
```ruby
r = (nil..)
# or
r = (..nil)
# or
r = (nil..nil)
```
...but just `(..)`, while theoretically nice, is too rarely necessary to complicate the parser.

The “infinite range” might seem just a curiosity, but it might be useful for consistency when produced dynamically (when some code conditionally decides whether some value should be limited from the top and from the bottom) or as a “catch-all” default pattern in some DSLs.

The existence of those new kinds of ranges, again, raises a question of covering/inclusion. The answers are mostly coming naturally, though there were a few edge cases to fix.

```ruby
# natural behavior:
('a'..).cover?('b') #=> true -- it is more than the beginning
```
Using the "linear object" behavior described above, `#include?` works like `#cover?` with real numbers (and `Time`):
```ruby
(1..).include?(1.5) #=> true
```

But only in **Ruby 3.2** trying to check `#include?` on the endless range for objects other than linear was fixed to raise an error immediately:
```ruby
('a'..).include?('bat')
# cannot determine inclusion in beginless/endless ranges (TypeError)
```

Before that, it just hung indefinitely (trying to iterate the "entire" sequence and never stopping if the element is not in it).

In **Ruby 3.3**, one more clarification was made: fully infinite range started to return `true` for linear objects:
```ruby
(nil..).include?(1)
# 3.3: => true
# 3.2: cannot determine inclusion in beginless/endless ranges (TypeError)
```
So it was deducing the range’s type by its begin/end and switching to a default behavior of “trying to iterate” as it wasn’t number/`Time`. Now it is considered that if the only “defined” value in this statement is number, then we are in a numbers (linear) space, where both `nil`s are representing infinities in this space.

By the way, before the introduction of beginless/endless ranges literals, it was common to use `Float::INFINITY`/`-Float::INFINITY` to designate a semi-endless range, but this, of course, worked only for numbers (and, accidentally, `Date`, because it is [historically comparable](https://docs.ruby-lang.org/en/3.3/Date.html#method-i-3C-3D-3E) with numbers, while `Time` is not)[^5].

[^5]: I wonder how the history might’ve gone if Ruby had some literal or at least a constant name for infinity, which would be more pleasant to type and read than `Float::INFINITY`. I remember one pre-Ruby 2.6 codebase where we just defined our own `INF = Float::INFINITY` because we had a ton of semi-infinite ranges in various patterns.

**That’s where the _modern_ history of range’s ends ends.**

But long before that, even before Rails 1.0 and the first edition of “Programming Ruby”, **Ruby 1.4** introduced ranges with _exclusive ends_:

```ruby
(1..5).cover?(5)  #=> true -- the range includes its end
(1...5).cover?(5) #=> false -- the range excludes its end
```

Only the first kind existed initially, unlike many other languages which have an exclusive-end range as their older and more basic form.

> Curiously, I can’t remember the idea of ranges that exclude their beginning to be proposed—maybe I am missing something, or maybe nobody was able to come up with good syntax or compelling use cases. And neither of other mainstream languages seem to have them.

That **1.4** release also introduced names/aliases `begin` and `end` for its boundaries. This change could be considered "prehistoric", but still interesting how the thought flew! The initial names of the boundaries were `first` and `last`, and they preserve this meaning as synonyms for `begin` and `end`, sometimes confusingly:

```ruby
r = (1...5)
r.last #=> 5, it would not be the last element of range as a sequence!
# ...and also inconsistent with "several last elements" call:
r.last(2) #=> [3, 4]

r = (1...1) # an empty range:
r.to_a #=> []
# still has "first" and "last"
r.first #=> 1
r.last #=> 1
```

There was [once an attempt](https://bugs.ruby-lang.org/issues/8739) to fix the inconsistency, at least for the first described case (exclusive range with integer bounds), but it exposed too much broken code/incompatibility, so it stayed that way.

To increase (or decrease!) confusion, the synonyms behavior isn’t maintained for beginless/endless ranges:
```ruby
r = (1..)
r.end #=> nil
r.last # cannot get the last element of endless range (RangeError)
```

> **How others do it:** Rust has all the _range_ of ranges that Ruby does: with only one boundary, inclusive and exclusive ends: `1..3` (exclusive), `1..=3` (inclusive), `..3`, `1..`, and even `..`; Kotlin and Scala have inclusive/exclusive pairs (`1..3` and `1..<3` in Kotlin, `1 to 3` and `1 until 3` in Scala), but no syntax/notion of ranges without a boundary. Python’s `range(1, 3)` is always exclusive, and no boundary can be omitted.

**And that’s what can be said about range ends! But... Not the end of the design space travel!**

## Range and iterations through the sequence

Usage of range for iteration is one of the most common (even Go, typically reluctant for this kind of abstraction, [has introduced](https://tip.golang.org/doc/go1.22) `range 10` recently—whilst before this change, `range` keyword was used to mean “range of keys in this collection”).

In Ruby, Range implements conventional `#each` method:

```ruby
(1..5).each { |v| puts v } # prints 1, 2, 3, 4, 5
```
Range includes [Enumerable](https://docs.ruby-lang.org/en/3.3/Enumerable.html) module, so all of its idioms are readily available:
```ruby
(1...4).to_a #=> [1, 2, 3]
('a'..'d').map(&:upcase) #=> ["A", "B", "C", "D"]
(Date.today..).find(&:monday?) #=> #<Date: 2024-07-29>
```

Actually, even the default implementation of `#include?` (when it is not specialized for “linear” values) is [provided](https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-include-3F) by `Enumerable`.

To be iterable, the range’s beginning value should implement only `#succ` method (returning the next successive value); internally, such types are called “discrete”. The type might be linear but not discrete (say, fractional numbers):
```ruby
(1.5..2.5).to_a
#=> `each': can't iterate from Float (TypeError)
```

In this case, we can use `#step` method to explain to Ruby how to iterate:
```ruby
(1.5..2.5).step(0.5).to_a #=> [1.5, 2.0, 2.5]
```

In the upcoming **Ruby 3.4** (hopefully: the change is [approved by Matz but not yet merged](https://bugs.ruby-lang.org/issues/18368)), I am trying to make `#step` more powerful for non-numeric values, so this would be possible:

```ruby
(Time.now..).step(1.hour).take(3)
#=> [2024-07-24 20:22:12, 2024-07-24 21:22:12, 2024-07-24 22:22:12]
```

...because when `#step` was introduced—much later than `Range`’s infancy, in **Ruby 1.8**, at the same time when `member?`/`include?` story started—it received two different implementations:

* for numbers, it works with `#+` (each next value is produced by `prev_value + step`);
* for everything else, it only accepts integers and means “just call `#succ` several times”:

```ruby
('a'..'z').step(2).take(3) #=> ["a", "c", "e"]
(Time.now..).step(1.hour) # can't iterate from Time (TypeError)
```

This seems less useful behavior and not consistent with how it is for numbers (which, for me, represents the “default” intuition), so I hope the change will happen!

This “numbers are special” is a repeated motive, as one could’ve probably noticed! Another example is `#reverse_each`’s specialized behavior, introduced as recently as **3.3**. The default reverse iteration method is provided by [Enumerable](https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-reverse_each), with the only way it is possible for a generic case: iterating through the entire sequence till the end, memoizing the result, and then iterating it backward. Obviously, for numbers it can be specialized to just use math—and even work with beginless ranges!
```ruby
(...5).reverse_each.take(3)
#=> [4, 3, 2]
```

It is not possible for any other type.

And another “numbers are (were) special” example, a somewhat comical one: the “second kind of `#step`” (that which just repeats `#succ`) was always raising on an attempt to use `0` step (which would just repeat the beginning value), while it was allowed for numbers:

```ruby
('a'..'c').step(0).take(3) # step can't be 0 (ArgumentError)
(1..5).step(0).take(3) #=> [1, 1, 1]
```

Unlike other cases, when the generic behavior was eventually made closer to the one numbers have, **Ruby 3.0** decided that `0` step isn’t semantically meaningful, and prohibited it from numbers, too:
```ruby
(1..5).step(0) # step can't be 0 (ArgumentError)
```
...even though there were some doubts about edge cases when the previous semantic might be useful.

But other than this curiosity, “steps over numbers” (and number ranges in general) remains the most powerful construct. Confirming this, Ruby **2.6** introduced a new type, [Enumerator::ArithmeticSequence](https://docs.ruby-lang.org/en/3.3/Enumerator/ArithmeticSequence.html), and an additional operator `%` to produce it:
```ruby
(0..10).step(2) #=> (1..10).step(2), an object of class ArithmeticSequence
# same, maybe more expressive in some contexts:
(0..10) % 2
# say, this:
array[(1..10) % 2] #=> each second element in the array

('a'..'z').step(2) #=> still just Enumerator
```
The ArithmeticSequence object can be used for iteration as a regular enumerator (which `step` returned before the change), it just exposes `#step` as an attribute, allowing to pass around `(begin, end, step)` set of values, and use it for, say, custom collection slicing. (The change was requested by the Scientific Ruby community for this purpose; only by Ruby **3.0** I’ve [pushed](https://bugs.ruby-lang.org/issues/16812) for adding this to the [standard Array](https://docs.ruby-lang.org/en/3.3/Array.html#method-i-5B-5D), too.)

> **How others do it:** All of the languages I am listing above that have ranges (Python, Rust, Kotlin, Scala), have `step` in them, too—though always only integer one—even languages that have not only integer ranges (Rust and Kotlin). E.g. in Kotlin `'a'..'z' step 2` is “each second item” in the sequence. Neither language even makes an exception for “float steps between numbers,” so the idea of step being “something else,” a custom iteration through the values space, seems less natural there.
>
> In Python, Kotlin, and Scala, `step` is an attribute of Range (so they are more like Ruby’s `ArithmeticSequence`), while in Rust [Range::step_by](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.step_by) is just a specification of a generic [Iterator::step_by](https://doc.rust-lang.org/std/iter/trait.Iterator.html#method.step_by)—and, consequently, can’t be used to slice arrays.

## ...and other usages?

The changes/questions above mostly cover the Range design, though there are two more areas of improvement/usage worth a brief mention for completeness.

One is math-like operations between ranges: in Ruby **2.6**, `Range#cover?` was changed to also accept another range (and check if the operand is fully inside), and in **3.3**, `#overlap?` method was added. One might imagine a lot of other “interval math” methods to be _theoretically useful_, yet as usual, the Ruby core team expects persuasive use cases and clear semantic definitions for those future possible methods (here is one [ongoing discussion](https://bugs.ruby-lang.org/issues/16757), not very active, though).

Another interesting topic is adopting Range for _other_ APIs where it is semantically sound. Besides `Comparable#clamp` already mentioned above, notable examples are:
* checks like `numbers.any?(3..5)` since Ruby **2.5** (not directly range-related, just methods [Enumerable#any?](https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-any-3F), [#all?](https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-all-3F), [#none?](https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-none-3F), [#one?](https://docs.ruby-lang.org/en/3.3/Enumerable.html#method-i-one-3F) started to accept `#===`-matching patterns);
* introduction in **1.9.3** `rand(begin..end)` API to generate a number in a given range;
* and representation of the standard library’s IPAddr as a range, which was introduced in **1.8** (apparently, a Big Version for ranges):
```ruby
IPAddr.new("192.168.0.0/16").to_range
#=> #<IPAddr: IPv4:192.168.0.0/255.255.0.0>..#<IPAddr: IPv4:192.168.255.255/255.255.0.0>
```

---

**This concludes the story of Range’s evolution.**

I hope I was able to share that feeling of a language being a living, breathing being, making its decisions and missteps, clarifying its behaviors, bearing the weight of legacies and habits, and still moving forward.

There hopefully would be more.

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now—including subscription options with secret posts! Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
