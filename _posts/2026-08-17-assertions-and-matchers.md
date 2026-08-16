---
layout: post
title:  'Thinking about tests: assertions and matchers'
date:   2026-08-16
description: >
  Why some of us are still bothered about the way we write tests and what the "matcher" concept has to do with it.
categories: ruby rspec testing
comments: true
---

**Why some of us are still bothered about the way we write tests and what the "matcher" concept has to do with it.**

During most of my Ruby career, I was that unpleasant person who honestly enjoys writing tests and is frequently concerned about the ways we write them.

This means treating unit tests like the rest of the codebase: like something that is supposed to be _read by humans_ and something that should be written _efficiently and expressively_. Basically, like something that wouldn't be boring and disgusting to read and write.

This also means that I find it useful, once in a while, to stop and reflect on _why_ we write test code the way we write it. And can this be improved?

> Let's move the elephant in the room from our way at once: from my point of view, this way of thinking does not become obsolete due to AI agents, who "can write any number of tests without being bored." If anything, short, readable, and expressive code means _more_ in this age. I extend this argument a bit in the last section of the post.

So, thinking about "how do we write tests" leads to a mass of related, tightly intertwined questions: How does the typical test in the codebase look? How hard is it to write a new one? How hard is it to read and maintain an existing one? How does it affect the overall codebase maintainability?

And how the design of the test framework and its utilities affects all these considerations and is affected by them?

To understand how this way of thinking might be useful, let's look at the lowest level of the test: **just one check, or assertion.**

## Starting from the beginning

Let's perform a small "from the first principles" journey (bear with me!).

How do we check that the code we just wrote does what expected of it?

It starts easy when you have just a small amount of new code to test: one script, one utility function, one small class, things like that.

The first, most naive approach, is to just run the code, see what it outputs (prints to the console, renders in browser, or makes any other user-visible effect), and compare it visually with what you'd like it to output. Frequently, this is enough for a quick prototype or a throw-away script: just write the code, run it, say "aha" or "oh no," tinker a bit till you are happy, and then move over.

Obviously, it becomes tiresome for any non-trivial code, or one that turned out to be not short-lived: you eventually need it to do more and more things under more and more circumstances, and just manually checking "my new case is working, and the old one is not broken" becomes a burden.

And so you need some kind of a "test script," with "when we run it like this, that happens" codified. This "check what happens" should be easy to write, and it should provide useful feedback: "in this part of the test script, this assumption turned out to be incorrect." This is what we frequently call "test assertion", "test check," or "test expectation," depending on the context and tools used.

**The API to assert things** is one of the first services that any test tool provides. And one that, in my opinion, affects the test library usage and developer's thinking process.

> Of course, we can go to higher levels to think how we organize many tests and groups of tests. And also how do we run them -- a lot of decisions can be made here: order of tests, their independence, running in parallel, rerunning only a subset. All of this unquestionably affects our thinking, the design of our tests, and the design of our software. But it all starts with one test -- and one assertion.
>
> Not everyone considers "how do we write one test" to be of any importance. In the "architecture-first" thinking, the particular code at the level of singular "paragraphs" and "phrases" -- its brevity, expressiveness, or ease of modification -- is frequently brushed off as insignificant. My way of thinking on ease of development and maintenance of software, though, gives this "low" level significance. I will follow this line of thinking for now without further argument (which I [expressed](https://zverok.space/blog/2023-10-02-syntax-sugar.html#how-i-think-about-the-programming-language) many times already). And I ask you to be with me here, if only out of curiosity, "how some of us approach what they do."

## So: a single assertion

The simplest of such APIs is `assert(expression)`, with `expression` expected to return either truth/truthy value (the test passed) or false/falsy value (the test failed).

Frequently, this `assert` is even a part of the language itself, or its standard library -- to be used as a debug or production guard against "impossible conditions."

In testing, it might be used like this (usual "arrange, act, assert" structure)[^1]:

[^1]: The examples below, if the language is not specified, have Ruby syntax/formatting, but use it to represent some generic "quasi-language."

```ruby
arguments = prepare_arguments()  # arrange
result = execute_code(arguments) # act
assert(result == expected_value) # assert
```

Here, only the last line has any calls that should be provided by a test library. Or, if it is a "core language" assertion feature, the only role of the test framework here is to provide a hook/handler for the signal that failed assertion produces (by raising an exception or other means).

Throw in some API or agreement how you put such fragments in separate tests and how are they executed (the common approach: every method/function in `tests/` folder files that is named `test_something` is run separately) -- and this is already enough for the smallest, yet useful, "testing library."

If not provided by the language itself, such an `assert` can be trivially implemented as a method that just throws an `AssertionError` exception if the passed argument is falsy. The exception's backtrace will point to the failed line, giving enough basic information to debug. To make it a bit more friendly, a message argument can be added to the `assert` signature, allowing the developer to write:
```ruby
assert(result == expected_value, 'Explanation of the case tested')
```
...and adding the explanation to the failure message.

In fact, the first JUnit library[^2] was not much more than this.

[^2]: The history of computer science is a fast river. I was _not_ "there, 2000 years ago" at the age of the first testing frameworks. So my description of their development is more an archeology study and might be quite shallow. The JUnit behavior I am describing is from the earliest version I've found online: JUnit 1 as can be seen [here](https://www.eg.bucknell.edu/~cs475/F2000-S2001/hyde/JUnit/). To the best of my understanding, it is the _first_ widely used testing library, with its predecessor/inspiration, the first version of Smalltalk's [SUnit](https://en.wikipedia.org/wiki/SUnit), being at that time more of a thought exercise/example of the approach.

But still, there was _some_ more. Even in the most basic case -- the "result should be equal to the expected outcome" -- if the assertion fails, "it was not equal" is not enough useful information; "but what it _was_" would be the immediate follow-up question. So the logical next step is to have a small utility wrapper:

```ruby
assert_equal(result, expected_value)
```
...which compares two values and, if they aren't equal, renders something informative, like `"expected: 1, was: 2"`.

> A pedantic note: In JUnit, the declared order is actually `assert_equal(expected, actual)`. Modern JUnit and some of the JUnit-derived libraries preserve this order; others switch to `(actual, expected)`; still others refuse to confine the developer and use neutral naming like `(left, right)`. Finally, there are those that, like LuaUnit, make it [a configuration option](https://luaunit.readthedocs.io/en/latest/4_reference_doc.html#equality-assertions). While this might seem a "boring nuance," we show that this order decision matters further in the article.

Once we have this helper function, one might think of other APIs in the same line of reasoning: assert that value is that of the expected type (and properly render what actual type it was otherwise); assert that it is a collection and has an expected number of items; assert that it includes some specific structural subpart, and so on.

These bunches of `assert_something` APIs are still, for all I can tell, the most popular testing API. All across the programming languages spectrum, the "xUnit-style" testing library is frequently the default/most used one, if not (in newer "batteries included" languages) part of the core distribution itself.

> To work on this article (and, hopefully, the next parts), I made a private quick comparison document of many test libraries throughout the mentioned "spectrum" of modern programming languages. I am thinking about publishing it as a separate post/document, as it turns out to be of interest for any living soul other than me.

Still, another style exists, and it is almost equally widespread. And, as far as I am able to research, its popularity (if not the style itself) had originated from Ruby.

## "Behavior-driven development" and the invention of a matcher

Around 2005-2007, there were a lot of blog posts ([here](https://dannorth.net/blog/introducing-bdd/) is one of the definitive ones) discussing the ideas of "behavior-driven development" -- mostly, a new way of thinking, or rather a shift from the familiar ways. While the initial approach to test-driven development made the code author think in terms of "how can I write _the test_ for my non-existent yet code, that will help me design it," behavior-driven development suggests thinking in terms of "how can I _describe the desired behavior_ of non-existent yet code."

The difference might sometimes seem subtle, though it was believed that this shift of perspective might mean a lot.

One of the influential articles demonstrates how subtle the shift might seem: 2005's [A new look at test-driven development](https://web.archive.org/web/20090419153557/http://techblog.daveastels.com/2005/07/05/a-new-look-at-test-driven-development/). Dave Astels makes a big distinction between "old" test-driven development with `assertEquals(expected, actual)` and "behavior-driven" `shouldBeEqual(actual, expected)`.

> From today's point of view, these two APIs might seem effectively indistinguishable. A lot of software developers today would frown at almost any syntax/API discussion as not making a difference for a "professional engineer." But in those old times people considered that choice of singular "words" and the shape of a "phrase" in the programming language affects the writer's thinking. I still believe this, and that this belief is one of the things that make me efficient.

In the article, Dave Astels also says that in Smalltalk, and possibly Ruby, it might be even more natural with (Smalltalk's syntax):

> `actual shouldEqual: expected` or `result shouldBeNull` or `[2 / 0] shouldThrow: DivideByZeroException`.

Soon the Ruby's RSpec library [was born](https://blog.davidchelimsky.net/blog/2007/05/14/an-introduction-to-rspec-part-i/), directly inspired by Dave's article and with significant contributions of Dave himself. The first version provided almost exactly the same syntax Dave had described:

```ruby
actual.should_equal expected
```

It went from several hard-coded `should_<something>`-methods like `should_equal` in 0.0.1, through metaprogrammed syntax variants like `should.do.something` in 0.0.4, then `should_do_something` (but now parsed into words metaprogrammatically), and, in version 0.8.0 (February 2007), introduced this[^3]:

[^3]: [Here](https://www.stevenrbaker.com/tech/history-of-rspec.html) is a much more precise first-hand account of the early days of RSpec by Steven Baker, its creator.

```ruby
actual.should eq(expected)
```

> Ruby's flexible syntax allows omitting method call parentheses in unambiguous situations. The code above is an absolute equivalent of
>
> ```ruby
> actual.should( eq(expected) )
> ```
> ...so `should` here is actually a method expecting one argument -- a **matcher**. But it also allows omitting more parentheses and writing the same code so `eq` would look like an operator between actual and expected:
> ```ruby
> actual.should eq expected
> ```
> The meaning stays the same: the **matcher object** produced with `eq(expected)` is a separate entity from `.should`.

These were the days when Ruby's flexible syntax and the APIs it allowed to envision were inspiring other communities to try to implement something similar in their language. This has happened with Rails in general and its various APIs, this also has happened to RSpec. In some languages the idea was ported more or less straightforwardly, for others, it required stretching the syntax tricks just for the sake of mimicking the `actual should matcher` formula.

> For a quick example, Python's [should-dsl](https://github.com/nsi-iff/should-dsl) had provided `actual |should| expected` by defining a special `should` object which had an operator `|` defined on it in a way that made the whole statement produce an expectation. I don't think it is widely used.

Ruby has its classes open by design (you can continue the definition of any class -- even a system one -- in any program). And in the Ruby community in those days, "just throw what you need into the core class" was a popular development approach. So `.should` method was just added to every `Object` in the first RSpec versions.

Eventually, "extend every object just to write somewhat nicer code" fell out of popularity in the Ruby community, and in a few versions [RSpec settled](https://rspec.info/blog/2012/06/rspecs-new-expectation-syntax/) onto the "wrapper object" API, that didn't require unconditional extending of the base `Object`:

```ruby
expect(actual).to matcher
```

Here, `expect(actual)` creates a wrapper object with methods like `.to(matcher)`/`.to_not(matcher)`, which is almost as compact as `actual.should`, but doesn't pollute every object in existence with test library's methods.

## Matchers in the wild

The API akin to this "new" one, or RSpec's initial `.should` syntax, can be nowadays found in many languages and testing libraries.

For example, Go's [Gingko](https://onsi.github.io/ginkgo/) uses `Expect(actual).To(Equal(expected))` which is exactly RSpec's API, save for Go's stricter punctuation[^4].

[^4]: As an aside note, I believe that Ruby's punctuational flexibility was a source of inspiration for many APIs, including the one we discuss. While technically being just "methods and objects," RSpec's initial `foo.should eq bar` _looks_ like a completely boilerplate-free test-specific sublanguage, and the early stages of invention and adoption of such a language was most probable in Ruby.

Both Scala and Kotlin enjoy their "infix function" notation for even greater punctuation flexibility than Ruby allows: [ScalaTest](https://www.scalatest.org/) with `actual should equal (expected)` and [KoTest](https://kotest.io/docs/assertions/matchers.html): `actual should eq expected`, respectively.

In JavaScript, two prominent libraries -- [Jest](https://jestjs.io/docs/expect) and [Chai](https://www.chaijs.com/guide/styles/#expect) -- both provide `expect` notation, but with a twist. In a bit weird turn of events, they both call their solution "matchers" while not providing separate "matcher objects": it is `expect(actual).to.something(expected)` in Chai and `expect(actual).toSomething(expected)` in Jest. While visually similar to other RSpec-like libraries, this approach has a significant difference: "matchers" here are not separately constructed arguments, but methods of a wrapper object that `expect` produces. As a consequence, creating custom matchers (which we'll discuss a bit later) requires extending that object, not just creating independent objects that correspond to matcher's interface. Lua's [Lust](https://github.com/bjornbytes/lust) follows the same road.

**It is worth noticing** that to use a matcher concept doesn't necessarily require the whole formula with "should" / "expect to". In fact, for all I can dig up through the developer thought archeology, the concept of matchers might've first emerged in the Java's [Hamcrest](https://hamcrest.org/JavaHamcrest/) library -- which even once became a part of JUnit 4 (but was later separated again), and then ported to [many other languages](https://hamcrest.org/).

Hamcrest just introduced the matcher concept into the familiar `assert<Something>` API with `assertThat`:

```java
assertThat(actual, equalTo(expected))
```

Several other libraries follow this or similar structure, like:

* C++ [googletest](https://google.github.io/googletest/) with `EXPECT_THAT(actual, Eq(expected))`;
* it's [Rust port](https://crates.io/crates/googletest) with `expect_that!(actual, eq(expected));`;
* C++ [catch2](https://catch2.org/) with `REQUIRE_THAT("Hello world", StartsWith("Hello") && EndsWith("world"))`;
* C# [nunit](https://docs.nunit.org/) with `Assert.That(phrase, Does.Contain("World"))`.

Considering the different choices of wording, we might say that the line is blurry here: whether, say, googletest should be considered "BDD-like"? If you squint out the punctuation, the difference of `expect(a).to(matcher(b))` vs `expect(a, matcher(b))` might be said to be "in the eye of the beholder." And the same can be said about the importance (or lack of it) of the word choice: "assert" vs "expect" (vs "require" vs "should" vs ...), for one writer, might shift the perception of their writing, and for another one, be just a familiar "sigil" they never much think of.

Finally, some testing libraries use the "matcher" concept even if it isn't directly related to assertions, like C#'s mocking library Moq, that introduces [matchers](https://docs.educationsmediagroup.com/unit-testing-csharp/moq/custom-matchers) only to specify mocked method arguments.

<div class="one-ukrainian-thing" markdown="1">
  <h3>A postcard from 🇺🇦</h3>

  **This interruption won't be long.** I just want to remind you we are still here. I am in Ukraine, still serving in the army. Russia still tries to erase us.

  In the last months, there was a glimmer of hope: while we are far from "winning," at least some of the Ukrainian strategies seem to make the war continuation more and more painful for Russia.

  In response to this (slight) change of the chances on the battlefield, Russia increased the barrages of ballistic missiles attacking our cities indiscriminately... And our dearest partners suddenly critically _decreased_ the number of anti-missile munitions that allowed us to handle that. As if _somebody_ really doesn't like to see Russia having a slight chance to lose.

  Oh, and one more thing: this summer, Russians increased deliberate targeting of the Ukrainian book industry (large book warehouse, printing houses and so on). Just in one strike on August 1, [8 million books](https://www.rfi.fr/en/international/20260813-a-war-on-words-why-russian-strikes-are-targeting-ukrainian-publishers) were destroyed. I wonder what it says about the goals of the war. And who should see that. UPD: And while I was editing the final version, Russian missile burned the largest Kyiv book market overnight.

  Let's proceed with the rest of the article.
</div>

## The value of matchers

It is easy to dismiss the examples with matchers as "syntax/API nuances" which doesn't change the general approach to writing tests. And, indeed, one can easily use matcher-enriched testing libraries to write tests in the exact style / order of `assert_equal` ones.

But when the concept of "assertion operator" and "matcher object" are separated, it becomes much easier (both mentally and technically) to construct new expressive checks by combining existing matchers or designing new ones.

### Combining matchers

In Ruby's RSpec, matchers can be combined with logical operators and nesting.

Here are examples of the logical combination:

```ruby
expect(user.email).to be_a(String).and match(/.+@.+/)
expect(user.occupation).to be_an(Occupation).or be_nil
```

Here, `be_a`, `match`, `be_nil` are separate matchers, and `matcher.and(matcher)`/`matcher.or(matcher)` produce new ones, which perform both checks and produce a clear resulting message.

Another way of the combination is nesting:

```ruby
expect(emails).to all be_a String
# or, with all parentheses Ruby allows to omit:
expect(emails).to( all(be_a(String)) )

expect(response).to include(
  name: starts_with('Admin'),
  occupations: instance_of(Array),
  badge: have_attributes(title: 'Hero')
)
```

...again, `all`, `be_a`, `include`, `starts_with` and so on are separate matchers, that can be put together to express a complicated _expectation_ of the subject under test, in approximately as many words as the most straightforward human description.

### Designing custom matchers

Most of the test frameworks that use teh "matcher" concept explicitly support and encourage designing your own matchers. Usually, it would be an object with a simple interface (frequently, consisting of three methods: `match`, `failureMessage`, and `failureMessageWhenNegated`, or their equivalents), with their implementation being pretty trivial.

So, for example, in our large production Rails codebase we use more than a dozen of custom matchers (not even counting several third-party matching libraries). They allow us to write code like

```ruby
expect { some_code }
  .to create_record(User)
  .with(name: 'Mary', email: 'mary@example.com')
# or:
expect(response).to be_successful(200...300).with_json(reloaded: true)
# or:
expect { some_code }
  .to call_service(InternalService)
  .with(some: 'arguments')
  .returning(fake_value)
```

RSpec, in the style of the classic Ruby, even has a DSL to define matchers right beside the tests if the matchers are simple, so you can have some code like

```ruby
# in the middle of some testing context,
# just "this local useful thing"
matcher :create_user do |username|
  match do |response|
    expect(response.status).to eq 201
    expect(response.body).to include("Successfully created user #{username}")
  end
end

# Usage for a subsequent several tests:
expect { some_call }.to create_user('alice')
```

...with the matcher itself being a two-liner, while still providing _all the matcher services_, like informative descriptions, backtraces pointing to right lines, and even diffs, when applicable.

## Does it even matter?

Even before the arrival of AI-assisted coding, the tests were frequent victims of "nobody will read that" syndrome. The common reaction in many discussions of more expressive tests and utilizing the high-level language's powers are full of monk-like resolve: tests _should_ be "boring" and verbose, as if this is that unpleasant duty that only "true grown-up professionals" are able to perform (I go on a much longer and more heated rant on the topic... [almost 10 years ago](https://zverok.space/blog/2017-11-07-on-culture-of-bdd.html); since than, I lost a lot of will to fight this particular fight with this much vigor, but kept the overall opinion on the matters).

So, to _breathe in, breathe out_ summarize my rants, I firmly believe that we should strive for **readable tests**, and not in a "token by token, all tokens are easy to recognize" way, but in a "minimal amount of reading to give the idea clearly" way. _In [earlier writing](https://zverok.space/blog/2023-10-02-syntax-sugar.html#how-i-think-about-the-programming-language) I use the term **"lucid"** to distinguish "the meaning is conveyed efficiently" readability from "every particular phrase is easy to consume" readability._

Thinking in "matchers", which are separate from the test/assertion itself, combinable and extendable, allows approaching the goal of "each test being a phrase that just describes what is tested" without resorting to high-level pseudo-languages of "Given/When/Then" (with a tar pit of implementation of every step being "somebody else's problem")[^5].

[^5]: Some readers might notice that the whole angle of "behavior-driven development" doesn't receive much attention in this text other than early in the history when it inspired the RSpec introduction. I believe that's "just how it is": many of the "X-Driven" so-called "paradigms" are actually just optical tools -- or, indeed, _angles_ at which one might look at our work and invent new things. When they turn into rigid, book-described, rule-constricted "true" disciplines... Most of them just lose their steam.

And the tsunami of AI coding agents doesn't make it all irrelevant -- quite the opposite. The code is now read much more _frequently_ than before: by humans who control their agents (and frequently _reading_ and an occasional nudge are all they do to achieve the final result), and by other agents, trying to get on board with the current work. In other words: more expressive code, including test code = smaller context window to keep in mind and fewer tokens to spend.

Clarity and precision are what matter more than ever. (Until we all are lost in waves of unreadable code goo which nobody even tries to open in the editor.)

I personally find it funny that it took the arrival of enormous and (objectively) inhumane language models to establish some principles that would've been hugely beneficial for humans many years ago. But until the "use your agent efficiently" discourse, everybody "was too busy" to actually write guidelines in simple no-BS markdown right beside the code, lead the work with clear descriptions, strive to keep the context obvious and short, endorse simple code-investigation tools like [ast-grep](https://github.com/ast-grep/ast-grep), and, in general, make the codebases _truly readable_. More humane, if you will. God works in truly mysterious ways.

But whether you cater to human colleagues, your own sense of beauty, or more token-efficient and more controllable agentic development, clear and efficient tests might (just might, OK?) deserve a bit of your attention.

And, surprisingly for some, that niche, rarely-heard-of-anymore language, Ruby, and its libraries, give me several interesting angles to look at the topic. Some of them are even half-forgotten by modern Rubyists.

I hope to continue this train of thought, but (looks into the last two years of blog regularity) we'll see.

## PS: Bonus section on pytest

...which should've been in the main flow, but it was too long already, so here is one aside observation about Python testing.

Speaking of approaches to a single assertion, pytest is an interesting case. It is one of the modern frameworks that actually seem to make _a step back_ to having just a single `assert value` statement. The trick is that the usual conveniences like detailed reporting of what exactly was not equal to what is provided via metaprogramming, so if you run a test saying something like

```py
x = 5
y = 6
assert x == y
```
...you'll have an output saying:
```
AssertionError:

>       assert x == y
E       assert 5 == 6
```

...and for collections comparison it is even more detailed, able to render the detailed element-by-element diff. Such a "back to simple" approach can be found in, say, in [Nim](https://nim-lang.org/docs/unittest.html), which has a single main test function `check`.

But the approach is somewhat limited to the operators known to Pytest: even simple `assert re.match('^test$', 'tost')` fail will print `Assertion Failed: Assert None`[^6] instead of something like "expected 'tost' to much '^test$".

[^6]: pytest mitigates it by printing the test itself and all of the local variables available in the scope.

Curiously, the library itself provides a "matcher-like thing" for floats comparison:

```py
assert 0.1 + 0.2 == 0.3 # fails for obvious reasons
assert 0.1 + 0.2 == pytest.approx(0.3) # succeeds
```

The [pytest.approx](https://docs.pytest.org/en/stable/reference/reference.html#pytest-approx) produces a special object that utilizes Python's `==` implementation, which tries both values' `__eq__` method if the left is not compatible with the right and signals it with `NotImplemented`.

It is fairly similar to a matcher concept! However, there is only one such object in the library itself. There is at least one third-party [library](https://github.com/jwodder/anys) that implements many possible matchers with this trick (like `actual == AnyMatch('^admin:')`)

It is almost as if "matcher" as a base component of a testing library has some significant merit, so it might suddenly emerge on itself.
