---
layout: post
title:  "It is not what you expect, but it is what you want: how Data#initialize is designed"
date:   2023-01-03
description: "A description of a curios core class design decision made for happier coding"
categories: ruby new-features
comments: true
image: https://zverok.space/img/2023-01-03/code-sample.png
---

> I am Ukrainian Rubyist from Kharkiv. There is still war in my country: a full-scale invasion that Russia started on Feb 24, continuing its 8-year-long hybrid war. We still need your support. **Please [donate](https://war.ukraine.ua/donate/), and don't be silent!**

Currently, I am working on updating my [annotated Ruby evolution site](https://rubyreferences.github.io/rubychanges/) with everything from the recent 3.2 release (ETA this or next week). But while I was on it, one frequently discussed topic about a new core class urged me to write a small explanatory post.

## A curios core class design decision made for happier coding

`Data` is a new class introduced in Ruby 3.2 to define simple, immutable value objects with nice API.

It is defined a used like this:
```ruby
Point = Data.define(:x, :y)

p1 = Point.new(1, 2)       #=> #<data Point x=1, y=2>
# or...
p2 = Point.new(x: 1, y: 2) #=> #<data Point x=1, y=2>
```

Now, a quick pop quiz: looking at the examples above, what would you **expect** to be `Data#initialize` signature? The "knee-jerk reaction" answer would be something like this:
```ruby
def initialize(*args, **kwargs)
  # now decide is it args or kwargs passed, and set internal variables
end
```

This would be the wrong answer because the signature is actually this:
```ruby
def initialize(x:, y:)
end
```

You can check it by redefining `initialize` and seeing for yourself what arguments are passed there:
```ruby
Point = Data.define(:x, :y) do
  def initialize(...) # define it to accept any argument
    p(...) # print all of the arguments as is
    super
  end
end

p1 = Point.new(1, 2)        # prints {x: 1, y: 2}
p2 = Point.new(x: 1, y: 2)  # prints {x: 1, y: 2}
```
So, the positional arguments are converted to keyword ones _before_ passing to `#initialize`!

## Why?

This is somewhat unexpected (and was already several times reported as a bug in the official tracker!), but designed this way for a good reason.

A few constraints we considered while developing `Data` was:
1. It should be uniformly initialized by positional and keyword arguments;
2. All arguments should be mandatory by default;
3. It should be convenient to redefine `#initialize` to provide default values for some arguments or do preprocessing before storing them.

To make it less abstract, let's imagine we want those to work in our `Data`-derived `Point`:
```ruby
Point.new(1, 2)
#=> #<data Point x=1r, y=2r> -- input converted to rational numbers
Point.new(1)
#=> #<data Point x=1r, y=0r> -- default value for `y` is provided
```
...without breaking the "can be initialized by positional and keyword args" contract:
```ruby
Point.new(x: 1, y: 2) #=> #<data Point x=1r, y=2r>
Point.new(x: 1)       #=> #<data Point x=1r, y=0r>
```
How would you implement task 1 (default values) if `#initialize` would need to accept positional & keyword arguments?

It would be something like this:
```ruby
def initialize(*args, **kwargs)
  # reimplement checking that one, and only one of them should be provided
  raise ArgumentError unless args.empty? != kwargs.empty?
  if args.count == 1
    # provide a default for the second positional arg
    args << 0
  else
    # or provide a default for keyword arg
    kwargs[:y] = 0 unless kwargs.key?(:y)
  end
  super(*args, **kwargs)
end
```
It is so tedious (and the implementation is still too naive) that most of the time, you would probably say, "screw it" and only implement support for one type of arguments.

With the approach `Data#initialize` takes, the redefinition is easy:
```ruby
Point = Data.define(:x, :y) do
  # just accept them and pass further while providing the default
  def initialize(x:, y: 0) = super(x:, y:)
end
# Check it works
Point.new(1)           #=> #<data Point x=1, y=0>
Point.new(x: 1)        #=> #<data Point x=1, y=0>
Point.new(1, 2)        #=> #<data Point x=1, y=2>
Point.new(x: 1, y: 2)  #=> #<data Point x=1, y=2>
```

Same for argument conversion:
```ruby
Point = Data.define(:x, :y) do
  def initialize(x:, y:) = super(x: x.to_r, y: y.to_r)
end
# Both positional and keyword args work:
Point.new(1, 2)
# => #<data Point x=(1/1), y=(2/1)>
Point.new(x: 1.5, y: 2.5)
# => #<data Point x=(3/2), y=(5/2)>
```

That's what it all was about!

## How?

One of the confusions that emerge at this point is, "how is this possible?" The usual habit of Rubyists is that `SomeClass.new`'s behavior is fully defined by `SomeClass#initialize` method, or, in layman's terms, "`.new` just calls `#initialize`" (and the fact that they _look_ like two methods with different names is just "some internal quirk").

The thing I love about Ruby, though, is that its small number of core concepts interact consistently, in a predictable way. So, `.new` is just a method, which in a default implementation does something like this:
```ruby
class MyClass
  def self.new(...)
    obj = allocate      # make an unitialized instance of MyClass
    obj.initialize(...) # call initialize method, passing all arguments .new received
    obj                 # return allocated and initialized object
  end
end
```

But nothing says that `new` can't be smarter and preprocess arguments somehow before initializing the object! In fact, lot of core classes do that. Say, [Array.new](https://docs.ruby-lang.org/en/3.2/Array.html#method-c-new):
```ruby
Array.new(5) { _1**2 } #=> [0, 1, 4, 9, 16]
```
...however it is implemented, looks like a "generator" method that hardly just passes its arguments to `#initialize`, right?

So, if `Data.define(:x, :y)` would've been implemented as a simple Ruby class, it would do something like this:
```ruby
class SimplePoint
  def self.new(*args, **kwargs)
    raise ArgumentError unless args.empty? != kwargs.empty?
    kwargs = {x: args[0], y: args[1]} if !args.empty?
    res = allocate
    res.send(:initialize, **kwargs)
    res
    # The last three lines could be just "do like other classes do:"
    #   super(**kwargs)
  end

  def initialize(x:, y:)
    @x = x
    @y = y
  end
end
SimplePoint.new(1, 2)       #=> #<SimplePoint @x=1, @y=2>
SimplePoint.new(x: 1, y: 2) #=> #<SimplePoint @x=1, @y=2>
```
The good thing here is that complexity of unifying arguments in most cases is handled by the `Data.new` internals, and the only thing you need to remember is "`#initialize` always receives unified keywords"!

## A few quirks

I can honestly say that I am a bit proud with the design decision, but it doesn't come completely without drawbacks.

**You need to be mindful redefining `#initialize`**

This wouldn't work, and, what's worse, wouldn't work in a confusing way:
```ruby
Point = Data.define(:x, :y) do
  # I know I would always use only positional args, so let me
  # redefine initialize simply!
  def initialize(x, y)
    super(x.to_i, y.to_i)
  end
end
# Expectation:
Point.new(1, 2) # => works, converts args
Point.new(x: 1, y: 2) #=> probably ArgumentError, I don't care!

# Reality:
Point.new(1, 2)
# ArgumentError: wrong number of arguments (given 1, expected 2)
```
That's because `new` already converted `1, 2` to `x: 1, y: 2`, and tries to pass _that_ to `initialize`, which is not ready to accept that now!

Possible fixes:
```ruby
# If you don't really care about signature, just want to convert arguments,
# follow the example above:
Point = Data.define(:x, :y) do
  def initialize(x:, y:) = super(x: x.to_i, y: y.to_i)
end

Point.new(1, 2) # => #<data Point x=1, y=2>
Point.new(x: 1, y: 2) #=> #<data Point x=1, y=2>

# If you _do_ care & want to limit it to positional only:
# This kind of clumsy, but works:
Point = Data.define(:x, :y) do
  def self.new(x, y) = allocate.tap { _1.send(:initialize, x:, y:) }
end
Point.new(1, 2) #=> #<data Point x=1, y=2>
Point.new(x: 1, y: 2) #=> ArgumentError
```
The latter example, if met frequently, can be made less clumsy with a small change in approach:
```ruby
class Point < Data.define(:x, :y)
  # call the default implementation, it will manage!
  def self.new(x, y) = super
end

Point.new(1, 2) #=> #<data Point x=1, y=2>
Point.new(x: 1, y: 2)
# in `new': wrong number of arguments (given 1, expected 2) (ArgumentError)
```
Here, `Data.define` creates an _anonymous_ data class, and our `Point`, its descendant, can refer to the parent's (default for `Data`) implementation of `new` easily.

**You can pass extra keyword arguments for free**

`.new` doesn't cares how many keyword arguments you have passed. It just cares that everything is converted into keyword ones, and the `initialize` will handle the rest:
```ruby
Point = Data.define(:x, :y)
Point.new(x: 1, y: 2, scale: 2)
# in `initialize': unknown keyword: :scale (ArgumentError)

# As the exception was raised in #initialize,
# we can handle extra arguments there if we want:
Point = Data.define(:x, :y) do
  def initialize(x:, y:, scale: 1)
    super(x: x * scale, y: y * scale)
  end
end
Point.new(x: 1, y: 2, scale: 2) #=> #<data Point x=2, y=4>
```

**...but not positional arguments**

```ruby
Point = Data.define(:x, :y)
# This is not something that can be handled only be redefining #initialize:
Point.new(1, 2, 3)
# in `new': wrong number of arguments (given 3, expected 0..2) (ArgumentError)
```
Note that the exception message says `in 'new'`. That's where the attempt to convert everything to keyword arguments is made, and if there are too many positional arguments, the code in `new` can't guess what keys they should belong to. So, the only way to make it work is redefining `new` as in the examples above:
```ruby
Point = Data.define(:x, :y) do
  def self.new(x, y, scale) = allocate.tap { _1.initialize(x:, y:, scale:) }

  def initialize(x:, y:, scale: 1)
    super(x: x * scale, y: y * scale)
  end
end
```
Redefining it so that both `Point.new(x, y, scale)` and `Point.new(x: ..., y: ..., scale: ...)` work would be quite tedious, though!

## PS: `Struct` does this differently

One interesting thing to notice is that since Ruby 3.2, Struct also can be initialized by both keyword and positional argument (and the `keyword_init:` param is left for the cases you want to specify it should _only_ be one of the two):
```ruby
MutPoint = Struct.new(:x, :y)
MutPoint.new(1, 2) #=> #<struct MutPoint x=1, y=2>
MutPoint.new(x: 1, y: 2) #=> #<struct MutPoint x=1, y=2>
```

But the separation of responsibilities between `new` and `initialize` differs. We can check this by redefining `#initialize` to print all of the arguments:
```ruby
MutPoint = Struct.new(:x, :y) do
  def initialize(...) # define it to accept any argument
    p(...) # print all of the arguments as is
    super
  end
end

p1 = MutPoint.new(1, 2)        # prints 1, 2
p2 = MutPoint.new(x: 1, y: 2)  # prints {x: 1, y: 2}
```

This is the opposite design decision to `Data`s, with the opposite combination of pros and contras: less confusing the first time you see it, but more tedious to redefine `#initialize` while preserving the contract. It is kinda "just happened," but it has its good reasons!

**1. In Struct, we can post-process arguments after assignment**

The `Struct` instances are mutable, and no arguments are mandatory. So we can just handle everything later!

```ruby
MutPoint = Struct.new(:x, :y) do
  def initialize(...) # define it to accept any argument
    super
    self.y ||= 0
    self.x = x.to_i
    self.y = y.to_i
  end
end

MutPoint.new('1') #=> #<struct MutPoint x=1, y=0>
```
This solution wouldn't work with `Data`, as its arguments are mandatory, and there is no way to set attributes in an already initialized object.

**2. Any other solution for `Struct` would be backward incompatible**

There is a lot of code out there that doesn't care about keyword initialization and just relies on the old behavior, like...
```ruby
Word = Struct.new(:word, :sentence_id) do
  def initialize(word, sentence)
    super(word.strip, sentence.id)
  end
end
```
This code worked perfectly before 3.2, and it should continue to work (even if it will behave weirdly if somebody tries to use `Word.new` with keyword arguments).

But with `Data`, we could introduce a cleaner, more narrowly defined yet usable API from scratch.

And so we did.
