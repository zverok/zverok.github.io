---
layout: post
title:  "A few words on Ruby's type annotations state"
date:   2023-05-05
description: "...that were written in a military training camp and accidentally grew to 5k words"
categories: ruby philosophy
comments: true
---

## ...that were written in a military training camp and accidentally grew to 5k words.

> I am writing this on my phone, in a barrack that houses some 200+ of my brothers-in-arms in the Ukrainian army's training camp; I use short periods of rest between training, mostly at night and on Sundays. TBH, since joining the army, I didn't expect to have time or inspiration to write about Ruby, and yet, here we are.

Recently, there was a [long and quite interesting discussion](https://www.reddit.com/r/ruby/comments/12kjtv0/why_i_stopped_using_sorbet_in_all_my_ruby_projects/) in /r/ruby, which started with an [article](https://betterprogramming.pub/why-i-stopped-using-sorbet-in-all-my-ruby-projects-9366bf6dd116) about one of Ruby's type annotation tools, Sorbet—or rather, the reason somebody would abandon attempts of using it.

The discussion quickly turned into a generic argument about typing, type annotations, dynamic and static languages, and so forth.

That discussion made me remember some thoughts on type annotation situation in Ruby I've distilled while working on my [Ruby book](https://rubyintuitions.substack.com/) (now postponed till our victory); some of those thoughts expressed an urge to be shared.

**The text below might be (or might not be) interesting for Rubyists as well for a general audience of programming language design space enthusiasts. It focuses on some design decisions, the reasons that forced them to be made, and the consequences that came after.**

Considering the conditions of its writing, the article will be somewhat imprecise in its definitions, preferring "layman" definitions to strict ones. It is also probably not very historically accurate: I write about decisions and discussions in Ruby design and how I remember them without trying too hard to validate it with remaining documents and discussion logs.

Still, I hope it to be interesting.

## The feature that might've been

A few years ago, when the Ruby 3 release was brewing, there were a lot of expectations laid on it. The main one was a 3-fold performance improvement compared to Ruby 2.0 (known as the "Ruby 3x3" initiative) while preserving full backward compatibility; but also, the "big digit" release obviously asked for new cool features.

**The baseline of what the modern high-level language should provide has shifted significantly in the last decade**, with many elements that were previously associated with academic/experimental languages gaining their attention in the mainstream. To name a few: first-class lambdas/closures, algebraic types, pattern matching, actor concurrency—

Type annotations in dynamic languages seemed to fall into this list, at least in the eyes of a part of the community.

Whether the latter is/was a sound idea, we'll discuss it a bit later. What should be said now is two things: that pressure to "do something about type annotations" was definitely present, and that Matz (Yukihiro Matsumoto, Ruby's extremely nice and thoughtful BDFL) was never quite fond of it. I vaguely remember a couple of presentations from him about the upcoming three-oh release, where in his usual laconic manner, Matz commented on the issue, saying that he acknowledges the demand but personally never liked the idea, basically considering annotations redundant, "not DRY."

I fail to remember any particular syntax proposal for in-language type annotations to be discussed at that point, but the Ruby community was already experimenting with DSLs and tools to make it happen. This, by the way, is pretty usual for Ruby folk: many features that the language currently has were born as experiments, DSLs, and prototypes.

One of those I remember to be [extensively discussed at RubyKaigi 2017](https://rubykaigi.org/2017/presentations/soutaro.html) was **Soutaro Matsumoto's [steep](https://github.com/soutaro/steep). Its idea was to annotate types in a separate file using a Ruby-like (but not Ruby) language**.

Considering the method `String#split` which accepts two parameters—separator (string or regular expression) and (optional) numbers of parts to split into:
```ruby
class String
  def split(separator, limit = nil)
    # ...implementation...
```

...its steep declaration would look like this:
```
# written in a separate .rbs file
class String
  def split : (String, ?Integer) -> Array<String>
            | (Regexp, ?Integer) -> Array<String>
```

**Another notable tool, released maybe a year later, was [Sorbet](https://sorbet.org/) by Stripe.** It was powered by an in-code DSL: valid Ruby statements that were expected to be written alongside method definition:

```ruby
class String
  extend T::Sig # adds `sig` method to the class

  sig { params(separator: T.any(String, Regexp), limit: T.nilable(Integer)).returns(T::Array[String]) }
  def string(separator, limit = nil)
    # ...implementation
```

> For non-Ruby devs: code above just calls a method `sig` that receives a block of code (akin to lambda in other languages), and inside this block of code, the method `params` is called with signatures passed to it. So all of this syntax is valid Ruby and doesn't require any additional tricks to work.

At the moment, Sorbet's syntax looked (for me) as an interesting experiment on what types of challenges in-language type annotations needed to solve—and (for me!) was proof that the feature should be supported by language's syntax natively: DSL version was kinda readable, yet clumsy and wordy.

These pre-Ruby 3.0 experiments exposed the fact that the design area was visibly huge, and I—probably like many others—was intrigued to see how it would look when it will become "real" (read: part of the language syntax).

## ...but wasn't

On Christmas 2020, Ruby 3 was [finally released](https://www.ruby-lang.org/en/news/2020/12/25/ruby-3-0-0-released/). It included [many new features](https://rubyreferences.github.io/rubychanges/3.0.html)—say, pattern matching[^1]: in a couple of iterations in the next versions, it will prove to be a powerful, clearly designed concept. The performance promise of 3x3 was kinda reached, too—though there are conflicting opinions and ways to measure it.

[^1]: Initially introduced in the previous year, in Ruby 2.7, as an experimental feature, but significantly changed by 3.0.

**As for type annotations, the compromise solution between community pressure and Matz's reluctance that the core team chose was [RBS](https://github.com/ruby/rbs).** I don't remember much of a public discussion about it; it just kinda came to be: a separate Ruby-like language for type annotations, meant to be written in separate `.rbs` files: the idea and syntax pioneered in steep became the official way.

There was an initial ecosystem of [tools](https://github.com/ruby/typeprof) introduced for type analysis, checking, and deduction; soon, a community-supported [repositories](https://github.com/ruby/gem_rbs_collection) of type annotations for standard and third-party libraries emerged. But the bottom line was that **type annotations became part of Ruby-the-tool, but not of Ruby-the-language, Ruby-the-way of expressing thoughts**.

At the same time, Sorbet DSL/toolset authors [decided to continue](https://sorbet.org/blog/2020/07/30/ruby-3-rbs-sorbet) developing it instead of replacing it with RBS. Nowadays, Sorbet also has its own [type annotation files](https://sorbet.org/docs/rbi) format (to accompany the third-party libraries that don't include annotations in code), which is the same DSL the in-code Sorbet uses, i.e., unlike RBS, it is valid Ruby code. There is also a [repository](https://www.reddit.com/r/ruby/new/) of Sorbet-compatible type definitions for many popular gems.

To this day, a constellation of tools and knowledge around the Ruby type annotations remains rich and actively developed but stays apart from the language itself.

I wouldn't try to give a comprehensive overview of this constellation and how various tools interact and compete for a very trivial reason: **I am not interested enough in types-as-tool.**

---

In my day job (the one before the army), I had a position we somewhat jokingly name _"chief code editor"_ (akin to chief magazine editor): a person who—through communication, code reviews, guides writing, tooling—takes a responsibility to keep a large production codebase readable and maintainable.

My main tool here is treating code as text and following **language intuitions** to maintain micro-narratives of separate methods and classes and the big structure of the system that these narratives make **lucid**, i.e., expressing intention as clearly and directly as possible.

In my OSS and blogging activities, I am mostly focused on the same concepts: Ruby intuitions and lucid code. Be it my [involvement in language development](https://zverok.space/ruby.html), [small](https://github.com/zverok/the_schema_is) [practical](https://github.com/zverok/saharspec) [gems](https://github.com/zverok/time_calc) extracted from my day job, [libraries for accessing open data](https://github.com/molybdenum-99), or experiments with "[explanatory](https://zverok.space/spellchecker.html) [ports](https://zverok.space/blog/2021-12-28-grok-shan-shui.html)," my [writing](https://zverok.space/writing/),--the main question is always **"how the language leads us to express this in the clearest way possible."**

From this point of view, an isolated type annotations language doesn't bring anything interesting (and even Sorbet, being implemented as a Ruby DSL, is a separate micro-language). It doesn't make us think of everyday language situations differently through insightful rhymes in syntaxes of annotations and other declarations, and it doesn't make usual phrases and idioms more powerful.

**I perfectly understand that tools like Sorbet have their practical purpose, and the end goal is essentially the same as my "lucid code" approach: long-term maintainability; just the path is different.**

But I can't stop to mourn, if even so slightly, what might have been, would we found a way of having type annotations as a _natural_ part of the language.

The "I am interested in annotations, but neither Sorbet nor RBS doesn't feel right" seems to be a pretty common sentiment. The reasons might differ (the original article that triggered that Reddit discussion lists three quite different ones), but the doubt seems common.

But...

## Did we need it, though?

Every time something typing- or type annotations-related is discussed in the Ruby community, one of the first comments would be something to the meaning of "if you want a language with _this_, take another language." (Honestly, the same argument frequently applied to other new features that eventually were successfully integrated into the language, like pattern matching.)

Frequently, the opinion is expressed as a categorical statement about the full impenetrability of paradigms, or "you can do it, but the harm (to clarity and simplicity) would be higher than any gains."

Is that so?

As a person who thinks a lot about ways of _expressing meanings_ in programming languages and design spaces of the means of expression, I follow the development and design of several modern languages, both new and shiny like Rust and old rivals like Python.

The latter adopted optional [type annotations syntax](https://docs.python.org/3/library/typing.html) a few versions ago, and it seems that several fruitful concepts have grown from it.

I had some experience with Python's approach while working on "[Rebuilding the spellchecker](https://zverok.space/spellchecker.html)" project, and that experience was mostly pleasant.

It went like this: the spellchecker (port of industry-grade [Hunspell](https://en.wikipedia.org/wiki/Hunspell), a complicated set of algorithms) was drafted to a "mostly working" state in a fully dynamic manner. Then, I started to polish the edges: catch special cases, clarify what depends on what (the intention of the port was a clear illustration of algorithms), streamline the flow—

And at that point, an ability to leave type annotations here and there, first as a "note to self," then, as a formally validated system, was a great tool. **And in most places, it also made code _clearer_, not wordier for no gain.**

---

In fact, we in Ruby have several incomplete systems of type-annotations-in-disguise:

* [YARD](https://yardoc.org/) documentation tool provides [a way](https://rubydoc.info/gems/yard/file/docs/Tags.md#types-specifier-list) to document argument/return types with comments:
  ```ruby
  # @param separator [String, Regexp]
  # @param limit [Integer]
  def split(separator, limit = nil)
  ```
* [Grape](https://github.com/ruby-grape/grape) (as well as several other API-first web frameworks) [allows to declare types](https://github.com/ruby-grape/grape#parameter-validation-and-coercion) as part of HTTP API definition:
  ```ruby
  params do
    requires :id, type: Integer
    optional :text, type: String, regexp: /\A[a-z]+\z/
  end
  put ':id' do
    # ...
  ```
* Ruby libraries [supporting GraphQL](https://github.com/rmosolgo/graphql-ruby) and many other schema-based protocols allow to [express types](https://graphql-ruby.org/getting_started.html#declare-types) of schema parts in Ruby DSLs.
* Many codebases use the pattern of a callable object with declarative typed arguments ("interactors," "actions," "commands," or just "service objects"), with one of the leading approaches for those declarations being dry-types gem:
  ```ruby
  class CreateUser < ApplicationObject
    parameters do
      required(:name).filled(:string)
      required(:email).filled(Types::Email)
      required(:age).filled(:integer)
      optional(:phone).maybe(:string)
    end
    # ...
  ```
  (from [here](https://luizkowalski.net/taming-complex-service-objects-with-dry/))
* Even ActiveRecord validations can be seen as an ad-hoc typing system, with all the "[validate numericality](https://guides.rubyonrails.org/active_record_validations.html#numericality)" and such stuff.
* All in all, Ruby's core docs have an [informal agreement](https://docs.ruby-lang.org/en/3.2/String.html#method-i-scan) to mark method params and return types:
  ![](/img/2023-05-05/ruby-docs.png)

The list can go on and on, but it already seems to be partial proof of the fact that _at least sometimes_ we might be better by having **a standardized way to say explicitly, "this value has this type."**

In modern Python, all of these cases—and much more—can be covered with standard, deeply integrated with the language types syntax. Be it documentation generation, [schema validations](https://docs.pydantic.dev/latest/), [HTTP API signature deduction](https://fastapi.tiangolo.com/features/#just-modern-python), [CLI calls generations](https://typer.tiangolo.com/)— "How do I say the type" is **already solved.**

Some examples definitely make me envious: not even the particular libraries, but this exact "already solved" feeling.

And _even without these advanced usages_, eventual type-checking might be extremely helpful.

Now, **I deeply empathize with Matz's "non-DRY" sentiment.**

Too frequently in codebases using YARD types for documentation, one can find preciously useless statements like

```ruby
# @param organization [Organization] Organization to process
def process_organization(organization)
```

We hate code elements that repeat each other, and rightfully so: they impede the reading, making the reader doubt themselves and reread repeated elements to understand whether they fully repeat or clarify each other. We frequently call such occurrences _boilerplate_ (and sometimes take the notion too far, producing "easy to write, hard to guess where it all comes from" code chunks based on the urge to remove "all the boilerplate").

But even in very clearly defined APIs, there remains room for doubt that one would be happy to clarify with a small additional declaration: is this parameter or return value nullable? Does this argument with a plural name accept arrays or DB scopes or what? Is that `client` argument in some service method meant to be our `Client` model, or `Stripe::Client` object, or, say, HTTP client? Too frequently, we need a way to declare (and, ideally, automatically check) this!

So, with years, I became persuaded that in a language-as-a-tool of expression thinking framework, **type annotations in _any_ language might give a powerful boost to expressiveness—but only if they are integrated with the rest of the language naturally.**

> I'll skip here the topic of IDE possibilities opened when type annotations are available: first, because it is self-evident, and second, because "external" to the language type annotations like RBS or Sorbet can—and do—enable this support, too.

## How would it be possible?

Ruby's development process is quite unlike other languages.

Last year, I described how it all works in a [series](https://zverok.space/blog/2022-01-06-changelog.html) [of](https://zverok.space/blog/2022-01-13-it-evolves.html) [posts](https://zverok.space/blog/2022-01-20-still-flying.html) (which also shares personal feelings of someone trying to participate in it). The main trait of this process is that it is highly informal: we don't have anything like Python's PEPs or Rust's RFCs, only a [bug/discussion tracker](https://bugs.ruby-lang.org/). An integral part of this informality is relying on Matz's taste and intuition for everything that affects the language's core.

Generally speaking, the progress on adding specialized classes like [IO::Buffer](https://rubyreferences.github.io/rubychanges/3.1.html#iobuffer), or an internal structure change, even a huge one, like JIT or GC changes, are usually easier to move forward than even one yet "core" [method](https://rubyreferences.github.io/rubychanges/2.6.html#then-as-an-alias-for-yield_self) or class, i.e., the things that directly affect how we write.

Needless to say that **a change as big and visible as adding type annotations would require miraculous persuasion powers and syntactical ingenuity to even be seriously considered.**

To have a better understanding of the gargantuan task one would need to solve, let's try to outline the design space and questions to be solved to have "natural" type annotations in Ruby (pretending Matz have never [said](https://bugs.ruby-lang.org/issues/9999#note-13) "we are not going to add any kind of type annotation")).

> If you are reading this as another programming language user, I encourage it to reflect upon how your language addresses these challenges. It might give some interesting insights into the language developer's design choices.

---

First and foremost, we'll need **a way to put type annotations in method definition syntax** _(...and probably in local variable definition and a couple of other places, but let's not overcomplicate this mental exercise)_.

Most languages go either with `name: Type`, or with `Type name`—with the former being somewhat more popular among languages that are close to Ruby in some of their design elements. Unfortunately, in Ruby, this first syntax is already taken by default values of keyword (named) arguments.

```ruby
# Here is quite a simple Ruby method definition:
class File
  def self.readlines(name, chomp: false)
    # reads the lines from file name, removing the line endings if `chomp` is true
  end
end

# Usage:
File.readlines('README.txt')              #=> ["First line\n", "Second line\n", ...]
File.readlines('README.txt', chomp: true) #=> ["First line", "Second line", ...]
```

> My limited knowledge says Ruby's approach to named arguments is quite rare. In Python, Kotlin, or C#, _the caller decides_ what to name on call. So there is no dilemma in the method definition. Say, in Python:

  ```py
  # the same method might be defined as:
  def readlines(name, chomp=False):
    # ...

  # ...and then called:
  readlines('README.txt', chomp=True)
  # or, also works, because it is the caller's choice to make it named:
  readlines('README.txt', True)

  # So when Python has added types to the syntax, it was just:
  def readlines(name: str, chomp: bool = False):
    # ...
  ```

> In TypeScript, something like named arguments is imitated by dictionary destructuring, and the typing problem is solved for the whole dictionary:

  ```ts
  function readlines(name, {chomp=false}) {
    // ...
  }
  // Call:
  readlines(name, {chomp: true})
  // With types
  function readlines(name, {chomp=false}: {chomp: boolean}) {
    // ...
  }
  ```

But OK, as we don't have much choice here, Ruby could probably go with `Type name`:

```
def readlines(String name, Boolean chomp: false)
```

_TBH, would I try to make a real proposal, my senses would scream at that point, "most Rubyists would reject it as weird!"_

But let's move to the next question.

We need to have a **type syntax** itself.

In Ruby, the common chant is "everything is an object and has its own class" (even primitive values like numbers), so the decision seems to be simple and obvious: just write `ClassName` and be done with it. Or— not be done?

**In most modern languages, types syntax also allows defining:**

* Union types, e.g., "`String` or `Regexp`";
* Nullable types in a handy shortcut: rather something like `String?` than `Optional(String)` or `String | NilClass`;
* Ad-hoc types consisting of a limited set of values: `role: ["admin", "manager", "user"]`;
* Parametrized types/generics, like "array of strings"—and another way to say "array containing exactly one string and one number" (Ruby doesn't have a separate heterogeneous tuple type and uses arrays instead of them).

One can say all of this is not necessary if it can't be expressed with Ruby classes—but requiring to use of only "natural," pre-existing Ruby classes as type annotations would severely limit an ability to infer/check types. Even a relatively simple concept "`array.first` returns one element of the same type as possible array's elements types, or `nil`" can't be expressed and validated.

There would also be a lot of small nasty problems unique to Ruby's approaches. Say, only boolean values pose at least two additional problems:

1. `true` and `false` in Ruby belong to separate `TrueClass` and `FalseClass`, not having any common base, so should the `Boolean` be introduced now (increases the area of language change)?
2. in Ruby (unlike many other dynamic languages) only `false` and `nil` are considered "falsy" values—which turned out to be quite handy, so there are many cases when not boolean, but boolean-ish values are passed around. E.g., any non-`nil` and non-`false` value can be passed instead of `true`, and it all works perfectly; giving up this advantage in favor of "stricter" typing will be backward incompatible ideological change; preserving it would require to have an additional type declaration like `foo: Booleanish` or something like that.

**And don't forget about duck typing** here, i.e., an ability to declare "this method doesn't care about argument's particular class as long as the argument has these methods." A typical way to solve it is to declare "interfaces" or "[protocols](https://mypy.readthedocs.io/en/stable/protocols.html)."

This is how it is [done in RBS](https://github.com/ruby/rbs/blob/master/docs/syntax.md#interface-declaration) (while Sorbet [consciously](https://sorbet.org/docs/faq#can-i-use-sorbet-for-duck-typed-code) [refuses](https://github.com/sorbet/sorbet/issues/2582#issuecomment-580796366) to support it). Say, the notion "`File.open` can receive anything that is convertible to string, as a file name" should be expressed in RBS as
```
interface _Stringable
  def to_str(): () -> String
end

def open(name: _Stringable)
```

As this and similar protocols (expect the argument to have only _one_ method, or sometimes two or three) are extremely common in Ruby, solving them with a whole separate "interface" declaration looks extremely verbose for Rubyists' intuition. YARD's type system acknowledges the fact by allowing to declare "anything that responds to a method" in place:

```ruby
# @param name [#to_str]
def open(name)
```

_**All of these problems are solvable—and to some extent, solved by RBS and Sorbet—I am just trying to do a thorough reality check.**_

If you are still following, here is the next question: **do those typing statements have a meaning as separate expressions or only can be allowed in method definitions and similar places?** E.g., is this valid?

```ruby
# put in the variable "array of strings or nullable integer"
# defined in some imaginary syntax
my_type_variable = Array[String] | Integer?
```

This question is quite important because if the code above is allowed, we are extremely limited to what syntax could be used for types. Because then the syntax should be unambiguously unused before! Only taking on a problem of generics, we **can not** use:

* `Array[String]`: already a valid syntax, calling [Array.\[\]](https://docs.ruby-lang.org/en/3.2/Array.html#method-c-5B-5D) class method;
* `Array<String>`: already a valid syntax, treated as `(Array < String) > ...missing code ...` (i.e., two comparison operators, BTW, class comparison operators [exist](https://docs.ruby-lang.org/en/3.2/Module.html#method-i-3C) and check for sub-/super-class);
* `Array(String)`: will be treated as method [Array()](https://docs.ruby-lang.org/en/3.2/Kernel.html#method-i-Array) with argument `String`;
* `Array{String}`: will be treated as method `Array()` with a block argument that just returns `String`.

So, what should you do?.. _(The best possible candidate is still `Array<String>` probably, complicating the parser to depend on spaces/sequence of lexemes to distinguish comparison from a generic type definition. But this leads to parser complication and, probably, some other consequences.)_

And that's only one of the elements necessary for a good type system!

> We'll talk a bit about pattern matching in a "[bonus section](#bonus-multiple-dispatch-and-pattern-matching)," but it seems suitable to mention here that when Ruby introduced it, the syntax `Array[something]` was used—but it was only made possible by the fact that PM-specific syntax elements are only valid inside PM statements (and `Array[something]` with _another_ meaning is invalid inside them), so no ambiguity is created.

But **why would we need to have type expressions being independent? Because metaprogramming!**

Which is very important in the context.

While Ruby doesn't have many distinct _tools_ for this (no more than Python or JS), it definitely has a distinct _style_ that favors metaprogramming.

For example, Ruby objects don't actually have _attributes_, only methods—and here, the language is quite unlike Python or JS. All object data is ultimately private. So, this code:

```ruby
class User
  attr_accessor :name
  # ...
```

...is actually a call of the _method_ `attr_accessor` with argument `:name`, that defines _two accessor methods_, e.g. dynamically generates code equivalent to this:

```ruby
class User
  def name
    @name # @<identifier> is Ruby's syntax for instance variable name (always private)
  end

  def name=(val)
    @name = val
  end
  # ...
```

Another example: Ruby's main web framework, Rails (and many other libraries) are full of DSLs like this:

```ruby
class Organization < ActiveRecord::Base
  has_many :users
```

Here, again, `has_many` is just a method that does a couple of things, and amongst them, defines that `Organization` class has methods like `users` (returning a DB scope with all organization users), `add_user(u)`, and so on. It also _guesses_ quite a handful of things: that the association links the organization to `users` DB table, represented by `User` model, via `organization_id` foreign key, etc.

The fact that this is a very poses questions like (not an exhaustive list by any means):

* How do we draw types on the fly? Because `has_many :users`, if it wants to define typed methods, needs a way to programmatically go from `:users` symbol to `User` type declaration (which proves the point above: type declarations _need_ to be first-class expressions, that can be generated on the fly, passed to other methods etc.).
* Some of such DSLs need to include type declarations (`attr_accessor name: String`? or what?..), others will draw them (see the point above); the "natural" for Ruby type system—and all the tools working with it—**should** support both cases.
* How many "small revolutions" and reevaluations of language design choices would've been necessary for the whole system to work? I mean, of the two small examples, one with `attr_accessor` already requires redefining an old, popular method, and _how to do this in a backward-compatible way_, I don't know.

Of course, there is an option to limit what types of language construct support typing and just give up on the rest—this is the way that Sorbet frequently chooses, relying on (as far as I understand) its maintainer, Stripe, which already has a policy of using only a subset of Ruby.

Obviously, that's not what a core language feature can allow itself.

**The "this can be typed, this can't" approach would most probably lead to a schism unseen before: splitting of the community into those who prefer type-annotated code and consequently became reluctant to any of the most expressive and dynamic features—and everyone else.**

> TBH, I am partially afraid this _might_ happen even in our timeline due to Sorbet and its support by a heavy-weight Ruby-powered company. Or, it might happen in one of the future attempts, performed in the mindset of "we need types, let's sacrifice whatever would be required to get them," which might pull Ruby out of its path too far.

---

**What we learned after this mental exercise?**

I honestly tried to perform it alongside the reader: since the early drafts of the article, my expectations were that I'd come up with some coherent idea about a possible Ruby typing syntax.

My attempt resulted in a deeper understanding of the level of design complexity making it hardly achievable. (I am not saying "impossible" because I am obviously not the smartest person in the room.)

_A side note consequence: while the question wasn't considered too deeply in public at the time of Ruby 3.0 development (and I have no inside knowledge about internal core team discussions on the topic), the reasons "proper" type annotations weren't introduced seem to be a tiny bit deeper than "nobody just thought about it" (which some Reddit comments suggested)._

## Bonus: multiple dispatch and pattern matching

One question about possible type declarations syntax that I've consciously omitted above is **method overloading by type**. There are a lot of methods in Ruby (especially in core classes) that do different things depending on argument count and types. Say, [Array#\[\]](https://docs.ruby-lang.org/en/3.2/Array.html#method-i-5B-5D) supports these protocols:
```ruby
ary = [1, 2, 3, 4, 5]
ary[0]          #=> 1, just integer index
ary[1, 3]       #=> [2, 3, 4], (start, length)
ary[1..2]       #=> [2, 3], Range of integers
ary[(0..3) % 2] #=> [1, 3], every second element from 0th to 3th
```

When type definitions are part of the language's initial design, the typical solution is defining overloaded methods:
```
# Not valid Ruby!
def [](index: Integer)
  # implementation 1
end

def [](start: Integer, length: Integer)
  # implementation 2
end

# ...and so on
```
...though some languages, like Rust, [consciously avoid](https://internals.rust-lang.org/t/justification-for-rust-not-supporting-function-overloading-directly/7012/7) overloading.

How we implement such methods in Ruby is just `if` or `case` in the method's body:

```ruby
def [](*args)
  if args.count == 1 && args.first.is_a?(Integer)
    # it is index
  elsif args.count == 2
    # it is (start, length)
  # ...and so on
end
```

Since Ruby 3+ introduction of pattern matching, we have a powerful dispatch-by-pattern tool in our hands:

```ruby
def [](*args)
  case args
  in [Integer => index]
    # it is index, handle the `index` variable
  in [Integer => start, Integer => length]
    # ...
  in [Range => range]
    # ...
```

The same syntax can also be used for validating or unpacking arguments for the method that only has one signature:
```ruby
def connect(db, options)
  options => Hash[user: {login:, password:}]
  # If options had correct structure, `login` and `password variables would be set,
  # ...otherwise, NoMatchingPattern would report wrong arguments were passed
end
```

Note that the syntax/approach is not _totally unlike_ typechecking. At least for the direct code reading, the two definitions of `[]` method (the imaginary one with overloads and the real one with pattern matching) aren't that different and look quite _declarative_, exposing what possible method signatures there are.

The pattern matching-based definition maybe even look closer to "what's realistically necessary" to declare without building of full type system and solving all of the complicated challenges described above.

But the pattern matching-in-body has no advantages for metaprogramming, documentation generation, etc. - what happens in the method's body stays in the method's body and is only available during the execution of the method (or for _very_ sophisticated static analysis tools). E.g., would `Array#[]` be defined with a top-level `case`/`in` statement as it is shown above (it isn't because it is implemented in C), it would've looked quite informative/declarative about acceptable signatures, but can't be queried about it with reflection, say, `[1, 2, 3].method(:[]).parameters` would only be able to say it accepts `*args`.

**The wild(ish) idea it gives us is that some combination of the pattern matching and method definitions syntax is somewhat easier to imagine than "classic" type annotations. Ruby's cousin Elixir does something like that.**

Say, what if the examples above could've been written like this (assume we have a new keyword `defp` -- "define a method with dispatching by patterns"):

```
defp []
in (Integer => index)
  # implementation 1
in (Integer => start, Integer => lengths)
  # implementation 2
in (Range => range)
  # implementation 3
end

# and for "connect to DB" example:
defp connect(db, options => Hash[user: {login:, password:}])
  # ...implementation, having access to unpacked `login` and `password`
end
```

... and be exposed as a first-class metainformation:
```ruby
[1, 2, 3].method(:[]).signatures
# => [#<Method::Signature (Integer index)>,
#     #<Method::Signature(Integer start, Integer length),
#     ....and so on
[1, 2, 3].method(:[]).signatures.first.parameters #=> [{name: :index, pattern: Integer}]
```

This is something that tickles my _Ruby intuition_ in _almost_ the right way. Not something to envision in the near future, but not something impossible either[^2]. The fact that it doesn't require a comprehensive type system can be seen as a strength or weakness, depending on how you'll tilt your head.

[^2]: Though we'll be back to that problem when _patterns aren't value objects either_. 🙃

WDYT?

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting a donation.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
