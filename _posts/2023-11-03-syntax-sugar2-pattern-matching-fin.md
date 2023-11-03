---
layout: post
title:  '“Useless Ruby sugar”: Pattern matching (Pt. 3/3)'
date:   2023-11-03
description: "The final part of the article about pattern matching in Ruby, putting it in a broader context of the industry state, possible future usages, and a general effect on the language design."
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2023-11-03/crazy-pm.png
---

> This is the third part of the article about pattern matching in Ruby; that article is itself a part of the series about recent language features, their design, and pragmatics. Please start (at least) from previous parts: [first](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html), [second](https://zverok.space/blog/2023-10-27-syntax-sugar2-pattern-matching-cont.html); the series intro and table of contents are [here](https://zverok.space/blog/2023-10-02-syntax-sugar.html).

In two previous parts, we looked into Ruby's pattern matching, introduced through a few recent language versions. We discussed how it was implemented and what problems and possibilities it brought to the language syntax and semantics.

Now, let's put it all into a broader context.

## How others do it

Nowadays, a lot of programming languages—including pretty mainstream ones—support structural pattern matching. To name (and link to) a few: [Python](https://peps.python.org/pep-0636/) and [C#](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns), [Rust](https://doc.rust-lang.org/book/ch18-00-patterns.html) and [Haskell](https://en.wikibooks.org/wiki/Haskell/Pattern_matching), [Swift](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/patterns) and [Elixir](https://hexdocs.pm/elixir/1.16/pattern-matching.html), [F#](https://learn.microsoft.com/en-us/dotnet/fsharp/language-reference/pattern-matching) and [Scala](https://docs.scala-lang.org/scala3/book/control-structures.html#match-expressions). (There are also proposals of various stages of readiness to introduce it in [JS](https://tc39.es/proposal-pattern-matching/) and [C++](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2020/p1371r2.pdf).)

From this list alone, we can see that the feature is no longer a privilege of "functional" languages (whatever this means in our postmodern multiparadigm times) with which it was initially associated. There are static and dynamic languages in the list, languages old and new, languages that had PM from the beginning, and languages that introduced it in later versions.

It is impossible and impractical (for a blog post, at least) to discuss and compare all the details of syntax and behavior, so I have just a few brief and shallow observations.

Mostly, pattern matching integration into language falls into two categories: **either it is the main binding/destructuring construct**, consistently used in conditions, assignments, function arguments decomposition, **or a separate `case`/`match` expression** with branches defined by patterns, and pattern syntax is only valid inside this statement. _(Though Swift seems to have it in assignment but not in function arguments.)_

Whether it is the main assignment construct or just a special expression seems to correlate with whether it was in the language from the beginning. _(Though Scala seems to have had it from the beginning, yet it is limited to a `match`.[^1])_

[^1]: Or I might be missing something, guided mostly by language docs and quick experiments in an online sandbox: according to this [changelog entry](https://docs.scala-lang.org/scala3/reference/changed-features/pattern-bindings.html), at least unpacking-on-assignment exists in Scala.

Pattern matching in function arguments typically means that the language allows method overloading by argument patterns. _(Though it is a somewhat orthogonal concept: many languages have overloading by arguments shape/types without full pattern matching, while Rust, having a powerful PM, doesn't have method overloading.)_

Ruby mostly falls into an "introduced late/separate expression" category, but its quasi-assignment with `=>` might be a leap of faith, which might or might not lead to interesting things in the future (see the next section).

Comparing particular pattern syntax, we might find a lot of different punctuation decisions (made to make it consistent with the target language, or, sometimes, vice versa, to make it stand as a singular construct), but the fundamental list of elements seems to be very similar:

* **literals and constants**: just `1` or `FOO`;
* **"skip this"/"matches everything"** pattern, most usually `_`;
* patterns that just **bind values to names**: just a `variable_name` that will match anything and put it into a named local variable;
* **class/type checks**: usually just represented by type name: `Foo` will match any value of class `Foo`;
* patterns that check **shape of sequences**, like arrays (`[pattern1, pattern2, ...]`), tuples, dictionaries; and mostly look like those sequence literals of the target language with patterns inside;
* **records/struct/objects check & unpacking**, mostly in some constructor-alike syntax: `Foo(x, y)`, `Foo { x:, y: }` or similar — this invokes many other smallish design choices (like whether we might unpack without explicitly stating the type, and just listing possible attributes; or, the type name is mandatory—which, surprisingly, is the case for a language as dynamic as Python);
* **check _and_ bind** at once: probably most syntactically convoluted element, options differ:
  * sometimes it is `variable_name @ pattern` (say, `val @ Integer` will match integer value, and put it into a `val` variable—Rust, Haskell)
  * Python uses `pattern as variable_name` (`int as val`);
  * C# made just `pattern variable_name` allowed;
  * in Elixir it is `pattern = variable` in the middle of pattern (e.g. you can have `[a, b] = c = [1, 2]` which will read "put 1 in `a`, 2 in `b`, and the whole array once again in `c`"—looking absurd as a standalone example, but useful in function definitions);
* **pattern combination** with "or" (usually `pattern1 or pattern2` or `pattern1 | pattern2`); sometimes also `and` and `not` are present (say, in C#, `foo is not null` is an instance of pattern matching);
* an imperative "escape hatch" in a form of a **guard clause**: when declarative structure is not enough to check, something like `if more_conditions` (or `when more_conditions`) can be written after the pattern, like `Foo(x) if x > 7`
* ...and so on :)

This list is not exhaustive, but it covers most of what usable pattern syntax should provide—and it seems most of the implementations provide it in similar ways (adjusting to their specific syntactical intuitions).  And we can observe Ruby's decisions aren't outstanding in either a good or a bad way.

However, one thing slightly stands out: most of the languages compare matched values with values in patterns by simple equality, which closes the door for custom objects that do some broader checks. A typical question is, "how do I match a value in range `(s, e)` declaratively," or "how do I check that the string in the middle of a pattern matches some regular expression."

Ruby has a generic answer derived from our old "[proto-pattern matching](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html#how-it-was):" all objects in patterns are compared with `===` "case equality" operator, which is the same as equality for most objects, yet some are redefining it: say, `Range` or `Regex`. This good old operator is utilized by a shiny new pattern matching too, so we can easily write `percent in (1..100)`, or `age in (18...)`, or `identifier in /^[a-z]+/`. Of all the languages I've checked, only Swift [made the same generic decision](https://docs.swift.org/swift-book/documentation/the-swift-programming-language/patterns#Expression-Pattern) (with redefinable `~=` operator)—and I suspect, inherited it from Ruby!

Many very high-level languages just [give up](https://stackoverflow.com/questions/3160888/how-can-i-pattern-match-on-a-range-in-scala) here, though some solve the "value in range" matching problem with range-specific solutions: Rust [allows](https://doc.rust-lang.org/book/ch18-03-pattern-syntax.html#matching-ranges-of-values-with-) ranges in patterns (but it is not a generic rule of some trait being implemented, just a core language feature); while C# has something called "[relational patterns](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/operators/patterns#relational-patterns)" with comparison operators embedded in a statement:
```csharp
static string Classify(double measurement) => measurement switch
{
    < -4.0 => "Too low",
    > 10.0 => "Too high",
    double.NaN => "Unknown",
    _ => "Acceptable",
};
```
...but neither of the solutions seems as generic as what Ruby (and Swift) did. So, at the end of the day, the new Ruby's feature does not completely ignore its predecessor (old forms of `case`), and even if the integration is not [as tight as some hoped](https://zverok.space/blog/2023-10-27-syntax-sugar2-pattern-matching-cont.html#irks-and-quirks), it is still fruitful.

And one last yet telling thing about the design space of the pattern matching syntax: a lot of mainstream languages (like C# in the example above) make a choice towards a pattern-mapping that looks and behaves not as a generic branching construct but as a "data to data mapping": there is a `=>` between the pattern and the branch, and the branch is expected to be one expression. (In C#'s case, it is also underlined by the form of the whole expression: `value switch { patterns }`, and the fact that its result needs to be assigned to something; it can't be just a standalone statement.)

While Ruby didn't make those choices, their prevalence seems to underline the thought from the "[Consequences](https://zverok.space/blog/2023-10-27-syntax-sugar2-pattern-matching-cont.html#consequences)" section of the previous article: **pattern matching is a construct that supports data-first thinking**. And with time, it might have deeper consequences than just simplifying some code.

<div class="one-ukrainian-thing" markdown="1">
  <h3>A weekly postcard from Ukraine</h3>

  _**Please stop here for a moment.** This is your weekly reminder that I am a living person from Ukraine, with some random fact or event from our last week._

  Last Tuesday, I learned that my best friend from the spring army training camp was killed in action. He was a ship engineer from Odesa. When the full-scale invasion started, he was stuck with a broken ship in Singapore. Once he managed to return to Ukraine, he got into the army. He was 36. He had a wife and a six-years-old son. His name was Olexandr Demydyuk.

  ![](/img/2023-11-03/sashko.png)

  Please proceed with the rest of the article.
</div>

## Taking it further

Since pattern matching is still young for a feature this size, there is obviously room for improvement here and there: like fewer requirements for "pinning" for non-variables or support of pattern checks for the "rest" of collections (think `Array[*Integers]`).

But one thing that bothers me most is _performance intuition_.

While I mostly talk about programming languages as a _tool of thought_ ("Programs are meant to be read by humans"), I never forget the pragmatic part of things (that small nasty "...and only incidentally **for computers to execute**"). I work on pragmatic projects in pragmatic companies striving to deliver value, so all the "tool of thought" talks are about _using language intuitions to deliver well_, not to produce abstract beauty or academic formulae.

Twenty years ago, I was raised by C++, which taught me to ask, "but what overhead does this nice syntax bring?" constantly. Or, maybe, vice versa, it allows to explain to the compiler the _intention_ for better optimization? Switching to Ruby was quite a paradigm shift: an interpreted dynamic language with a reputation of being "slow" even in its class (I regret nothing).

And still, even in this "slowest" language, I recognize and praise the ability of the structure of the code phrases to inform the reader and the writer about the computational complexity. In the most naive phrasing, I believe that what looks _clear and simple_ should perform well; features that break this _intuition_ might create nasty dilemmas[^2].

[^2]: To some extent, those dilemmas are unavoidable, especially in Ruby: deep profiling of very "hot" code paths frequently leads to "less idiomatic" rewrites. This is OK when we are talking about the specific optimizations in rare cases, and not OK if "idiomatic" and "reasonably performant" _always_ require opposite approaches.

As a core construct, pattern matching's _intuition_ seems to be the **efficient branching** by the tree of possibilities: say, taking the example from the previous part:
```ruby
case event_data
in type: 'create', role: 'admin', **data
  # ...
in type: 'create', **data
  # ...
in type: 'update', **data
  # ...
```

One might imagine some "compiled" internal form of the entire statement that can immediately analyze that checking by `type:` once might, in the best case, immediately show the relevant branch, or, at least, it might guess to check the `type:` exactly once. That's not what actually happen, which we can check with a bit of black magic:

```ruby
class String
  # redefine `===` to log its execution
  def ===(*)
    puts(comparing: self)
    self.==(*) # it was simple == anyway
  end
end

case {type: 'update', role: 'admin'}
in type: 'create', role: 'admin', **data
  # ...
in type: 'create', **data
  # ...
in type: 'update', **data
  # ...
end
```
This code prints, on Ruby 3.2.2, a confirmation of _three_ checks being performed:
```ruby
# {:comparing=>"create"}
# {:comparing=>"create"}
# {:comparing=>"update"}
```

I.e., in the aspect of the _current implementation_, the pattern matching _is_ just a form of syntax sugar on top of a bunch of `if`s, each of which is performed independently. I believe that is a case where we might **hope** that an actual implementation will catch up with the syntax promise.

But even if choosing by patterns would be optimized away[^3], we shouldn't forget that `case`'s argument would frequently be an object, not a bare hash/array, and pattern matching is only as efficient as that object's `#deconstruct_keys`[^4].

Or, we can put it another way: it depends on whether the object's author considered that the suitable behavior for pattern matching is just (efficiently) exposing some internal data structures or took the liberty of doing non-trivial calculations, navigating object graphs, materializing missing part of the structure and so on. In other words, we are back to whether it was a **data object** or an **active object**.

> Random fun fact: on a wave of pattern matching enthusiasm, Rails once [have merged](https://github.com/rails/rails/pull/45035) a PR that provided pattern matching for ActiveModel, but soon [reverted it](https://github.com/rails/rails/pull/45553) in favor of more discussion. Since then, a more ambitious idea is [discussed](https://github.com/rails/rails/pull/45070), that will allow even match by nested associations.

[^3]: Which, as every optimization in Ruby is non-trivial exactly due to its flexibility: like my "debug script" demonstrated, somebody might've redefine `#===`, and in a general case, the optimizer can't rely on the fact that "for strings, it is just equality." Of course, redefining it for a core object is not a good practice, and a comparison which depends on the order of calls is a horrible idea, but as we know, [every change breaks someone's workflow](https://xkcd.com/1172/).

[^4]: Which, to add insult to injury, in the current Ruby is called repeatedly for every branch (so if there are ten branches, the same object compared against patterns would be deconstructed ten times). I hope this one **will** be optimized away! The current specification [slyly declares](https://docs.ruby-lang.org/en/3.2/syntax/pattern_matching_rdoc.html#label-Appendix+B.+Some+undefined+behavior+examples) the number of calls as "undefined behavior."

**And consequently**, if we want to think about "**data objects**," one might imagine some slow mind shift in how Ruby thinks about itself (and how Rubyists think about it): however we liked the "opaque objects sending messages" paradigm, its time seems to be running up. Some new types of objects/APIs/concepts might emerge that _do_ consider "how it is shaped inside."

In fact, **this process is already happening** for reasons unrelated to pattern matching! Last year's Ruby 3.2 release brought a large internal optimization of **[object shapes](https://bugs.ruby-lang.org/issues/18776)** (I highly recommend [this talk](https://rubykaigi.org/2022/presentations/jemmaissroff.html) by Jemma Issroff, one of those who implemented the concept). It was introduced as an "under-the-hood" change, but with time, it [turned](https://island94.org/2023/10/writing-object-shape-friendly-code-in-ruby) [out](https://railsatscale.com/2023-10-24-memoization-pattern-and-object-shapes/) that to make an "object shape optimization-friendly" object, one _needs_ to think about its internal structure.

So, between object shapes, pattern matching, `Data` introduction (sorry for the shameless plug!), and everything that happens in programming language design in industry, we _might_ see some more of "data-first" Class/Object APIs in Ruby in the future.

**And consequently**, it is possible to imagine "data objects" to affect pattern matching acceptance, maybe even endorse it to "blend" more into the language's API? Say, in the [types article](https://zverok.space/blog/2023-05-05-ruby-types.html) I suggested that we might once see a `defp` ("define with patterns") clause—which is almost like `def` but is able to unpack its arguments with pattern matching:
```ruby
# Enforce type to always be "create",
# unpack "role",
# check and unpack "temperature"
defp handle_create(type: 'create', role:, temperature: 0..40 => temp)
  # ...
end
```

---

BTW, in that same article, I imagined that `defp` could also be used for method overloading, but that statement I now consider a mistake. As I said in the current article, overloading seems to be somewhat orthogonal to pattern matching (C++ has the former, but not the latter; Rust has the latter, but not the former), and Ruby definitely made its mind about it.

But as a small "future is already here" joke, here is an implementation of "polymorphic `Array#slice`" (that was used in a types article as a demonstration of branching by shape and types of data), which works in Ruby 3.2 and looks exactly like a polymorphic method with no spare part:

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

Love it or hate it, I find the fact that it can be written, and is a valid Ruby, and makes perfect sense, at least amusing.

### Conclusions

Phew! Those three articles were _a lot_ to pack—and I am probably still missing a lot of things and wrong about even more of them.

In the way of conclusion, I'd like to say this: **In university, they said to me that "programs = data + algorithms." It took me 20 years to start to _suspect_ that maybe they were right. Like, you know. Data. And algorithms. Like, separated.**

To be honest, it seems to be a common sentiment, at least in some parts of the industry: the "good practices" of writing big systems are definitely shifting from a soup of polymorphic active objects to passive "structs" (which totally could have utility methods and operators to make working with them easier) and larger, frequently short-living or completely stateless "services" to handle them.

Pattern matching is one of the "Trojan horses" that can bring this shift closer to the many unsuspecting language users. It is objectively "cool" for many (though not for everyone! Otherwise, it wouldn't make its way into this series— and my life would've been a lot easier). It allows to say things in a clear and declarative manner, improving the code's structure here and there—frequently, in good alignment with the values I [laid out](https://zverok.space/blog/2023-10-02-syntax-sugar.html#how-i-think-about-the-programming-language) in the first article of the series: stating the intent clearly; avoiding to repeat the obvious; make the big picture easy to follow, etc.

But once you start thinking—at least sometimes—in pattern matching terms, the "simple" questions of "what is matchable," or "how to match efficiently," or "how to make the objects pattern matching-friendly" _might_ affect someone's—not everyone's!—persuasions and expectations about the code layout in general, and splitting into objects, classes, and modules. And bring the whole worldview, eventually, quite far from what was considered good object-oriented code once.

And yes, the irony of saying that from the position of the contributor and evangelist of one of the most "classically object-oriented" languages is not lost on me. Somehow, I still believe that this beautiful mess of Ruby, with its internally minimalist set of concepts, complicated implementation, and slightly chaotic design process, will get on top of it.

It is kinda interesting to observe where that can bring us.

---

Next in the list are **hash/keyword argument values omission.** One of the most controversial features, I definitely hope it wouldn't require me as much resource to discuss! You can subscribe to my [Substack](https://zverok.substack.com/) to not miss the next part, or follow me on [Twitter](https://twitter.com/zverok).

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

