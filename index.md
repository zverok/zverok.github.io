---
layout: default
---

<img src="/img/me-odesa-2021.jpg" style="float:right; margin: 10px;"/>

<div class="callout">
<p>I am Victor Shepelev aka zverok. I am a software architect at <a href="https://hubstaff.com">Hubstaff</a>, committer of the <a href="https://ruby-lang.org">Ruby programming language</a>, author of several open-source projects, and writer.</p>

<p>In tech, my main areas of interest are open data and writing <i>lucid</i> code. I approach the code—and the world in general—as a text that we should learn to read and write better. <small><a href="/about/">More about me →</a></small></p>
</div>

**Follow me on [Twitter](https://twitter.com/zverok) and [Substack](https://zverok.substack.com).**

<div style="clear: both;" />

## Writing

I am currently working on my first Ruby book, titled "[41 Ruby Intuitions](/writing/#ruby-intuitions)" and writing a regular [blog](/writing/#blog)/[substack](https://zverok.substack.com) on the adjacent topics and my open-data related projects.

### Recent posts

{% for post in site.posts limit: 3 %}
* <a href="{{site.url}}{{site.baseurl}}{{post.url}}">{{post.title}}</a> (<time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %-d, %Y" }}</time>)
{% endfor %}

### Featured older posts

* [Grok {Shan, Shui}\*: Advent of understanding the generative art](/blog/2021-12-28-grok-shan-shui.html)
* [Game of Life in one Ruby statement... inspired by APL](/blog/2020-05-16-ruby-as-apl.html)
* [On sustainable testing](/blog/2017-11-07-on-culture-of-bdd.html)
* [Please stop calling it "magic"](/blog/2017-10-22-stop-magic.html)
* [MiniTest is not "Just Ruby", it is "Just Rails"](/blog/2016-10-09-minitest.html)

### [All posts →](/writing/#blog)

## Projects

### Current

* [Contributing to Ruby programming language](/projects/#ruby)
* Designing [WikipediaQL](/projects/#wikipedia_ql) query language for expressive parsing of Wikipedia and other wikis
* Writing that book!

### Other notable

* [Spylls](/projects/#spylls) – Pure Python spell-checker, “explanatory” full port of Hunspell;
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
