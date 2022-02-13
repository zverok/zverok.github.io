---
layout: post
title:  "Using Wikipedia as an Impromptu RottenTomatoes API"
date:   2021-05-31
categories: opendata wikipedia ruby
comments: true
---

Last week I posted an _errhm_ Tweetstorm making a few quite important points:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">Some of the hardest problems to bring in the software development ecosystem are those that &quot;intuitively&quot; have an easy answer.<br>My favorite one is about common sense knowledge.<br><br>1/Thread →</p>&mdash; zverok (@zverok) <a href="https://twitter.com/zverok/status/1397651367909629953?ref_src=twsrc%5Etfw">May 26, 2021</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

To rehash the two most important points:
1. It will be generally good (for the development community and humanity and progress) to have programmatic access to common-sense knowledge.
2. The best candidate for the source of this knowledge is Wikipedia—however chaotic and irregular it sometimes might be.

(The thread makes a lot of other good points, [go read it](https://twitter.com/zverok/status/1397651367909629953)!)

## The task

To make a practical demonstration of Wikipedia's ubiquitous knowledge-bearing power, I experimented on the weekend with one small piece of the data: movie ratings from [Rotten Tomatoes](https://www.rottentomatoes.com/).

There are several interesting traits for this data:
1. It is useful (more than IMDB ratings anyways)
2. There is no open API to fetch it (the official one is [invites-only](https://developer.fandango.com/rotten_tomatoes))
3. But most movie-related Wikipedia articles have RT ratings.

[!["Nomadland"'s one. Yes, it is that awesome](/img/2021-05-31-rt-nomadland.png)](https://en.wikipedia.org/wiki/Nomadland_(film)).

BTW, formalized [Wikidata item](https://www.wikidata.org/wiki/Q61740820) doesn't have this data free-form Wikipedia text has.

## The experiment

So, can we extract this data automatically? Yep. Not without some quirks, but we can:

```ruby
Infoboxer.wikipedia.category('Best Drama Picture Golden Globe winners').each do |page|
   # "Titanic (1991 film)" => "Titanic"
  title = page.title.sub(/ \(.*film\)$/, '')
  print "#{title}: "

  # Find top-level section which might contain a reception.
  # Most of the time it is "Reception" or "Release and response".
  reception = page.sections(/reception|response/i).first ||
    page.sections(/release/i).first or
    (puts "no Reception section"; next)

  # In all section sentences, find one containing "Rotten"
  rating = reception.text.split(/\n|\. /).grep(/Rotten/).first or
    (puts 'No RT ratings'; next)

  # There are many ways of say it in Wikipedia, but we know that
  # the phrase will contain something like "58% according to 20 reviews"
  # plus (not always) average in form of "8.8/10".
  puts("%{rating} (%{of}), avg %{average}" % {
    rating: rating.slice(/\d+%/),
    of: rating.slice(/\d+ (critics|reviews)/),
    average: rating.slice(/[\d.]+\/10/) || '—'
  })
end
```

This code uses a high-level Ruby client for Wikipedia [infoboxer](https://github.com/molybdenum-99/infoboxer) (created by yours truly), which effectively fetches page sources in 50-page batches, parses them, and provides a nice DOM API for the parsed Wikitext.

It prints this:
```
12 Years a Slave: 95% (365 reviews), avg 8.91/10
1917: 89% (454 reviews), avg 8.40/10
All the King's Men: No RT ratings
Amadeus: 93% (95 reviews), avg 8.92/10
American Beauty: 87% (190 critics), avg 8.2/10
Anne of the Thousand Days: No RT ratings
Argo: 96% (358 reviews), avg 8.40/10
...
```
(And so on, 81 movies in a blink of an eye.)

**See [this gist](https://gist.github.com/zverok/04c8c6eee593425e96746d45161f57b1) for self-contained example and full output.** Try changing the category name to [something else](https://en.wikipedia.org/wiki/Category:Film_award_winners), or extract "critics consensus" from the article, if you wish.

## The quirks?

I, myself, was surprised by how easy it was with the first movie I've tried.

Of course, I polished _infoboxer_ for many years just for the tasks like this. But honestly, even using raw Wikipedia API with some generic HTTP client and parsing data with regexps— It would be maybe twice as much code, no big deal either.

Then I've tried to run it on the whole category, and that's when the [irritating irregularities](https://twitter.com/zverok/status/1397665705185759235) of plain-text encyclopedia started to manifest.

There almost always were "Critical response" section... or "Reception". Or "Reception" nested in "Critical Response". Or "Release > Reception". The phrase itself has as many forms as there are movies, like:

* ...has a 96% "Fresh" rating at Rotten Tomatoes, based on 56 reviews...
* ...has an 79% rating on the review aggregation website Rotten Tomatoes based on 195 reviews...
* ...75% of critics gave the film a positive review, with an average rating of 7.50/10, based on 103 reviews...
* ...records that 83% of 217 critics gave the film positive reviews....
* _and so forth._

### The formalization?

There is a funny sideways story here: some of the movies articles (not from this list, but [here](https://en.wikipedia.org/wiki/The_Resort_(film)) is an example) tried to introduce formalized "template" for the task, so the source wikitext would look like this:
```
{% raw %}{{Rotten Tomatoes prose|40|3.80|15}}{% endraw %}
```

(still rendering to the one variation of the phrase above.)

Next thing that happened?..

![Delete it!](/img/2021-05-31-rt-for-deletion.png)

Wikipedia editors want to delete it. The [discussion](https://en.wikipedia.org/wiki/Wikipedia:Templates_for_discussion/Log/2021_May_18#Template:Rotten_Tomatoes_prose) is quite enlightening:

> There is zero consensus for establishing an overly-specific wording for how to write the aggregate score that Rotten Tomatoes has for a film. ...<br/>
> ...that discussion doesn't need to happen before deletion; even if everyone is happy with one fixed wording for this - which we shouldn't be, forcing it one way can be awkward...

So, Wikipedia editors are explicitly _against_ the formal specification of how to write some phrase. Is it almost like they are against automated parsing? (Winks.)

Note that turning a very regular part into some widget/infobox row is out of the question here. Being outside of the editors' community, I can't guess the reason. The status of this data quoting/copyright is questionable (is it [CC BY-SA](https://en.wikipedia.org/wiki/Wikipedia:Text_of_Creative_Commons_Attribution-ShareAlike_3.0_Unported_License), as Wikipedia?). So maybe they _are_ trying to make parsing it automatically harder?

BTW, stories like this one are why (after working on infoboxer for several years) I [suddenly understood](https://twitter.com/zverok/status/1397671159047544832) that parsing resulting HTML would be more robust than the source Wikitext. With Infoboxer, you can handle the template easily:

```ruby
Infoboxer.wp.get('The Resort (film)').templates(name: 'Rotten Tomatoes prose').first.variables.map(&:text)
# => ["40", "3.80", "15"]
```

...but to parse data with this, you need to know when somebody has introduced a template like this. They emerge, change names and syntax, and get removed all the time, so we can't actually rely on them for the automated parsing.

The (rendered) text of the article is much more robust.

## The conclusions

It is up to you. I'd rather ask some questions.

Is it a sane approach to data extraction? How well can it be scaled? What's the legal status of the data?

What's the value of presenting the regular facts in such irregular prose? Why does Wikipedia hold for it? Is there a middle way between "everything's prose" and "everything's RDF statement" (Wikidata), which WikiMedia project will follow?

Is there maybe an automated way to handle almost regular data, shouldn't some cool ML shtick help here?
