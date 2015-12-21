---
layout: post
title:  "Infoboxer 0.2.0: Effective Wikipedia data extraction"
date:   2015-12-21 16:30:00
categories: ruby gems
comments: true
---

**We are releasing [Infoboxer](https://github.com/molybdenum-99/infoboxer)
version 0.2.0 today -- which is still young but already
highly useful MediaWiki (including Wikipedia) hi-level _data client_**.

Please look at Infoboxer's [showcase](https://github.com/molybdenum-99/infoboxer/wiki/Showcase)
to have a feeling of what it can do for you.

Since [first announce](http://zverok.github.io/blog/2015-08-18-infoboxer.html)
in Aug 2015, Infoboxer already made a huge advance in stability and usability.

In new release:

* new backend library, allowing to receive various **lists of pages**;
* new shiny `infoboxer` **interactive console** for experiments;
* various cleanups and fixes.

## Lists of pages

Previously, Infoboxer allowed you only to fetch one or several pages from
MediaWiki by their titles:

{% highlight ruby %}
Infoboxer.wikipedia.get('Argentina')
# => #<Page(title: "Argentina")>

# or
Infoboxer.wikipedia.get('Argentina', 'Bolivia', 'Chile')
# => [#<Page(title: "Argentina")>, #<Page(title: "Bolivia")>, #<Page(title: "Chile")>]
{% endhighlight %}

Number of pages you could fetch this way was limited to 50 (MediaWiki
API limitation) in version 0.1.

Currently, with version 0.2.0 and new backend library, you can fetch as
many pages as you like in one `#get` call. Internally, it will be translated
in several API requests. Overall process can take some time, but eventually
you'll have them :)

Moreover, new version introduces `#category`, `#search` and `#prefixsearch`
methods for receiving whole lists of pages:

{% highlight ruby %}
# Category
# --------

cars = Infoboxer.wikipedia.category('Cars introduced in 2015')
cars.count
# => 31
cars.map(&:title)
# => ["Alfa Romeo Giulia (952)", "BMW 7 Series (G11)", "Borgward BX7", "BYD Tang", "Cadillac CT6", "Chevrolet Camaro (sixth generation)", "Chevrolet Volt (second generation)", "Ferrari 488", "Ferrari FXX-K", "Fiat 124 Spider (2016)", "Fiat Aegea", "Honda S660", "Hyundai Creta", "Jaguar F-Pace", "Koenigsegg Regera", "Maserati Levante", "Mazda CX-3", "McLaren 570S", "Mercedes-Benz GLC-Class", "MG GS", "Tesla Model X", "Opel Karl", "Renault Kadjar", "Renault Kwid", "Renault Talisman", "Rolls-Royce Dawn (2015)", "SsangYong Tivoli", "Tata Zica", "Ferrari F12berlinetta", "McLaren 650S", "Mercedes-Benz M-Class"]
cars.infoboxes.fetch('body_style').map(&:to_s).uniq
# => ["4-door saloon", "Sport utility vehicle (SUV)", "4-door Sedan", "", "5-door hatchback", "2-seat berlinetta\n2-seat convertible", "2-seat berlinetta", "2-door roadster", "4-door sedan\n5-door hatchback\n5-door station wagon", "5-door SUV", "2-door Targa top", "2-door coupé", "2-door SUV", "4-door saloon\n5-door estate", "3-door 2+2 coupé", "2-door convertible", "2-door berlinetta", "2-door coupé\n2-door roadster", "5-door crossover/sport utility vehicle"]

# Search
# ------

Infoboxer.wikipedia.search('"I am Groot"').map(&:title)
# => ["Griffon Ramsey", "Groot", "Guardians of the Galaxy (film)"]

# Prefix search (titles starting with)
# -------------

Infoboxer.wikipedia.prefixsearch('List of town tramway systems in').map(&:title)
# => ["List of town tramway systems in Africa", "List of town tramway systems in Argentina", "List of town tramway systems in Asia", "List of town tramway systems in Austria", "List of town tramway systems in Belarus", "List of town tramway systems in Belgium", "List of town tramway systems in Brazil", "List of town tramway systems in Central America", "List of town tramway systems in Chile", "List of town tramway systems in Croatia", "List of town tramway systems in Denmark", "List of town tramway systems in Europe", "List of town tramway systems in Finland", "List of town tramway systems in France", "List of town tramway systems in Greece", "List of town tramway systems in Hungary", "List of town tramway systems in Italy", "List of town tramway systems in Japan", "List of town tramway systems in North America", "List of town tramway systems in Norway", "List of town tramway systems in Oceania", "List of town tramway systems in Poland", "List of town tramway systems in Portugal", "List of town tramway systems in Romania", "List of town tramway systems in Russia", "List of town tramway systems in Scotland", "List of town tramway systems in Serbia", "List of town tramway systems in South America", "List of town tramway systems in Spain", "List of town tramway systems in Sweden", "List of town tramway systems in Switzerland", "List of town tramway systems in Ukraine", "List of town tramway systems in the Czech Republic", "List of town tramway systems in the Netherlands", "List of town tramway systems in the Republic of Ireland", "List of town tramway systems in the United Kingdom", "List of modern tramway & light rail systems in England", "List of street railways in Canada", "List of streetcar systems in the United States", "List of urban tram networks in Germany"]
{% endhighlight %}

**Warning**: currently, there's no way to limit number of pages fetches.
So, saying something like `Infoboxer.wikipedia.search('ruby')` may
require you to wait... a lot of time. Better lists fetching granularity
is on the way!

## New backend

For Infoboxer's needs, we started to develop new generic low-level
MediaWiki client library, called [mediawiktory](https://github.com/molybdenum-99/mediawiktory).

It is still really young, yet, unlike all the rivals, it targets **all**
MediaWiki API, which is pretty hard yet useful. It tries to provide
structure, chanable, Arel/Sequel-alike query interface for perfoming
any task MediaWiki allows. A first glance to MediaWiktory functionality:

{% highlight ruby %}
client = MediaWiktory::Client.new('https://en.wikipedia.org/w/api.php')

response = client.
  query.
  generator(categorymembers: {title: 'Category:Countries_in_South_America', limit: 30}).
  prop(revisions: {prop: :content}).
  perform

# MediaWiktory handles "next page fetching" for you, if you want
response.continue! while response.continue?

# MediaWiktory parses response and provides smart shortcuts
p response.pages.map(&:title)
{% endhighlight %}

## Interactive `infoboxer` console

Installs as a binary alongside infoboxer gem.

Generic usage with access to any wiki:

<pre>
> infoboxer
$ wiki('http//mywiki.com/w/api.php').get(....)
$ wikipedia.category(....)
</pre>

With predefined wiki:

<pre>
> infoboxer -wikipedia
Default Wiki selected: https://en.wikipedia.org/w/api.php.
Now you can use `get('Pagename')`, `category('Categoryname')` and so on.

$ get('Argentina')
</pre>

With any wiki:

<pre>
> infoboxer -w http://mywiki/w/api.php
Default Wiki selected: http://mywiki/w/api.php
Now you can use `get('Pagename')`, `category('Categoryname')` and so on.

$ get('MyProduct')
# you can still use any other wiki:
$ wikipedia.get('Argentina')
</pre>

## Cleanups, fixes, shortcuts and so on

Internally, there were a lot of work done for cleanup of code, providing
and documenting new smart features, like `Template#to_h` allowing to
convert entire large infoboxes into hashes in a blink.

But, there's a long way ahead of us! You can contribute to infoboxer
development by describing (or fixing!) [issues](https://github.com/molybdenum-99/infoboxer/issues)
or just by giving it a try and provide us with your precious feedback.

Thanks.
