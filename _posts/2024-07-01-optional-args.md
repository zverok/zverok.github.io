---
layout: post
title:  'Vignettes on language evolution: discovering an old syntax feature history'
date:   2024-07-01
description: "One Ruby thing I never noticed before"
categories: ruby evolution
comments: true
image: https://zverok.space/img/2024-07-01/args.png
---

**One Ruby thing I never noticed before.**

While working on Ruby Evolution-themed [articles](https://zverok.space/blog/2024-06-14-method-evolution.html) (and looking for a shape for the future book), I am starting to look deeper and deeper into the history of the language—and into other languages, too, trying to understand when some solutions became common in the industry, or, vice versa, when something has fallen out of fashion.

With Ruby, I spend a couple of hours now and then looking through the [NEWS](https://github.com/ruby/ruby/tree/master/doc/NEWS)/[Changelog](https://github.com/ruby/ruby/tree/master/doc/ChangeLog) files, making my own structured lists of when something changed, and frequently making some (mostly minuscule) discoveries. Mostly, those “discoveries” are along the lines of “Ah, that’s when it was introduced” or “Wait, it wasn’t always like this.” The latter is sometimes more exciting because I spent a long time with Ruby—since 2004, versions 1.6/1.8. Suddenly remembering that a syntax that I am now taking for granted and feeling “natural” wasn’t that way sometimes feels like clarifying your childhood memories.

It is rare that I notice something introduced a long time ago that I was not aware of, even more rare with core syntax features and not some convenient yet rarely needed method.

But recently, just that happened.

## The feature

In the [NEWS file for Ruby 1.9.1](https://github.com/ruby/ruby/blob/master/doc/NEWS/NEWS-1.9.1#language-core-changes-)[^1], there is a concise note:

[^1]: The version history before Ruby 2.0 is quite complicated. Commonly used were versions 1.6, then 1.8 (with odd minor versions, like 1.7, considered experimental), but then “patch versions” 1.8.* were introducing a lot of new features, and 1.9.* were several big leaps forward preparing for 2.0, with 1.9.1 being the first stable 1.9 release, and subsequent 1.9.2 and 1.9.3 changing a lot upon it. Since 2.0, version number adjustment is more predictable: each notable version changes minor number, and is released on Christmas each year.

> **Mandatory arguments after optional arguments allowed**

Wait, does it mean what I think it means?

I don’t remember this being possible. So, I try it:

```ruby
def foo(optional = -100, mandatory)
  p(optional:, mandatory:)
end

foo(1, 2) #=> {:optional=>1, :mandatory=>2}
foo(1)    #=> {:optional=>-100, :mandatory=>1}
```

Yup, this works.

And works in a reasonable way (that is, if one would consider this order of optional/mandatory arguments reasonable, but we’ll get to it): depending on the count of arguments passed, the language decides when the optional argument should use its default value.

A mandatory argument can even follow several optional ones:
```ruby
def foo(optional1 = -100, optional2 = -200, mandatory)
  p(optional1:, optional2:, mandatory:)
end

foo(1)        #=> {:optional1=>-100, :optional2=>-200, :mandatory=>1}
foo(1, 2)     #=> {:optional1=>1, :optional2=>-200, :mandatory=>2}
foo(1, 2, 3)  #=> {:optional1=>1, :optional2=>2, :mandatory=>3}
```
...but the mandatory-after-optional construct might happen only once in the method definition:
```ruby
def foo(optional1 = -100, mandatory, optional2 = -200)
#                                              ^ syntax error, unexpected '=', expecting ')'
end
```

OK, so the behavior is simple and predictable, and still, its existence has surprised me. Unlike many other things in Ruby, I never discovered this by myself (even though Ruby has been my primary language for 20 years, and “how to use it to write clean and expressive code” is my main topic of thinking) because the thought that this might be possible never came to me.

Of course, the reason might be that my previous experience with other languages and CS education taught me that “optional arguments in the end” is the only possible choice[^2].

But on the other hand, I can’t, from the top of my head, think of an API that would require this order of arguments. I _vaguely_ remember defining such methods once or twice or seeing them in others’ code, which did something like...
```ruby
# So one can use "natural" order of
#   post('/foo', {'Content-Type': 'JSON'}, 'data')
#   post('/foo', 'data')
def post(endpoint, headers, body = nil)
  headers, body = nil, headers if headers && !body
  # ...
end

# ...but I didn't know I could just
def post(endpoint, headers = nil, body)
```

...but honestly struggle to remember when that was and whether it was used.

[^2]: For positional arguments, I mean. But at the time and in the languages I learned to program, all arguments were positional.

So, I wanted to know, when this feature was introduced, how was it justified?

<div class="one-ukrainian-thing" markdown="1">
  <h3>A postcard from 🇺🇦</h3>

  _**Please stop here for a moment.** This is your regular mid-text reminder that I am a living person from Ukraine, with the Russian invasion still ongoing. Please read it._

  **One news item.** Just when I was finishing this article, Russia [bombed a postal company terminal](https://x.com/United24media/status/1807449120681943269) in my home city Kharkiv (as they frequently do, in attempts to cripple civilian logistics and make life in the cities closer to the frontlines unbearable).

  **One piece of context.** [Almost all Ukrainian POWs return from Russian captivity in shocking condition, having been subjected to starvation and torture. This is clearly Kremlin policy.](https://x.com/Biz_Ukraine_Mag/status/1807129810147115167)

  **One fundraiser.** Over 25% of UA territory is contaminated with the explosive objects because of RU invasion. [Please, help us demine UA.](https://x.com/nastasiaKlimash/status/1807370509685760129)
</div>

## Why? The investigation

I have maintained “[Ruby Changes](https://rubyreferences.github.io/rubychanges/)” for many years and always try to provide the _reason and thinking_ behind each language change ([example](https://rubyreferences.github.io/rubychanges/3.0.html#endless-method-definition)). I don’t rely only on my own reasoning and language intuitions and always check the original bug tracker discussion (or several, sometimes spanning years) that led to the change.

The problem is, deeper down in history, Ruby’s NEWS/Changelog files become more and more concise, eventually losing links to discussion tickets—and actually, before some point in the past, most of the discussions happened on mailing lists and not on the tracker.

So, the only source we can rely upon is, well, the source (code)!

As the feature of allowing mandatory arguments after optional requires support from the parser, the reasonable place to look for the answers is in `parse.y`: the YACC/Bison definition of the Ruby grammar. Looking at [its state at 1.9.1](https://github.com/ruby/ruby/blob/v1_9_1_preview1/parse.y) and squinting hard enough around all the C code embedded into the grammar definition, we can relatively quickly find [the necessary fragment](https://github.com/ruby/ruby/blob/v1_9_1_preview1/parse.y#L4191): the definition of a method arguments node (I skip here all the C code that was embedded into grammar):

```
f_args    : f_arg ',' f_optarg ',' f_rest_arg opt_f_block_arg
            | f_arg ',' f_optarg ',' f_rest_arg ',' f_arg opt_f_block_arg
            | f_arg ',' f_optarg opt_f_block_arg
            | f_arg ',' f_optarg ',' f_arg opt_f_block_arg
            | f_arg ',' f_rest_arg opt_f_block_arg
            | f_arg ',' f_rest_arg ',' f_arg opt_f_block_arg
            | f_arg opt_f_block_arg
```

This reads: `f_args` node is a sequence of `f_arg` (defined below as a list of mandatory arguments, just names), then a literal comma, then `f_optarg` (optional arguments with default values), comma, `f_restarg` (the "rest" arguments `*rest`), then optional block argument; or, mandatory arguments, comma, optional, comma, rest, comma, mandatory again, block; or... and so on, you get the idea.

We are interested in the lines when `f_arg` (mandatory arguments) are after the `f_optarg` (optional arguments/arguments with defaults); and via `git blame` we can find the commit where it was introduced: [this one](https://github.com/ruby/ruby/commit/ead9b197be96f9eacf462b5539f71c43422495d0). Fortunately, it even has an identifier for the mailing list discussion (`[ruby-dev:29014]`), which is easy to find in an [online mirror](https://public-inbox.org/ruby-dev/87mzbghkrm.fsf@fsij.org/T/#m1b2d41e34a698d03b43d8462af9ac4b6869972a2). Well, it is an old `ruby-dev` Japanese-language mailing list, but we are blessed with Google Translate, so here is the conversation starter:

> Suddenly, in 1.9, you can omit the first argument (like `TCPServer#initialize`). Isn't it possible to define an optional method as `def m(a=nil, b)`? I thought about it and tried it, but it doesn't seem to work.

So, the API mentioned as one of the possible usages is [TCPServer#initialize](https://docs.ruby-lang.org/en/master/TCPServer.html#method-c-new) of the language standard library, which indeed has a protocol `new([hostname,] port)` (the _first_ argument is optional, the the second is mandatory).

In the ensuing discussion, Matz is initially reluctant to introduce what he considers a questionable possibility but eventually implements it himself. It was not because somebody provided many specific usages for this particular syntax but because the question turned out to be an edge case of the more general change in Ruby 1.9: the possibility of having a singular argument (mandatory or optional) after a _variable number of arguments_:

```ruby
# Was a syntax error in Ruby 1.8
# API like:
#   handle_files '*.rb', '.*js', Compiler
def handle_files(*extensions, handler)
#                ^              ^
#        any number of args   mandatory last
end

# A usual idiom in Ruby of those days: the last argument is
# an options dictionary:
def handle_files(*names, options = {})
  # ...
end

# usage:
handle_files('README.md', 'script.rb', remove: true)
#                                      ^^^^^^^^^^^^
# The last part would be treated as `options` dictionary
```
The change was introduced in a [large commit](https://github.com/ruby/ruby/commit/9b383bd6cf96e1fe21c41528dec1f3ed508f335b), which doesn’t have a link to some discussion. Nevertheless, it seems to be less inexplicable than the “optional argument before the mandatory.” The `options` dictionary after the variable list of arguments might’ve been the main driver (confirmed in Matz’s [old blog post](https://matz.rubyist.net/20060601.html#p03)), but there are other usages even today, when real keyword arguments replaced the options hash. _I was able to `grep` such occurrences in our production app codebase, in [Rails](https://github.com/rails/rails/blob/v7.1.3.4/actionview/lib/action_view/template/handlers.rb#L31), in [Rubocop](https://github.com/rubocop/rubocop/blob/v1.64.1/spec/support/multiline_literal_brace_helper.rb#L25) — though in very moderate quantities in all of them._

But, if this is possible (and Ruby can properly “understand” that the only argument goes into the last one):
```ruby
def foo(*optional, mandatory)
  p(optional:, mandatory:)
end
foo(1) # {:optional=>[], :mandatory=>1}
foo(1, 2) # {:optional=>[1], :mandatory=>2}
```
...then, if just for consistency’s sake, another way to define a method with a variable number of arguments _before the last one_ should be possible, too.  Just in the case of optional arguments, the “variable number” is 0 or 1 (or, more precisely, from 0 to the number of optional arguments defined before the mandatory ones). And so, it happened.

## Is this useful?

As I’ve said earlier, I fail to remember many APIs that I would’ve liked to implement with an optional argument before mandatory (if I’d remembered it existed). The Ruby core API/standard library has, if I am not missing something, two such examples (they might be implemented in C, but the principal existence of such signatures shows when they might be useful): the aforementioned `TCPServer#initialize([hostname,] port)`, and [exec](https://docs.ruby-lang.org/en/master/Kernel.html#method-i-exec) method to run external commands, that has a signature `exec([env, ] command_line, options = {})`—i.e., allows to provide `ENV` for the subprocess. The order here is probably intended to imitate how we do this in the shell:
```bash
VERBOZE=1 ruby run_script.rb
```
to provide the “natural” order in Ruby:
```ruby
exec({'VERBOSE' => 1}, 'ruby run_script.rb')
```

As with my “theoretical” `post(endpoint, headers = nil, body)` example above, all of those examples seem to solve the conflict between what is perceived as natural for some API: natural ordering vs. what’s natural to omit.

But if we look wider—at the general “mandatory argument after a variable number of other arguments”—it seems to find its usages, as the examples above show. And in the spirit of symmetry that I talked about in the [previous article](https://zverok.space/blog/2024-06-14-method-evolution.html), making this available in method _definitions_ meant that the same syntax became available, say, in variable assignment, so you can just...
```ruby
*folders, filename = '/home/zverok/blog/_posts/optional-args.md'.split('/')
folders #=> ["", "home", "zverok", "blog", "_posts"]
filename #=> "optional-args.md"
```

There is another interesting API, though, which maybe not a lot of Rubyists frequently implement, but which is still a part of the language’s abilities, and which makes “last mandatory argument” make sense. It is a redefinition of the indexed assignment operator, where the last argument of `[]=` method is the assigned value, and all before it are parts of the key:
```ruby
# With unpacking
class Multidimensional
  def []=(*key, value)
    p(key:, value:)
  end
end

m = Multidimensional.new
m[1, 2, 3] = 4 # {:key=>[1, 2, 3], :value=>4}

# Or with optional in the middle
class Matrix
  def []=(col, row=:all, value)
    puts "Assigning [#{col}:#{row}] = #{value}"
  end
end

m = Matrix.new
m[1, 2] = 3
# Assigning [1:2] = 3
m[1] = [4, 5, 6]
# Assigning [1:all] = [4, 5, 6]
```

## How others do it

There are very few languages that allow to define methods/functions with optional arguments before (positional) mandatory ones. PHP had this, but rather as a design flaw (`foo(1)` for a function with optional-then-mandatory arguments would still consider the value for the first one is passed, and the second is missing), finally [deprecating](https://php.watch/versions/8.0/deprecate-required-param-after-optional) it in PHP 8.

JS has this, but requires passing an explicit `undefined` to use the optional value:
```js
function foo(optional = 1, mandatory) {
  console.log({optional, mandatory})
}
foo(2)            // {optional: 2, mandatory: undefined}
foo(undefined, 2) // {optional: 1, mandatory: 2}
```

Python, Kotlin, and Scala all allow other mandatory arguments defined after optional or unpacking ones, but then they can be passed only by name:
```kotlin
fun foo(i1: Int, i2: Int = 0, v: String) {
  println("foo($i1, $i2, $v)")
}

fun main() {
  // Type mismatch: inferred type is String but Int was expected
  // Kotlin can't guess we intend to omit i2
  t.foo(1, "foo");
  // Works:
  t.foo(1, v="foo"); // foo(1, 0, "foo")
}
```

If we look at how other languages handle cases like `TCPServer([host,] port)` from above—we might find different solutions, like, [in Python](https://docs.python.org/3/library/socket.html#socket.create_connection) it is passing a _tuple_ of `(host, port)` as one argument, where `host` can be `''`; while, say, NodeJS [just inverts](https://nodejs.org/api/net.html#serverlistenport-host-backlog-callback) the order to `(port [, host])`. Anyway, the direct need for such APIs seems to be rare enough to not be too bothered.

It is interesting to note here that newer languages strive to avoid the question altogether: Rust, Zig, Nim, and Go all made a choice of not allowing optional arguments at all, instead asking their users to use param structs (maybe with default initializers for some fields—here is [Zig’s version](https://github.com/ziglang/zig/issues/485#issuecomment-440489525)). This, and other approaches to pass named arguments to indicate their meaning rather than using positional ones, seem to become the most widespread solution for defining APIs with many parameters with non-trivial defaults and order.

In other cases (like in Java's [ServerSocket](https://docs.oracle.com/javase/8/docs/api/java/net/ServerSocket.html)), a possibility to have several definitions for the same method might be used instead, just defining several versions of the method to be chosen based on the actual arguments provided instead of one with many optional arguments.

The one last case I was curious about is how other languages handle the “indexed assignment” operator redefinition. Here, Python just gathers the entire key in one tuple, thus removing the question of defaults and unpackings (or, rather, moving it into an imperative logic inside the method `if len(key) ...` and so on):

```py
class Multidimensional:
  def __setitem__(self, key, value):
    print(f'{key=}, {value=}')

m = Multidimensional()

m[1, 2, 3, 4] = 5
# key=(1, 2, 3, 4), value=5
```

...while Kotlin actually does a “trick”:

```kotlin
class Multidimensional {
  operator fun set(i1: Int, i2: Int = 0, v: String) {
    println("set[$i1, $i2] = $v")
  }
}

fun main() {
  var m = Multidimensional();
  // Works, doesn't require an explicit name for v!
  m[1] = "foo";      // set[1, 0] = "foo"
}
```

## So what?

Unlike the “[Useless Ruby Sugar](https://zverok.space/blog/2024-01-23-syntax-sugar-fin.html)” series, this text is dedicated to a syntax feature that is neither new nor overwhelmingly expressive. I think it might have its usages (in the upcoming months, I’ll try to reflect more on code I write to understand where it might be appropriate), and a bigger feature of allowing a singular argument after unpacked one definitely has some.

What I was trying to do here is rather show how the design emerges, what choices are made, and how they might be different for similar programming languages, and how uncovering those choices many years after might be similar to a mystery investigation.

Hope you enjoyed it.

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now—including subscription options with secret posts! Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

