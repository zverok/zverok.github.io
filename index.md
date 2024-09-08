---
layout: default
---

<img src="/img/me-war.jpg" style="float:right; margin: 10px;"/>

<div class="callout" markdown="1">
I am Victor Shepelev aka zverok. Since March 2023, I am part of Armed Forces of Ukraine amidst [Russian invasion](https://war.ukraine.ua).

Besides that, I am a software architect at [Hubstaff](https://hubstaff.com), committer of the [Ruby programming language](https://ruby-lang.org), author of several open-source projects, and writer.

In tech, my main areas of interest are open data and writing _lucid_ code. I approach the code—and the world in general—as a text that we should learn to read and write better. <small><a href="/about/">More about me →</a></small>
</div>

**Follow me on [Twitter](https://twitter.com/zverok) and [Substack](https://zverok.substack.com).**

**[→ Support on "Buy me a coffee"](https://www.buymeacoffee.com/zverok)** <small>(Till the war end, all donations go to Ukrainian organizations supporting the military, or for my own equipment when necessary.)</small>

<div style="clear: both;" />

## Writing

When my other duties allow, I am writing a [blog](/writing/#blog)/[substack](https://zverok.substack.com), mostly dedicated to the programming languages' evolution, writing and reading code; mostly (but not exclusively) based on my Ruby experience.

Before the full-scale Russian invasion, I was working on an ambitionus Ruby book, titled "41 Ruby Intuitions". This project is postponed till after the war, but some smaller book is in the works now. Stay tuned.

### Recent posts

{% for post in site.posts limit: 3 %}
* <a href="{{site.url}}{{site.baseurl}}{{post.url}}">{{post.title}}</a> (<time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %-d, %Y" }}</time>){% endfor %}

### Featured older posts

* [That useless Ruby syntax sugar that emerged in new versions](/blog/2023-10-02-syntax-sugar.html) (an intro + ToC of a long series on various new Ruby syntax elements, their justification, and consequences)
* A series on writing Ruby Changelog and observing the evolution of the language:
  * [What you can learn by merely writing a programming language changelog](/blog/2022-01-06-changelog.html)
  * [Following the programming language evolution, and taking it personally](/blog/2022-01-13-it-evolves.html)
  * [Programming language evolution: with all that, we are still flying](/blog/2022-01-20-still-flying.html)
* [Grok {Shan, Shui}\*: Advent of understanding the generative art](/blog/2021-12-28-grok-shan-shui.html)
* [Game of Life in one Ruby statement... inspired by APL](/blog/2020-05-16-ruby-as-apl.html)
* [On sustainable testing](/blog/2017-11-07-on-culture-of-bdd.html)

### [All posts →](/blog/)

## Projects

### Current

* [Contributing to Ruby programming language](/projects/#ruby)

### Other notable

* [Spylls](/projects/#spylls) – Pure Python spell-checker, “explanatory” full port of Hunspell;
* [WikipediaQL](/projects/#wikipedia_ql): an experimental query language for expressive parsing of Wikipedia and other wikis;
* Rails libraries:
  * [the_schema_is](https://github.com/zverok/the_schema_is) -- Rails DSL for model annotation with DB schema, done right;
* Ruby libraries
  * [saharspec](https://github.com/zverok/saharspec) -- a set of RSpec addons for DRY-er specs;
  * [time_calc](https://github.com/zverok/time_calc) -- Simple time arithmetic in a modern, readable, idiomatic, no-"magic" Ruby;
  * [whatthegem](https://github.com/zverok/whatthegem) -- information about any Ruby gem in your terminal: general information, usage examples, popularity stats, changes and more;

### [All →](/projects/)

## Talks

* [The struggle for better documentation (for Ruby itself)](https://www.youtube.com/watch?v=2VVEcOyeYLA) at NoRuKo'20 ([slides](https://bit.ly/noruko2020zverok))
* [Language as a tool of thought](https://www.youtube.com/watch?v=iMBqqjkbvl4) at RubyConf Nashville'19 ([slides](http://bit.ly/rc19zverok))
* [Towards the post-framework future](https://www.youtube.com/watch?v=5UiBQtfRDUI&list=PLoGBNJiQoqRDJvwOYLuu7jnprRKhuc7Cp&index=10&t=1165s) at WrocLove.rb 2019 ([slides](https://docs.google.com/presentation/d/1ve4At8Vwww9ww3iM7BrQTTkBN9bWkOXmuSK2mmugSOQ/edit?usp=sharing))
* [When the whole world is your database](https://www.youtube.com/watch?v=x9GePP3B0oE&t=1s&list=PLe872Yf6CJWGYKLny9jFs9mLv0Z94m8k4&index=26) at RubyConf India 2018 ([slides](https://docs.google.com/presentation/d/1I4mznHUBhVVDxWfO2DRzxP4wNhs9Mmtx09SizLqIbaE/edit?usp=sharing))

[More talks →](/talks/)
