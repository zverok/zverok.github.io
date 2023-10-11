---
layout: post
title:  '"Useless syntax sugar": Numbered block parameters'
date:   2023-10-11
description: "Are they as ugly as I was told?"
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2023-10-11/what.png
---

> This is a part of a blog post series about "useless" (or: controversial) syntax elements that emerged in recent Ruby version. The goal of the series is not to defend (or criticize) the features, but to share a "thought framework" for analysis of their reasons, design, and effect the new syntax has on a code that uses it. See also [intro post](https://zverok.space/blog/2023-10-02-syntax-sugar.html).

Let's start with the feature that at the moment of its introduction I disliked to the extent of taking it as almost personal offense (the story was already told [here](https://zverok.space/blog/2022-01-13-it-evolves.html#taking-offense-for-method-references)): **Numbered**, or **anonymous block parameters.**

## What

Since [Ruby 2.7](https://rubyreferences.github.io/rubychanges/2.7.html#numbered-block-parameters), instead of writing this:

```ruby
[1, 2, 3].each { |x| puts x }
```
...one might've written this:

```ruby
[1, 2, 3].each { puts _1 }
```

...where `_1` stands for an "unnamed first argument"[^1].

[^1]: There are some nuances about the "first," but we'll get to it.

Argument names from `_1` to `_9` are supported, so, theoretically, one might've even written something like this:

```ruby
# From each row, take a values from columns 1 to 4, and process them this way:
CSV.read('somedata.csv').map { [_1, (_2 + _3) * _4] }
```

...though I doubt that any of the bravest feature's proponents would recommend this coding style! _(Still, might be tempting for one-off scripts and experimentation in the console.)_

## Why

The search for a way of not repeating the block argument for a short one-statement block has a long history. We'll look into it in the next section, but for now, let's try to understand _why_ it seemed important for many to make some improvements here.

**Blocks are a crucial element of Ruby's syntax.**

> My homegrown theory is that the idea of `array.each { ... }` as the main syntax for cycles was the first "aha!" of the new language development for its author—a seed, around which the rest grew.

But many Rubyists in love with the clarity and succinctness of the language were always uncomfortable with a slight irritating redundancy in short blocks. This code, from some point of view, feels formal, even ceremonious:

```ruby
numbers.map { |x| x**2 }
```

We think "square each number" (or, in Ruby word order: "numbers — map to — _their_ squares"), but we write like in a slow school exercise: "in a set of given numbers, for each x, square this x."

Dropping this name repetition is like replacing a subject with a pronoun ("their") to not repeat it too much. It gives a few slight, yet notable gains some people cherish[^2]:

* The liberty to **not name things that are obvious**. Which is important, because naming is hard in many ways ("should I use one letter? which one? `i` looks like a counter, not '`i`ntegration'! should I spell it fully, as one-letter names are bad taste? should I change the name in subsequent blocks following the transformation of the values?").
* The **visual lightness**, especially in the chains of short blocks like
  ```ruby
  numbers.select { |x| x.odd? }.map { |x| x**2 }.partition { |x| x < 20 }
  ```
  ...the swarm of <tt>|x|</tt>'s aggressively stands in the way of the _what code has intended to say_.
* The **screen space**—that, like _avoiding stating the obvious_ will be our repeating motive. Despite all the cool multi-monitor setups and retina displays, we can safely say that _a page_ of code should still be considered something like 80-100 chars wide and 30-40 lines high. How much context can be seen _at once_ (without cramming everything mercilessly) is an important thing for comprehension. And when small blocks spend a quarter of the space to just underline that we have a `|user|` inside `users.each {}`— Well, might've been better!

[^2]: And some consider it not worth thinking about—which is totally OK, till they start to persuade others, or, worse yet, judge others for being "picky, bike-sheddy, unprofessional" for looking after small-scale improvements.

## How

For the simplest case of a block containing just `some_argument.some_method`, Ruby 1.8.7[^3] adopted the trick of defining [`Symbol#to_proc`](https://docs.ruby-lang.org/en/3.2/Symbol.html#method-i-to_proc), invented in Rails:

[^3]: The versions logic was different back then. 1.9 was a major new version with a lot of features (like the `hashkey:` syntax), and 1.8.7 was "the last in the 1.8 family, already introducing some of the new features." Like 2.7 before 3.0 did.

```ruby
# Before 1.8.7:
users.map { |u| u.name }
# 1.8.7+:
users.map(&:name)
```

Clear and to the point!

Here, `:name` is still just a [`Symbol`](https://docs.ruby-lang.org/en/3.2/Symbol.html), but it had a new method `#to_proc` converting it to a [`Proc`](https://docs.ruby-lang.org/en/3.2/Proc.html) object equivalent to `proc { |arg| arg.send(self) }`. And `&` operator invoked that method.

> For some reason, the same Ruby version introduced an option to pass a bare Symbol instead of a block to exactly one method: [`Enumerable#reduce`](https://ruby-doc.org/core-1.8.7/Enumerable.html#method-i-reduce) allowed to just do `numbers.reduce(:+)` (without `&` operator). It is a mystery for future historians!

So far, so good. But only worked in a simple case of one method, belonging to a block's parameter, and not receiving any arguments. Those were still impossible to improve:

```ruby
things.each { |thing| puts thing }
numbers.map { |n| n**2 }
users.each { |user| user.update(status: 'active') }
```

There were some [wild(ish)](https://bugs.ruby-lang.org/issues/15302) [proposals](https://bugs.ruby-lang.org/issues/15301) to allow at least syntax like:
```ruby
users.each(&:update.with(status: 'active'))
```
...but they were rejected mainly because that would extend the `Symbol`'s behavior, turning it from a clear and immutable "internal name" to "that thing that stands for method names." (That's why I called `Symbol#to_proc` a trick: it doesn't create a new entity/type/concept that would be possible to extend in the future, just uses a clever operator redefinition to pretend the concept exists.)

Another approach to solving some of the cases above is to use the `method` method. It allows to do this:
```ruby
things.each(&method(:puts))
# or
filenames.map(&File.method(:read)).map(&YAML.method(:parse))
```

...which is using the [`Method`](https://docs.ruby-lang.org/en/3.2/Method.html) object to pass as a block.

It is "conceptually" DRY (doesn't need to name the parameter, at least!), but, honestly, still awfully wordy—including "stating the obvious," like, friends, I know `puts` is a method, why should I spell that?..

There was [an idea](https://bugs.ruby-lang.org/issues/13581) once to make "get a method" an operator:

```ruby
filenames.map(&File.:read).map(&YAML.:parse)
```

...which eventually was [rejected](https://bugs.ruby-lang.org/issues/16275) (and I already told a [sad story](https://zverok.space/blog/2022-01-13-it-evolves.html#taking-offense-for-method-references) about that rejection hurting my feelings).

There were several more high-concept functional-programming-friendly ideas to allow "constructing" block contents out of callable objects, [currying](https://docs.ruby-lang.org/en/3.2/Proc.html#method-i-curry), and function composition (some of them even [got merged](https://rubyreferences.github.io/rubychanges/2.6.html#proc-composition), neither, to the best of my knowledge, got much use).

Eventually, it was agreed to just introduce a default designation for a block parameter, allowing not to name it explicitly.

As it is frequent in the Ruby design process, _how_ would you designate that anonymous thing went through many discussions and iterations.

The initial proposal was submitted in [2011](https://bugs.ruby-lang.org/issues/4475), and ended by accepting and merging `@1` syntax (allowing also `@2`, `@3` etc.) during 2.7 development in 2019 (eight years later). Then, even before the release, the appropriateness of `@1` was [questioned](https://bugs.ruby-lang.org/issues/15723) (with the ticket proposing to use some "normal" keyword like `it` or `this`), which ended, after some 100+-comment discussion with Matz rethinking it to use `_1`, `_2` etc., and introducing `_0` as "all parameters." The latter, again before the release, [was dropped](https://bugs.ruby-lang.org/issues/16178).

> For many people in the community, the discussion is still burning! [Here](https://bugs.ruby-lang.org/issues/18980) is the latest open ticket arguing to change the name again (in favor of something like `it`).

## Irks and Quirks

One of the frequent sentiment against the new syntax was "it just looks ugly" (or "wrong," or "unintuitive," or, "like an ignored parameter"). While arguing with "I don't like it visually" position is useless, I once [tried](https://bugs.ruby-lang.org/issues/18980#note-20) to do an objective analysis of the syntax choice and the design space surrounding it. To my own slight reluctance, I found out that I could think of nothing better. To repeat that analysis here:

> From my PoV, the design space can be described this way:
>
> 1. It should be something following the rules of scoping by its name (e.g. `_1` is a valid name of local variable, implying locality). This rules out `@1`, which implies "some special instance variable".
> 2. It should look special. Even if you don't know its meaning (just learning Ruby, or haven't upgraded your knowledge of the language for a long time), it should immediately imply "it is not just a regular name like all other names". This rules out `it`. Even besides "somebody could've used this name already" (and somebody could indeed, besides RSpec, I saw codebases which used this abbreviation to mean "iterator", "item", or "i(ndex of) t(ime point)"), it just doesn't give a strong feeling of "this is a local name, but also a special name."
> 3. As far as I understand, we are currently quite reluctant towards introducing "Perlisms" like "this character means something new in that context." [...]
> 4. (Have mixed feelings about this one) It probably should allow a sequence of similar names (like `_1`/`_2` do).
>
> TBH, I can't think of much better naming scheme which would satisfy at least 1-3.

A much more irky thing is related to the reason why the numbers were used, in the first place.

Whether `_2` or `_3` will be to some use is debatable: after all, the idea of numbered parameters is to "avoid stating what's completely obvious," and when there are several of them, the reader will probably need to stop and count anyway— at which point, "just give it a name" seems more reasonable already!

Worse yet, the _presence_ of `_2` might change the semantics of `_1`, like in the following example:

```ruby
{name: 'Victor', country: 'Ukraine'}.each { puts "_1=#{_1.inspect}" }
# Prints:
#   _1=[:name, "Victor"]
#   _1=[:country, "Ukraine"]
{name: 'Victor', country: 'Ukraine'}.each {
  _2 # I just mention it, not even use!
  puts "_1=#{_1.inspect}" # This stays exactly the same statement as above, right?..
}
# Prints:
#   _1=:name
#   _1=:country
```

The reason is how the arguments are treated. The code above is an equivalent of the following Ruby code before 2.7 (before numbered parameters were introduced, and one could've used the names as just local variables):

```ruby
# When it is just _1 mentioned, it is treated "all arguments"
{name: 'Victor', country: 'Ukraine'}.each { |_1| puts "_1=#{_1.inspect}" } # _1 contains [key, value] pairs
# When _2 is mentioned, _1 is treated as "just the first argument"
{name: 'Victor', country: 'Ukraine'}.each { |_1, _2| puts "_1=#{_1.inspect}" } # _1 contains only keys
```
...e.g. it is all about block parameters _[deconstruction](https://docs.ruby-lang.org/en/3.2/Proc.html#class-Proc-label-Lambda+and+non-lambda+semantics)_. But without explicit declaration, Ruby just guesses how to deconstruct by mentions of numbered parameters, which might be deep in the body of the block. The result is— confusing, to say the least. (Repeating myself: once there [was](https://bugs.ruby-lang.org/issues/15723) an idea to have `_0` meaning "all parameters, no unpacking," and `_1` to mean "the first parameter, however many there are," but it [was dropped](https://bugs.ruby-lang.org/issues/16178) for some reason.)

All in all, most of the code I saw in the wild which used numbered parameters to a good effect, just stuck to `_1`. And that's probably OK.

## Consequences

Whatever you think about `_1` aesthetics, the thing is, **people are lazy** (and it is frequently a good thing). And most of them are eventually **happy to have a way to save a few moves**—both keystrokes and "mind moves" like inventing the name for a parameter, and then being irritated at immediately repeating it in the next word.

Once using "pronoun parameters" becomes a habit, it seems to encourage some kinds of code and discourage others. Which is what we are looking for in this blog post series: what effect on the general shape of the code small syntax changes have?

The style that flourishes with this new shortcut is representing computation as a chain of small blocks, _declaratively describing transformations from inputs to outputs_. The style that is _native_ to Ruby, the first mainstream language to have `enumerable.transformation { ... }` as the _main_ cycle construct, and, I believe, played a role in normalizing "map", "reduce", and "filter" as a way of thinking about how things are handled.

In other words, the code like this:

```ruby
teams = []

users.each do |user|
  next if user.admin?

  user_teams = find_teams(user)

  user_teams.each do |team|
    teams << team unless teams.include?(team)
  end
end
```

...in Ruby would rather frequently be seen rather written like this:
```ruby
users
  .reject(&:admin?)
  .flat_map { |user| find_teams(user) }
  .uniq
```

...which (at least from some point of view) pronounces the intention as a clearly readable phrase: "for all users except admins, gather a unique set of their teams."[^4]

[^4]: Let's pretend we already had that usual conversation about "But this way you produce many intermediate arrays, it is inefficient!" All the arguments and counter-arguments in this conversation are already known. For me, in most cases, clarity of intent is the first goal, and only when proven necessary, eventual loss of clarity is possible for a necessary performance gain (unless both clarity _and_ performance are achievable at once, which is frequently the case).

In the two code samples above, the former will become more cumbersome with the introduction of `_1` (and, BTW, where to? Inner block, for better visibility, or outer one, for better character economy?). The latter sample will become even easier to read, avoiding the obvious reminder "it is about users!" and making the reading more fluent:

```ruby
users.reject(&:admin?).flat_map { find_teams(_1) }.uniq
```

So, there are cases when trying to apply that cool trick with `_1` will expose the code inadequacy—and, if we are lucky, encourage a small rethinking into more atomic chunks.

> Of course, this requires some amount of self-reflection from code author or reviewer. I've seen people abusing "character economy" that `_1` gives, to produce monstrosities like `array_of_pairs.map { _1[0] + _1[1] }` (or, when the ActiveSupport is present, making it "nicer" with `_1.first + _1.second`) without a second thought. What can you do!

One personal joy of mine is applying a chain syntax to singular values, not only collections, with `.then` operator, introduced in Ruby 2.5-[2.6](https://rubyreferences.github.io/rubychanges/2.6.html#then-as-an-alias-for-yield_self).

`then` allows to apply the same "chain from input to output" intuition when one is not dealing with an enumerable.

Look at this example:

```ruby
value = Time.parse(YAML.parse(File.read(ENV.fetch('CONFIG_PATH'))).dig('metadata', 'created_at'))
```
This is a lot of parentheses nesting, and the reading feels backward: parse what time?.. Ah, that from some YAML, which we get from where again?.. Oh.

The following is "clearer," but we introduce a lot of one-off variables—and in a large algorithm, it is not obvious for the reader whether they are truly one-off, or should be remembered for the later. Also, if some of the statements are non-trivial, it is easy to lose track of what's calculated to what end:

```ruby
config_path = ENV.fetch('CONFIG_PATH')
config = File.read(config_path)
data = YAML.parse(config)
time_str = data.dig('metadata', 'created_at')
value = Time.parse(time_str)
```

By using `.then`, we can write the code that follows the flow of "input to output" directly:
```ruby
ENV.fetch('CONFIG_PATH')
  .then { |config_path| File.read(config_path) }
  .then { |config| YAML.parse(config) }
  .dig('metadata', 'created_at')
  .then { |time_str| Time.parse(time_str) }
```

...and now, implicit block argument makes extra one-off names completely unnecessary with transparent references to "what the previous statement did:"

```ruby
ENV.fetch('CONFIG_PATH')
  .then { File.read(_1) }
  .then { YAML.parse(_1) }
  .dig('metadata', 'created_at')
  .then { Time.parse(_1) }
```

...and now it is reasonable to rewrite in fewer lines, going from "child / rhymes / with / one / word / per / row" to a meaningful phrase:

```ruby
ENV.fetch('CONFIG_PATH').then { File.read(_1) }.then { YAML.parse(_1) } # read and parse config
  .dig('metadata', 'created_at').then { Time.parse(_1) } # and extract some data from it
```

## How others do it

There are some languages that have also decided to have an "implicit parameter," though they typically limit themselves to support only one.

Say, in Kotlin, [`it` is an implicit name of a single parameter](https://kotlinlang.org/docs/lambdas.html#it-implicit-name-of-a-single-parameter) for a "trailing lambda"[^6]. And Ruby's distant cousin Groovy [does the same](https://groovy-lang.org/closures.html).

[^6]: Funny that when Ruby was the first language in relative mainstream to use `method { code }` approach, lambdas and higher-order functions were so esoteric that Ruby tutorials tended to avoid the term, talking about just "blocks of code." Much later, when the same syntactical element turned out to be useful in younger languages, they could allow themselves to call it straightforwardly "trailing lambda" (just like "regular lambda everybody knows" + some syntax sugar if it is the last argument to the method). The baseline of understanding concepts in the mainstream has definitely shifted through the years!

In Scala, [it is underscore](https://docs.scala-lang.org/tour/higher-order-functions.html), allowing code as short as `numbers.map(_ ** 2)`. Though, Scala's approach is quirkier (or smarter—depends on who's asking): there, each entry of `_` refers to the _next_ in the list of the parameters, allowing, say, to write this:

```scala
                                  // ↓ refers to the first parameter
List(1, 2, 3).zip(List(4, 5, 6)).map(_ * _) //=> List(4, 10, 18)
                                  //     ↑ refers to the second parameter
```

In a somewhat different approach, Ruby's niece Crystal (which once, I believe, was born as a "typed Ruby,"  but grew into an independent wild child) took inspiration from Ruby's `&:symbol`, making it into a [standalone syntax](https://crystal-lang.org/reference/1.9/syntax_and_semantics/blocks_and_procs.html#short-one-parameter-syntax), which allowed to extend it at least for accepting parameters:

```crystal
numbers.map &.**(2)
# same as
numbers.map { |n| n ** 2 }
```

Another niece, Elixir, is the only language I know to [actually use numbers](https://elixir-lang.org/crash-course.html#partials-and-function-captures-in-elixir) for implicit parameters, thus allowing several of them:

```elixir
Enum.reduce(list, %{}, &Map.put(&2, &1, &1 * 15))
```
(Here, `&` before `Map` is akin to Ruby's Proc-conversion operator for the rest of the statement, while `&1` and `&2` refer to the first and second parameters of the proc.)

> We _might_ also start look into the concept of the "[tacit programming](https://en.wikipedia.org/wiki/Tacit_programming)", which, from some point of view, is also about "not repeating the arguments," but this would be a much longer post—while it is obscenely long already. _But once I shall write about application of the high-level functional programming ideas in Ruby in more natural ways than just defining `class Monad` and such._

## Taking it further

> This is some "fantasy" section I want to have in every issue of the series: trying to imagine "how else this approach might've evolve"—not necessary an idea for Ruby language proposal, but just as an exercise that might shift some understanding a bit more.

One small thing comes to mind regarding the possibility of the evolution of the "block with numbered arguments" idea.

Small lambdas with implicit arguments are shining in DSLs, but still give a feeling of "too much punctuation":

```ruby
serialize :salary, format: -> { csv_currency(_1) }
```

One is tempted to ask here: what would be conditions/language changes so we could've just written this?

```ruby
serialize :salary, format: -> csv_currency(_1)
```

I didn't analyze this deeply, but "intuitively" this seems enough syntax for the interpreter to understand "the next code is one-statement lambda's body."

While looking like the whim of a person too lazy to write an extra couple of characters, this change might affect the DSLs design, focusing on the approaches that allow to utilize the "super-compact lambdas" while staying readable—and, in the end of the day, just improve the overall design to not expect cramming wordy clarifications in the middle of declaration (which lambda parameters allow, and sometimes encourage).

## Conclusions

> As it happens with me _all the effing time_, I imagined the series would consist of a very short notices... But even the most simple of the "syntax sugars" eventually brings a whole baggage of opinions, points, views, and analogies with it!

As a matter of conclusion, I wanted to reiterate some points of focus that will be repeated throughout this post series (I am also managing expectations here, you see? If neither of those resonates with you, I don't want to waste your time).

So, some of the beliefs about code and syntax I grew throughout my career:

* Even small syntax elements the language provides might affect the code of those using them, even for pragmatic purposes of "press fewer keys": how it looks as a whole frequently encourages (even so slightly) particular ways of writing code; in some cases, it might not be even the code using new syntax—but the calling code that makes it possible; sometimes, whole classes design is affected. And it is important for new features perception to understand what it encourages.
* An opportunity (not an obligation!) to skip the obvious and to refer to things instead of repeating their names might create a natural flow and improve the _code reader's experience_.
* Screen space, code layout, visual lightness and the amount of code that fits on one screen are all important for big systems design in subtle ways.
* In the end, it is all about _how the whole reads_.

Next entry (in a week or so? maybe. hopefully.) will be dedicated to the **pattern matching**. Please like and [subscribe](https://zverok.substack.com/), or whatever they young people say!

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

