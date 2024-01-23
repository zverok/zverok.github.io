---
layout: post
title:  '“Useless Ruby sugar”: Endless (one-line) methods'
date:   2023-12-01
description: "Or, about the virtue of exactly one phrase."
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2023-12-01/code.png
---

> This is a part of a blog post series about "useless" (or: controversial) syntax elements that emerged in recent Ruby version. The goal of the series is not to defend (or criticize) the features, but to share a "thought framework" for analysis of their reasons, design, and effect the new syntax has on a code that uses it. See also [intro/ToC post](https://zverok.space/blog/2023-10-02-syntax-sugar.html).

Today's post covers the feature that was one of the most divisive in the community (sometimes even more so than [numbered block parameters](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html)): **one-line method definitions**.

## What

The usual Ruby method is defined like this:
```ruby
def my_method(args)
  body
end
```

Since [Ruby 3.0](https://rubyreferences.github.io/rubychanges/3.0.html#endless-method-definition), this alternative syntax is also allowed for methods consisting of exactly one statement:

```ruby
def my_method(args) = body
```

## Why

Ruby is one of the rare mainstream languages that doesn't have C-like `{}` as its base code blocks wrapping punctuation. It also doesn't use  significant whitespaces to designate where the code block ends, unlike Haskell or Python. (Almost) every construct ends with `end`, like in Pascal or Lua.

```ruby
if condition
  # ...body...
end

items.each do |item|
  # ...body...
end

class C
  # ...

  def m
    # ...
  end
end
```

This is mostly OK to type, and a modern IDE might do that for you, but when the body and the header of the code block are tiny, the syntax might feel bulky ("feel bulky" is very imprecise here, but we'll get to more concrete reasons soon).

Most of the code constructs have more compact versions, though—and not just mechanically compact, but expressing small and simple things a bit differently:

```ruby
return [] if denied?

items.map { |item| process(item) }

# Produce a body-less class, just to designate a new type of exception
MyError = Class.new(NetworkError)
```

...but not methods! There is only one way to write them.

One _could've_ forced methods to fit into one line using `;`:

```ruby
def my_method(args); body; end
```

However, the views of the Ruby community developed in the way that using `;` is deemed bad taste: it is a sign that you are cramming too much—several logical **phrases**—into one line[^1].

> **UPD:** As it was pointed by several people in comments, this actually works without `;`:
>
> ```ruby
> def my_method(args) body end
> ```
>
> What can I say! Once in a while, I forget how to Ruby :)
>
> There is a reason for that, though (for me forgetting it is possible): **Yes**, many Ruby constructs allow to be flattened this way, but **no**, nobody writes code this way: an unclear transition from header to the body, and, most visibly, the dangling `end` makes the structure of the phrase messy. So while the syntax is _theoretically_ available, it is _practically_ unused. But it probably should've centered the original explanation around the community's view at `;`.

[^1]: The syntax is helpful, though, when Ruby is used by its old vocation: as a scripting language to be invoked from a console, write one-time quick scripts, and fast, focused experiments.

Many languages were forced to invent shortcuts for one-expression functions when function iteration became mainstream, going, in JS's case, from `function(arg) { return val }` to `arg => val`. But Ruby already had code blocks for that, so no evolution for methods syntax was necessary[^2].

[^2]: Standalone lambdas—which in Ruby aren't directly related to methods—were following the common trend and changed from `lambda { |arg| body }` to `->(arg) { body }`.

***

"But why would a _small syntax non-optimality_ matter?"—one might ask. (And, depending on the mood, mention code golf as a main association for the "whether it can be put in one line" question.)

Throughout this series, I talk a lot about the comfort of a reader and the perception of the code as a continuous narrative. In this context, "how much of it fits in one page" matters. This doesn't mean that cramming everything into tight subsequent paragraphs, like a serious book, is a good idea: code isn't supposed to be primarily read paragraph-by-paragraph.

On the other hand, a two-words-per-line, twenty-words-per-page nursery rhyme-like layout means that one might need to scroll through dozens of pages to get "what's this all about."

I imagine a good code layout somewhat like an entertainment printed magazine: reasonably short articles, a lot of breathing space, pull quotes, lists, and schemas to emphasize and draw attention to various parts, removing small details to footnotes, and so on. (Of course, our layouting tools are different, but the effects to achieve are frequently the same.)

But the only pre-Ruby 3.0 syntax for methods turned many of them into nursery-rhyme style text:

```ruby
# A small value object encapsulating a word in text-processing algorithm:
class Word
  attr_reader :text

  def initialize(text)
    @text = text
  end

  def inspect
    "#<#{self.class} #{text}>"
  end

  def ==(other)
    other.is_a?(Word) && text == other.text
  end

  def <=>(other)
    text <=> other.text if other.is_a?(Word)
  end

  def punctuation?
    text.match?(/^[[:punct:]]+$/)
  end

  def capitalized?
  # ...and so on, I just started!
```

It might be a "value object" fully consisting of such small methods, like shown above[^3], or a few small methods in a larger object (`#inspect`, trivial predicates, `#to_h`, this kind of stuff), the problem stays the same: a page or several pages of context might be easily eaten by code with saying "Hello, my name is Jane"-level things.

[^3]: Yes, inheriting from `Struct` or `Data` would make _some_ of these methods unnecessary, but that's not the point.

An unspoken consequence of this situation is that people start to avoid "adding unnecessary (but useful!) stuff" like convenience methods or whole convenience objects because it was "just that small thing" in your head and two pages of code in reality.

So... Can those small helper methods become shorter?

## How

The solution was born as an April's Fool joke.

There is a long-standing tradition of proposing absurd features once a year (here are [some](https://bugs.ruby-lang.org/issues?utf8=%E2%9C%93&op%5Bstatus_id%5D=*&f%5B%5D=issue_tags&op%5Bissue_tags%5D=%3D&v%5Bissue_tags%5D%5B%5D=joke) selected by a tag, but I think there were many more before). The proposal is frequently supplemented with a dead-serious justification; dedicated jokers frequently provide a patch to the language proving the change is possible. A good-natured discussion arises, with other tracker participants alternating between those who haven't noticed the date or felt the absurdity and those who support the joke by discussing syntax details or submitting equally absurd counter-proposal.

[Yusuke Endoh's proposal of 2020](https://bugs.ruby-lang.org/issues/16746), though, was stated in an emphatically unserious tone:

> Ruby syntax is full of "end"s. I'm paranoid that the ends end Ruby. I hope Ruby is endless.
>
> So, I'd like to propose a new method definition syntax.
>
> ```ruby
> def: value(args) = expression
> ```

What happened next was somewhat singular. Matz (Ruby's BDFL) left a comment:

> I totally agree with the idea **seriously** \[...] but don't like the syntax.

And so it happened.

A more natural syntax

```ruby
def value(args) = expression
```

was initially considered impossible, but stellar @nobu (Nobuyoshi Nakada) [implemented it](https://github.com/ruby/ruby/pull/2996) in one night. (Apparently, this still required a lot of careful juggling with the parser: some limitations the new syntax brought were [resolved](https://rubyreferences.github.io/rubychanges/3.1.html#inside-endless-method-definitions-method-calls-without-parenthesis-are-allowed) only in the next version, and some nasty quirks are still remaining and discussed below.)

So, there are times when a lighthearted joke might produce an important change to the language (and give it a goofy name: "endless method" is still its semi-official moniker, frequently used on the tracker, though [in docs](https://docs.ruby-lang.org/en/3.2/syntax/methods_rdoc.html), it is referred to as "shorthand method syntax").

## Irks and quirks

One confusing and unintended problem with one-line methods is related to non-obvious parsing priorities:

```ruby
class Test
  def initialize(active)
    @active = active
  end

  def invoke = puts "works" if @active
end
# Trying to use it
Test.new(true).invoke
```

Instead of printing `"works"` at the last line execution, this code will fail with a confusing message " undefined method \`invoke'". That's because of the aforementioned confusing parsing order:
```ruby
# Expected:
def invoke = (puts "works" if @active)

# Real:
(def invoke = puts "works") if @active
```

This is most definitely an unintended behavior, and one that apparently incredibly hard to fix, so the discussion is [still ongoing](https://bugs.ruby-lang.org/issues/19392).

As usual with syntax quirks, parentheses help!

```ruby
# This will work as intended
def invoke = (puts "works" if @active)
```

Another example of the parsing problem:

```ruby
# valid
def initialize(one_value) = @one_value = one_value

# throws syntax error:
def initialize(two, values) = @two, @values = two, values

# because it is parsed as
(def initialize(two, values) = @two), @values = two, values

# the remedy, again, is to put parentheses around:
def initialize(two, values) = (@two, @values = two, values)
```

I have a small hope that a tectonic process of switching to a new parser, [Prism](https://github.com/ruby/prism) (it is awesome, [read about it](https://kddnewton.com/2023/06/12/rewriting-the-ruby-parser.html)), might help to fix the case eventually.

## Consequences

One of the things frequently pointed out while criticizing a new syntax is that it makes one-statement methods "special," in a sense that once you need a second statement, you'll need to change the shape completely:

```ruby
# You had this...
def owner_name = @user.name

# ..but what if it becomes a tad more complicated?
# We can't just insert a new line of code right above the existing one:
# need to push code around, remove =, etc.
def owner_name
  default = I18n.t('that_thing.default_owner_name')

  @user&.name || default
end
```

This property of the syntax is not unusual for Ruby, though. A simple `objects.map { do_something }`, once you need a second statement inside the block, requires splitting into lines (and in many code styles, changing block wrapping syntax for multiline blocks[^4]) and is generally _inconvenient_.

[^4]: Ruby has two syntax constructs for block wrappers: `do`/`end` and `{}`. There are two styles of choosing one of them: a part of the community uses `{}` only for one-line blocks (and switches to `do`/`end` for multiline ones), another part, including yours truly, prefers to keep `{}` for "functional" blocks that return values (like `map` or `filter`) and use `do`/`end` only for multi-line, imperative iteration.

We can look at it not as an inconvenience, though, but as a _suggestivity_ of the syntax. At the point when your small and elegant one-line method suddenly needs a second line, one might stop (for a brief microseconds, after all, we think and type pretty fast, we just _perceive_ some things as unnecessary obstacles) and consider one of two scenarios: maybe there is a way to keep it a one-liner? For the method above, it could've probably been something like:

```ruby
DEFAULT_OWNER = I18n.t('that_thing.default_owner_name')

def owner_name = @user&.name || DEFAULT_OWNER
```

...which, depending on the case and the codebase, could represent a cleaner separation of concerns.

There might be another case when two or more statements are what really represents the method's needs. In this case, those few strokes of "rewrite" are also useful: they allow to update an "internal model" of the method from "one phrase" to "several phrases" (and this might lead to adjusting the name, say).

The "one phrase perception" is a key here—and the main thing that the new syntax added: **one-phrase methods** that are written as such: just like trailing `if`. Because the "classic" method, even the smallest one:
```ruby
def size
  @objects.count
end
```
...the internal voice would read: "There as a method `size`. It is calculated as `@objects.count`. That's it."

While the one-line one:
```ruby
def size = @objects.count
```

...is read "The method `size` is `@objects.count`."

The character count here is not that important. Heck, even, paradoxically, the line count is not! While the shorthand syntax is frequently dubbed "one-line method syntax," this is perfectly valid code:

```ruby
Event = Data.define(:kind, :context, :timestamp)

def event(kind) = Event.new(
  kind: kind.to_sym,
  context: self,
  timestamp: Time.now
)
```

This still reads as exactly one phrase: "`event` is a method that produces `Event` instances."

On the other hand, in the code that makes good use of such "one-phrase" methods, one might consciously leave the one-expression method _multi-phrase_, emphasizing its non-triviality (and a general feeling of "here, stop and read"):

```ruby
def send_event(kind, payload)
  EventQueue.instance.push(Event.new(kind:, **payload))
end
```

So it is not about making all one-expression methods endless _mechanically_, but about a tool of thought, a tool of communication.

> And here I'll again repeat the convoluted yet expressive example from "[Pattern Matching / Taking it further](https://zverok.space/blog/2023-11-03-syntax-sugar2-pattern-matching-fin.html#taking-it-further)," where several new syntax features play together, to the effect of "multi-body pattern matching method", with one-line method syntax bringing final touches to the structure:

```ruby
def slice(*) = case [*]
in [Integer => index]
  p(index:)
in [Range => range]
  p(range:)
in [Integer => from, Integer => to]
  p(from:, to:)
end

slice(1)    # prints {:index=>1}
slice(1..3) # prints {:range=>1..3}
slice(1, 3) # prints {:from=>1, :to=>3}
```

And while I don't expect using endless methods for multi-line, yet "logically one phrase" methods to have a large mind share anytime soon, having it as an expressive tool might adjust community outlooks with time.

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please stop here for a moment.** This is your weekly mid-text reminder that I am a living person from Ukraine, and a bit of useful related information._

  **One news item.** Besides everything that happens on the frontline, a couple of days ago, Russians [shelled](https://eng.obozrevatel.com/section-war/news-russian-forces-targeted-residential-buildings-in-thesumy-region-with-multiple-rocket-launchers-three-people-died-including-a-child-photo-29-11-2023.html) Seredina-Buda in the Sumy region, killing two adults and seven-years-old girl. To understand that better, I advise you to look at the town on a [map](https://www.google.com/maps/place/%D0%A1%D0%B5%D1%80%D0%B5%D0%B4%D0%B8%D0%BD%D0%B0-%D0%91%D1%83%D0%B4%D0%B0,+%D0%A1%D1%83%D0%BC%D1%81%D1%8C%D0%BA%D0%B0+%D0%BE%D0%B1%D0%BB%D0%B0%D1%81%D1%82%D1%8C,+41000/@49.9513545,27.1652341,6.13z/data=!4m6!3m5!1s0x412c6733c53b5325:0x9dea326f922bc4a3!8m2!3d52.1837429!4d34.0412051!16zL20vMGd0MmRw?entry=ttu) (and compare it to a [map](https://deepstatemap.live/en#6/49.438/32.053) of active warfare). Russians constantly shell Northern parts of Ukraine to terrorize and in the hope of provoking "an unjustified attack on Russian soil."

  **One piece of context.** Last Saturday was a Holodomor Victims Remembrance Day. Here is the [Ukraine Explainers thread](https://twitter.com/uaexplainers/status/1596109449579696129) giving an important context of _one of the previous_ genocides Russians attempted upon our country.

  **One fundraiser.** Finland-based Ukrainian game designer Sergey Mohov and his charity Polubotok Treasury have an [active fundraiser (with convenient donation options)](https://twitter.com/krides/status/1727967054076969210) to help the Ukrainian Armed Forces. Please consider donating!

  Please proceed with the rest of the article.
</div>

## How others do it

It goes without saying that in many functional (or, in our post-modern times, "functional-first") languages, `name = expression` is the main way of defining them. (And how do _multi_-expression functions look in such languages is a separate question.) Haskell:

```haskell
add x y = x+y
```

In the context of this article, it is more interesting, though, to look at how the problem is addressed in languages of the closer paradigms.

As was pointed out at the beginning, most mainstream languages nowadays use C-style `{}` to wrap their code blocks, and so the problem doesn't manifest itself as pressing: you always can just write `header { body }` in one line as necessary, and the wrapping punctuation neither prohibits this nor takes too much space; it is also easy to mentally skip while reading as a phrase.

Even so, the _special_ role of one-phrase methods is recognized, say, by [C#](https://learn.microsoft.com/en-us/dotnet/csharp/programming-guide/classes-and-structs/methods#expression-body-definitions) and [Kotlin](https://kotlinlang.org/docs/functions.html#single-expression-functions):

```kotlin
// regular function:
fun double(x: Int): Int {
  return x + x
}

// single-expression function:
fun double(x: Int): Int = x + x
```

(For both languages, dropping parentheses _and_ `return` clause, with a value implicitly returned by a single statement, seems to be a point of those shorthands.)

Another group is languages with significant whitespaces: many of them allow writing a short function body in the same line as the header, like Python:
```py
# regular function:
def is_even(x):
  return x % 2 == 0

# ...can be written this way:
def is_even(x): return x % 2 == 0
```
...bringing both shortness and "it is just one phrase" effect (though Python's mandatory `return` might sometimes feel redundant).

[Scala](https://docs.scala-lang.org/overviews/scala-book/methods-first-look.html) and [Nim](https://nim-by-example.github.io/procs/) even use `=` as a symbol between the header and the body, which makes one-line method definitions look almost exactly like Ruby's.

Julia, which is, like Ruby, one of a few languages with `end` keyword, [has](https://docs.julialang.org/en/v1/manual/functions/) like Ruby, `=`-driven shorthand (yet, unlike in Ruby, it creates a value that is possible to pass around—see more about it below):

```julia
# regular
function f(x,y)
  x + y
end
# shorthand:
f(x, y) = x + y
```

As a counter-example, some newer languages with `{}` in syntax and a default formatter in the toolbox, not only avoid special one-expression forms (without parentheses) but tend to set the default formatting rules to prohibit writing a short one-expression body with parentheses in one line. Here are corresponding discussions for [Go](https://github.com/golang/go/issues/4363) and [Rust](https://github.com/rust-lang/rustfmt/issues/262), the latter stating in no unsure terms (emphasis mine):

> Some people like to fit a whole function (decl and body) on one line. **They are wrong**, but we should support it as an option.

## Taking it further

Quite a few Rubyists were displeased with the new syntax specifically due to usage of `=`, which is "too similar to assigning the value," and therefore muddies the distinction between values and methods.

It would be fair to say that the strong distinction between "values" and "methods" is a fruit of strictly the imperative upbringing of the developer. Today, it isn't necessary to take a full course of functional programming to be accustomed to the fact that, yes, you actually can put a function into a variable, and treat it as a data type. After all, the ubiquitous JS has it!

```js
// A definition!
function foo() { return 3 }
// A variable!
foo = function() { return 3 }
```

(Though I admit, even for users of those languages, it sometimes requires a good mentor or a book to reveal the "function can be a value" idea. I met an experienced and productive Rubyist who felt "weird" to pass around a block of code that is passed to a method as `&block`, and when they _needed_ a functional value, they just used `-> {}` lambdas.)

So using `=` to define a function is not an esoteric choice.

On the other hand, it might feel misleading!

As I [explained some time ago](https://zverok.space/blog/2022-01-13-it-evolves.html#taking-offense-for-method-references), Ruby doesn't have a natural way to use a method as a value. The shortest you can do is to invoke `method(:method_name)` and it _creates_ a [Method](https://docs.ruby-lang.org/en/3.2/Method.html) object on the fly, which has both performance and reading penalties.

So, while in, say, C#, when you write an assignment-like definition:
```c#
int multiply(int x, int y) => x*y;
// you can do this:
var m = multiply;
// or this:
call_something(multiply);
```
...so, you really have _assigned_ some value. But in Ruby:
```ruby
def multiply(x, y) = x * y

# No, this immediately attempts to call multiply, and fails
# due to missing arguments.
m = multiply
# That's the only way
m = method(:multiply)
```
...and "assignment-like syntax" feels less justified.

So I think: maybe, there is a possible future where everybody so _got used_ to writing `def method(...) =` that the idea of method _values_ becomes naturally necessary, and there might be one more attempt to bring them to the language.

## Conclusions

I fully expect at least some readers to catch on the "April Fools' joke" theme and use it as proof of the "language awfulness" and "syntax features uselessness" (especially considering my honest covering of the implementation's shortcomings).

But I didn't come here to preach Ruby's superiority (nor to expose its unworthiness).

My theme here is how the language changes in response to our understanding of what and how we want to say and how our understanding is adjusted by the language changes.

For programming languages, the process is as natural, inevitable, and perpetual as for the human ones.

Unlike human languages, many programming languages change not only in response to their factual usage but also in response to their inherent values. What draws me to Ruby is what I feel like its _inherent values_: **phrase-level expressiveness for story-level clarity**.

I am not saying that it is not the "only expressive language" (and not even "one of the most expressive ones"), and many of its design decisions with years became questionable. The only thing I am saying is that it is—for me—the language that **consciously thinks** this way and makes me think, too—and, hopefully, produce not the most mundane texts from those thoughts.

On this note, I have finished covering the last feature in the series I've planned. There would be one more post with some general conclusions and probably a bit of bonus content: a list of Ruby syntax elements that I actually don't like (there are some!) and a list of those that could but haven't (yet?) materialized.

But now December is upon us, and it means the upcoming Ruby release, which, in turn, means I need to work on this year's [changelog](https://rubyreferences.github.io/rubychanges/), and it usually takes quite some time. This year, I plan to publish a few diary notes from this work—and then get to other topics, including the "useless sugar" series wrap-up, after the New Year.

You can subscribe to my [Substack](https://zverok.substack.com/) to not miss it, or follow me on [Twitter](https://twitter.com/zverok).

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

