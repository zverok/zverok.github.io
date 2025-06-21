---
layout: post
title:  'Notes on code, text, and war. Week 2: If code is text, then what?'
date:   2025-06-21
description: >
  Last time, I've highlighted "treating code as text" idea as something that I want dedicate a few posts to. Let's talk in generalities some more this time.
categories: code-as-text
comments: true
image: https://zverok.space/img/2025-06-21/cover.png
---

Last time, I've highlighted "treating code as text" idea as something that I want dedicate a few posts to. Let's talk in generalities some more this time.

<div class="blurb" markdown="1">
  **OK, if we look at code as a text, then what?**

  If this metaphor-based approach – bringing the mental model of one domain into another – is useful, it should have some consequences. What are we gaining by using the culture’s experience with natural language texts in software development?
</div>

For now, I want to briefly outline some consequences and then expand on them, hopefully, in future posts.

The code is frequently compared to a cultural artifact, and most of the time, as a juxtaposition. Sometimes, this juxtaposition seeks reconciliation, as in literate programming. Sometimes it is antagonistic, asserting the superiority of “precise engineering work” over something as intangible as culture. However, even those who compare coding to writing as a compliment frequently go for it as an emotional argument for coding being a creative work, too. In those discussions, you might hear that “good code reads almost like poetry”, meaning a synonym for generic “beauty,” something that is “joyful” to read or write.

Honestly, I am the last person to frown at defining code as “craft” and at statements that it might bring “joy.” I have been writing code (and texts) all my life, and I am in it because I really like to do it. But also, I know that to become better in coding, as well as in writing, “looking for joy” and “listening to your heart” is not enough. At least, for me.

And so we can use the comparisons for something other than emotional effect. We can translate our experience with texts to working with code.

Say, **on the level of a single phrase** (the level which Ruby’s expressiveness taught me to emphasize and cherish), we can talk about choice of “words”, the thesaurus that the language provides, and its expansions with the names we chose to become “words” in our code. We can talk about idioms and set phrases that are typical for language, and explore how they might be thought-provoking or thought-obscuring, or become thought-terminating cliches.

We can talk about even seemingly insignificant aspects, such as _layout_ and _punctuation_, and how line-breaking choices might emphasize or muddle the structure of what is said.

An answer to a primitive question like "assume `x` is a list of users, how would you produce a list of user names from it?" and its typical solutions in every language and community might expose, like a drop of water, a whole ocean of approaches and values.

In software development, many of those questions are often dismissed as “bikeshedding” and addressed with prescriptivist rules and tools that rewrite the code to adhere to a “common style.” This is frequently done without a distinction between genuine typos/cleanup (e.g., fixing a stray space where it is not intended to be) and enforcing mechanistic uniformity, where the style previously emphasized the flow of thought. The rise of “opinionated” (see: non-configurable, “because there is nothing to discuss”) code formatters is part of this problem.

Next, **on the level of code units** that structure and organize your phrases (methods/functions, modules, classes), we might, again, think about what structuring of _texts_ teaches us.

Visible **page** of text/code seems an important concept here: how much can you comprehend without scrolling/jumping around? The question is frequently neglected -- despite being of utter importance for the human perception -- in favor of, again, prescriptivistic rules, like requiring to structure everything in extremely small methods, or code layouting conventions that prefer very short lines with roughly one "word" (method call, argument, etc.) per line.

We might examine how well-written, comfortable-to-read texts are broken into sections, how important headers and call-outs emphasize key thoughts, while build-ups to them are packed together into paragraphs, and useful yet tangential facts are extracted into footnotes and sidenotes. We might discuss how section and paragraph sizes differ depending on genre and intended effect.

Moving to higher levels, such as **the whole project or a large component**, we might again look at the experience of organizing large groups of heterogeneous texts.

Obviously, most modern software projects (save maybe for some narrow and specific libraries) don’t provide a linear narrative like a book does. But I like to think about organizational principles of large magazines and modern media outlets as an inspiration for some ideas, without mindless borrowing of what is irrelevant for the sake of extending the metaphor.

Think of the difference in styles to match the purpose, but under the same guidebook.

Or, about the variety of roles and competences in text work. A genius editor is not necessarily a good writer; a literary editor and a managing editor who overlooks a rubric and the editor in chief being all different “hats” (even if not always different people). There are reporters, analysts, and fact-checkers, and they work in an elaborate dance of creating a whole. It is not hard to find equivalents for those roles in software projects. It is not that hard to imagine how people with distinct skill sets and temperaments find a matching role that makes them most efficient and productive. How they are keeping a common schedule, contributing their angles of view, and working towards the general integrity of the software project/publication.

* * *

This is a weird year to speak of any integrity, though. Ukrainians know it like no one else, ever bitter and growing more bitter by the day.

For a week now, the world seems to be forgetting about Ukraine completely. (Forgetting even more than before, I mean.) You know what I am talking about, what’s on the news and social networks everywhere.

I wouldn't share my opinions on the incredibly tangled Middle East politics.

But I want you to imagine, if just for a moment, how Ukrainians feel about the world’s “integrity” when we read about Western joint efforts to protect against an attack by a hundred Shahed kamikaze drones. The same Shahed Iran is being supplied to Russia. The same Shaheds that, for the last months, have attacked Ukrainian cities in quantities of **two to four hundred per night**. Frequently, many nights in a row.

No Western aircraft ever tried to help intercept them in our skies. The idea of using our “partners’” aviation for this task (for cities that are very far from the front lines!) was briefly entertained in discussions last year, and then swiftly and decisively abandoned by everybody with a great sigh of relief.

They were also very public and clear about this, not even maintaining some strategic public ambiguity, which in itself might deter Russia’s boldest moves for some time. And they do it all the time: after the first rumors of a possible “ceasefire” this May (which were baseless but sparked some “what next” discussions), most Western governments were very quick to emphasize that no peacekeeping contingent from _their_ country would be possible on Ukrainian soil. As clearly as possible, for Russia to hear and take into account. Again: not that we expected any! But those rushed assurances are a message in themselves, too.

It is like they perceive Russia with some split-brain malfunctioning: it is, at the same time, entirely harmless for them (so there is no need to help Ukraine push it back decisively) but also extremely powerful (so the "provocation" as moderate as UK or French plane helping to patrol Ukrainian skies 1000 km from the active battles might cause catastrophic consequences). I once theorized that [Western perception of Russia is kinda religious](https://zverok.space/writing/onwar/2024-04-09-1777789363545379180.html), like that of an unpredictable yet vengeful god.

Sorry for this (probably pointless) venting.

It is somewhat hard to build a deliberate thought structure, trying to translate years of thinking about being better in our professional skills in this world. In a world that seems to be in agreement in its collective drive to fall apart into incoherent fragments, and the whole industry of IT, the industry about building and maintaining systems, realigns with this fragmented vision.

I am well aware that my notes are fragmented, too. But I hope they will eventually amount to some cohesive vision. The vision that will be helpful to fellow programmers, if only in small ways.

See you next time.

---

**Thank you for reading. Please support Ukraine with your donations and lobbying for military and humanitarian help. [Here](https://war.ukraine.ua/), you'll find a comprehensive information source and many links to state and private funds accepting donations.**

**If you don't have time to process it all, donating to the [Come Back Alive](https://savelife.in.ua/en/) foundation is always a good choice.**

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
