---
layout: post
title:  '“Useless Ruby sugar”: Argument forwarding'
date:   2023-11-24
description: "Or, how the feature that allows to write less names might make the code more explicit."
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2023-11-24/callable.png
---


> This is a part of a blog post series about "useless" (or: controversial) syntax elements that emerged in recent Ruby version. The goal of the series is not to defend (or criticize) the features, but to share a "thought framework" for analysis of their reasons, design, and effect the new syntax has on a code that uses it. See also [intro/ToC post](https://zverok.space/blog/2023-10-02-syntax-sugar.html).

Today's post covers the feature that, unlike most of the others in the series, caused very little pushback: a set of shortcuts for **argument forwarding**.

## What

Since [Ruby 2.7](https://rubyreferences.github.io/rubychanges/2.7.html#keyword-argument-related-changes), this is possible (please note that `...` in the code below is the exact valid syntax, not "code omitted for a blog post"):

```ruby
def foo(...)
  # bar receives all positional, named and block arguments
  # that foo received
  bar(...)
end
```

Since [Ruby 3.1](https://rubyreferences.github.io/rubychanges/3.1.html#anonymous-block-argument), an anonymous block argument can be "forwarded" separately:

```ruby
def iterate_through_data(&)
  data.each(&)
end
```

Since [Ruby 3.2](https://rubyreferences.github.io/rubychanges/3.2.html#anonymous-arguments-passing-improvements), one can also "forward" positional and named (keyword) arguments separately:

```ruby
def split_arguments(*, **)
  pass_positional(*) # passes all positional arguments
  pass_keywords(**) # passes all keyword arguments
end

split_arguments(1, 2, 3, a: 4, b: 5)
# after this call, those methods would be called:
#   pass_positional(1, 2, 3)
#   pass_keywords(a: 4, b: 5)
```

## Why & How

...are inseparable here!

The introduction of `...` was not something discussed for years (at least, I am unaware of such discussions). It was rather an impromptu invention on the wave of the Big Changes in Ruby 2.7.

Version 2.7 was the last preparatory version before 3.0, and it introduced some of the "big oh" changes so everybody had time to prepare. One of such changes was a _final separation_ of positional and keyword arguments: making some rules of treating arguments stricter and less cumbersome than they historically were.

This is a complicated topic, well covered by the [official explanation](https://www.ruby-lang.org/en/news/2019/12/12/separation-of-positional-and-keyword-arguments-in-ruby-3-0/). In the context of the shorthand invention, we need to know that one of the side effects of the separation was that writing a method that just passes all of its arguments to another method became complicated. The pre-2.7 way of doing so was this:

```ruby
def reader(name, mode:)
  # does some reading
end

def writer(name, content)
  # does some writing
end

def wrap(method, *args)
  # a typical wrapper method, which might, say, log execution,
  # catch extra errors, do a more complicated dispatching and so on.
  send(method, *args)
end

wrap(:reader, "file.txt", mode: 'wb')
wrap(:writer, "file.txt", 'some content')
```

`*args` in the method signature and the method call was enough to pass all positional and keyword arguments around. In Ruby 2.7, the separation would become more formal[^1], so to handle every kind of method, one needed to write:

[^1]: Again, the official explanation gives all the details. Here it is enough to say that the separation was necessary to solve many confusing edge cases that bit people in random body parts at random times.

```ruby
def wrap(method, *args, **kwargs)
  send(method, *args, **kwargs)
end
```

In a generic situation, the third type of the argument should've been considered: a block (Ruby's special "tail lambda"). So for a truly universal delegator, one needed to write this:

```ruby
def wrap(method, *args, **kwargs, &block)
  send(method, *args, **kwargs, &block)
end
```

That's a whole lot of syntax _and_ naming to do a thing that is so simple by its idea!

So the practical solution for a shortcut "just pass whatever arguments you have" was [proposed](https://bugs.ruby-lang.org/issues/16253) and quickly accepted in the form of `...`.

A few technicalities and edge cases were discussed and settled, some immediately, some in the next version. Say, [support for passing a leading argument before forwarding syntax](https://rubyreferences.github.io/rubychanges/3.0.html#arguments-forwarding--supports-leading-arguments) was introduced in 3.0 but considered so useful it was then backported to the 2.7 branch and has been available there since around 2.7.3.

It was kind of a big deal because "delegate everything" is a very widespread phraseology in Ruby, used for all kinds of effects:

```ruby
# hide the real call sequence complexity:
def log(text, level:, **payload)
  Loggers::Registry.fetch(:http_logger).log(text, level: level, **payload)
end

# dynamically choose an implementation
def make_event(type, sender, **details)
  EVENT_CLASSES[type].new(sender, **details)
end

# make a nice DSL with dynamically defined methods:
class HTML
  def method_missing(name, content, **attributes)
    # construct tag string from tag name, content and attrs
  end
end

HTML.new.a('Ruby', href: 'https://ruby-lang.org')
#=> "<a href='https://ruby-lang.org'>Ruby</a>"
```

All of those cases can be made much shorter with `...`, without losing the _real_ meaning of "just pass everything"!

This syntax change was a rare case when many groups with frequently conflicting views directly saw an immediate gain:

* those who are usually curious about syntax changes and shortcuts found it pretty;
* those more cautious and frequently asking "what's the use case", in this case, immediately knew plenty of them (especially considering that after ruby 2.7, a lot of delegating code should've been rewritten anyway, either to `...` or to `*args, **kwargs`, so making peace with a shortcut seemed acceptable);
* finally, those with an emphasis on language as a pragmatic engineering tool saw a gain of the improved performance.

The latter has a simple explanation: what you don't name, you don't need to put in the object. E.g., this allocates an intermediate array and hash to put positional and keyword arguments into:
```ruby
def delegator(*args, **kwargs)
  # `args` is Array, and `kwargs` is Hash here
  # ...but we needed them only to immediately unpack
  delegatee(*args, **kwargs)
end
```

While this code doesn't make additional local variables available, and therefore no need to allocate an array/hash:
```ruby
def delegator(...)
  # no extra local vars here
  delegatee(...)
end
```

So, it wasn't a surprise or a scandal when, a couple of versions later, separate shortcuts to pass only positional and only keyword args were proposed.

Moreover, the change was small; those signatures were already valid syntax:
```ruby
def ignore_my_args(*)
end

def ignore_keyword_args(some, positional, **)
end
```
...to say, "the method accepts any numbers of positional or keyword args (maybe for compatibility with the same method in neighbor classes), but ignores them." So the only change [in Ruby 3.2](https://rubyreferences.github.io/rubychanges/3.2.html#anonymous-arguments-passing-improvements) was to additionally allow to say "...and passes them further":

```ruby
def pass_my_args(*)
  other_method(*)
end

def pass_keyword_args(some, positional, **)
  other_method(**)
end
```

Considering the intuitive feeling of "no new syntax" and that `...` was already there, and with the same "no unnecessary allocations" argument, the change was quickly accepted. The fact that it was [proposed](https://bugs.ruby-lang.org/issues/18351) (and the high-quality implementation supplied) by Jeremy Evans, author of prominent libraries like Sequel and Roda and a member of Ruby core helped, too.

Interestingly enough, the new syntax is acceptable not only for delegation to another method but almost everywhere where unpacking of named variables was supported:

```ruby
def with_anonymous_args(*, **)
  ary = [*]
  hash = {**}
  p ary, hash
end

with_anonymous_args(1, 2, 3, a: 4, b: 5)
# Prints:
#   [1, 2, 3]
#   {:a=>4, :b=>5}
```

This can be considered more of a curiosity (at least the "don't instantiate an array/hash" gain is lost here), but might be at least useful for temporary debugging statements in the pass-everything methods:
```ruby
def make_event(type, **)
  puts "DEBUG!" if {**}.key?(:password)  # temp
  EVENT_CLASSES[type].new(**)
end
```

Or, as the slightly-over-the-top example at the end of the "[Pattern Matching / Taking it further](https://zverok.space/blog/2023-11-03-syntax-sugar2-pattern-matching-fin.html#taking-it-further)" shows, the "we don't care about the name" can be repurposed to further pattern match the argument list as several possible signatures with different meanings (and, therefore, names) for parts of the argument list.

---

The story with `&` for anonymous block forwarding is a bit more complicated.

Unlike `*` or `**`, there was no `&` for "just ignore this block" because block arguments in Ruby are always optional, and there is no way neither to demonstrate in the method signature "we require it" nor that we don't. (It is sometimes a problem when blocks are erroneously passed—and ignored—to methods that never expected them, but it is quite [hard](https://bugs.ruby-lang.org/issues/19979) [to solve](https://bugs.ruby-lang.org/issues/15554).)

So, when the standalone forwarding with `&` [was proposed](https://bugs.ruby-lang.org/issues/11256)—long before the 2.7's "argument forwarding" work—mainly as an optimization for block allocation, it was met with great caution. At that time, the optimization part was [implemented](https://bugs.ruby-lang.org/issues/14045) on its own as just optimization of passing the block around even if it was named. Later, though, when the basic argument forwarding with `...` was already in the language, the six-year-long discussion about the acceptability and readability of `&` was ended with its introduction in [Ruby 3.1](https://rubyreferences.github.io/rubychanges/3.1.html#anonymous-block-argument).

That's the same effect we saw [while discussing](https://zverok.space/blog/2023-11-10-syntax-sugar3-hash-values-omission.html#how) keyword argument omission (which became acceptable after we got used to other cases of value-less `key:` syntax). Once a "bigger" feature takes its mindshare, the smaller ones might follow more easily.

## Irks and quirks

A small irk around ellipsis-based delegation is related to parentheses. The nature of the problem is similar to what we saw in the keyword omission case:

```ruby
def my_method(...)
  p ...
end
```
Is actually
```ruby
def my_meth(...)
  (p()...) # empty method call + a range from its result to infinity
end
```

This affects only ellipsis (not other forms of anonymous forwarding) and is remedied, as usual, by adding parentheses:

```ruby
def my_method(...)
  p(...)
end
```

The worse problem is that anonymous forwarding is not supported in blocks/procs. This is especially confusing considering that an old syntax of "anonymous splat" _is_ supported, and therefore one might potentially write code with a very confusing effect:

```ruby
def process(*)
  # ...a lot of code...
  ['test', 'me'].each { |*| puts(*) }
end

process('input')
```

This looks as if it will print "test" and "me" (inside of proc, `*` accepts its arguments and passes them to `puts`), but _actually_, it prints:
```
input
input
```

What happens here is:

* `each { |*|` is treated in old logic "accept all arguments and discard them;"
* `puts(*)` is treated in the new logic "see if the context has anonymous forwardable arguments"—and consider _the method's arguments_ as such.

This is an [open discussion](https://bugs.ruby-lang.org/issues/19370) on the matter, with a confusing (for me, at least) outcome: the Ruby developers' meeting seems to be leaning toward an idea of just prohibiting the case like above (`*`-forwarding inside of a block with `*` arguments) while allowing more unambiguous cases.

## Consequences

What happens on a (mindful) usage of argument forwarding shortcuts is the **onset of explicitness**.

This might sound confusing because the "explicitness" is frequently associated with adding more names to the code or more steps to construct the value. Like splitting the formula into a few named local variables or, instead of passing the result of some method to another, first attaching it to some name. (In pathological cases, it is "every non-trivial call/check/calculation should be its own method with the name explaining its usage.")

But here, I am talking about _the explicitness of the intention_ of some sizeable chunk of code, a "page" or a "chapter" of it. (In the same way in the intro article [I underlined](https://zverok.space/blog/2023-10-02-syntax-sugar.html#how-i-think-about-the-programming-language) we'll be talking about the reader's comfort in comprehension of the narrative instead of the "readability" of a single line.)

Imagine a code like this:

```ruby
def event(type, sender:, content:, details:)
  EventBus::Registry.instance.push_event(type, sender: sender, content: content, details: details)
end
```
Such "intermediaries" are frequent in layered systems: the params are already checked and defaults assigned by the layer above; the handling itself would be performed by the layer below; and this current method is just a shortcut in the current module (that probably invokes it many times, so repeating the verbose call of the underlying layer is tiresome).

There are several ways to make this definition shorter, like using [values omission](https://zverok.space/blog/2023-11-10-syntax-sugar3-hash-values-omission.html)
```ruby
def event(type, sender:, content:, details:)
  EventBus::Registry.instance.push_event(type, sender:, content:, details:)
end
```
...or "keyword-rest" splatting:
```ruby
def event(type, **event_data)
  EventBus::Registry.instance.push_event(type, **event_data)
end
```

But the real intention of this method is just to "pass everything further," what the author thinks about its argument names is closer to "like, _you know,_ everything" or "yeah, _whatever,_ just pass it" (and frequently would just call the rest arguments `**kwrest`, or, `**options`—the latter is a long yet frequently misleading tradition of naming the last hash/keyword argument).

So, eventually, we can be just **explicit** in expressing these "you know" or "whatever":

```ruby
def event(...)
  EventBus::Registry.instance.push_event(...)
end
```
Or, with one-line method definitions (a topic of the next article), just
```ruby
def event(...) = EventBus::Registry.instance.push_event(...)
```

Such simplifications might appear in various stages of design. Sometimes, "just pass everything through" is an early-prototype version that allows to quickly assemble the reasonable stack of layers; and later clarify on each step what are the particular responsibilities besides pass-through.

Other times, after a long period of design and clarification, it becomes obvious that a bit of meaningful realignment of layers with each other's capabilities and expectations allows dripping the trivial things in favor of the literal embodiment of "you know",  `...`. The intention to do so sometimes would uncover an unjustified signature change through the layers and, as a _consequence_, might lead to useful cleanups.

In any case, an ability to designate "what's obvious/doesn't matter here" allows the reader to focus better on the other parts: those that _are_ non-trivial and important. Or: if everything is important (and underlined by language means like long explanatory names), then nothing is.

That's why I am talking about explicitness: akin to the case of [numbered block parameters](https://zverok.space/blog/2023-10-11-syntax-sugar1-numeric-block-args.html), sometimes giving a name to a value is just **pretending** to explain something. In these cases, it is good to have a syntax that allows to be **clear and explicit** about "nothing more to explain here."

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please stop here for a moment.** This is your weekly reminder that I am a living person from Ukraine, with some random fact or event from our last week._

  A few days ago, there was an "anniversary" of sorts: my home region have seen its 3000s air raid alert since the beginning of the full-scale invasion. (And it is already more than 100 days by the summary length of the alerts.)

  ![](/img/2023-11-24/alerts.png)

  Please proceed with the rest of the article.
</div>

## How others do it

Like last time (with "value omission syntax"), I actually struggled to find the _exact_ correspondence of the "pass every argument" syntax: partially, maybe because there is no other languages to arrive to `(*args, **kwargs, &block)` as the shortest _normative_ way to say "everything that can be passed to a function."

The design space here is related to accepting to the function a variable number of arguments—the concept is frequently called "[varargs](https://en.wikipedia.org/wiki/Varargs)" (frequently demonstrated by formatted-printing functions such as `printf`). It seems that languages that have them (some, like Rust, consciously do not; Zig had them in the early versions but then decided [against](https://github.com/ziglang/zig/issues/208#issuecomment-393777148)), there are a few groups:

**Group 1, "The old school":** make a declaration "variable number of arguments is accepted here" in the function signature, and provide some special name (of variable/function/macro) to access them. Say, in C:

```c
int sum(int count, ...) {
  va_list args;
  va_start(args, count);
  // handle `count` of `args`, calling `va_arg(args, int);` each time
  va_end(args);
}
```

**Group 2, "The old school, but dynamic":** in the old JS and old PHP any function, regardless of its signature, could accept[^2] any number of arguments and exposed them with `arguments` (JS) or `func_get_args()` (PHP):

```js
function variadic() { console.log(arguments) }

variadic(1, 2, 3, {foo: 'bar'})
// Prints
//   Arguments(4) [1, 2, 3, {foo: 'bar'}]
```

[^2]: This still works but is mostly frowned upon in favor of newer syntaxes.

In a radical variant of this, Perl's `sub` doesn't have a syntax for arguments declaration at all[^3], and all arguments are accessible inside the subroutine in a list variable `@_`, and the way to designate their names is to assign them to local variables:
```perl
sub test {
  my($x, $y) = @_;
  print "x=$x y=$y\n"
}

test(1, 2)
# prints x=1, y=2
```

[^3]: As [pointed](https://news.ycombinator.com/item?id=38406739) by _chrismorgan_ on HN, it [does](https://perldoc.perl.org/perlsub#Signatures), since version 5.20.0 (May 2014), and they’re considered stable since 5.36.0 (May 2022).

**Group 3, "New school":** Many languages have a special syntax for a function signature to declare a named parameter that would "catch" the variable list of parameters. Like in modern JS:
```js
function new_variadic(...myargs) { console.log(myargs) }

new_variadic(1, 2, 3, {foo: 'bar'})
// Prints
//   [1, 2, 3, {foo: 'bar'}] -- note no special "Arguments" object wrapper
```

The designation used is frequently `...` or `*`, though C# uses [`param`](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/method-parameters#params-modifier) keyword for such cases, and Kotlin uses [`vararg`](https://kotlin-quick-reference.com/130-R-vararg-parameters.html).

It seems to be a generally agreed-upon practice nowadays.

The symmetric question is also more or less agreed upon[^4]: if you have a list/array/tuple of values and want to pass them as separate arguments to some function, there is an operator for that (usually looking the same as "rest arguments" declaration, and frequently called "splat" or "spread"):

[^4]: At least in dynamic languages. The situation in static ones and their challenges produces quite interesting design space with many nuances and can be a topic for a separate article.

```js
function function_with_3_args(arg1, arg2, arg3) {
  console.log({arg1, arg2, arg3})
}
args_in_array = [1, 2, 3]
function_with_3_args(...args_in_array)
// Prints: {arg1: 1, arg2: 2, arg3: 3}
```
...which provides a way to perform pass-through (accept in one function "whatever arguments passed" and pass them to another function).

It wasn't a given in old times! In `arguments` days of JS, "pass all arguments further" was as cumbersome as:
```js
function b(){
    console.log(arguments); //arguments[0] = 1, etc
}
function a(){
    b.apply(null, arguments); // pass them through, first `null` is for `this`
}
a(1,2,3);
```

So today's situation (with, usually, just one named "splat" argument to be "splat" further down the layers) is enough for most pass-through situations.

Though, wait!

The interesting thing happened to Lua (as I discovered while writing the article).

As of [version 5.0](https://www.lua.org/manual/5.0/manual.html#2.5.8), it belonged to "group 1": `...` in signature to designate "accepts any number of arguments," a special variable `arg` with a table of all arguments inside of a function.

```lua
function f(...)
  print(arg)
end

f(1, 2, 3)
-- prints  1  2  3
```

And yet, in [Lua 5.1](https://www.lua.org/versions.html#5.1), `...` [replaced](https://www.lua.org/manual/5.1/manual.html#2.5.9) the `arg`, so now it looks the closest to Ruby's shortcuts:

```lua
function f(...)
  print(...)
end

f(1, 2, 3)
-- prints  1  2  3

-- even can be used as a simple variable:
function f(...)
  print(...+5)
end

f(1) -- prints 6
```

The world of programming languages evolution is full of wonders!

---

An aside note: the temptation to use the `...` as a syntax construct along the lines of "and so on" is tempting not only for Ruby and Lua: it is present at least in Python, where bare `...` produces an object `Ellipsis`, which can be passed around as a regular value and has various uses:
```py
def method():
  ... # do-nothing method

# specify type of fun as a callable with any input arguments and str result
fun: Callable[..., str]

matrix = np.matrix([[1, 2], [3, 4]])
# take all data by all dimensions, but only 0th column
matrix[..., 0] #=> matrix([[1], [3]])
```

## Taking it further

There is one idea that is unlikely to be implemented yet still tempting: can we sometimes shift the place where "well, you know" can be said?

Imagine this code: a frequent idiom for a "callable class" (which might encapsulate a complicated multi-step algorithm that is split into multiple private methods):

```ruby
class MyOperation
  def initialize(some, arguments, of:, various: "kinds")
    # arguments assignment
  end

  def call
    # ... implementation ...
  end

  # public interface:
  def self.call(...) = new(...).call
end
```
Its intended usage is simple:
```ruby
MyOperation.call(with, some, of: :arguments)
```

...which just creates an instance with all arguments passed and immediately invokes its `#call` (the meat of the implementation) method.

The problem here is that the only public method of this class doesn't give any information in its signature: neither to render into autogenerated documentation nor to extract via introspection:
```ruby
m = MyOperation.method(:call)
#=> #<Method: MyOperation.call(...)> -- not informative!
m.parameters
#=> [[:rest, :*], [:keyrest, :**], [:block, :&]] -- not informative either!
```

The "callable wrapper" is not the only situation demonstrating this problem: a typical HTTP client implementation might have something like this (if it is written in a modern Ruby):
```ruby
def get(...)
  make_request(method: :get, ...)
end
```

Here, again, "public interface" method `get` doesn't demonstrate any information about its signature (which "private implementation" method `make_request` knows).

So, maybe it would be possible to still support the "pass everything" syntax in the presence of explicit arguments declaration? As a compromise between "say it the shortest way possible" and "spell everything like a beginner exercise":

```ruby
def get(endpoint, params: {}, headers: {}, redirect: false) # spell it here
  make_request(method: :get, ...) # "just pass it further" here
end
```

The ticket, describing the idea, [exists](https://bugs.ruby-lang.org/issues/16296), but it never received much attention.

BTW, there is place in Ruby where something like this works! Calling `super` ("parent class' version of this method") implicitly passes all arguments of the current method[^5], which allows for some very nice shortcuts. Say, here is everything you need to do to add some defaults to a `Data` initialization:

```ruby
class Measurement < Data.define(:amount, :unit)
  def initialize(amount:, unit: '-none-') = super
end

Measurement.new(100, 'm') #=> #<data Measurement amount=100, unit="m">
Measurement.new(100)      #=> #<data Measurement amount=100, unit="-none-">
```

So, the idea of "accept params declared explicitly, then just pass everything" is not _unimaginable_, at least!

[^5]: Actually, all current values of local variables with names of the arguments of the current method... Which have some interesting consequences, but let's not allow ourselves to be carried away.

## Conclusions

So, here are some things to round up today's entry:

1. **Not all things should be named**—we already talked about this while discussing numbered block parameters.
2. **Being explicit about the absence of additional meaning** is a useful kind of explicitness that should be considered to underline the meaningfulness of important parts.
3. **Sometimes, performance optimization and clarity optimization align like stars**, producing a feature that (while _still_ meeting its haters) might be explained and defended from several points of view at once!

The next part will be dedicated to the **one-line (endless) methods**, the syntax feature that was born out of April Fools' joke.

You can subscribe to my [Substack](https://zverok.substack.com/) to not miss it, or follow me on [Twitter](https://twitter.com/zverok).

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

