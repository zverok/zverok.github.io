---
layout: post
title:  "Language, perception, and empathy: Ukrainian's gaze (Notes to the talk rejected by RubyConf)"
date:   2023-11-17
description: "Notes to the talk-that-never-was. No pretty illustrations, no code examples even, just a dotted line of thoughts I once intended to make into a talk."
categories: ruby philosophy
comments: true
image: https://zverok.space/img/2023-11-17/me.jpg
---

This week, I am breaking the flow of my "Useless Ruby sugar" article series for a text that would be quite different.

This week, the largest US Ruby conference, [RubyConf](https://rubyconf.org/), was held in San Diego. When, a few months ago, its call for papers was announced, I suddenly had an idea for a talk that seemed quite interesting: the talk that would be drawn both from my experience of Ukrainian living through war and my experience as a long-term dedicated Rubyist, part of the core team and a maintainer of [Ruby Changes/Ruby Evolution](https://rubyreferences.github.io/rubychanges/).

Obviously, I knew I wouldn't be able to attend a conference in the US to give a talk in person, so I wrote to organizers asking whether it would be possible to consider—as an exception—returning to a practice of remote presentations (that seemed to work well at COVID times).

I was suggested to submit a proposal, and "if it gets accepted," the organizers promised to consider the possibilities for further actions. So I did.

For some reason (call it "victim entitlement"), I half-assumed that the perspective I proposed, in combination with my background, was unique/interesting enough to be accepted. So, while waiting for a final decision, I spent my free time (at the frontlines then—and you would be surprised how much bored free time one has there between the "busy" parts) plotting the talk and making some drafts of the key points.

The talk was rejected, though—with a standard automatic notice that also suggested buying a ticket to the conference at a rejected-speaker discount or coming to volunteer. Can't say I took it very well, though I blame nobody. Victim entitlement didn't work, who might've thought!

So, today, I am publishing bare notes to that talk-that-never-was. No pretty illustrations, no code examples even, just a dotted line of thoughts I once intended to make into a talk.

_(The proper way of handling the situation would be to record the talk nevertheless and put it on YouTube. I played with the idea for some time while polishing the plan/notes, but then I understood I didn't have enough resource. Video is a media that is hard for me to handle well, and without an incentive in the form of "it's for the conference"— I just wouldn't do it. Even with the understanding that I am losing a potential audience who is more comfortable with video, at least in the context of "something like a conference talk.")_

So, here goes.

---

## Language evolution, perception, and empathy: Ukrainian's gaze

**(Notes to the talk rejected by RubyConf)**

## How I write

Ruby is almost thirty years old. I am writing in it and about it for about twenty of them.

For me, Ruby is much more than a tool for a day job. A lot of what I love to do and spend my free time on is about Ruby: some [language core contributions](https://zverok.space/ruby.html), including several minor and middle-sized features and documentation improvements; a [fair share](https://github.com/zverok) of open source libraries and contributions to libraries others did; a [Ruby Changes](https://rubyreferences.github.io/rubychanges/) annotated changelog that already has gained some notoriety; a lot of [blog posts](https://zverok.space/writing/) and [fun experiments](https://zverok.space/blog/2020-05-16-ruby-as-apl.html); many years of mentoring; maybe a book, once. This year, I was also honored to be a beta-reviewer for the upcoming [new edition](https://pragprog.com/titles/ruby5/programming-ruby-3-3-5th-edition/) of "Programming Ruby," updated for modern times by [Noel Rappin](https://noelrappin.com/).

The constant theme for all these work/leisure activities is how Ruby's (and, in general, programming language— or, any language) affects how we express thoughts. This is the reason I am obsessed with language's _evolution_, too: I am immensely curious about all the ways small changes in syntax bring tectonic shifts in understanding and all the trends of community practices evolving and requiring the language to adjust and rethink itself.

When, a few years ago, I started to focus on gathering my views on the language into a coherent system—by systematically writing about it (frequently in public, but not always)—I understood that a lot of things about this _computer_ language I find valuable are rather _humane_ than strictly logical. For me, good code is further from a strict mathematical system and closer to a good text that people are pleased to read: sometimes succinct, sometimes redundant, slightly playful, maybe witty on occasion. But always direct, truthful, and insightful.

If this sounds like a bunch of poetic bullshit to you, no problem: I was a poet for quite a long period of my life. But this system of views, when implemented consistently, is observed to be having a good pragmatic effect on large real codebases—and is frequently contagious throughout the team, to a healthy effect.

All in all, I hope that it is safe to say that my activities and my views brought some value to the community—maybe not that huge, but not negligible either. (Including all the times I am spending hours on Ruby tracker finding a proper justification for a new feature or a fix for a deep inconsistency. Frequently, it isn't even "credited" to me, neither should it—the mere fact of the language adjustment is rewarding.)

So I felt seen, and intended to continue to "be me" and focus more on my ways of thinking and how they can be useful to fellow developers—even if my approaches are frequently expressed in such vague terms they might seem the opposite to _engineering_: feelings, perception, intuitions.

## Then, the war

When the full-scale Russian invasion started, it was a continuation of the eight-year span of hybrid war Russia already waged against my country and hundreds of years of colonial oppression before that. And yet, when it unfolded it its full ugly scale, it definitely shifted my _feelings_ and _perception_ of things, that's for sure!

I told my personal story—unremarkable by Ukrainian standards—a few times already (for example, [here](https://zverok.space/blog/2023-02-07-changelog2023.html)), so I wouldn't go into details again.

I lived with my children through the first two weeks of the constant bombing of Kharkiv, then evacuated my family and returned to my city to volunteer and observe it taking the hard blow. Finally, joined the Armed Forces of Ukraine in March'23. I never became a hero worth talking about: three months on the frontlines near Robotyne, and then I got transited to a— more computer-related position, let's put it this way.

![](/img/2023-11-17/me.jpg)

There are many people there who also were civilians recently (from all trades of life, software developers included) but became much more efficient soldiers than I did.

Surprisingly, even for myself, all that time (including the AFU serving time, which is still continued, till the victory), I had enough attention and time to spend on my day job, contributing to Ruby, and writing. Much less than before the invasion, but still, weird. As if nothing happened.

I am still shocked, honestly. You just learn to function despite the shock. (And no, it is not some personal heroism or other enviable trait of this particular individual; it is just how we learned to live.)

One of the additional shocks of the first months (besides the direct experience of being bombed and other war stuff) was learning how little "the community" I felt belonging to cared. I [published](https://zverok.space/blog/2022-03-03-WAR.html) [articles](https://zverok.space/blog/2022-03-15-STILL-WAR.html). I literally begged for retweets. I looked into feeds of prominent Rubyists, known for their attention to empathy and injustice (they were busy discussing whether DHH should be banned from the RailsConf due to one transgression or the other). I tagged them personally. Mostly, in vain. (There were a few notable exceptions, which I never forgot.)

_(Yes, I probably was unpleasant and demanding. After one particularly unfortunate encounter, I was forcibly unsubscribed from Ruby Weekly, and my email was blocked. So it goes.)_

"But how it was related to us, and what can we do?" one might ask.

A lot.

The war that Russia wages is not an isolated attack by a bunch of armed extremists. It is a war performed by a **big system**—and in a modern globalized world, it means a war in which the entire world is participating, willingly or unwillingly. By selling necessary tech to the aggressor country; by supporting its economy via buying its export; by maintaining diplomatic ties with it.

But the most important thing today is **public opinion**—and the public spread of the information.

Public shame on those who support the aggressor, and refusal to work with them (do you know, say, how many big corporations from _your_ country never left Russia, helping it to maintain the war budget? Do you have them as contractors or clients?).

Public pressure on politicians to provide all necessary help to those attacked (only during the last month, only on one hot section of the frontline—near Avdiivka—Russia lost more tanks than the number of modern tanks all international partners provided to Ukraine throughout the entire war, after a year of begging) and severe all ties with the attacker.

Public pressure on Russia itself and all millions of Russians who support the war—some enthusiastically, some by their indifference—a public pressure that will clearly tell them what they are doing will never become "some political nuance."

Public pressure on your own compatriots, who first try hard to ignore all information about what's _really happening_, and then put _their_ pressure on the politicians to "stop spending so much attention and money on that nonsense we don't really know a thing about."

At the very least, in those first naive months, I hoped for small tokens of public support, like a stray Ukrainian flag on a homepage, a few retweets, a clear and unambiguous statement. With very few exceptions, I saw none. I hoped to see my fellow Rubyists as members of the awesomely whimsical and incredibly helpful "[fellas](https://en.wikipedia.org/wiki/NAFO_(group))" movement; remembering earlier days of the community, it had just the right kind of appeal— But, alas, I personally know no Rubyist who became a "fella."

(I heard many had—at least in the early days—helped Ukrainian refugees in their countries. We are thankful for that, and it is incredibly important, but the weakness of _public_ reaction to events means that those refugees might never have proper relief.)

I was additionally shocked by the conferences: **for the Ruby community**, many of the prominent conferences were a place to discuss important humane topics: equity and equality, oppression and race—and I always appreciated that.

But genocide turned out invisible for them, too.

I vividly remember that on one, European, a talk fully dedicated to empathy in software and cautioning from the opinion that "technology doesn't have anything to do with politics," Ukraine wasn't mentioned at all. The conference was held in Finland, which by that moment hosted tens of thousands of Ukrainian refugees.

On another, a similar themed talk was listing the Russian invasion at the end of a long list of "bad things that are currently happening in the world" (far below income inequality in IT and DHH's and Musk's behavioral problems) before switching to "what should we do" (which never bothered by "maybe stop supporting evil dictators by doing business with the companies that pay taxes to them").

But you know what?

## I get it. I think I do.

We, modern people (especially Western ones, probably? because I mostly have experience of a European, even if Eastern one), actually have very few tools to cope personally with events as huge and hard as imperialistic genocide performed by a seemingly normal country in the middle of Europe, with the highest officials of the said country openly admitting their deeds and intentions of taking the land and killing or "reeducating" everybody in it.

This is just too big a fact to recognize and admit and too "black-and-white" one to ring true. (And once you admit it, it [requires a lot of involvement](https://twitter.com/JuliaDavisNews/status/1701565551942709408) to keep thinking of yourself as a good person!)

So, some kind of emotional "bikeshedding" happens here!

> I am using "bikeshedding" in its original meaning from the "Parkinson's law". There, people are spending 5 minutes on discussing 5-cent problem (too small to care), another 5 minutes to discuss 2-bln problem (too big to really relate to), and then 2 hours on discussing how much should be spent on building a bike shed: something everybody can imagine, evaluate, and relate to. The important difference from today's IT meaning ("bikeshedding" = "discussing something that isn't worth discussion at all") is that "bike shed"-scale problems are still important, yet they take _unproportional_ amount of attention compared to much bigger ones.

So, yeah, in this meaning, income inequality in a generally well-paid industry might _to some extent_ be a "bike shed"-scale problem when compared to a [large city being besieged](https://en.wikipedia.org/wiki/Siege_of_Mariupol) and destroyed, with some 25k of its residents being killed and thousands of others (many of them children) forcibly relocated into the attacking country.

But nobody wants to discuss the latter at programming conferences, even on their "humane" tracks. Nobody wants to imagine that might've happened to them, too.

The human culture conveniently provides a lot of convenient fables and vantage points, from "local political events in the far and turbulent country" through "it is always generals and politicians fighting, nobody's right, just stay away from that dirty matters," and all the way to "but what about US imperialism?"

In other words, we  use **thinking patterns** to organize problems too large to handle.

## Are you still with me?

_Remember, it was intended as a talk for a Ruby conference, right?_

By the end of the summer of '22, I managed to partially get back to Ruby and programming not only for a bare money-earning minimum but also as something I am interested in and reflect upon. (That's when the [`Data` class](https://rubyreferences.github.io/rubychanges/3.2.html#data-new-immutable-value-object-class) was made.)

Still, I had much less resource to spare, both physically and mentally, and that made me focus and rethink what parts of my programming activities are essential to me and how I can do them better.

In my civilian job as a staff software engineer, I frequently play a role we half-jokingly call "chief editor" (as in "magazine's chief editor"). Basically, it is a person who reads a lot of code others write, shares the preferred approaches through the team, and helps maintain generally sound structure.

But frequently, it is manifested in review comments to small parts of the code, even singular phrases, with a common motive of "Can we make this _clearer_?" or "Let's try to restructure it a bit for _clarity_" (or, at times, "I don't get it, can you please _clarify_ what this code intended to do?..")

Mostly, this is not about _clean_ code—i.e., the code that is visually pleasant or matches some common style—but about _clear_ code: the one that provides enough context and exposes the intent (I use term the "lucid" to not constantly stumble upon this one-letter distinction). "Clean" and "clear" are neither mutually exclusive nor automatically follow from each other.

When met with this "per-phrase" approach, some people are initially baffled by "inexplicable" attention to code or, worse yet, explain that by malicious intent ("nitpicking to establish dominance"). Some are coming from the teams where discussion of "how you write the code" is a bad tone and only Big Serious Architecture deserves the discussion and should be argued about, while the rest is just the "subjective personal style" and can be written in whatever way first came to author's mind (that's where the "bikeshedding" word is frequently repurposed).

The same frequently happens in online discussions when I am trying to _explain_ how some way of structuring a phrase or two or some small syntactical feature works towards the improvement of the overall experience of the reading. The response is frequently not just a disagreement (which I totally OK with), but a _bored disagreement_, along the lines of "You are wrong, but the problem is so minuscule I wouldn't even try to explain" (which frequently means "wouldn't bother trying to understand how _I perceive something myself_, and that's why I can't explain it").

I'll tell you what.

In my "chief editor" life, there is a repetitive story of how a struggle to state a few phrases of code _clearer_ uncovers an architectural-layer problem, otherwise invisible through a massive of visibly _clean_ code that follows the common style and gives descriptive names to everything and split into small methods yet loses "what was intended to say here" somewhere in the middle. Not a small amount of bugs and performance problems were timely caught this way, and some of the best ideas for structuring big subsystems were invented by my bright colleagues when we discussed "what is really intended here" for a few back-and-forth comments.

And in the most interesting cases, people already had some idea in mind about "how to say that better" (or, at least, an internal feeling of "something is murky here")—which they didn't follow, either because it felt like an unnecessary perfectionism, or, frequently, for a reason like "it would be good to have that one class here, but it is neither model, nor interactor, nor decorator, so probably it doesn't fit into our codebase."

Which gets us to the following.

## Hope and fight

I don't separate my "programmer self" from my "human self." Software development isn't "just a source of money" for me (actually, I rather identify as a writer who, _by a lucky coincidence_, also loves to write in programming languages). I think about the code in the categories of the text, and my life experience and my coding experience enrich and inform each other.

So, what am I thinking as a person at war? What insights integrate two themes that I am dealing with above: being shocked by the community's indifference to genocide and using phrase-level code reading to improve the architecture?

Maybe it would be just a too big leap of reasoning, but **I think the code might tell the truth**. And we have all ways to obscure this truth by putting the bare facts of "what comes from where and how it should be processed" in convenient boxes of patterns, frameworks, and idioms.

That's not an inherent flaw in the way we organize our thinking or our code.

The real world—and software implementations corresponding to its aspects—bears a lot of complexity, and to not be overwhelmed, we _need_ our shortcuts, our maps, and our alphabetically labeled drawers. An ability to recognize the repeating problem with a known solution, the system of classification of entities and their relations, DDD, MVC, interactors, components, and conventions are all helpful tools that allow us to handle systems of a scale unimaginable before. We became really friggin' good at it.

What we aren't that good at is noticing those systems' inadequacy to a changed or just better understood reality. Defending ourselves from complexity, we frequently eventually start defending from the truth.

Am I comparing the truth about genocide and the truth about everyday coding problems? I do. People will be people and do the same thing in all the contexts, being this context a world-scale tragedy becoming "old news" or a new kind of tabular report that doesn't fit into the old reporting system, but if we add those two boolean parameters to every method—

We mix our core beliefs and values, like "unprovoked aggression is bad" or "decoupling of concerns is good," with the particular system of thinking which holds them currently, without noticing when it limits the ways we can look at the problem: "unprovoked aggression is bad, and there can't be any serious war, it is just some dirty politics and news hype," or "decoupling is good, and we should put every thing in either model, view, or controller."

Of course, I am not urging you to abandon all the patterns and frameworks and run to the woods barefoot, singing (though some might consider it a useful exercise).

I just warn you to be vigilant and understand where old patterns do not match new facts, and when thinking frameworks we outgrew should be ditched. To drop structures and idioms once they are inadequate to the truth about changed reality—drop without mercy, and if necessary, accept the guilt and responsibility for following them in the first place.

Look for lucidity, not evenly-spaced, well-named "no-smart-code" trivialities (that should be obvious by this point, but let's spend 100 lines to repeat them).

See the real people. Write for the real people—and read what real people wrote, with all the redundancy, idiosyncrasies, joy, somberness, and struggle to understand. Be involved and take action whenever the truth is in danger.

And—if it wouldn't be too tremendously huge thing to ask—please [STAND WITH UKRAINE](https://twitter.com/walter_report/status/1684735384201117697).
