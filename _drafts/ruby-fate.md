---
layout: page
title: The Fate of Ruby
permalink: /ruby-fate/
---


**This a pretty long text about Ruby programming language: why and how it is an important point in a history of computer science, how it continues to live and evolve, and what forces are moving it to the roadside of said history.**

Partially it is a continuation/retelling of my RubyConf'19 talk [Language as a tool of thought](https://www.youtube.com/watch?v=iMBqqjkbvl4) (but this version is grimmer, at the conference I felt obliged to end on an uplifting note), partially an answer to the question "What is the Ruby Legacy?" [asked by Avdi Grimm](https://avdi.codes/what-is-the-ruby-legacy/), but mainly it is an attempt to state **one person's current vision on one language's place in history, culture, and technology**.

The text is unusually long (tbh, at some point I expected it to grow to a book size... but it hopefully will not), so I'll try my best to use structuring/typographic tools at hand to make it easier to navigate and read. I also feel the text of this length and boldness requires some context about its author.

**About me:** I am a 37-year-old Ukrainian developer. I started to use Ruby as the main language somewhat around 2004, after a long search of the language that would feel "right" (went from Pascal/Delphi, through a long romance with C++, briefer encounter with Perl, and picked up a bit of every known programming language at the time... was a bit of PL junky). Unlike many of that wave, I came to Ruby not through Rails, and managed to avoid Rails-based projects till around 2015. During my time with Ruby I've created a lot of gems (nothing too popular), mentored students (online and offline, paid and volunteer), spoken on lots of domestic and international meetups and conferences, and for the last several years seriously involved into language's development and documentation. (All of my activities and "achievements" are listed [here](https://zverok.github.io/public.html).)

Also, I am a writer/poet (obscure, but sometimes published), which is surprisingly relevant when we try to talk about _language_ and _expression of thoughts_.

All of the above is stated to demonstrate only one thing: I had a lot of time, and a lot of passion to form and evolve my opinions on the language. By no means I am stating those opinions are "universally correct" or "the only possible", but I do believe they are _consistent_ and _could be interesting_. So, I am not asking to _believe_ or _agree_, only to _try my optics_.

> And the one last thing: to illustrate some points, I'll mention particular libraries and quote concrete people. That sometimes _could_ look like a personal attack, but I want to state clearly that it is _never_ my intent to say that somebody is less right than me or less smart than me or have some malicious intentions or something like that. It just so happens that a lot of things are better shown on real examples and naming real actors instead of "one prominent library at some point made one [unstated] decision..." roundabouts. For the purposes of this text, I prefer clarity over nicety.

## Importance of the language

It seems in any programming language-related discussion there always would be one side claiming that "a language is a tool, it doesn't matter", that "professional developer picks new language in a few weeks" or something along those lines. Those are "top-down" thinkers, who maintain that principles stay the same and architecture mostly doesn't depend on the language.

Another similar claim is that **"most languages are pretty similar, that's just syntax details"—and proponents of this claim mostly stick to some common mainstream "quasi-language"** (namespaces, classes, methods, variables, if/while/for, etc.) regardless of what concrete language they use at the time. Concrete languages are judged on the base of their performance, availability of libraries and developers, popularity in the industry, and some "real" features (like support for async) as opposed to "just syntax" features—which are sometimes even thought as peculiar disadvantages, like "why you just not do what all the mainstream does?"

> Ironically, "what's mainstream" is constantly changing, with ideas coming exactly from outside of "what everyone does". I, for one, do believe that general widespread of `array.map(function).filter(function)` as a "mainstream", more modern than "`for` for everything", was at least _affected_ by a brief period of Ruby's universal popularity.

Then, there are others, myself included, who believe that **language we use shapes how we think about implementing stuff**, and attention to some particular language, its idioms and its peculiar differences from others may be insightful and change a lot.

I tend to describe it this way: **the ideas, practices, designs are _tools_ we build with, while the language and libraries are _materials_ we build of**. And as most of the non-trivial work we do is mostly "[post-industrial](https://www.youtube.com/watch?v=5UiBQtfRDUI)" in a sense that each task is at least slightly unique, slightly different than the previous one, being comfortable with your material, following its qualities and noticing its changes is important for success.

This being said, I consider all of those activities beneficial for one's productivity:

* studying your main language's features, techniques, and idioms, constantly looking "is there a way to express it simpler/shorter";
* studying (at least through skimming through language reference/READMEs of popular libraries) other languages idioms and differences -- both from "sister" ones and very different ones;
* following language's (your main and others you are just fond of) evolution, analyzing _why_ something considered as a useful addition, why some practices are deprecated, what new libraries popularity says about their API design, etc.

Look, I am not saying that if you are "not into this kind of stuff" you are bad/unprofessional/unproductive, OK? What I _am_ trying to say, is: **there are people out there who think in their programming language. And for those, Ruby has some unique appeal**.

### What's cool about Ruby

So what's the "promise" of Ruby? What's so good about this language?

One of the usual mottos is "Optimized for programmer's happiness" which I believe _is_ true, but also quite useless: it is one of those sayings that you understand when you already understand it, and it explains nothing if you don't.

My way to say the same thing—utterly subjective, but a bit easier to explain—is this: **Ruby tries hard to allow expressing things exactly how you think them**, with as much verbosity as necessary to properly explain, as much nuance as matters, and as logical order as it could possibly be. (Note that this definition is significantly different from "reduces boilerplate" or "minimal" or "simple". Note also "allow", not "enforce".)

Ruby achieves this goal by following three principles:

1. Small, consistent and discoverable **core**, based on _"everything is an object"_ idea;
2. Explicit and clear **data flow**, expressed by _methods chaining with blocks_;
3. _Flexible syntax_ to **intone** and underline intentions of the code;

Let's take a look at all three of them.

#### 1. "Everything is an object"

It seems kinda vague (and kinda "nothing new, it is just a description of OOP language"), but what it means in Ruby is this: (almost) everything you can do is call a method on the relevant object (including implicit "current context"); everything you need to understand "how to do something" is to ask what methods relevant object has. It makes the code consistent and discoverable, both at writing ("what's the way to do it?") and reading ("where it came from") time.

For example, given array `a`, whatever array operations you'll think of, are consistently performed by methods of [Array](https://ruby-doc.org/core-2.7.1/Array.html) class, being it check for emptiness (`a.empty?`), enumerating (`a.each{|item| ...}`), adding new items (`a.push(item)`) or testing if it includes one (`a.contains?(item)`).

> Compare it with your language of choice <small>(especially if it claims to "always have one obvious way to do it")</small>! But keep in mind languages cross-pollination for last years: totally possible that your one (say, modern JS) is already not that different—but it was Ruby that was _initially_ designed this way.

Another example: Ruby doesn't have any special language construct for class-level "macros". So, this:

```ruby
class Cat
  attr_reader :name
```

...is just a call of the _method_ `attr_reader` on the implicit `self` of that context (e.g. `Cat` class—object of `Class` class), which is properly documented in a [`Module`](https://ruby-doc.org/core-2.7.1/Module.html) (ancestor of `Class`) docs, alongside with everything you can do about methods visibility, dynamic definitions, constants and everything else Ruby's object system allows: it all is exposed in a regular way.

Ruby doesn't take this principle to the extreme, though, staying short of "esoteric" languages camp, unlike, say, its ancestor Smalltalk: in Ruby, there are non-method-calls too. Some have method call equivalent (`class SomeName` vs `SomeName = Class.new`), others do not (`if`—unlike Smalltalk's!).

#### 2. Data flow

To the best of my knowledge, Ruby was the first language to become mainstream which doesn't have `for` loops. Well, formally it does, but they are strongly discouraged in favor of functional enumeration methods with blocks: `array.each{...}`, `array.map{...}`, `array.select{...}` and so on.

I imagine (but too shy to ask Matz directly), that the ability to write typical loops as a sequence of `data => transformation => transformation` was one of the biggest inspirations on the initial Ruby design, both on transformation methods being attached to core classes, and on the concept of "code block" being borrowed from Smalltalk.

> Once again, "how Ruby does it" may not feel that special in 2020, with modern JS and other languages having now syntaxes like `map(x => x**2)` as a nicer way to say `map(function(x){return x**2;})`—which is how it looked back then when Ruby made its blocks-based enumeration exposed into the mainstream.

The most important thing about this approach to enumeration—the main data-processing construct—is that it made _chaining_ of operations natural, providing the possibility to write something like this, as an _obvious_ statement "what's going on in this code":

```ruby
users
  .reject { |u| u.admin? }
  .chunk { |u| u.role }
  .each { |role, group| notify(role, group.sort_by { |u| u.registered_at }) }
```

_(I am smirking every time encountering the frequent "functional programming isn't natural to Ruby". Wake up already! The essence of functional programming—the explicit flow of data through higher-order functions without mutations—is what Ruby was built upon.)_

#### 3. Syntax flexibility

Ruby's <abbr title="There is more than one way to do it">TIMTOWTDI</abbr> principle is frequently misunderstood (especially from outside of the community) as a sign of the "chaotic" or "lone ranger" nature of how Rubyists code. But that's not what this principle is about. The real meaning is _humanity_ of the language: with the small core of concepts (just objects and methods!), syntax flexibility plays a role of intonation/punctuation, allowing to underline the different intentions of the code built from the same blocks.

Like this: both statements built from the same elements (method-arguments-block), but they look differently, clearly separating the semantics:

```ruby
%[Ruby Python Scala].map.with_index(1) { |v, i| "#{i}. #{v}" }

create_table :languages do |t|
  t.column :id
  t.column :name
end
```

With community growth, of course, syntax flexibility had its share of questionability, leading to deviations like different personal styles of small influential groups, and backlashes like the introduction of over-detailed linting rules, effectively prohibiting the "intonation" intent of syntax in lots of cases. But it is hard to overestimate the influence of flexible syntax and optionality of some elements on the acceptance of the language's inherent feature minimalism (like an absence of a separate concept of "object's attribute", which parentheses-less method calls like `array.length` made unnoticeable).

---

> Obviously, I don't consider Ruby "the ideal language". Besides the obvious questions "ideal when", "ideal who whom" and "ideal for what", there are some parts of the language that I'd dream were different (and which are very hard to fix in the hindsight). To name a few, it is modularity system (maybe Python's localized imports are easier to tame), weirdly shaped parts of the standard library, and the fact that most popular `Enumerable` methods return arrays immediately instead of providing enumerators. I may only imagine that some habits of Ruby developers could've been shaped differently, which probably could change the course of evolution.

In general, nothing feature-wise was particularly "new" in Ruby: object model and blocks from Smalltalk, TIMTOWTDI and regexp literals from Perl, nice accessibility of code objects in runtime from Lisp, etc., etc. What _was_ unique, though, was that intent that I am not ashamed to repeat: **Ruby tries hard to allow expressing things exactly how you think them**.

Even more than that: for me, if I paid attention, **Ruby always provided a way to _expand_ my way of thinking about things in a gentle and natural manner**. In my early twenties, when I started to work with Ruby, I was quite ignorant and haven't had an internalized `reduce` concept. But having it under the fingertips (on the same help page, in the same classes as obviously useful `map`/`select`/`group_by`), I quickly turned to understand and trust it, and a bit later, when I was exposed to "map-reduce" distributed processing paradigm, it felt just natural.

> When preparing to RubyConf's talk mentioned above, I came with what felt a good metaphor for Ruby: [Babel-17](https://en.wikipedia.org/wiki/Babel-17), the language from an old yet awesome Sci-Fi novel by Samuel R. Delany. That imaginary language, being learned, dramatically changed _how_ the language user is thinking about problems. I then was pretty fascinated to find [slides from Matz's 2003 OSCON talk](https://web.archive.org/web/20030811071449/http://www.rubyist.net/~matz/slides/oscon2003/mgp00001.html) named, wait for it: "The Power and Philosophy of Ruby (or, how to create Babel-17)".

## Evolution of the language

Ruby is not frozen in amber. It has a relatively short development cycle—new release every Christmas—and pretty informal development process: mostly just via discussions on the bug tracker. (There are different opinions whether the latter is a good thing, or somewhat more formalized process like Python's PEPs should be implemented; but anyway, the process is quite straightforward and it became more open in recent years.)

But the question I find interesting is _why_ language evolves, why should it? Maybe all we need is fix some unfortunate misconceptions, and then just focus on the implementation quality? "Ruby is _good enough_, just make it (faster/more async-friendly/optionally typed/choose yours)" is a popular stance. All in all, **if you believe something is good, if you are such a self-proclaimed fan of Ruby, why would you want it to change**, right?

Wrong.

What _is_ important about language (not only programming language) is it changes constantly—and **its change is the feedback to its own qualities**. As the language allows us to express complex, non-trivial ideas, some of those ideas become more widespread, more common-place, and you now can use them as _atoms_ to build new, more interesting ideas.

That is what sometimes signified with the (much valued in Ruby community) acronym **DRY**—Don't Repeat Yourself. But what's the deep meaning of DRY? It is not about sparing some keystrokes, neither it is just irrational aversion to copy-paste. **It is more about pattern recognition: once something (some way of doing things, some idea, some approach) becomes regularly repeated, it deserves its own way to say it unambiguously, atomically.** Descriptive paragraphs got represented by one word; new precise adjectives emerge to replace vague definitions.

In Ruby—Ruby-as-a-language—most of the noticeable changes serve the main idea of the _clear dataflow_: what follows what, what is subject and what's just an elaboration. Here are three semi-random examples of recent language changes, to illustrate the point.

### Example 1. `Comparable#clamp`

It is a frequent situation that after calculating some number (of items per page, of hours to schedule, of angels on the head of a pin) we need to make sure it is not lower than allowed minimum and/or higher than allowed maximum. In "mainstream quasilanguage" there are two typical ways to express it. First is just `if`s:

```ruby
value = calculate.that.damn.value(with, lot, of params)
value = allowed_minimum if value < allowed_minimum
value = allowed_maximum if value > allowed_maximum
```
...which is readable, but quite verbose (especially if it is the final step of a _really_ complex calculation). And another one, kinda shorter:
```ruby
[[value, allowed_minimum].max, allowed_maximum].min
```
...which is a common idiom, but not the most readable one. The greatest problem here is that the relations between value and limits aren't that straight: what is the "main" variable of the calculation, and what are just auxiliary thresholds, is lost in the monotonous list of values compared.

Also, neither of the two is really chainable to the previous calculation. But `#clamp` is:
```ruby
# As it was introduced in Ruby 2.4
calculate.that.damn.value(with, lot, of params)
  .clamp(allowed_minimum, allowed_maximum)

# As of Ruby 2.7, clamp allows using ranges, which
# communicate intention better:
calculate.that.damn.value(with, lot, of params)
  .clamp(allowed_minimum..allowed_maximum)

# ...and considering endless (2.6) and beginless (2.7) ranges,
# clamping from both sides, or from either one is always the
# same statement:
calculate.that.damn.value(with, lot, of params)
  .clamp(..allowed_maximum)
```

> Now, that would frequently be the case people will argue that the pair of `if`s is "more understandable". Yet the difference between `clamp` and `if`s is like the difference between those phrases "now, pour the broth; you need 2-5 cups" vs "...if you have less then 2 cups, you should make sure it is at least two; if you have more than 5 cups, you should make sure it is at most five". The situations when the latter, formal, is preferred, obviously exist; but a lot of times the former makes both _writer_ and _reader_ more productive.

### Example 2. `Object#then`

Chained processing of sequences were available in Ruby since the beginning of times, enabling coding style like already shown above:

```ruby
users
  .reject { |u| u.admin? }
  .chunk { |u| u.role }
  .each { |role, group| notify(role, group.sort_by{|u| u.registered_at }) }
```

But with singular value, chaining was not initially possible, so algorithms working with one object in several steps still only could've been written with lots of intermediary instance variables, or with nested calls:

```ruby
paragraph = "Hamlet: To be, or not to be, that is the question"

# verbose option:
author, text = paragraph.split(/:\s*/, 2)
data = {author: author, text: text}
payoload = JSON.generate(data)
client.send(payoload)

# nested calls option -- extract hash construction to a method "for brevity":
client.send(JSON.generate(construct_data(paragraph.split(/:\s*/, 2))))
```

"Nothing bad about it", obviously, it is the mainstream way to do things. But the experience with chaining on Enumerables makes (some of) us want the same approach to singular objects: with steps following each other in a logical way, and explicitly short lifetimes of intermediate variables. `#then` (initially introduced as `#yield_self` in 2.5, and _then_ renamed in 2.6) allows this:

```ruby
paragraph.split(/:\s*/, 2)
  .then { |author, text| {author: author, text: text} }
  .then { |data| JSON.generate(data) }
  .then { |payoload| client.send(payoload) }
```

This approach, while looking enlightening for some and "too weird" for others, has its pragmatic benefits: methods written this way are much harder to turn into spaghetti (at once, or eventually, while adding details) with unrelated calculation happening in between algorithm steps, a temporary variable from line 5 being suddenly reused on line 20, and three different things happening in one method.

It also combines extremely well with "safe navigation operator" `&.` introduced in 2.3, providing Ruby-idiomatic variety of "maybe monad"—safe calculation with some steps probably producing nothing:

```ruby
site.url
  &.then { |url| client.get(url) }
  &.then { |response| try_parse(response) }
  &.then { |data| data[:user_id] }
  &.then { ... }
```

I'll send the reader to my [two](https://zverok.github.io/blog/2018-01-24-yield_self.html) [other](https://zverok.github.io/blog/2018-03-23-yield_self2.html) articles, showcasing some fascinating and weird techniques enabled by this "trivial" addition to the language.

> One interesting step further this road of "logical chaining" was introduced recently with experimental "right-way assignment" in Ruby's master. It allows rewriting statements looking like

```ruby
value = a.long
  .multiline(...)
  .chain(...).of
  .calls { ... }

# now use value variable, but its definition is quite far
```
> as

```ruby
a.long
  .multiline(...)
  .chain(...).of
  .calls { ... } => value

# assignment is just before you'll really use it
```
> It is not that clear at the moment how much of code will become better with this addition—and whether this addition will stick (we have a history of discarding some experimental stuff before the release), but it shows the importance of data flow for Ruby's syntax.

### Example 3. Short blocks

As the block is one of the main ...well, building blocks of Ruby code, it was noted early in Ruby's life that their existing brevity still feels like there is a room for improvement in simple cases like `strings.map { |s| s.upcase }`. The only "non-trivial" thing inside that `map` is `upcase`, and _of course_ I am applying it to the item that is passed to the block! The need to name the item, just to repeat the name immediately, felt almost mockingly unnecessary.

In Ruby 1.8.7 (god, I am old! I still remember my feelings when this version came...), the simplest case was solved with adding [`Symbol#to_proc`](https://ruby-doc.org/core-1.8.7/Symbol.html#method-i-to_proc) which allowed to just call `map(&:upcase)`. The best thing about this witty, yet somewhat semantically questionable solution, at the time, felt that it hadn't required any new syntax, and showed Ruby's flexibility and extendability.

About ten years we spent chasing the great whale of "how do I shorten the block if it is _a bit_ more complex", like `files.map { |f| File.read(f) }` or `numbers.map { |x| x**2 }`. The first one could've been rewritten as `files.map(&File.method(:read))` which is kinda longer character-wise, but shorter conceptually (you don't introduce new variable name). BTW, this `method(:read)` is a rare obvious case of the price paid for flexibility of syntax: in Python and JavaScript, where `()` are mandatory to call a method, `File.read` would be a reference to method object, but in Ruby, it is already a call, so...

One of the ideas (which I was a vocal proponent of) was to make a new syntax for method references—so the first example above could be rewritten as `files.map(&File.:read)`—and later, probably, introduce some syntax for partial application, so additional params could've been passed as `files.map(&File.:read(mode: 'rb'))` or something like that. Between 2.6 and 2.7, the `.:` syntax was even merged to `master` for some time, but eventually got discarded. Instead, Ruby 2.7 introduced numbered block parameters: `files.map { File.read(_1) }`, and `numbers.map { _1**2 }`.

With all due respect, I felt, and still feel, very ambiguous about the change. It _makes_ everyday life better (after playing for some time with 2.7 I can say it for sure), but it seems a bit like "defeat", in a sense that new solution introduced for a long-standing problem shows no connection to other parts of the language, and has no visible "interesting consequences" making it more powerful and more interlinked with Ruby's consistency than it initially seemed. Nevertheless, the story of "shortening the blocks" and heated arguments around it demonstrates how the atomicity of data flow elements are in the focus of the language development.

---

The changes like those shown above are frequently met with criticism (often surprisingly hostile for reportedly "nice" community, especially when performed by influential community members). The mildest thing to be said is "pointless" or "nobody cares about", but many strike as hard as "unreadable mess", "unnecessary bloat" and general "ugliness".

Another interesting angle of attacks can be seen as based on a half-formed understanding of _why_ Ruby initially felt comfortable to the accuser. People tend to say that the language "was just like English!" (it never was nothing near), or "was so simple" (nope). The formulated above "tries hard to allow expressing things exactly how you think them" seems (to me!) to describe language's appeal better. But it implies that, in order to follow the language's evolution, one needs constantly present reflection on "how can I say it shorter and clearer" at a singular phrase level, which a lot of developers trade for "bigger picture".

I, nevertheless, testify that for me and the team I am working with, Ruby's recent changes are leading to the constant discovery of new ways to write "phrase-level" code, which, in turn, changes how we approach subsystems design (so the "phrases" stay as lucid as Ruby allows), which affects the whole architecture, in a good way.

And yes, **on the way to expressing more complex ideas in the clearest way, language tends to grow its vocabulary, and even new syntaxes emerge**. From where I stand, it is not an evidence of "unnecessary bloat"—but, as long as changes are consistent with the core (and I believe most of them are), it is rather the evidence of alive language that has all the characteristics to adapt to constantly shifting landscape without cheating on its values.

### It would all mattered if

So...

You have at your hand a language which has a small core that is easy to grasp, and rich discoverable vocabulary growing from the said core. The language which encourages and values design leading to high "phrase-level" readability, short "paragraphs" and visibly structured "chapters". The language which is open both for "pocking around" (what's here? how it can be used? where that came from?) and wild experiments (changing any behavior, even the most core one, designing task-specific sub-languages, porting idioms from other languages). The language whose main ideas are "clear flow of data, based on small powerful self-sufficient objects".

**Where is its place? Where it would be really good?**

Try imagining you the answer about some _abstract_ language characterized by properties stated above. You'll probably come to a list of applications at the same time quite obvious, and rather surprising.

The language we described probably should strive in the academy—both as a subject of teaching modern programming and as a tool of experimentation. In schools, for teaching and playing. As a classic "scripting" or "glue" language, the beloved tool of DevOps, sysadmins and automation engineers. The naturality of chaining and data flow should draw the attention of authors and users of data-processing libraries. Boilerplate-free DSLs design will lead to emerging of brilliant modeling software. And, of course, such a language will be the tool of choice for all kinds of R&D departments and early prototyping groups. How they can pass a language that tries so hard to allow expressing things exactly how you think them?..

Now, returning to the real Ruby situation, do you notice something strange about this list? **The one usage area that is weirdly (though reasonably) missing? Which is, by an ominous coincidence, also known as an only area where Ruby is massively used: developing of large production web applications.**

So, let's finally talk about...

## That elephant in every room

Let's talk about Rails.

The topic of Rails is to massive to cover it in entirety. **Rails, in my view, is a singular event in the history of IT.** Many will argue about the reason for its initial stellar popularity, and what should be considered the main factor: framework's design, language's quality, approaches to marketing, or "just the right time" in the industry? Rails can be seen as being highly beneficial, in a pragmatic sense, for the people who were brought to development through Rails projects, for the products it powered, for startups it made available. It was, probably, the turning point in the history of web development, when middle ground between (those days) "unstructured PHP pages" and "enterprise Java frameworks" was found, and that "middle ground" seemed a proverbial "10x" effective tool. One may also wish to focus on the disenchantment that followed and arguably led to a shift to explicitly limited simplicity of Go. Or, on the design and architecture that the framework strongly implied, and its relevance to the modern understanding of software engineering.

Books can be (and possibly are, or will be) written on all of those captivating subjects.

And yet, for the point of this narrative—narrative of Ruby-the-language and its fate—only two statements matter, and I put them boldly:

1. Rails is not Ruby
2. But Rails is the only Ruby we have

Let's elaborate, starting from the latter as more obvious.

### Rails is the only Ruby

As far as the most of (development) world concerned, Ruby=Rails. Ruby is the only mainstream language currently being categorized (be it job site, or bookstore) as "language/framework", like the two only go as a pack. Whether one starts to learn Ruby, falls in love with it, and evaluates their prospects to find some work with it; whether one, being another language's user, monitors "what new the neighbor community invented"; whether one has a prototype to be turned to startup—almost universal answer in Ruby-land will be "Rails". **There _are_ alternative web-frameworks, and non-web-framework related activity, obviously—we will talk about some of it in the next chapter—but all of it is dwarfed by Rails behemoth.** (Or let's stick to the "elephant", probably?)

It means, besides other things, that for many years, "the Rails way" shaped how Ruby is read and written, and new libraries and new language features are considered in the light of whether they can be used in the Rails environment, and whether they fit well into it. It is not unusual to meet professional and productive developers who studied Ruby half-heartedly, just enough to understand "the interesting stuff".

I am far from shaming "Rails-first" Rubyists. They are professional and productive, many of them are really good, some are overwhelmingly great. So, their approach to the language is good enough for them and good for products they deliver; but my sorrows are with the fate of the Ruby.

Because...

### Rails is not Ruby

Rails were _made possible_ due to Ruby: some group of people expressed how they imagined it would be convenient to write the software, basing on that days' understanding of a good approach to server architecture. Ruby, being powerful as it is, allowed to express what they thought, with grace. But the code that the framework expects of its users, the "Rails language" those users are expected to write in, is quite different in its principles and values from Ruby itself.

* Where Ruby values **consistency** allowing to mix discoverable concepts freely, Rails enforces undiscoverable **convention** for things to work: you should put things in those folders, and name them that way, and some _outside force_ will make them work;
* Where Ruby carefully designs for **expressiveness** (through the small core and flexible syntax), Rails is satisfied with **convenience**: classes are open, so let's add any method to any class to make things "look nice", like (in)famous `1.year` durations syntax, or [`Enumerable#including`](https://twitter.com/zverok/status/1103970067589533696). If something may look "nice", semantic can just go through the window—like `Time.current` which, _depending on a global `Time.zone` setting_, may return object of class `Time`, or of class `ActiveSupport::TimeWithZone`, "but that's not a problem", because `TimeWithZone` redefines its class `.name` to pretend it is named `"Time"`!
* Where Ruby prefers **small self-sufficient objects**, Rails is built upon giant "all-happening-here" controller classes, with instance variables magically passed to views and all "helpers" in the world somehow _helpfully_ available;
* Ruby's **blocks** are mostly avoided by Rails APIs; whenever there is a need to pass code-to-perform to a method (like ActiveRecord `scope` definition), Rails prefers lambdas—probably because they are "like in other languages";
* With a notable exception of ActiveRecord query interface, Rails APIs are not friendly (to say the least) to **chaining**, preferring singular mutation statements to chained transformations.

In general, code that Rails expects of its users is the code in that exact "mainstream quasilanguage" I am mentioning above: just classes-methods-variables-imperative statements (as if preparing the developer to "evacuate" from Ruby in any time... so to not become fond of its features and idioms).

One dreadful consequence of this state of things is that with years, **significant part of the community has reattached labels of "[magic](https://zverok.github.io/blog/2017-10-22-stop-magic.html)" or "(too) smart code" to the code that uses native Ruby features and idioms**. It is a frequent thing to meet nowadays in Reddit comments—"not everybody familiar with `Proc` concept, so I'd not use this in production". (Note that you'll hardly happen to hear "not everybody familiar with DB indexes, so don't use them"—but in contrast, one of the most foundational language features considered optional knowledge that hard-working people probably shouldn't spend time upon.)

One small illustration of this paradigm shift to an aversion to "Rubyisms" (as opposed to "mainstream-isms") is **testing frameworks story**. It goes like this.

Rails currently uses (and promotes) MiniTest library—which, to my knowledge, is the only library in the Ruby ecosystem which includes lost of quotes roughly saying "We are better than %main alternative%!" in every promotional material—every README, every release notice, even [gem description](https://rubygems.org/gems/minitest) ([RSpec's](https://rubygems.org/gems/rspec), to compare, just says "BDD for Ruby"). The _exact_ formulation of one of those phrases, quoted from praising blog post, is this:

> rspec is a testing DSL. minitest is ruby.

Now, if we look at this distinction, and then look at APIs both libraries provide, we'll notice that RSpec leverages Ruby's features (methods with blocks, some metaprogramming, small independent objects, chaining for matcher composition) as building blocks, while MiniTest provides the familiar XUnit _convention_: it is just classes and methods, but they should be named in a special way, and then _the framework will do the rest_. This convention emerged in less flexible languages for lack of in-language tools allowing to build APIs like "test(description, behavior)" or easy composition of "asserts"—the tools that Ruby _has_. But somehow, **following the "mainstream quasilanguage" approach is now the advantage—and not only the advantage, but the advantage that is described (and perceived) as "being more Rubyish"**.

That's how things are in the post-Rails Ruby landscape.

> Some time ago I covered those topics in more detail in a pair of blog-posts: [on MiniTest/RSpec](https://zverok.github.io/blog/2016-10-09-minitest.html) and on [tests expressivenes](https://zverok.github.io/blog/2017-11-07-on-culture-of-bdd.html).

And to end this lengthy tirade, there is a special say...

### About ActiveSupport

It is "bag-of-language-enhancements" library that adds a lot of new methods to Ruby's core objects, changes the behavior of other methods, provides new core-alike objects and APIs; and that's what generally power "nice" Rails-isms like `3.years.ago` or `1235551234.to_s(:phone, area_code: true)`. Nowadays, there seem to be only two polar reactions to ActiveSupport: universal adoration (because of nicety and convenience), or universal disgust (because changing core objects is an unholy abomination).

The thing is, the fact that Ruby classes are open, even the most core ones, is mostly beneficial to the language development. In earlier days, there were several popular libraries with all kinds of extensions (like [facets](http://github.com/rubyworks/facets)), and every keen Rubyists had their own (I had!). That was perfectly in line with Ruby's philosophy: sometimes you notice the repetitive concept, and you give a name to it, and yes, in the name of chainability and self-sufficiency of objects, it could make sense to attach it to core (or other library's) objects. Some of those methods have proven to be of general usefulness and made their way into the language (that's how we got `Symbol#to_proc` or `Hash#transform_keys`—both, and a lot of others, exactly from ActiveSupport); other just made code better in a domain-specific context (I [used some](https://github.com/zverok/drosterize/blob/master/lib/drosterize.rb#L83) when experimenting with math and images, for example).

So, the problem with ActiveSupport is not "thou shalt not extend core classes". The problem is that ActiveSupport turned to be "take it or leave it" whole, that was _forced_ to be used on production projects. You can't use Rails and avoid ActiveSupport's quirks; you can't just pick some concept from ActiveSupport (technically, you can, but it will either pull a lot of neighbor concepts or just wouldn't work without them: ActiveSupport and Rails code internally assumes all of ActiveSupport is present). And being just a _bag_ of things, it is the very irregular bag: some of the features are obviously useful, like aforementioned `Hash#transform_keys`, some underdesigned, like `Enumerable#including`, some encourage bad practices, like `Object#present?` ("fixing" the fact that Ruby have stricter falsehood concept than, say, PHP), some are semantically meaningless but look "nice", like `1.year`.

And that is omnipresent **existence of ActiveSupport—huge, unavoidable, chaotic—that made Rubyists aversed to the idea of "fixing" some language parts (temporary or for good) with core extensions**. That drowned the idea of hygienic extensions (refinements) which _seemed_ promising ("add only features that particular module needs without polluting other modules"): Rails folk doesn't need them, "anti-Rails" groups know for sure any change to core object is "bad, like Rails".

Funnily enough, the whole idea of _sometimes we can extend core classes for expressiveness_ seems to be semi-dead in Ruby community: even in Rails-first projects, what is provided by ActiveSupport is taken for granted, but any custom changes are seen as a "bad taste".

### On the other hand

It would be stupidly unfair to state that Ruby life outside Rails doesn't exist at all. It totally does!

First, there always been alternative web-frameworks like Sinatra or Grape, or alternative ORMs like Sequel, and other scattered counterparts to Rails ecosystem. They are relatively well-known in the community (and, arguably, "borrowed from" from time to time, making "the same, but better" options available under the Rails umbrella).

Every now and then, a new gem not being "just new Rails plugin" (and even not "alternative to some Rails part") got released, and some of them are surprisingly cool.

To name **a few "breeding grounds" of interesting things**: there is [SciRuby](https://github.com/SciRuby) organization (which I proudly belonged to, and then shamefully abandoned) trying to bring a decent base ecosystem for scientific research; there is its Japanese counterpart, grouped around [Numo](https://github.com/ruby-numo) set of libraries; there are [Gosu](https://www.libgosu.org/) and [Ruby2D](https://www.ruby2d.com/) game engines, [Kiba ETL](https://github.com/thbar/kiba) data processing engine; and also [Andrew Kane's](](https://github.com/ankane)) fascinating single-handed streak to provide Ruby wrappers for a wast set of machine-learning libraries. (I am naming here only some of the projects I am aware of, and only the recently active ones. Also, there are _rumors_ that lot of interesting Japanese non-Rails projects do exist but don't have PR materials in English; to name one I've become aware [at RubyKaigi'17](https://rubykaigi.org/2017/presentations/takaokouji.html) is [Smalruby](https://smalruby.jp/) Scratch alternative.)

What is saddening when we try to talk about all this richness, is most of the projects are falling or already fell into a "negative feedback loop": a lot of work is done → (almost) nobody notices or uses the project → it doesn't get integrated into bigger ecosystem → author(s) become disenchanted and stop to evolve the project → nobody notices. Say, in Python, pandas is a default go-to solution whenever one needs to work with table data—even if it is just "load CSV and make a report for a client" in some web startup. But in Ruby, it is totally possible you haven't even heard of [Daru](https://github.com/sciruby/daru) dataframe library—which is not that mature, but quite feature-rich Ruby's version of Pandas.

---

One particular recent development in the Ruby ecosystem (that the reader probably already thinks I've missed) is particularly hard for me to discuss. This development is related to a loose **set of libraries, frameworks and plugins, some developed by the same people, some just depending on each other, some sharing ideas and approaches: Hanami web framework, dry-rb set of libraries, ROM persistence, and (somewhat aside) Trailblaizer "high-level architecture"**.

First thing first: I have a deep respect for the people involved, and a strong belief they are enriching the community with ideas, approaches, and views; with exposure to ideas those new libraries are built upon, one may become if not a devotee of these libraries, but at least more aware of some _saner_ approaches to design and code structure. With some of the dry-rb community, I regularly communicate and exchange ideas, so now I struggle to explain my disagreement with some of their... not even ideas, but, I don't know, core impulses and assumptions?

So, what always felt weird for me in all of the aforementioned projects is this: they struggle hard to have a saner design (than "default Rails") on a "top-down" scale—whole application and general modules design. But on a language level, on a phrase level, they are inheriting the approach used by Rails: defining their own "language-inside-language", which, be it consistent by itself, wouldn't necessarily be consistent with the "rest" of the Ruby, or aligned with its design.

To give just one brief illustration: dry-rb quickly adapted to Ruby 2.7's new pattern matching (as it is seen as a powerful and unavoidable new feature, close to those of functional languages, inspiring dry-rb's development), but at the same time has no respect to `#then` or, in general, Ruby's chaining—instead redefining Ruby's `yield` semantics to [mimic do-notation](https://dry-rb.org/gems/dry-monads/1.0/do-notation/) of purely functional languages (which, ironically, was invented as a tool to expose "flow" in an imperative-alike way).

> As an aside note: interestingly enough, a lot of "Rails-alternative" libraries also seem to be unable to walk far enough away from "convention/convenience-based design": for example, Hanami, just like Rails, has its conventions about "what should be in which folder" to establish relations between classes; Hanami [Router](https://github.com/hanami/router)'s reportedly "Beautiful DSL" allows syntax like `get '/flowers/:id', to: 'flowers#show'`—with what exactly `'flowers#show'` mean being decided by "outside forces". On another account, ROM, just like other Ruby persistence libraries (and _unlike_ persistence libraries of almost any other language), prefers to "infer" what columns the relation has, instead of complying with a user-written readable description of expected data structure.

To wrap it up, we may also notice that those libraries—the most vocal "new things" in the community—are still staying in a narrowest "like Rails, but better" domain; even in a far-fetched fantasy where Hanami and ROM and dry-rb are the most popular Ruby libraries in the world, there would be no new answers to big questions: _what's Ruby for_? and _what's interesting happening in Ruby world_?

Which moves us to the biggest question of all:

## Where are we, now?

**Is Ruby "dead"? The answer, as of 2020, is definitely "no", but the intonation of this "no" is quite different depending on what angle you are taking.**

For example, one may pragmatically ask: is it easy to find a job with Ruby? Well, it is _not that hard_ in large cities of my home country (Ukraine), considering how many outsourcing/outstaffing companies we have; it is probably not-that-hard in a lot of other countries, starting with US (the biggest startup hub of the world) and Japan (Ruby's home country). Yet the market is definitely shrinking, and, for what I know, land at a _junior Ruby developer_ job is much harder today, as the majority of Ruby-powered projects are old production codebases expecting a new developer to bring a lot of competence at once, or lean startups that just don't have enough resource to spend on growing their own professionals. And it creates a feedback loop that will strike hard in years to come: less career-start-friendly market → fewer new courses and bootcamps → fewer fresh minds and ideas → fewer people to possibly hire for new products → fewer new products started in Ruby → ...

Tell yourself whatever you want about "productivity plateau" we allegedly "reached", list whatever "major products (still) written in Ruby", but in the broader picture of the software development world, **I'll tell you what Ruby eventually became: _Irrelevant_**.

From the outside of the community, what was only a few years ago infuriating "remember that big thing, Ruby? it is dead" now given place to _not mentioning Ruby at all_ when talking about the actual picture of technologies and the job market. Just yesterday I've stumbled upon some YouTube video of local IT training courses guru, who had discussed Python's usage, future, benefits, and flaws, and he just casually said at one point (talking about which language takes which market share in different domains) "...oh, and Ruby? I honestly don't know, my Rubyist acquaintance says it is still not dead, but I haven't heard what's happening with it, for a long time".

That's it. In a space of ideas, Ruby-the-language almost doesn't matter anymore, for a long time not making it into the broader community's perception with "cool new tool (written in Ruby, btw)", "interesting approach to coding (which Ruby enables, but you can try it in your language)", "new state-of-the-art library (which, it so happens, written in that weird Ruby, but it IS awesome)" or something like that. Ruby took place of "PHP of the 2010s": yeah, it is one of known tools-of-choice if you just "need work done", and it may power the half of the internet, but it is not something that is _interesting to think about_.

And Ruby—Ruby-the-tool of thinking—deserves so much more.

#### What is next

So, one may ask at this point, "what's your proposal"? Why am I writing this all, do I have some ridiculous "plan to save Ruby"?

I don't.

**For quite some time, I hoped for a miracle.** It could've been something like what happened with Rails—new library so different of what everyone had that people start to study Ruby just to use it; just it should've been "done right" this time: exposing language's concepts and idioms instead of parasitizing on them. Another kind of possible miracle was that some huge company like Google will choose Ruby as their "go-to scripting language for everything", and will use its human and marketing resource to change language's image in the minds of industry players. Neither seems to happen (though, either still might... it is not an unseen event for language seemingly dying to find a new momentum suddenly).

> Somewhat embarrassingly to acknowledge, I've once had started a ["miracle" project](https://github.com/molybdenum-99/reality) myself, yet despite some interesting results it never flew high enough. Most probably it is due to my naivete and lack of energy to develop and promote it much faster and more aggressively, or maybe the idea itself was shitty. It made a few good [conference](https://www.youtube.com/watch?v=oqsX8kNq94I) [talks](https://www.youtube.com/watch?v=x9GePP3B0oE), at least.

I expect the current state of things in Ruby land to be more or less stable for upcoming years. Maybe Hanami and ROM and dry-rb will take some share of usage, though I find much more probable that Rails ecosystem, as it happens, will cannibalize their ideas once they will become popular enough (so brace yourself for ActiveMonad in Rails 8).

For a living, I still work on Ruby-on-Rails app, though our team nowadays has most of the code structured outside the framework, and follows rather Ruby's than Rails' idioms (and I am well aware that working with such a team _is a priviledge_). I don't expect it to change soon: all in all, I spent _most of my conscious career_ with Ruby, and total breakup is unimaginable at the moment.

For my hobby projects, though, I am currently switching to Python. The work is not fast—partially because I am lazy, and partially because of my mindset being quite different from Pythonic. If it was for the language, I'd happily stay with Ruby... But in my area of interests (open data, text processing, dictionaries) making OSS in Ruby feels like shouting into the void (or, rather, like growing cacti in a pine forest: your plant is OK, and the forest is OK, you are just in a wrong ecosystem).

Looking at other languages as a potential career directions—mindset-wise, I feel some bonding with Elixir (obviously) and Rust—which, while initially feeling quite different than Ruby in almost every aspect (statically-typed performance-oriented system language without classes!), at a phrase level seem to share Ruby's approach to data flow, to operations belonging to their types, to chaining, and to general love fore expressive code. Crystal (which started as "statically-typed Ruby", and then evolved on its own) has some appeal, but I am not quite sure it will find target audience large enough: for Rubyists, it is unnecessary, and for everybody else—too like "that thing, Ruby, that seemed promising... a decade ago".

I beg your forgiveness for such a downer ending. Maybe trying to make you feel as sad as I am is egoistic. But I, for one, prefer to have a clear and coherent picture of the world around me, and just wanted to share this picture.

I just hope that sometime in the future, when the history of programming languages will be studied as a _cultural_ history and not just of that of engineering tools, Ruby's place in changing the world around us, and minds of those present at the time, would be properly acknowledged.
