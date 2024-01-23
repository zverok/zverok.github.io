---
layout: post
title:  'The end of “Useless Ruby sugar”: On intuitions and evolutions'
date:   2024-01-23
description: "I wrote the analysis of “useless sugar” features of Ruby for two months, and I regret nothing."
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2024-01-24/cover.png
---

At the end of September, I returned to— more peaceful parts of Ukraine, let's say. And a Reddit discussion inspired me to start writing what I expected to be a short blog post.

The discussion was about new Ruby syntax elements from recent versions, and, as usual, a fair share of participants expressed some variety of displeasure. I don't care much for "I like it"/"I don't like it" kind of discussion, yet I frequently find myself hooked by "I don't see any explanation why this might be a good idea" or "This is absolutely against everything I love in Ruby."

While this kind of statement can be—and frequently is—just another way to say "I don't like it," my mind perceives it as a challenge: can the reasoning behind the change be understood and explained? What was the design space of solving the problem?

That's how the "Useless Ruby sugar" was born: as an attempt to look for a consistent explanation of the changes in the syntax and to analyze whether the changes have deep consequences.

> "Syntax sugar" is another liberally used term to mark elements of syntax in a derogatory way, implying that its only effect is saving a few keystrokes at a cost of complicating the language's logic. The named was chosen as a sign of irony, but I should regretfully acknowledge that some readers took it at a face value (sometimes cheering to "finally saying the truth," sometimes arguing).

## Are you done yet?

When I started to write "That useless Ruby sugar," I imagined it as a short blog post made of concise sections (a paragraph or two each), dedicated to several latest language changes and investigating where they came from, how they were designed, and how they change the perception of the code.

**The intention was not to "defend" some particular feature or the language as a whole.** I wanted to honestly share my _optics_, my way of looking at means of expression and integrating them into my mental model.

I decided that each of the chosen syntax features (anonymous block parameters, pattern matching, argument forwarding, and so on) can be covered from the same angles, from reasons of its introduction to consequences and changes in code style, and compared to similar designs in other programming languages.

A "small post" grew into "a very large post," then into a series of posts published over two months (one of the topics—pattern matching—fractally grew into [three](https://zverok.space/blog/2023-10-20-syntax-sugar2-pattern-matching.html) [consecutive](https://zverok.space/blog/2023-10-27-syntax-sugar2-pattern-matching-cont.html) [posts](https://zverok.space/blog/2023-11-03-syntax-sugar2-pattern-matching-fin.html) on itself— and, trust me, I am praying that the current text wouldn't turn out too long for a singular final article!)

<big>Anyway, the series is now finished. The **[introductory post](https://zverok.space/blog/2023-10-02-syntax-sugar.html)** explains the thinking behind the project in more detail and ends with a table of contents for the entire series.</big>

## So what?

Despite the appalling underestimation of the amount of writing which cost me some two months of weekly posts, I kinda enjoyed doing it. And it seems to have found its share of grateful and attentive readers (thank you!). For me, it turned out not to be about particular language features. And, to some extent, not even about the particular language (which, though, became a load-bearing pillar of the argument).

I started with a slightly expanded version of "rules of thumb" for understanding, guessing, and memorizing language behaviors I frequently share with colleagues. Eventually, the texts grew into some **"thought framework": the way of looking at the inevitable change of a living language**—its cause and the design space of possible implementations, its consequences, and effects.

The essential property of this framework is considering the programming language as a tool for writing and reading first and foremost. I always attempt to see the driving forces behind its evolution as an intention to formulate the intention better.

It is obviously not the only way of looking at programming languages, but it seems to be interesting and insightful enough for the series to have some modest success and a healthy dose of comments. The individual articles frequently caused interesting cross-language discussions, where most participants were not native to Ruby[^1].

[^1]: In all honesty, not all of the discussion was healthy. Well, talking about the way of thinking and expressing thought is probably destined for some absolutely hateful comments about Ruby (from outside the community) or particular new features (from inside of it). Probably, causing strong emotions is better than causing none—though, I can't say I really enjoyed the experience (but, well, as a Ukrainian, I can take a lot). My own strong emotions were more triggered by people responding to their idea of the article (or its title) while completely ignoring the actual reasoning it presented.

While amounting to a small book already, the series has a pretty narrow premise: just one way of looking at just a few changes of just one language in a vast universe of them. Still, I think two of the main "driving forces" are important and relevant not only for Ruby.

The first, which I think is common for many mainstream languages and styles, is the **tectonic shift from the "classic OO" style** of structuring programs (and envisioning them!) to the "small immutable objects + functional algorithms" style. Obviously it is not the most breaking news (entire languages of Go and Rust that can already be considered mainstream are built on foundations that are quite far from the "classic OO"); it is still interesting to see how the established languages adapt.

The second force of the change is more perpetual: it is a **natural expansion of language's—any language's—thesaurus** in response to more and more things previously non-trivial and complicated becoming everyday ones. Concepts that used to require explanation and education become something that you understand from a single mention and want to be expressed in one word. Some decades ago, not every programming language could handle strings properly (Unicode strings, even less so). Today, things like lambdas, sum types, or pattern matching are almost must-haves for expressing everyday thoughts. The baseline shifts constantly.

A lot of changes I discussed in the context of Ruby are more universal: most widely used languages continue to adjust as thinking tools to the changing landscape of _what we are thinking about_. It seems as universal as a pushback from some part of every language's community: a pushback from those whose thought process is performed at other layers and who believe that the fabric of the language should stay stable to not "distract" from the materialization of already formed thoughts with the new features.

And every language's evolution is probably a compromise between those two forces.

## But what about Ruby?

While I hope that some of my observations and conclusions in the series were curious for a broader audience, I am still thinking and writing in and about Ruby.

One may say that Ruby is my "lens" for looking at problems, both on the pragmatic layers (writing code to understand something) and on the meta-/philosophical layer of looking at how the language adjusts to the changing world to understand the world better.

The important thing about Ruby, which is its curse and blessing, is **it isn't an interesting language**. There is no particular "shtick" or wittily-named concept everybody tries to imitate in other languages (well, I think it was part of Ruby's doing to introduce the functional iterators like `map` in the mainstream, but they weren't invented here).

I believe the premise of the language is something along the lines of "take the modern ideas and put them together with a reasonably minimal number of base concepts, as clearly and expressively as possible without becoming esoteric."

_And I think the reputation of being "weird and whimsical" that the language earned in the 2000s, both amongst its fans and haters, tells more about the state of the industry at that point. An industry where the desire to write in one line what "should've" required a class and a package and several methods and initializations and type definitions— when that desire alone made a developer look "unserious."_

In many aspects, Ruby (with its Smalltalk-inspired internal "almost everything is just objects exchanging messages") achieved remarkable clarity and consistency from a very small set of core concepts—yet not as small to become a "language constructor" instead of a language.

Through the years, it managed to continue staying _reasonably modern_ without losing the internal consistency and discoverability (though, of course, some Rubyists would have strong dissenting opinions: I think one of the commentators of the series said something along the lines of "I love Ruby but nothing good happened since 2.0"—with 2.0 being ten years old).

Of course, not all of the language's initial design decisions and accumulated weight is "optimal" or provably best.

I can list a lot of things I am personally not a big fan of. An early draft of this article had a planned section for some of them— But I recently developed an irrational fear of fractally expanding articles! Though, to name a few:

* textual `require`s and module system could've been much cleaner; as an indirect result, the Ruby community eventually adopted the "autoloading" concept, and while the [current autoloading solution](https://github.com/fxn/zeitwerk) is a very well-engineered one, I personally don’t believe in the approach;
* while "code blocks" aka `Proc` were a witty way to introduce higher-level functions, the current situation with "callable objects" in Ruby (non-fist-class `Method` objects, semi-convention of `call` method handling) makes it less powerful than it might've been;
* the uneven and confusingly supported standard library (my favorite example is `Date`/`Time` types situation, with `Time` being a core object, `Date` being part of the standard library mostly dedicated to scientific calculations and many small discrepancies between them);
* (I can continue).

The best things are still a reasonably small core of concepts, with the development team working hard to "factorize" any new feature into those concepts, and the flexible syntax that is instrumental for making the core small (like an ability to call a method without parentheses makes `foo.bar` attribute-like syntax possible and removing a need to have separate "method" and "property"/"getter" concepts).

Those characteristics make Ruby—still—an incredible fit for developing _domain-specific thesauri_: new, task-specific parts of the language that would integrate naturally with its rest and would make "thinking in the domain" much easier. I'd distinguish that from a concept of a "domain-specific language," which frequently implies a _new_ system of expression, with its logic not necessarily aligned with the implementation language.

> I think Rails was/is a great demonstration of how natural Ruby is in development of that "domain-specific thesaurus." I just wish sometimes that it would be taken as an example/inspiration for development of many others. Instead, Rails became _the_ Ruby shtick, and its popularity at some point put Ruby in the domain of "development of support of big systems," with its reputation and evolution being severely affected by those new circumstances.

Still, for me, Ruby stays a great _modeling clay_ for experimenting with things and investigating new ideas.

<div class="one-ukrainian-thing" markdown="1">
  <h3>A postcard from Ukraine</h3>

  _**Please stop here for a moment.** This is your regular mid-text reminder that I am a living person from Ukraine, and a bit of useful related information._

  **On the news.** Usually, I put "one news item" here. Yet, honestly. It is hard to choose one that happened since my [last post](https://zverok.space/blog/2023-12-28-advent-of-changelog-fin.html). We had a lot happening. A prominent young poet [was killed](https://twitter.com/PenUkraine/status/1744075793855029721) on the front line ([here, I translated](https://twitter.com/zverok/status/1744416917392179531) some of his poems).  Massive missile/drone [bombardment](https://twitter.com/United24media/status/1742224719011594673) around the New Year, including in my home city (which has continued to be regularly shelled since; say, last week, a missile [hit](https://twitter.com/FeministsUa/status/1747695174602420381) a private gynecological clinic, and the week before that—a [hotel](https://twitter.com/United24media/status/1745340529699696643) with foreign journalists). That's not [even](https://twitter.com/den_kazansky/status/1743684511320133876) [mentioning](https://twitter.com/United24media/status/1747896573990781438) [everyday](https://twitter.com/United24media/status/1748633966255686091) [attacks](https://twitter.com/United24media/status/1748680827620270192) on civilians in the smaller cities nearer to the front lines. I wonder how much of this gets in the mainstream media of my reader's countries.

  **One piece of context.** Today (Jan 20, the day I am finishing this post) is [Memorial Day of Defenders of Donetsk Airport](https://twitter.com/DefenceU/status/1748647821803245895). For 242 days, they defended the airport, which disrupted Russia’s plan to transfer forces through it and prevented more Ukrainian territory from being lost in 2014-2015. Yeah, Russia was waging its war then, already.

  **One plea.** If you are in the US, please call your representative! In DC, you can also join the [Ukraine Rally](https://twitter.com/ukrainerallydc), even if for a small amount of time.

  **One fundraiser.** You can donate to the [Ambulances for UA](https://www.ambulanssi.org/en) initiative, which buys ambulance vehicles to serve on evacuation missions near the front lines.
</div>

## There is always more

While the series, as it was invented, is finished, there are a lot of interesting things that are left to think about, at least for me. Maybe some of those things would turn into future articles!

One thinking direction is diving deeper into the "code as readable text" consequences: I think I have some interesting angles on the (arguably aged and trivial) comparison—from the "software project as a magazine" metaphor (slightly touched in the [last](https://zverok.space/blog/2023-12-01-syntax-sugar5-endless-methods.html) post) to what does it mean to think about a "page of code" (repeated motive of the series), or what would be a modern take on the old idea of "literate programming" with our current knowledge.

Another promising direction is juxtaposing design decisions made by several different programming languages in the same area: after doing it for those few syntax features I covered, I now believe it is a powerful tool for understanding some programming concepts, their design space, and the choices and sacrifices some particular language makes to adopt them.

And in general, I think a lot about the concept of "language intuitions" and other semi-conscious forces that make us consider some code "good" or "bad," choose approaches and methods of solving problems, and change our views over time. Trying to find out why we have those intuitions and persuasions, as well as expose, challenge, and formalize them, seems like an interesting topic. (Though, arguably, less "marketable" than "just write software my way and you'll be good, period.")

After all, it is not a language study; it is the study of how we think and write. Or at least that's what my poet's mind tells me.

## P.S.: What about a book?

"Intuition" is an important word for me in the context of programming language understanding (as if somebody hasn't noticed yet!). As I stated in the intro article of the series:

> I believe a well-designed, consistent language imposes a lot of implicit _intuitive_ knowledge about "how things are typically done, what this combination of elements might mean, what this particular name is related to," and alike. Acquiring those intuitions creates _fluency_ in reading and writing the code that follows them.

Naturally, "Ruby Intuitions" became the name of the book I planned to write in 2022.

Writing the book on the language that tries to expose this implicit knowledge of Ruby and reflect upon it seemed like a good idea. Still seems, actually. Yet the idea of the book became something akin to my "big white whale": something that I need a lot of luck, time, focus, and experience to work on, and well— it looks like the time for it hasn't come yet!

But, while working on the series of articles, I had an idea for another, simpler (hopefully!) book, which would allow me to gather into one text years of my work on understanding the language, documenting it, and observing its change.

The title I came up with was just "Ruby Evolution," and currently, in my head, it looks like a reasonable cross-over between my [Ruby Changes](https://rubyreferences.github.io/rubychanges/) site and essays like this "Ruby Sugar" series, taken over the span of Ruby's lifetime: an orderly discussion of the most notable changes by versions/topics, the reasons behind them, and a reflection on how it changed the language.

I have a hope that a book like this might be of interest both for Rubyists and for those from other language communities, in a general "history of ideas" context.

Currently, I am in the early stages of playing with the idea and drafting a table of contents. But I wonder if there would be some interest in such a book?

In the meantime, I'll be writing on other topics that will catch my curiosity, more or less related to Ruby, or at least writing software in general.

_**You can subscribe to my [Substack](https://zverok.substack.com/), or follow me on [Twitter](https://twitter.com/zverok).**_

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

**If you've found the post (or some of my previous work) useful, I have a [Buy Me A Coffee account](https://www.buymeacoffee.com/zverok) now. Till the end of the war, 100% of payments to it (if any) would be spent on my or my brothers' necessary equipment or sent to one of the funds above.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

