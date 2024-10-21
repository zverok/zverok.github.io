---
layout: post
title:  'There is no such thing as a global method (in Ruby)'
date:   2024-10-21
description: "What Ruby’s top-level methods actually are, who they belong to and how they are namespaced"
categories: ruby
comments: true
image: https://zverok.space/img/2024-10-21/code.png
---

**What Ruby’s top-level methods actually are, who they belong to and how they are namespaced.**

A few days ago, a curious question [was asked](https://www.reddit.com/r/ruby/comments/1g5ma34/question_about_kernelrand/) on /r/ruby, which can be boiled down to this: **How are the methods of the [Kernel](https://docs.ruby-lang.org/en/3.3/Kernel.html) module available in the top-level scope?**

The question was dedicated to `rand` method, but (as the author correctly suggests) it also applies to many seemingly “top-level” methods documented as belonging to the `Kernel` module, even as base as `puts` (print a string), `require` (load code from another file), or `raise` (an exception).

We know that in Ruby, all methods belong to some objects and are defined in their classes or modules. The documentation suggests that all of those “global” methods are coming from the `Kernel` module, yet you typically call them without referring to any module, object, or any other ceremonies (like loading some namespace or adding it to the current scope). So, **how to understand this working?**

```ruby
puts "Hello World"
```

## It is always a method of `self`

In Ruby (unlike most other OO languages), the bare lowercase identifier `foo` always refers to either a local variable or (if there is no such variable in the current scope) _the method of the current object_ (the one that `self` refers to in the current scope).

So,
```ruby
puts "Hello world"
```
is always[^1] the same as
```ruby
self.puts "Hello world"
```

[^1]: As we’ll see below, `puts` is a _private_ method, and in older versions of Ruby, you couldn’t call `self.private_method`, only as a bare word (as a part of the general rule “no explicit receiver for private methods”); since Ruby 2.7, the requirement was relaxed to allow explicit `self.private_method` (but only `self`), see the Ruby Changes [entry on it](https://rubyreferences.github.io/rubychanges/2.7.html#selfprivate_method).

What is the `self` in the top-level scope? It is a special object, which is called `main` (though you can’t access it by this identifier, so "main" is just a representation thing):
```ruby
self                 #=> main
self.class           #=> Object
self.class.ancestors #=> [Object, Kernel, BasicObject]
```

So, we can see that it is just an instance of the generic `Object`, available as a top-level scope, and as such, it includes the module `Kernel` and all methods from it, therefore making `puts` available:

```ruby
m = method(:puts)  #=> #<Method: Object(Kernel)#puts(*)>
m.owner            #=> Kernel -- who defined it
m.receiver         #=> main -- object which will receive the call
```

But... What’s the deal with this `Kernel` module anyway?

### Confusing legacy quirk: `Kernel` vs `Object`

_Ideologically_, it was intended for `Kernel` module to be a storage for those “global” methods, available everywhere. All of them are private[^2], i.e., available only to call on the current object from inside this object, thus making them look global:
```ruby
self.private_methods.include?(:puts) #=> true

ref = self
ref.puts "Something"
# NoMethodError: private method `puts' called for main:Object
```

[^2]: In Ruby, _unlike other languages_, `private` methods are available to the children of the class who defined them, and `protected` is used to mark what some other languages might call "friend": methods that are available to the current object and _other objects of the same class_.

At the same time, methods defined in a base [Object](https://docs.ruby-lang.org/en/3.3/Object.html) class, the common ancestor of all other classes[^3], are _public_ methods that are available on every object for other objects, like `#inspect`, `#to_s`, `#respond_to?(method)`, `#is_a?(some_class)`, and so on.

[^3]: In some contexts, one needs a very lean class without any extra conveniences defined, and for that, the class might be explicitly inherited from [BasicObject](https://docs.ruby-lang.org/en/3.3/BasicObject.html). But without this special approach, any class will inherit from `Object`.

But that’s how it was _meant to be_. In fact, _most_ of the above public methods are also defined in `Kernel`, which is easy to check:

```ruby
Object.instance_method(:is_a?)       #=> #<UnboundMethod: Kernel#is_a?(_)>
Object.instance_method(:is_a?).owner #=> Kernel
Object.instance_methods - Kernel.instance_methods
#=> [:!, :equal?, :__id__, :__send__, :==, :!=, :instance_eval, :instance_exec]
```
As you can see, a very small batch of what is “ideologically” public methods of every object are actually defined in `Object` class.

**But**, looking at the [`Object`’s docs](https://docs.ruby-lang.org/en/3.3/Object.html), you’ll see much more of them. This is literally a hack in RDoc (Ruby’s documentation system) to make it _look_ like it is meant to be.

**But**, this hack is old and unaware of core methods being defined with Ruby (not C) code, and some of the public (ideologically `Object`’s) methods “slip” back into `Kernel`, like [#then](https://docs.ruby-lang.org/en/3.3/Kernel.html#method-i-then) or even [#class](https://docs.ruby-lang.org/en/3.3/Kernel.html#method-i-class) (e.g. `obj.class`). There is a [long discussion](https://bugs.ruby-lang.org/issues/19304) about handling it in a saner way (which includes some explanation for how it happened), but it hasn’t moved much yet.

## When you `puts` inside an object

So, if `puts` is actually not a “global” method, but a private method which is included from `Kernel` in every object, then... When you call `puts` from inside some other class’ method, _whose_ `puts` is this?

Of the object that calls it!

```ruby
class A
  def test
    p method(:puts)          #=> #<Method: A(Kernel)#puts(*)>
    #                                      ^ who owns the method actually
    p method(:puts).receiver #=> #<A:0x00...>
  end
end

A.new.test
```

This is _different_ from most other languages (and, therefore, probably, from the default intuition of many developers), where “global” methods really do belong to some “global” scope and not to the current object.

This might be considered just an esoteric internal quirk, but understanding this fact may be useful sometimes. Say, in testing some Text UI class, one might want to write test code that checks that some class prints elements of the UI on a method call. (There are special [RSpec matchers](https://rspec.info/features/3-13/rspec-expectations/built-in-matchers/output/) to check that, but let’s use this example for simplicity; there are many other Kernel methods that one might want to stub/expect in tests):

```ruby
class MyUI
  def header
    puts "-----"
  end
end

RSpec.describe MyUI do
  let(:instance) { described_class.new }

  describe '#header' do
    it "outputs header (would fail)" do
      # No, that `puts` from inside the class wouldn’t call Kernel.puts
      expect(Kernel).to receive(:puts).with('-----')
      instance.header
    end

    it "outputs header (correct way)" do
      # because it owns its `puts`!
      expect(instance).to receive(:puts).with('-----')
      instance.header
    end
  end
end
```

<div class="one-ukrainian-thing" markdown="1">
  <h3>A postcard from 🇺🇦</h3>

  _**Please stop here for a moment.** This is your regular mid-text reminder that I am a living person from Ukraine, with the Russian invasion still ongoing. Please read it._

  **One news item.** There are reliable reports that soldiers from North Korea (as many as 12000) [are currently training in Russia to participate in the war in Ukraine](https://x.com/KhaterDiana/status/1847253931203670365).

  **One piece of context.** A year ago, on Oct 21, 2023, my best friend from the army training camp was killed in the front line. I wrote a bit about him in a mid-text “postcard” [here](https://zverok.space/blog/2023-11-03-syntax-sugar2-pattern-matching-fin.html). Here is a small [memorial Twitter thread](https://x.com/zverok/status/1848284613048729777) about him.

  **One fundraiser.** Please help [raise money](https://x.com/zverok/status/1845118020110139857) for a volunteer, Olena Samoilenko, who helps the elderly, disabled, and other disadvantaged inhabitants of the Kherson region.
</div>


## What about custom top-level methods?

So, if all that looks like “standard global methods” are actually methods of the current object (included from `Kernel`), what about _user-defined global methods_?

```ruby
def my_method
  puts "who am i? #{self}"
end
```

The situation is almost exactly the same: such methods become _private instance methods_ of the `Object` class, and as such, _they are available in every object_:

```ruby
method(:my_method) #=> #<Method: Object#my_method() test.rb:1>

my_method # called in the context of the main object
# prints: "who am i? main"

class A
  def test
    my_method
  end
end

a = A.new
a.method(:my_method) #=> #<Method: A(Object)#my_method() test.rb:1>
a.test # invokes my_method that belongs to A
# prints: who am i? #<A:0x0...>
```

So, once again: **all top-level methods are actually present in every object**.

This is a clear and consistent system… Which might sometimes lead to extremely weird quirks with the metaprogramming code, which checks for some method’s presence by name and changes the behavior depending on it.

We actually caught one just last week: deep in Rails insides, there was serialization code that relied on whether the current object `respond_to?(:avatar_url)`—and in some completely unrelated place, a helper module was included into the global scope, which made _all_ objects to have `avatar_url` method... But not the one that would be expected there in the serialization code. Extremely funny to debug!

The outtake is: _keep your top-level scope clean of random methods, especially those with generic names, and including those that might come from included modules._

> **How others do it:** To the best of my knowledge, there is no other languages (at least amongst relatively mainstream ones) with such an approach to “global” methods. Most of the OO languages either fully prohibit those (Java/C#, you only can have `SomeClass.static_method`); or there is no object context (no `this`/`self`) in those methods (Kotlin, Python, PHP), or the methods are truly global, and `this` in them consistently refers to some global object (JS with its `globalThis`, Scala). But I might miss something!

## Is the main scope special?

The "main scope is special, everything there goes directly into the `Object` class" is a good enough heuristic to remember, but one might be interested to observe that the top-level code behaves like _any method body_: as if it is all performed in a method of an instance of the `Object` class. In Ruby, you can have nested method definitions, but they don’t define local methods and instead go to the parent class:

```ruby
class A
  def outer
    # See, we are defining helper inner method!
    def inner(i) = print("iteration #{i}")

    5.times { inner(_1) }
  end
end

a = A.new
a.outer # prints "iteration 0", "iteration 1", etc.
a.inner(1000) # prints "iteration 1000" -- the method is now defined in a!
# ...and moreover
A.new.inner(2000) # prints "iteration 2000" -- it belongs to the A class
```

Like many things in Ruby, it is “unlike most of other languages, but self-consistent.”

With this knowledge, we can model `main` method and its definitions behavior this way:

```ruby
my_main = Object.new

def my_main.implicit_top_level
  # that's where all your top-level code go
  def other_method
    puts "OK!"
  end
end

my_main.implicit_top_level
# `my_main` is an instance of object,
# so `other_method` is now defined in object
# Let’s check:
Object.new.other_method
# prints "OK!"

# And as everything, including the top-level scope, inherits from Object,
# we have it here:
other_method
# prints "OK!"
```

Here is Ruby for you, keeping on its “slightly peculiar yet consistent” side for most of the time.

> This equivalence breaks when we talk about constants. All constants (including classes and modules) in the main scope are nested into the `Object`, too; but it wouldn’t work in our `main_scope_method`. So... It is a bit special thing, after all! Or, rather, it behaves consistently with a method body for everything other than constants; and consistently with class/module body for constants.

## Summarizing it

Just to reiterate:

* In Ruby, there is no such thing as a top-level/global method; the method without an explicit receiver (the core one or user-defined one) is always a **method of the current object**;
* Methods defined on the top level become **instance methods of every object**; modules `include`d into the top-level scope, are included into the `Object`;
* This distinction—that methods are never really “global”—can be mostly ignored, but it is useful to have an accurate mental model when using metaprogramming or debugging large codebases;
* Everything is inspectable and should be inspected;
* Many things are peculiar yet self-consistent.

Hope this helps :)

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
