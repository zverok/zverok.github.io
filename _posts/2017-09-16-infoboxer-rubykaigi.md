---
layout: post
title:  "Making data API of MediaWiki sites"
date:   2017-09-16 04:30
categories: ruby gems
comments: true
---

**NB: This small case study was published as pre-[RubyKaigi 2017](https://rubykaigi.org/2017/) appetizer for my [talk](https://rubykaigi.org/2017/presentations/zverok.html).**

Wikipedia and its "sister sites" (Wiktionary, Wikivoyage, Wikisource etc.) currently is the most comprehensive index of entire human knowledge. This information is precious, yet hard to consume programmatically.

One reason is information's informal, "human-first" structure—yet it turns out there are a lot of established patterns and enough of formalities to be useful in automated environments.

But another problem of access to the knowledge is lack of the tool for MediaWiki markup parsing, which is surprisingly hard. The thing is, markup has grown organically, and, with time, from simple "text with minimal ASCII formatting", it grew into really complicated grammar, highly context-dependent and with a lot of special cases.

[infoboxer](https://github.com/molybdenum-99/infoboxer) tries hard to handle this problem gracefully, providing not only the parser, but also a way to easily navigate parsed syntax tree and write information extractors in a simple and readable way.

On RubyKaigi 2017, I am talking about this library, its design, and usage, in a lot of (nasty) details, but here I'd like to just show a quick, somewhat complicated and hopefully interesting showcase.

So, today I am using infoboxer to make a Kanji database. That's how we go:

* [This Wikipedia page](https://en.wikipedia.org/wiki/List_of_kanji_by_concept) contains list of kanji grouped by topic, which is engaging for gaijin like me;
* Each of kanji is linked to corresponding [Wiktionary](https://en.wiktionary.org/) ("wiki dictionary") article, which has a lot of formalized details on it.

Now, the story goes like this:

```ruby
root = Infoboxer.wikipedia.get('List of kanji by concept')
# => #<Page(title: "List of kanji by concept", url: "https://en.wikipedia.org/wiki/List_of_kanji_by_concept"): This Kanji index method groups ...>
links = root.wikipath('//wikilink[interwiki=wikt]')
#  => [#<Wikilink(link: "色", interwiki: "wikt"): 色>, #<Wikilink(link: "白", interwiki: "wikt"): 白>, #<Wikilink(link: "黒", interwiki: "wikt"): 黒>, #<Wikilink(link: "赤", interwiki: "wikt"): 赤>, #<Wikilink(link: "紅", interwiki: "wikt"): 紅>, ...1172 more nodes]
```

Let's stop at the second line (first one, I hope, is self-explanatory).

The `.wikipath()` method is used for navigating Wiki AST (abstract syntax tree) in the same manner that Nokogiri uses XPath to navigate through HTML AST.

To find what you need in parsed page, you need to know a bit about logic of MediaWiki markup. The good news is, infoboxer is designed to be very inspectable and understandable. For example, knowing nothing about "wikilinks" and stuff, you can try this for starters:

```ruby
root.children
# => [#<Paragraph: This Kanji index method groups ...>, #<Paragraph: Kanji with multiple meanings ma...>, #<Paragraph: >, #<Heading(level: 2): Physical properties and attribu...>, #<Heading(level: 3): Colours>, ...107 more nodes]
puts page.children[5].to_tree
# <Paragraph>
#   <Template[nowrap]>
#     <Var(1)>
#       色 <Wikilink(link: "色", interwiki: "wikt")>
#        Colours; <Text>
#      <Text>
#   <Template[nowrap]>
#     <Var(1)>
#       白 <Wikilink(link: "白", interwiki: "wikt")>
#        white; <Text>
#      <Text>
# ....and so on
```

Don't think a lot about the `Template` and `Var` things. What matters is: we see that what was represented as links in a browser, called `Wikilink` in MediaWiki/infoboxer terms, and the fact it links to a different wiki (not Wikipedia) is represented by `interwiki` modifier. So, we can continue.

```ruby
links = root.wikipath('//wikilink[interwiki=wikt]')
pages = links.follow
```

Here you'll need some patience because fetching 1000+ pages is not that fast... Though, not that slow either: on my notebook, it takes ~50 seconds, much less than you'll expect of, well, _fetching 1000+ pages_. That's because MediaWiki API supports batch fetching in 50-page chunks, and infoboxer, of course, supports it too and handles grouping links in batches by itself.

Now, we can parse the pages (I'll skip the "investigation" part, but it was also done by `.inspect`-ing infoboxer's parsed pages:

```ruby
def parse_kanji_page(page)
  section = page.sections('Japanese')
  base = section.wikipath('//template[name=ja-kanji]').first&.to_h || {}
  readings = section.wikipath('//template[name=ja-readings]').first&.to_h || {}
  {'name' => page.title}.merge(base).merge(readings)
end

results = pages.map(&method(:parse_kanji_page))

p results.first
# => {"name"=>"色", "grade"=>"2", "rs"=>"色00", "goon"=>"しき", "kanon"=>"しょく", "kun"=>"いろ-, いろえ", "nanori"=>"しか, しこ"}

File.write 'kanji.yml', results.to_yaml
```

...or, considering we started from "kanji by concept" page, slightly more complicated (entire script this time):

```ruby
gem 'infoboxer' # You'll need 0.3.1.pre, the freshest version
require 'infoboxer'

def parse_kanji_page(page)
  section = page.sections('Japanese')
  base = section.wikipath('//template[name=ja-kanji]').first&.to_h || {}
  readings = section.wikipath('//template[name=ja-readings]').first&.to_h || {}
  {'name' => page.title}.merge(base).merge(readings)
end

start = Time.now

page = Infoboxer.wikipedia.get('List of kanji by concept')
links = page.wikipath('//wikilink[interwiki=wikt]')

data = links
  .group_by { |link| link.in_sections.map(&:heading).map(&:text_).join(' / ') }
  .flat_map { |section, links|
    links.follow.map(&method(:parse_kanji_page)).map { |k| k.merge('concept' => section) }
  }

p data.first
# => {"name"=>"色", "grade"=>"2", "rs"=>"色00", "goon"=>"しき", "kanon"=>"しょく", "kun"=>"いろ-, いろえ", "nanori"=>"しか, しこ", "concept"=>"Colours / Physical properties and attributes"}

File.write 'tmp/kanji.yml', data.to_yaml
# => 196166

puts format('Finished in %.1f seconds', Time.now - start)
# => Finished in 82.9 seconds
```

That's it :)
