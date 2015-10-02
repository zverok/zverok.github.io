---
layout: post
title:  "Designing A Good Library or Readme-Driven Development"
date:   2015-10-02 15:24:30
categories: ruby gems design
comments: true
---

**TL;DR**: Readme-driven development (writing README and usage examples
**before the code**) is considered a useful way for designing public
interface for new library. Tool for automating readme-driven design
proposed, named [dokaz](https://github.com/zverok/dokaz).

## The problem

When you are starting to create a new library (either for internal needs
or for public release as a gem), the toughest thing is oftenly designing
a good public interface. Classes, modules, methods, service objects, return
values, exceptions and more and more.

Making things natural and easy to remember, providing consistent,
transparent and useful behavior, keeping both integrity and loose coupling
of concepts—all of those were never been easy, but it is possibly the
hardest when you just starting to create something.

## Existing approaches

One of possible approaches is **following the internals**. You do, for
example, some new database-abstraction gem, or data processing one, or
wrapper for some complicated web API, and you just start this car with
an engine and hope that seets and dashboard and doors and steering wheel
will emerge somehow at some point. And even if those "seets" and "doors"
would be ugly, or uncomfortable, you don't give a damn, as engine is
powerful and fast.

Shortcomings of this approach is obvious: unless your domain is truly
unique (like first and the only gem for accessing some exotic database),
nobody like to use your work. Or at worst nobody'll ever understand what
you did and how is it important.

More reasonable approach we are all familiar with is **behavior-driven
development**. Before pulling right method names and class dependencies
out of nowhere, you are starting to write specifications with real
use-cases for (non-existent still) library's public interface.

> I'm talking here only about RSpec or MiniSpec-like approach, where you
write just specs code to show the reader (and yourself) how the library
code [expected to] work. I'm really assured tools like Cucumber (providing
hi-level description of functionality in natural languaes) are next-to-useless
in case of library interface design.

> Ironically, RSpec itself uses Cucumber to specify its hi-level behavior.
You can look at some of [official documentation](https://www.relishapp.com/rspec/rspec-core/docs/example-groups/shared-examples)
to see all of those `Given`s and `When`s and `Then`s and decide yourself
whether it provides some "added value" of clean behavior documentation.

While practice of behavior-driven development proved to be useful and
reliable technique, it have its own drawbacks when it comes to interface
design. First, it tends to be a bit too verbose for "thinking in code",
tends to draw you into details and corner cases and fixtures too early.

Second, it fails to provide design document with intended use of the
library. At times, specs and behavior-driven development are praised as
the way of documenting "how it works", but for complex library you'd rather
prefer some well-written README and usage examples than diving right into
specs.

## Proposed solution

And here comes **README-driven development** (I've thought out the term
myselft, but of course I'm [not the first](http://tom.preston-werner.com/2010/08/23/readme-driven-development.html)—this
article explains the same concepts I am trying to). The idea is simple:
_you just write small examples of code using your future library, and
explanations in natural language_, like you already have created the library
and planning to release it. This way, trying to explain to invisible
reader how cool it **should** be some day, you are likely to come with
much better and cleaner interface than "pure imagination"-driven or
behavior-driven design.

> **NB**: of course, it all works only if you can write good explanation
(and decide whether explanation is good). Though, I'm pretty sure you
should not design complex library if you can't explain it.

### Real-life example

When I have been creating first version of [infoboxer](https://github.com/molybdenum-99/infoboxer),
there was two consecutive parts of the library:

1. Wikipedia markup parser, and
2. Navigation and information extraction from parsed data.

For those two parts, signs of successful development differ dramatically.
For (1), the only public interface is something like `Parser#parse`, and
it either parses complex text pages successfully, correctly and quickly, or not.
It was hard to program, debug and profile, but easy to expose as a public
functionality and easy to explain.

The (2) task is completely different. Navigating through complex and
heterogenous data structures can be really puzzling, and one of my
hugest concerns was how to make it natural and consistent.

You can estimate what I did by yourself at Infoboxer's [showcase](https://github.com/molybdenum-99/infoboxer/wiki/Showcase).
For me, it is still far from ideal, but concise and consistent and useful.

The secret of how it was done is exactly what I'm talking about: almost
all pages of Infoboxer's [wiki](https://github.com/molybdenum-99/infoboxer/wiki)
were written before the final design of navigation module. It was really
helpful and almost enlightening experience.

And also, there was some cool by-product of this approach I want to share.

## The tool

[dokaz](https://github.com/zverok/dokaz) is a tool (in early stages of
its life cycle) allowing you to automate _readme-driven development_, as
well as just test your docs with usage examples and showcases for correctness.

Dokaz uses your Markdown documentation (README, GitHub wiki, whatever)
with integrated Ruby code as a loose specs for library you design.

You just run `dokaz README.md` (or `dokaz path/to/any/docs`) and the
tool does following:

* extracts all blocks of code surrounded by <tt>\`\`\`ruby</tt>
  and <tt>```</tt>;
* runs those blocks one-by-one and checks their output.

Two main modes of Dokaz is "spec" (does it work?) and "showcase" (what
it returns/outputs/raises?).

"spec" mode is useful for quick check when you refactor library or docs,
and can be easily integrated into your CI system. Its outcome is like
this, like other testing tools:

<img src="https://raw.github.com/zverok/dokaz/master/screenshots/spec.png" alt="--format spec"/>

"spec" mode not only check if some example in your docs fails, but also
can check sample output value you provide in your docs. For example,
this code in README.md:

{% highlight ruby %}
10 / 3
# => 2
{% endhighlight %}

...will also fail with message like "Expected sample output not match real".

"showcase" mode is useful for seeing by yourself what your documented
examples output:

<img src="https://raw.github.com/zverok/dokaz/master/screenshots/showcase.png" alt="--format showcase"/>

It also useful for inserting real and accurate return values into your docs
(future versions of dokaz should have option for updating the docs with
real output values automatically).

-

Dokaz is still on early stages of development, and I'd be happy with ideas
and opinions of how this tool should develop further. I'm confident as
for quality of idea, but still considering the good implementation. Please
feel free to drop me a line at comments, or by email zverok.offline@gmail.com,
or issue in [dokaz repository](https://github.com/zverok/dokaz/issues)!
