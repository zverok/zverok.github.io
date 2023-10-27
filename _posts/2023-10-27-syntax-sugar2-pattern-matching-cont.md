---
layout: post
title:  '“Useless Ruby sugar”: Pattern matching (Pt. 2)'
date:   2023-10-27
description: '...or, “but mah polymorphism and encapsulation!”'
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2023-10-27/result-monad.png
---

> This is the second part of the article about pattern matching in Ruby; that article is itself a part of the series about recent language features, their design, and pragmatics. Please start (at least) from [the first part](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html); the series intro and table of contents are [here](https://zverok.space/blog/2023-10-02-syntax-sugar.html).

Last time, we discussed why there was a demand to introduce pattern matching in Ruby and how it was introduced. _As it is rather a big feature with a lot of details to handle, I wouldn't fully cover all those details in the blog post. If you are interested in the full feature definition, I suggest reading [the official docs](https://docs.ruby-lang.org/en/master/syntax/pattern_matching_rdoc.html); they are quite accessible[^1]._

[^1]: (Well, I [participated](https://github.com/ruby/ruby/pull/2786) [in writing](https://github.com/ruby/ruby/pull/2952) the initial version, so I might be not really objective here).

Now, let's discuss the pains and gains it have brought!

## Irks and Quirks

With all the good things that I've said in the previous article about chosen pattern syntax looking natural, it still stands as a separate syntactic area of the code. This is mostly unusual for Ruby, which, once you get to know it closer, is characterized by a uniformity of semantics.

A lot of things that in other languages are represented by separate grammar elements, in Ruby, are just methods calls on objects:

```ruby
class A
  # here just a regular code execution environment
  # where you can do anything, like...
  p self #=> prints "A", current object inside which we execute

  # It is just a `attr_reader()` method that receives `:a` argument,
  # and defines a getter, not a special language construct
  attr_reader :a

  # This is just private() method, which receives an argument with
  # `:foo` (name of the method defined), which `def` statement returns.
  private def foo
    # ...
  end

  # Class methods could are defined like this,
  # but that's not a special syntax, any `def obj.method` works,
  # and `self` is the class object here
  def self.bar
    # ...
  end
end

# Even object creation is not a special construct, but
# call to `.new` method of `A` object, which can be introspected,
# redefined, and removed.
a = A.new
```

In these circumstances, pattern syntax stands out: what you see in it is _like_ things matched, but not because it is the same grammatical entity: it is a completely different entity, making sense only in patterns context, but made to _look_ the same. Or, if you wish, patterns are **visually integrated** into the language but not fully **semantically integrated**.

This _separateness_ might manifest in patterns themselves when the check against non-literal objects "suddenly" requires pinning, as if the illusion of full integration is broken:

```ruby
# This (literal range) works:
number in 1..10

# This (almost the same?) is syntax error:
time in Time.new('2023-10-01')..Time.now

# Can be written like this, though:
time in ^(Time.new('2023-10-01')..Time.now)
```

Modern pattern matching also defies the intuition that Ruby users previously had about pattern-alike things (like regexps or ranges): that "pattern" is an object that can be put into a variable, sent to a method, etc.

```ruby
# I can do this:
# "old" `===` check to select collection items that are integer
arguments.grep(Integer)

# ...but not this:
# seems similar: select "pair of integers"?.. But wouldn't work
arguments.grep([Integer, Integer])

# The closest we can have is this:
arguments.select { _1 in [Integer, Integer] }
```

Consequently, there is a "pattern" noun in terminology but no `Pattern` class to extend and play with: say, define [custom unpacking strategies](https://bugs.ruby-lang.org/issues/18583). Again, this is _logically explainable_ (patterns might include references to local variables, and it would be just borderline impossible to implement a pattern object that wraps them), but it is still _intuitively irritating_.

On the other hand, "pattern as a statement" applicability is also quite limited. In functional languages where pattern matching **is** the only form of assignment, it works in "implicit assignment" contexts, too, like passing arguments to a method. Say, in Elixir,

```elixir
# If this works:
{x, y} = data

# ...this works too, by the same match/unpack logic
def m({x, y}) do
  # do something with x and y
end
```

In Ruby, `=>` might _look_ like a form of assignment, but there is no way to use it implicitly. In other words, there is no way to define a method or code block that declares to accept patterns.

Lets, for example, look at a common case of handling an array of uniform hashes or structs:

```ruby
events = [
  {type: 'create', role: 'admin', some: 'details'},
  {type: 'delete', role: 'user', buy: 'coffee'},
  {type: 'create', role: 'user', some: [:other, :stuff]},
]
```

You can do this (explicit assignment):

```ruby
events.first => type:, role:, **data
```

And even this (check type, then unpack role and the rest):

```ruby
events.first => type: 'create' | 'delete', role:, **data
```

But as it is not a real assignment, but a completely separate syntax, nothing "follows" from it. Like, there is no way to do this (implicit check/assignment of params)[^2]:
```ruby
# Not a real Ruby!
events.select { |type:, role:| type == 'create' || role == 'admin' }
```

[^2]: This simplest form—just unpacking keyword arguments—has worked without pattern matching before full keyword argument refactoring in Ruby 2.7 but was dropped to simplify the parser. I still mourn it a little.

The closest you can go is this, but having an **explicit** rightward assignment:

```ruby
events.select { _1 => type:, role:; type == 'create' || role == 'admin' }
```
...which is almost as compact by character count but has a different perception: two statements (which most linters would like you to write on separate lines), and the unpacking reads as a "work done in the block" instead of a "part of the _block declaration_."

---

This "shallow" integration and many ways it breaks intuitions in more complex usages (say, the pattern `Array[*Integer]`—"array of any size of integers"—looks _logically possible_, but it is a syntax error) is not a show-stopper, of course. It is just a bit of bitterness to recognize while embracing the style that pattern matching brings.

> **Note:** Those "irks and quirks" are analyzed from the perspective of a Ruby user who is given a new tool and tries to apply their intuitions to it. If we'll approach it from a perspective of somebody already familiar with pattern matching in functional languages, a common "irky" thing is seemingly "inverted" order of pattern and value: in most of them, it is `pattern = value`, more corresponding to "matching-as-assignment" logic. But all things considered, "rightward assignment" aligns in an interesting way with chained computations style, as shown in the [previous article](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html#how-it-arrived). On the other hand, a developer coming from the perspective of other pattern matching implementations, might find the "why is it not an object?" question irrelevant.

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please stop here for a moment.** Since this article, I intend to make small reminders right in the middle of the text about how we live now. I am a living person from a country at war. If you enjoy my writings about tech, please take a toll of knowing what happens here. It is not a news digest a situation briefing, but a small event of the week  that passed._

  Last Saturday, Russia bombed a post terminal in my home city, Kharkiv. Six post workers died, and seventeen others were injured.

  <blockquote class="twitter-tweet"><p lang="en" dir="ltr">Our colleagues spent their last seconds alive helping - sorting parcels with medicine and humanitarian aid for the civilian suffering from the war. The russia destroyed a civilian building and should be convicted of war crimes. Please, spread the word <a href="https://twitter.com/hashtag/RussiaIsATerroristState?src=hash&amp;ref_src=twsrc%5Etfw">#RussiaIsATerroristState</a> <a href="https://t.co/2GkIlE2FRp">pic.twitter.com/2GkIlE2FRp</a></p>&mdash; Нова пошта (@NP_official_ua) <a href="https://twitter.com/NP_official_ua/status/1716072586939973845?ref_src=twsrc%5Etfw">October 22, 2023</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

  Please proceed with the rest of the article.
</div>

## Consequences

There are many consequences of introducing a feature that big into a mature language—and most of them will probably manifest themselves slowly with further adoption of pattern matching (if it will happen).

But the most important thing here is making the **specific code style easier and more acceptable: the style where _data_ and _algorithm_ are separated**. (I know how it sounds in 2023, but stay with me for a minute!)

The classic Object Oriented Code (from the times when you were supposed to follow _exactly one clearly defined paradigm_) answered the problem of branching by data shape/content with polymorphism.

Like, if you have several `if event.type == ... process_that_type(event)`, you just wrap events into different classes and invoke `event.process()`, with every class encapsulating "how I should be processed." OO programmer who writes a big branchy `if` is frequently guilt-tripped into "Of course, it is dirty code, should be refactored into proper small classes with small methods as soon as we have time to handle all that technical debt!"

However, in our multi-paradigm and pragmatic times, the common wisdom seems to value both approaches: big classes with matching interfaces, polymorphically implementing same algorithms for different contexts (say, DB adapters) and small passive structs/objects, whose processing is chosen by external (to those objects) code.

Comparing those two ways of handling things:

```ruby
def handle_event_oo(event_data)
  # Choose the class dynamically
  event = Event::EVENT_TYPES[event_data[:type]].new(event_data)
  # Implementation is defined in a corresponding class
  event.process
end

def handle_event_case(event_data)
  case event_data[:type]
  when 'create'
    if event_data[:role] == 'admin'
      admin_create(event_data.except(:type, :role))
    else
      call_create(event_data.except(:type))
    end
  when 'update'
    call_update(event_data.except(:type))
  # ...
end
```

...the developer might choose one or another depending on the length and complexity of event processing code, desired layering architecture, and many other factors. When branches are reasonably small (like in the code  above), some 15 or 20-line `case` clearly providing a **full overview** of what can happen depending on the event contents might increase the reader's comprehension velocity magnificently.

And in that case, pattern matching is a welcome ability to make it even clearer, making "what kinds of events handling we are talking about" a flat, declarative list:

```ruby
case event_data
in type: 'create', role: 'admin', **data
  admin_create(data)
in type: 'create', **data
  call_create(data)
in type: 'update', **data
  call_update(data)
# ...
```

> This line of reasoning is not even about how we organize _branching_, and rather about "do we put data and algorithms to handle it in the same object." In this context, it is interesting to meditate on the co-existence of two patterns of object-relational mapping in the modern discourse: _Active Record_ with big classes of data and related methods, and _Repository_ fetching small, mostly-passive, context-appropriate structs.

Following this logic, introducing pattern matching in Ruby, which always was a pragmatic multi-paradigm language, seems reasonable enough, right?..

There is one small problem, though.

### Where's all that data?

Being a child of "OO-first" times and having extremely object-oriented Smalltalk as its main inspiration, Ruby has no concept of data at all, just opaque objects sending messages to each other. Ruby follows the terminology of "calling methods" as other mainstream languages do, but even "call method by name" is still `object.send(name)`, as a bow to that Smalltalk's terminology about "sending messages."

This is an important difference from the "neighbor" languages, even if it isn't that noticeable at first sight.

E.g., if in Python or JS code, you see `dog.name` and `dog.bark()`—this transparent `dog` object has attributes (core language concept) `name` and `bark`, the latter is also callable if you attach `()` to it.

In Ruby code, when you see `dog.name` and `dog.bark()`, it means the opaque `dog` object _responds to methods_ `name` an `bark`: both are just method calls—which in Ruby allows to omit `()`. Unlike Python or JS, you could also write `dog.name()` and `dog.bark` with exactly the same effect.

Even Ruby's [Struct](https://docs.ruby-lang.org/en/3.2/Struct.html) (and the recently added[^3] [Data](https://docs.ruby-lang.org/en/3.2/Data.html) with more narrow and strict semantics) are just shortcuts to create _objects_ with necessary _methods_ that would look like data structures of the desired shape (immutable in `Data`'s case).

[^3]: [By yours truly](https://github.com/ruby/ruby/pull/6353).

This Ruby's deep distinction from similar languages might not be that obvious to pragmatic Rubyists and might even be perceived as "fun/boring trivia," but it severely affects the language's perception of itself and the way new features can be integrated.

Getting back to pattern matching, Ruby's feature design allows to match not only bare data structures like dictionaries and arrays but (some) objects, too:

```ruby
class Point
  # skip
end

# Tries to match and unpack Point to pattern
Point.new(0, 0) => x:, y:
# this works, too (with explicit class check):
Point.new(0, 0) => Point(x:, y:)
```

But if all Ruby objects are opaque (and there is no such public concept as "what's this object's data," only internal representation), how would pattern matching know whether `Point` instance matches the pattern? With the help of the methods, of course!

Here is the minimal `Point` implementation which would work with pattern matching:

```ruby
class Point
  def initialize(x, y)
    @x = x
    @y = y
  end

  # That's the method pattern matching will call when it will see deconstruction
  # with keys, like `point => x:, y:`
  def deconstruct_keys(*) = {x: @x, y: @y}

  # And that's the method it will call to check for a positional deconstruction,
  # like `point => [x, y]`
  def deconstruct_keys = [@x, @y]
end
```

So, _the deconstruction-friendly representation of the object_ is fully object's author's responsibility: for custom classes, Ruby neither helps to define trivial implementation nor limits the non-trivial definitions.

There are default implementations, though, for [`Struct`](https://docs.ruby-lang.org/en/3.2/Struct.html#method-i-deconstruct_keys) and [`Data`](https://docs.ruby-lang.org/en/3.2/Data.html#method-i-deconstruct_keys)[^4], so the _actual_ minimal implementation of the `Point` is just this:

[^4]: But again, it just means that they define `deconstruct` / `deconstruct_keys` methods by default, not that they need some deep core support by the language.

```ruby
Point = Data.define(:x, :y)

Point.new(0, 0) => x:, y:
# x = 0, y = 0
```

> This, BTW, opens a lot of small yet important possibilities for borrowing approaches from functional programming, like tagged data and "Result" monad:

  ```ruby
  Success = Data.define(:value)
  Error = Data.define(:message)

  # The definitions above are enough to handle
  # some function's results this way:
  case my_cool_func(something)
  in Success(value)
    # work with bare `value`
  in Error(message)
    # work with bare error's `message`
  ```

> Note that an effect that in some other languages requires separate concepts like "enums" or "case classes," here achieved by very minimal and generic means.

But most importantly, it **shifts the perspective to (probably passive) _value objects_ that are matchable and _algorithms_ that work on them** (which also can be contained in classes, passed as stateful objects, and even use polymorphic implementations!).

Pretty far from "a soup of objects sending messages to each other,"— So I might imagine why some are angry. On the other hand, one might say that this is how we write the most maintainable code anyway.

---

**You know what?.. This is still not the end of it,** but the end of the text that can reasonably be written in a week by a person mostly working on it at night. So, it seems there would be a third part in roughly a week! Which will close this investigation with

* How others do it;
* Taking it further;
* Conclusions.

That's the price of taking a big and complicated feature and trying to cover it in the same way small evolutions of syntax are covered! (A person should have the strength to recognize mistakes but also should have the persistence to follow the promises, so the pattern matching part of the series _would_ be finished, and the series will continue further.)

You can subscribe to my [Substack](https://zverok.substack.com/) to not miss the next part, or follow me on [Twitter](https://twitter.com/zverok).

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

