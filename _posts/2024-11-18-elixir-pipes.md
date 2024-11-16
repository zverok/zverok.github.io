---
layout: post
title:  'Elixir-like pipes in Ruby (oh no not again)'
date:   2024-11-16
description: "On a new approach to implement that long-envied feature"
categories: ruby
comments: true
image: https://zverok.space/img/2024-11-16/code.png
---

**On a new approach to implement that long-envied feature.**

In Elixir[^1], there is a [pipeline operator](https://hexdocs.pm/elixir/main/Kernel.html#%7C%3E/2) that allows rewriting this code:

```elixir
a(b(c(d)))
```

into this:
```elixir
d |> c() |> b() |> a()
```

which helps to write code in clearly visible “pipelines” corresponding to the order in which data processing is happening.

The concept is so captivating that many languages that don’t have an operator like this have either [language proposals](https://github.com/tc39/proposal-pipeline-operator) or libraries that implement one. Ruby is no exception.

This is a story of my take on implementing it.

[^1]: ...and in several other languages, but Elixir’s is the most widely-known one… except for shells, of course, which probably have originated the concept. But in many cases, when describing modern mainstream languages, the feature is referred to as “Elixir(-like) pipe operator.”

But first...

## Why we don’t need it

There have been many proposals to introduce the operator to Ruby over the years. The crucial reason they are typically met with skepticism is that Ruby’s API is very different from Elixir’s. In Elixir, most of the methods belong to modules, while in Ruby, they belong to objects we are processing. So the usual Elixir’s [motivating examples](https://exercism.org/tracks/elixir/concepts/pipe-operator) like
```elixir
"go "
|> String.duplicate(3)
|> String.upcase()
|> String.replace_suffix(" ", "!")
```
...wouldn’t require any additional handling in Ruby to make it flow in the right direction:
```ruby
("go " * 3)
  .upcase
  .sub(/ $/, '!')
```
Even most operators can be written in method-chaining style, so we also can do this (though not to everybody’s liking):
```ruby
"go "
  .*(3)
  .upcase
  .sub(/ $/, '!')
```

When the method we want to call doesn’t belong to the last object in the chain, we have an escape hatch of `.then` (introduced in [Ruby 2.5](https://rubyreferences.github.io/rubychanges/2.5.html#yield_self) as `yield_self`, and renamed to `then` in [Ruby 2.6](https://rubyreferences.github.io/rubychanges/2.6.html#then-as-an-alias-for-yield_self)):

```ruby
"https://api.github.com/users/#{username}/repos"
  .then { URI.open(_1) }
  .read
  .then { JSON.parse(_1, symbolize_names: true) }
  .map { _1.dig(:full_name) }.first(10)
  .then { pp _1 }
```

There was even [an attempt](https://bugs.ruby-lang.org/issues/15799) to introduce something looking like a “pipeline operator” in Ruby, giving up to the pressure to have it but making it just another way of writing the classic chains. I.e. the example with strings above might’ve been rewritten:
```ruby
("go " * 3) |> upcase |> sub / $/, '!'
```

The idea was backed by Matz and even merged for a short period before Ruby 2.7, but it caused so much distress and confusion (for bringing the new syntax, which didn’t _do_ anything new) that it was [reverted](https://bugs.ruby-lang.org/issues/15799#note-46) even before the release.

Another reason why introducing something like a pipeline operator in Ruby is problematic is that **we don’t have first-class method references**. Due to how Ruby methods [are designed](https://zverok.space/blog/2024-06-14-method-evolution.html), something like `URI.open` is _not_ a reference to a method object but an immediate call, and the only way to refer to the method is to do `URI.method(:open)`—which, to the best of my understanding, is not only wordy but also is not “cheap” because it would create an OO-representation of a method, the `Method` object, on the call. So, trying to optimize the “parse from URL” example above with a pipeline operator will require to put a reference to `URI.open` and `JSON.parse` into it, and there is no nice way to do that.

And still...

## Why I did it anyway

My experiment **is not a proposal as a library or as a core feature**.

It was inspired by yet another [discussion](https://bugs.ruby-lang.org/issues/20770) about introducing a pipe operator in Ruby. Initially, it started as they all go (with pipe operator being some syntax sugar on top of `.then { code }`). After some arguing, though, it took an [interesting turn](https://bugs.ruby-lang.org/issues/20770#note-34), where Alexander Magro, the submitter of the initial ticket, proposed this:

> What I (re)propose is to define the pipe operator as a statement separator, similar to `;` [...]
>
> This way, we could write:
>
> ```ruby
> "https://api.github.com/repos/ruby/ruby"
>   |> URI.parse(_)
>   |> Net::HTTP.get(_)
>   |> JSON.parse(_)
>   |> _.fetch("stargazers_count")
>   |> puts "Ruby has #{_} stars"
> ```

I still doubt this proposal has a chance to make it into the language core, but there is at least some fresh take here and an interesting justification.

Another piece of the puzzle that finally [nerd-sniped](https://xkcd.com/356/) me was recently released Python’s [pipe_operator](https://github.com/Jordan-Kowal/pipe-operator) library (like in Ruby, there were many attempts through the years). It provides two alternative implementations, one of them is regular “just a DSL”:

```py
PipeStart("3")                        # starts the pipe
>> Pipe(int)                          # function with 1-arg
>> Pipe(my_func, 2000, z=10)          # function with multiple args
>> Tap(print)                         # side effect
>> Then(lambda x: x + 1)              # lambda
# ...
```

but the other one has tickled my curiosity because I couldn’t, from the top of my head, guess how it is implemented:

```py
from pipe_operator import elixir_pipe, tap, then

@elixir_pipe
def workflow(value):
    results = (
        value                           # raw value
        >> BasicClass                   # class call
        >> _.value                      # property (shortcut)
        >> BasicClass()                 # class call
        >> _.get_value_plus_arg(10)     # method call
        >> 10 + _ - 5                   # binary operation (shortcut)
        >> {_, 1, 2, 3}                 # object creation (shortcut)
        >> [x for x in _ if x > 4]      # comprehension (shortcut)
        # ...
```

As it turned out, the approach that makes it work is [an interesting one](https://github.com/Jordan-Kowal/pipe-operator?tab=readme-ov-file#-elixir-like-implementation): when the `@elixir_pipe` decorator is applied to a method, it _transforms the method’s AST_ and defines a completely different method, where the code is rewritten in a more traditional way. Basically, it is full-fledged load-time syntactic macros. Curious.

**That’s what I immediately wanted to try to reproduce!** Not the pipe operator (which is just a pretext for the experiment), but this “local rewriting” of methods using their syntax tree. I am not sure I ever saw such an approach used in Ruby, though I might miss something (the [ruby-next](https://github.com/ruby-next/ruby-next) transpiler does code transformation at load time, but on the level of the whole file, which seems quite a different thing; “transpilers” and “inline syntactic macros” are different by perception and usage).

And, lo and behold, [here we are](https://github.com/zverok/not_a_pipe):

```ruby
require 'not_a_pipe'

extend NotAPipe

pipe def repos(username)
  username >>
    "https://api.github.com/users/#{_}/repos" >>
    URI.open >>
    _.read >>
    JSON.parse(symbolize_names: true) >>
    _.map { _1.dig(:full_name) }.first(10) >>
    pp
end

repos('zverok')
# prints: ["zverok/any_good", "zverok/awesome-codebases", "zverok/awesome-events", "zverok/backports", ...
```

Basically:

* `pipe` is a _decorator_[^2] to mark methods inside which `>>` works as a “pipe operator”;
* every step can reference `_` which would be a result of the previous step;
* but it also can **omit the reference** and just specify a method to call; the result of the previous step would be substituted as the _first argument_ of the method.

[^2]: In truth, Ruby does not have a separate “decorator” concept. `pipe` here is just a module-level method that receives the result of `def repos` (which is the method’s name, `:repos`), and then uses this name to somehow process the method.

It works. It is [small](https://github.com/zverok/not_a_pipe/blob/main/lib/not_a_pipe.rb) and doesn’t introduce any monkey-patches to core objects. It is [fast](https://github.com/zverok/not_a_pipe?tab=readme-ov-file#benchmarks), too, unlike typical situation of adding some new expressiveness DSL slapped on top of Ruby.

It is also, probably, an awful idea.

<div class="one-ukrainian-thing" markdown="1">
  <h3>A postcard from 🇺🇦</h3>

  _**Please stop here for a moment.** This is your regular mid-text reminder that I am a living person from Ukraine, with the Russian invasion still ongoing. Please read it._

  **One news item.** [Russia sentences Ukrainian mother of four to 14 years on ‘treason’ charges for supporting Ukraine.](https://x.com/khpg/status/1857466568453537800)

  **One piece of context.** “A reminder that the 1st Chechen War started by Russia ended not in RU's victory.” Read [this thread](https://x.com/nastasiaKlimash/status/1621960987371773952) that gives a grim picture of how the peace treaties with Russia usually go.

  **One fundraiser.** Please help [boost this fundraiser](https://x.com/KonstantinTeam/status/1852700048615784780) from always reliable and transparent Project Konstantin.
</div>

## How it works

The reference to the Python library that inspired me already gave up the trick

The code above is syntactically valid Ruby. But it is Ruby code that can’t be made to work with some small library/metaprogramming additions: `URI.open` is a method that requires at least one argument, as well as `JSON.parse`, and there is no way to convert them to some kind of “deferred method references”[^3]. Also, no tricks will allow to introduce `_` local variable at the middle of the pipe without it being explicitly defined (it can be made some contextual method, probably).

[^3]: [This library](https://github.com/LendingHome/pipe_operator) _has found_ a way to do so by introducing a whole world of “proxy objects” inside the pipe context.

But this is not a problem, as this code is **never executed**.

Instead, `pipe` “decorator,” when applied to a method, does the following:

* loads the method’s source;
* parses it with [parser](https://github.com/whitequark/parser) gem into AST (abstract syntax tree);
* transforms the AST’s shape into different AST that would work as the pipe expected to work;
* converts the new AST into Ruby code with [unparser](https://github.com/mbj/unparser);
* `eval`s the new code in the target class, basically redefining the method.

The code of the method that is really becomes defined is roughly this:
```ruby
def repos(username)
  _ = username
  _ = "https://api.github.com/users/#{_}/repos"
  _ = URI.open(_)
  _ = _.read
  _ = JSON.parse(_, symbolize_names: true)
  _ = _.map { _1.dig(:full_name) }.first(10)
  _ = pp(_)
end
```

> As an aside note, I would’ve be happy if this parsing/transformations could be performed by Ruby’s standard library, but I am not aware of any “unparser” (AST-to-source code) transformation solution based on either Ruby::AST or Prism. So, that’s that. I am also not aware of any standard method of obtaining the method’s source code other than “take location and look in file” which [method_source](https://github.com/banister/method_source) gem implements. Would be cool to have some standard introspection tools in core/standard library, too.

The transformation code (even as written as proof-of-concept/demo) is robust enough to handle the usage of `>>` pipelines in the middle of a longer method, not only “the whole method should be a pipeline.” So this would work, too:

```ruby
pipe def repos(username)
  repos = username >>
    "https://api.github.com/users/#{_}/repos" >>
    URI.open >>
    _.read >>
    JSON.parse(symbolize_names: true) >>
    _.map { _1.dig(:full_name) }.first(10)

  p "the rest of the code (doing something with #{repos.count} repos)"
  # Anything can be here, including other >> pipelines
```
...or, with `=>` right-hand assignment (that is brought by pattern-matching), this:
```ruby
pipe def repos(username)
  username >>
    "https://api.github.com/users/#{_}/repos" >>
    URI.open >>
    _.read >>
    JSON.parse(symbolize_names: true) >>
    _.map { _1.dig(:full_name) }.first(10) =>
    repos

    # ...the rest of the method, `repos` contains the result
```
...which seems to combine various flow operators handsomely!

## Some conclusions

Once again, I am not saying that this experiment is dedicated to the introduction of a new operator in Ruby. The “library” is rough, experimental, sarcastically named, and will probably stay this way.

What I tried to do here is to **experiment with the approach** that would give macros-like capabilities without runtime overhead and deep invasion into core classes.

When might this approach be useful?

First, in cases like this: in a “laboratory” of the new language features. Would I be more sympathetic to the idea of this operator in general, I would try to work in some hobby project for some time with this addition to gather more data about whether it is really frequently useful and might deserve a place in the language.

Another possible usage is a _strictly limited_ application: not something that is used throughout the entire codebase, but a thin/low-overhead implementation of a domain-specific language (emphasizing the **domain**-specific here) like Rake, or Sinatra/Grape, or Arel, or description of some ML/data-processing algorithm in a clear and concise manner.

Also, such an approach might be used for _small_ rewrites of the method code (though I am not sure that possible gains are worth the added complexity), like, say, `safe_sql def my_query` that wraps all literal strings in a method into `Arel.sql` to make the overall code clearer.

BTW, would I want to make `not_a_pipe` a (somewhat) production-ready library, I would also

* work through “unhappy paths,” from invalid syntax in the method to some parts of the pipeline throwing an error (we need to make sure that the backtrace would show the accurate file/line);
* perhaps introduce an argument-less form of `pipe` method to handle _all_ methods below the call (like argument-less `Module#private`), by catching `method_added` hook.

Some aside conclusions from this exercise are that:

* it would be cool to see now, that Prism becomes Ruby’s default parser, its further development into something that would allow AST rewriting and code introspection;
* I personally find “decorators” (the `something def my_method`) underused in modern Ruby practices, and I would like to see them more used; even if not as powerful as Python’s decorators, they allow for some interesting
* Structural pattern-matching is stunningly convenient for handling AST transformations;
* This was a fun way to spend half a Saturday, after all!

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
