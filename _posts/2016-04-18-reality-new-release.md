---
layout: post
title:  "New Reality release: comprehensive docs and other cool things"
date:   2016-04-18 16:20:00
categories: ruby gems
comments: true
---

Remember the [Reality](https://github.com/molybdenum-99/reality) library?
That thing which wants to make the entire world inspectable and computable
through Ruby, Wikipedia, Wikidata and other data sources? Like this:

{% highlight ruby %}
E('Eiffel Tower').coord.weather # => #<Reality::Weather(13°C, Clear)>
{% endhighlight %}

We have a new release today! It's not some breakthrough, more "way to
maturity" release, but contains some things you'd possibly wants to check
out.

## Docs

**Reality** is pretty well documented now. We have both [YARD docs](http://www.rubydoc.info/gems/reality/0.0.4)
for core classes and not-so-small [Wiki](https://github.com/molybdenum-99/reality/wiki)
on GitHub, where you'll find tutorials, some philosophy and tech details,
as well as detailed roadmap for future.

## `reality` command-line tool

**Computable world in your console**

Created by magnificent [@codenamev](https://github.com/codenamev), here
is command-line tool for query some things about reality right from your
terminal:

```
$ reality "Neil Armstrong"
----------------------------------
#<Reality::Entity(Neil Armstrong)>
----------------------------------
         awards: #<Reality::List[Order of Culture?, Air Medal?, Collier Trophy?, Congressional Space Medal of Honor?, Gold Star?, Korean Service Medal?, Eagle Scout?, Sylvanus Thayer Award?, Arthur S. Flemming Award?, Distinguished Eagle Scout Award?, Langley Gold Medal?, Silver Buffalo Award?, Presidential Medal of Freedom?]>
    birth_place: #<Reality::Entity?(Wapakoneta)>
       birthday: #<Date: 1930-08-05>
    citizenship: #<Reality::Entity?(United States of America)>
  date_of_death: #<Date: 2012-08-25>
         father: #<Reality::Entity?(Stephen Koenig Armstrong)>
     given_name: "Neil"
    occupations: ["astronaut", "professor", "test pilot", "United States Naval Aviator", "aerospace engineer"]
  organizations: #<Reality::List[Boy Scouts of America?, Kappa Kappa Psi?, Phi Delta Theta?, Purdue All-American Marching Band?]>
        part_of: #<Reality::List[NASA Astronaut Group 2?]>
 place_of_death: #<Reality::Entity?(Cincinnati)>
            sex: "male"
         spouse: #<Reality::Entity?(Janet Shearon)>
```

It can be called as a chainable set of methods!

```
$ reality "Chiang Mai" country head_of_state father
Mahidol Adulyadej
```

And, for those alread accustomed for reality interactive console, it
is still here, just add `-i` key:

```
$ reality -i
reality#1:001:0> E('Haruki Murakami').birth_place.describe
------------------------------
#<Reality::Entity(Kyoto):city>
------------------------------
      adm_divisions: #<Reality::List[Ukyō-ku?, Nakagyō-ku?, Yamashina-ku?, Shimogyō-ku?, Nishikyō-ku?, Minami-ku?, "Kita-ku, Kyoto"?, Fushimi-ku?, Higashiyama-ku?, Kamigyō-ku?, Sakyō-ku?]>
               area: #<Reality::Measure(827 km²)>
              coord: #<Reality::Geo::Coord(35°1′0″N,135°45′0″E)>
            country: #<Reality::Entity?(Japan)>
 head_of_government: #<Reality::Entity?(Daisaku Kadokawa)>
         located_in: #<Reality::Entity?(Kyōto Prefecture)>
          long_name: "Kyoto"
   official_website: "http://www.city.kyoto.lg.jp/"
         population: #<Reality::Measure(1,474,410 person)>
          tz_offset: #<Reality::TZOffset(UTC+09:00)>
```

## Experimental `Reality::Names` module

**Every constant in the world**

That's pretty new idea and I'm still vague about it, but experimental
module `Reality::Names` allows you now to think about world objects
as just a Ruby constants:

{% highlight ruby %}
# standalone:
Reality::Names::GuidoVanRossum.age
# => 60

# include it elsewhere:
include Reality::Names

London.coord.distance_to(Paris)
# => #<Reality::Measure(343 km)>
{% endhighlight %}

Still can't deside, whether it is cool or harmful, looking forward for
your feedback!

## Other goodies

There's a lot of small and not-so-small changes in Reality internals;
and more useful data properties here and there.

Also, there are some useful new features, like:

**Basic country economy indicators** integrated from
[Quandl/ODA](https://www.quandl.com/data/ODA) dataset:

{% highlight ruby %}
E('Nepal').economy.gdp
# => #<Reality::Measure(21,000,000,000 $)>
{% endhighlight %}

**`Entity#to_h`** converting entity to basic types, ready to be serialized:

{% highlight ruby %}
require 'pp'
pp Reality::Entity('Bernie Sanders').to_h
# {:name=>"Bernie Sanders",
#  :birth_place=>"Brooklyn",
#  :sex=>"male",
#  :father=>"Eli Sanders",
#  :spouse=>"Jane O'Meara Sanders",
#  :citizenship=>"United States of America",
#  :position=>"United States Senator",
#  :occupations=>["writer", "politician"],
#  :awards=>
#   ["Congressional Activism Award",
#    "Col. Arthur T. Marix Congressional Leadership Award"],
#  :residence=>"Burlington",
#  :birthday=>"1941-09-08",
#  :given_name=>"Bernard",
#  :official_website=>"http://www.sanders.senate.gov",
#  :twitter_username=>"SenSanders"}
{% endhighlight %}

...and [more](https://github.com/molybdenum-99/reality/blob/master/CHANGELOG.md)!

## Call to action!

Reality is still at the beginning of long and adventurous road! We'd be
happy to see your ideas, pull requests, questions and application examples.

Our Wiki now have [Contributing](https://github.com/molybdenum-99/reality/wiki/Contributing)
page, which roughly lists the areas where you can lay your hands on.
[Roadmap](https://github.com/molybdenum-99/reality/wiki/Roadmap) and
[Applications](https://github.com/molybdenum-99/reality/wiki/Applications)
pages are listing our near and far plans. [GitHub issues](https://github.com/molybdenum-99/reality/issues)
list current ideas and problems.

We really want to do something cool, want to do it with Ruby and for
Ruby community, and want you to be part of it!

