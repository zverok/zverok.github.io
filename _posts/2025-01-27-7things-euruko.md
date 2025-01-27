---
layout: post
title:  'Seven things I know after 25 years of development'
date:   2025-01-27
description: "The transcript of the talk given at EuRuKo'24"
categories: ruby rant
comments: true
image: https://zverok.space/img/2025-01-27-euruko/slide1.png
---

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide1.png"/></div>

> This is a loose transcript of a keynote I gave at the [EuRuKo conference](https://2024.euruko.org/) in September 2024. The video of the talk is [here](https://youtu.be/r9EQjBPU474?si=Yc-0ftOTqGu-DhVh). Unfortunately, I only could talk in recording, but it was a great honor nevertheless. The topic is pretty important to me, so I decided to share a slightly edited transcript for those who prefer text form.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide2.png"/></div>

I have been writing software for about ~25 years. For 20 of those, I have been using Ruby as my primary language. I’ve made some [contributions to the language](https://zverok.space/ruby.html) and [other open source](https://github.com/zverok). I [write a lot](https://zverok.space/writing/blog/) about Ruby, and I maintain the [Ruby Changes](https://rubyreferences.github.io/rubychanges/) project, which some of you probably heard of.

I am kind of principal engineer in the US company [Hubstaff](https://hubstaff.com/), but only "kind of," because...

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide3.png"/></div>

The most important thing about me is that I am Ukrainian who serve in the Armed Forces. Currently, I am not in a combat position but doing other things that I am better at (and the rest of my activities are happening besides these duties).

I got into the army not because I am very militaristic or like fighting but because I have a duty to protect my homeland.

It is the same for many Ukrainians.

There are all kind of people serving in our Armed Forces currently, and I have a small personal project of [translating poets](https://7uapoems.substack.com/) who serve in the army into English.

So the actual title of the talk is

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide4.png" alt="Seven things I know after 25 years of development and 10 years of war"/></div>

It is still about development and Ruby, but I allowed myself to draw some parallels with my experience as a Ukrainian. I hope it makes the talk richer. Nothing too graphic/disturbing, I promise.

Let's get to the matter, those seven things I wanted to share.

## 1. You outgrow every framework

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide5.png"/></div>

In any area, not only in tech, we frequently rely on various frameworks and foundations to know what to do and what to expect.

Say, in international relations, we Ukrainians, for a long time, have relied on being a sovereign country whose security is guaranteed by global powers. Which turned out to be, let’s say, not as solid as we’d hoped.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide6-1.png"/></div>

So, as programmers, we frequently rely on frameworks, and as Rubyists, we rely on The Framework, but I am not about to criticize any particular one.

The point is that frameworks provide us with [rooms and boxes](https://www.youtube.com/watch?v=lI77oMKr5EY) to put our code in, and to feel confident, only implementing the business logic.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide6-2.png"/></div>

But we all know that eventually, as code grows, we have more concepts, and then even more, some of them provided by your framework, some of them introduced by a team.

And however we extend it, something always sticks out.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide7-1.png"/></div>

And like a fractal, every particular concept starts small and then grows into a framework of its own. Like common nowadays callable object, which is a simple idea. When you introduce it, it might seem like a small step you need to achieve the completeness of the mental model.

And then there are new requirements and new use cases, and eventually, you have a dozen DSL options to define one.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide7-2.png"/></div>

And then something sticks out anyway, not fitting into any of the previous definitions. And if you don't handle it, you'll either have very bloated concepts or some garbage bin, like `lib/` folder or `app/services` folder, and soon nobody is sure what is were.

**To handle that, you should be ready to make your own architectural decision, regardless of the frameworks you rely upon. But there’s a problem.**

## 2. Patterns and methodologies fail

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide8.png"/></div>

The second thing I am sure of is that whole patterns and methodologies fail you and get outdated.

Like in our Ukrainian experience, the whole international security architecture became a laughing matter, international humanitarian structures proved themselves useless, and human rights defenders defended anything else other than human rights.

As a developers, when we need to make a decision, we...

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide9-1.png"/></div>

...might rely on some methodologies, trusted approaches, books, patterns, or funny mnemonics encoding simple principles.

However, as code grows, the project matures, as the industry develops new opinions, and as challenges change, a lot of doubts and questions emerge.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide9-2.png"/></div>

You might learn that inheritance was an idea that is good only in textbooks or that the whole OOP failed us. You start to doubt whether Active Record is a viable approach or whether passive entities and repositories are saner.

You might be reminded that SOLID is an acronym coined for marketing purposes by one person, and it groups probably useful but vaguely defined and contradictory principles. And in general, for any common approach, there are a lot of arguments about its usefulness or harmfulness (and counter-arguments, and counter-arguments to that).

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide10-1.png"/></div>

And so we discover for ourselves more high-level guiding principles, like DDD, which follows a very sane idea of isomorphism of the domain and its coding representation. However, it still leaves many blanks in design decisions, such as when a small domain concept requires a big algorithm to implement or when important domain concepts correspond to a small implementation.

It also introduces a lot of new terms that people love to turn into literal classes, so eventually, you have, besides your MVC classes, also entities, aggregates, bounded contexts... Or, as a [wise man put once](https://x.com/robsmallshire/status/1055085690260721664):

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide10-2.png"/></div>

Or we might talk about “hexagonal architecture”, right?

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide10-3.png"/></div>

...Which is notable for never explaining why is it hexagonal, and for leaving the design decisions about the whole “core” architecture for yourself. (I am not here to say it doesn't provide some useful concepts.)

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide10-4.png"/></div>

**Ultimately, those might help, but eventually, you will be on your own in designing new things. Why is it so?**

## 3. The scale only grows with time

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide11.png"/></div>

Because the scale only grows with time. This is true about many things in the world, and it is also quite a trivial statement, but it has frequently overlooked consequences.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide12.png"/></div>

Imagine any Rails app you ever maintained for some time. I think you probably agree that all it did was grow.

We live in a capitalistic world where you need to run fast to stay relevant. Whether it is some new tech like switching to SPAs, extending your market share to customers of a smaller or bigger scale than before, trying to adjust your market fit without losing old customers, or just linear growth.

Whatever happened, there was more code, more tables, more endpoints, more tests, more everything.

As an example, we might look at the [GitLab open-source codebase](https://gitlab.com/gitlab-org/gitlab), which at first looks like your regular Rails app.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide13-1.png"/></div>

Initially, it was what? Just a UI for collaboration on top of git.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide13-2.png"/></div>

Now if you look through its main folder, you’ll see a bit more things than in a new Rails app, and it into its [`app/`](https://gitlab.com/gitlab-org/gitlab/-/tree/master/app?ref_type=heads) folder, it has a multitude named concepts of its own, and if you’ll open [one of those folders](https://gitlab.com/gitlab-org/gitlab/-/tree/master/app/services?ref_type=heads), you see more, and more, and more. (Note I only got up to "c" letter on the last screenshot.)

Maybe not all of our projects are at a scale of GitLab, but talking about any commercial project, eventually, you’ll need things like reports, notifications, better role management, payment systems, new APIs, bulk operations, business analytics, and so on and so on.

The simple yet frequently overlooked consequence I was talking about: **it is perfectly natural that your old ways of handling problems would stop working eventually.**

The way of thinking about ten controllers, with four standard methods each, just doesn’t scale to hundreds of non-trivial endpoints.

**What do we do then?.. Well, I have a paradoxical answer!**

## 4. Pay attention to stories

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide14.png"/></div>

And my answer is: Pay attention to singular stories. When working with any feature of our software (implementing a new one, adjusting an old one, or trying to understand why it behaves in the wrong way), we always have a narrative to follow: there are some causes and consequences: inputs, outputs, transformations, effects, which can be told linearly.

But by jumping through layers and conventions and discussing the "bigger picture," we are frequently losing those particular threads—and then losing the bigger picture.

A small illustration: once upon a time, I was [trying to add a new feature](https://github.com/rubocop/rubocop/pull/8071) to Rubocop (it didn’t work out but for an unrelated reason), and I had this AHA moment when I spent, like, a day trying to understand an algorithm split in 2-3 “simple” line methods, and eventually rewrote it into one, half a screen-sized. Which, though, didn’t match expected “complexity metrics”:

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide15-1.png"/></div>

Here, for comparison, is the fragment of the [original algorithm](https://github.com/rubocop/rubocop/blob/v0.85.1/lib/rubocop/comment_config.rb#L44) (how it looked at the time), but I don’t have a big enough screen to screenshot the whole thing:

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide15-2.png"/></div>

That’s not about "badly written" Rubocop (it is an awesome software project maintained by extremely competent people), but about some deep difference in the approaches.

Splitting everything into ultimately small methods and extracting every condition to a predicate is actually a widespread approach in the Ruby community (which the default linters’ complexity thresholds reinforce).

As a counter opinion, at Hubstaff, we have this fragment in the "small coding rules" document, which underlines the idea of "trying to put everything together first, and impose architecture only after that":

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide16-1.png"/></div>

The "single method" is not a key here!

What IS a key is the point of view from which focusing on "exposing what's happening here" with code is the main goal. The rough list of principles we follow are:

* Dense code;
* Expressive language;
* Rich _thesaurus_;
* Frequent small rewrites;
* Attention at the level of a singular page (screen) of code.

It leads to the discovery of good names and structures of related concepts—much more reliably than just trying to invent them on the architectural planning stage and fit into the "big picture."

But most of the time, there is no single simple "big picture" at all (as anybody who tried to make a class or flow diagram of a mature project will agree).

**It is just a bunch of interlinked stories, a garden with paths through it, not a concrete skyscraper. But why are these stories so important?..**

## 5. The goals are truth and clarity

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide17.png"/></div>

Because our final goals, as developers and as human beings, are truth and clarity. And it emerges from listening to stories. One might say that "truth" is a weird word when talking about software architecture; it is more from philosophy (or law) domain...

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide18.png"/></div>

...But my persuasion is that on an intuitive level, all our principles and methodologies are introduced as a way to search for the truth.

Like OOP was invented in simpler times in the hope of modeling the whole world, and behavior-driven development is a way to honestly describe what is expected from a piece of software before implementing it, and so on.

Our problem is that we turn any good and imprecise idea into a discipline, formalize it, and then eventually, instead of trying to truthfully mirror the problem, we replace it with rigorously following the rules.

As a small example, the Ruby community once leveraged the language expressiveness to invent an incredibly powerful testing DSL named RSpec but then eventually persuaded itself that tests shouldn't be expressive and they should follow some rules of the most primitive code possible:

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide19.png"/></div>

As a consequence, many Rubyists are taught to write tests that are more verbose than the code it tests, sometimes orders of magnitude more verbose. They hate it rightfully and are forbidden from even dreaming of thinking it could be another way.

Well, it actually can.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide20.png"/></div>

Whether you like this example or hate it, I really hope you can see here a point where the example tells the story of the expected behavior. An attempt to make it work also illustrates the effect the "storytelling" approach has on the thesaurus. To make it work, you'll need a few new high-level abstractions that are defined once and then improve the life of many spec authors.

**However, it is a well-known fact that approaches like this frequently encounter fierce pushback.** And here lies my sixth thing.

## 6. This might be a lonely experience

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide21.png"/></div>

Trying to stick to the true stories might be a very lonely experience. First of all, because people have their “big pictures,” and the first thing they’ll expect from your story is to see how it fits in those.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide22.png"/></div>

So, when trying to "sell" the approach that focuses on "what's really going on here"—be it on code reviews, pair programming, or discussing style guides—one might meet reactions ranging from total indifference to active hostility.

Because looking for truth is about adjusting your mental models constantly and questioning your own experience. In general, people, especially established professionals and especially-especially software developers, aren’t up for it. I mean, we might change tools every day, right, but internalized persuasions and mechanical habits? Never!

One might notice that lately, in industry, there is a lot of strong sentiment against "smart code" and expressiveness in general, or, broader, about the unimportance of the particular ways we write code and, as an offspring of it, a strong sentiment against code review practice.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide23-1.png"/></div>

I get it, I do.

In general, when solving problems, people prefer to write and avoid reading and editing. This leads to the sad state of things when the practice of code review turns into a laborious formal duty for both participants (who don't know what exactly to do about it other than go through the moves) and eventually is badmouthed as a harmful and meaningless way to waste time and establish dominance.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide23-2.png"/></div>

But in my view, reading and discussing each other's code is the only way to make sure the story you are telling makes sense to others, that it doesn't contradict other stories, and that it doesn't misuse a common thesaurus to say things that shouldn't be said that way.

**This is a deeply humane activity, and nobody will persuade me otherwise.** Which leads me to my last point. Kind of sentimental one.

## 7. Never give up seeking truth

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide24.png"/></div>

Never give up seeking truth, however uncomfortable it is. Search for knowledge. Adjust your worldview. Ask. Rewrite outdated code. Drop faulty hypotheses and unreliable foundations.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide25.png"/></div>

Software author is, first of all, a writer. They are a person who stands upright and says: "that's what I know for now, and that's my best attempt to explain it." Having this stance, preferring it to everything else, and hiding behind terms, concepts, and authority are invaluable qualities for long-term project success.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide26.png"/></div>

Or, basically, for any long-term human activity success.

This is the most important thing I learned. Probably not much for a career this long, but that’s where I stand personally and professionally.

<div class="talk-slide"><img src="/img/2025-01-27-euruko/slide27.png"/></div>

Thank you. I hope to see you all in person one day.

And please, even if not remembering this talk, remember to help Ukraine.

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
