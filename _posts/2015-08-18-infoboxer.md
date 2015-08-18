---
layout: post
title:  "Infoboxer: use Wikipedia as a rich data source"
date:   2015-08-18 16:30:00
categories: ruby gems
comments: true
---

I remember times when Wikipedia seemed a new thing (and the times with
no Wikipedia at all). Despite all the critique, sometimes reasonable,
Wikipedia have grown to be a fascinating source of "common sense" data
about almost everything.

Being a programmer and open data enthusiast, I'm also kinda fascinated
about how hard to access this data in automated way.

Yes, Wikipedia have an [API](https://www.mediawiki.org/wiki/API:Main_page),
and rather impressive one. Yet at some moment API leaves you with just
a flow of Wikitext in your hands: a text that definitely HAS repeating and
structured parts, which contains lots of data. Only it's really hard
to extract it in reasonable way (who said "regexps"?..)

DBPedia tries to target this problem by converting (part of) Wikipedia
information into structured RDF data, yet interaction with it is kinda
complicated, and selection of data is very limited.

Several monthes ago I've started a project of Wikipedia client and parser
which should make Wikipedia (and, by the way, any other MediaWiki wiki,
like Wikiquote, or [Doctor Who wiki](http://tardis.wikia.com/wiki/Doctor_Who_Wiki))
to rich data source, available for information extraction.

_That early May day the volume of work the project needs
seemed something between weekend and two... and here we are, at August 18._

Now we are talking!

## Meet Infoboxer

As I've already said, [**Infoboxer**](https://github.com/molybdenum-99/infoboxer)
is a MediaWiki client and parser. It goes as simple as that:

{% highlight ruby %}
Infoboxer.wikipedia.
  get('Argentina').infobox.fetch('leader_name1', 'leader_title1')
# => [#<Var(leader_name1): Cristina Fernández de Kirchner>, #<Var(leader_title1): President>]
{% endhighlight %}

What's going on here?

1. Infoboxer receives page "[Argentina](https://en.wikipedia.org/wiki/Argentina)" via Wikipedia API;
2. Then you can query page's [infobox](https://en.wikipedia.org/wiki/Help:Infobox);
3. Params names we are using here (`leader_name1` and `leader_title1`)
  you can see in [page source](https://en.wikipedia.org/w/index.php?title=Argentina&action=edit)
  or inspect on the fly:

{% highlight ruby %}
Infoboxer.wikipedia.get('Argentina').infobox.variables.map(&:name)
# => ["conventional_long_name", "native_name", "common_name", "image_flag", "image_coat", "national_motto", "national_anthem", "other_symbol", "other_symbol_type", "image_map", "map_width", "map_caption", "capital", "latd", "latm", "latNS", "longd", "longm", "longEW", "largest_city", "official_languages", "ethnic_groups", "demonym", "government_type", "leader_title1", "leader_name1"
{% endhighlight %}

Infoboxer, despite the name, is **not only about infoboxes!** You have
full page tree parsed and easily navigable:

{% highlight ruby %}
page = Infoboxer.wikipedia.get('Argentina')

# headings
page.headings
# => [#<Heading(level: 2): Name and etymology>, #<Heading(level: 2): History>, #<Heading(level: 3): Pre-Columbian era>, #<Heading(level: 3): Colonial era>, #<Heading(level: 3): Independence and civil wars>, ...42 more nodes]

# page content grouped in sections:
page.sections('History' => 'Pre-Columbian era').paragraphs.count
# => 4 

# tables and pretty output
puts page.tables.first
# +----------------------+----------------------------+-------------------+
# |                        Government of Argentina                        |
# +----------------------+----------------------------+-------------------+
# |                      |                            |                   |
# | Congressional Palace | Casa Rosada                | Palace of Justice |
# | Seat of the Congress | Workplace of the President | Supreme Court     |
# +----------------------+----------------------------+-------------------+

# images...
page.images.map(&:caption).join('; ')
# => "The Cave of the Hands in Santa Cruz province, with indigenous artwork dating from 13,000–9,000 years ago; The surrender of Beresford to Santiago de Liniers during the British invasions of the Río de la Plata; Portrait of General José de San Martin, Libertador of Argentina, Chile and Peru; ...."

# jQuery/Nokogiri-alike navigation
page.
  sections('Geography' => 'Regions').
  lookup(:ListItem).
  lookup_children(:Wikilink, index: 0).      # link which is direct first child
  map(&:link)                                # of list item
# => ["Argentine Northwest", "Mesopotamia, Argentina", "Gran Chaco", "Sierras Pampeanas", "Cuyo, Argentina", "Pampas", "Patagonia"]


# ....and bunch of other cool stuff
Infoboxer.wikivoyage.get('Chiang Mai').
  sections('See' => 'Elephants').templates(name: 'see').
  fetch_hashes('name', 'address', 'price')
# => [{"name"=>#<Var(name): Baanchang Elephant Park>, "address"=>#<Var(address): 147/1 Rachadamnoen Rd>, "price"=>#<Var(price): 4500 baht a day (can be split b...>}
#     ...and so on...

{% endhighlight %}

There is _plenty_ more features, all of them (hopefully) well-structured
and thorougly documented in project's [wiki](https://github.com/molybdenum-99/infoboxer/wiki)
and [API docs](http://www.rubydoc.info/gems/infoboxer).

Using of all of it requires some preparation and studies (for example,
in most cases you need to understand what MediaWiki
[templates](https://en.wikipedia.org/wiki/Help:Template) are), yet, I
hope, it can be useful and usable tool.

## What next?

For now, Infoboxer is able to extract single page (or several of them)
from any MediaWiki-powered site, parse them and provide you with
easily-navigable parse tree. You also can follow links between pages with
a great ease, like:

{% highlight ruby %}
Infoboxer.wp.
  get('Argentina', 'Bolivia', 'Chile').  # several pages in one API query
  infobox.fetch('leader_name1').
    lookup(:Wikilink).
    follow.                              # "Follows" wikilink, extracting linked pages
                                         # as navigable array
    infoboxes.
    fetch_hashes('name', 'office', 'birth_date')
# => [{"name"=>#<Var(name): Cristina Fernández de Kirchner>, "office"=>#<Var(office): President of Argentina>, "birth_date"=>#<Var(birth_date): 1953-02-19>},
#    ...and so on
{% endhighlight %}

Yet many data-extraction capabilities (which API provides) are still
lacking. In nearest versions, Infoboxer will move towards things like
"pages from some category", "pages on search request" and other
list-of-pages-alike functionality. Ideally, things like "list of capitals
of all world countries with their mayors and population" should be done
in as few Infoboxer statements as it could be imagined.

There's plenty of room for further enchancement, cleanup and experiments.
That's exciting.
