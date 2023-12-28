---
layout: advent
title: "Advent of changelog: Day 16"
next: day17
next_title: "Day 17"
prev: day15
prev_title: "Day 15"
---

Following the [yesterday's plan](day15.html), I've started the day with a [small PR](https://github.com/ruby/pathname/pull/33) to a `pathname` standard library, adding to it a long-awaited `#chdir` method, consistently with a newly introduced `Dir#chdir`. I am not sure it isn't too late for the feature to make it into 3.3, but I hope it would be merged some day.

Now, let's put a structure into that drafty changelog!

For that, I am starting to rendering it locally as a site: easier to see the list of everything to think about organization. The site is Jekyll-based, but "rendering" also includes the preparatory transformations: my `rake` tasks are converting the markdown that is easy to write in `./_src/3.3.md` to markdown that renders nice in `./3.3.md` (say, replacing any Markdown links to `docs.ruby-lang.org` with literal `<a` HTML tags with custom classes to stand out as "official docs links"); also those scripts produce `_data/book.yml` file that powers the site's nice ToC, from links to the headers extracted from the markdown source.

The conversion scripts are working as expected, but to run Jekyll locally on my (currently default) Ruby 3.2 turns out to be a small challenge. First, I need to update some gems Jekyll relies upon: some of them still used `Object#taint` that was deprecated [since Ruby 2.7](https://rubyreferences.github.io/rubychanges/2.7.html#safe-and-taint-concepts-are-deprecated-in-general) and removed [in 3.2](https://rubyreferences.github.io/rubychanges/3.2.html#removals).

Next, it wouldn't start because `webrick` needs to be added to `Gemfile` since being dropped from standard library [in 3.0](https://rubyreferences.github.io/rubychanges/3.0.html#libraries-excluded-from-the-standard-library). (Hm, I might've not rendered the changelog with the latest Ruby for a while... But it is understandable: for a long time, I was stuck on a work project still using 2.7, so it was my system's default. But it also meant I had less possibilities to use and appreciate/hate newer features, which I compensated with various open-source and private experiments.)

> It is all pretty trivial stuff, just a reminder (including a reminder to myself) that moving through Ruby versions is not completely frictionless even for a small hobby project. Though most of the problems, thankfully, are solved just by updating dependencies.

But soon,

```ruby
bundle exec jekyll serve
```

...and finally, we are online! (With some of the structure already started to be introduced before I thought about rendering.) It suddenly feels like a _lot_ of progress—even if it is just a representation of what's already there.

![](/img/advent2023/image20.png)

After some moving things around, a restructuring is finished:

![](/img/advent2023/image21.png)

Now it became more obvious that there are a lot less changes than the [last](https://rubyreferences.github.io/rubychanges/3.2.html) [several](https://rubyreferences.github.io/rubychanges/3.1.html) [years](https://rubyreferences.github.io/rubychanges/3.0.html): the ToC is much less nested than usual.

It is important to emphasize here that I prefer to keep "humane" structure, so if there is just one change related to refinements, or to modules, I wouldn't nest it into a "generic" section called "Refinements" or "Modules." This year, there are not a lot of grouping sections necessary (and even those present have 2-3 items each.)

In general, this whole business of ordering/structuring language constructs makes me equal parts proud and uneasy: I "invented" and maintaining some "logical" way of organizing everything in the language throughout version changelogs, generic [evolution](https://rubyreferences.github.io/rubychanges/evolution.html) list, and my go at semi-automated rendering of the whole [Ruby Reference](https://rubyreferences.github.io/rubyref/). In each case (and every for each year's version) the order/structure is somewhat different to better correspond to the narrative of the particular case. I really think that it helps navigation/reading, but also by doing so I am taking on myself some kind of "authority" to decide what is related to what...

Anyway, I think that (plus some quick copy-pasting into a future "Standard library" section) this would be it for today. Not that much accomplished, despite a slightly larger timespan, but it feels like we became a bit closer to the final form.

**Commit: [cbd60c2](https://github.com/rubyreferences/rubychanges/commit/cbd60c2)**
