---
layout: post
title:  "The case study for The Last API Wrapper"
date:   2017-07-31 12:30
categories: ruby gems
comments: true
---

**TL;DR:** This post shows why you probably should consider using ["The Last API Wrapper"](https://github.com/molybdenum-99/tlaw)
next time you need to get something from HTTP API.

[GIPHY](https://giphy.com/)'s API is taken as an example.

## Preface

Idea of this example was born when I've seen recently created [giphy_api](https://github.com/faragorn/giphy_api)
gem. I don't want to offend author of this (clean and well designed) gem in any way, but it inspired
me to show an example of what it takes to create API wrapper from scratch, even for simple API, and
what other options we can have.

`giphy_api`'s `lib` folder contains 300 lines of code in 13 files. It follows usual best practices for
this kind of gems:

* defines its own `Connection` abstraction (wrapper around HTTP client);
* defines its own configuration DSL (global in this case, e.g. you can't use two different API keys
  in the same code, which, in fact, is _not_ the best practice);
* wraps all domain objects (`Gif`, `Sticker`, `Image`, `User`) in their own classes.

## What this all about?

Lots of people have done this kind of thing lots of time, for all kinds of APIs you can imagine,
starting the wheel over and over again: connection, exceptions, configurations, domain objects...
Repeat.

And still, there are much more APIs that are still lacking decent Ruby wrapper, not talking about
unmaintained or badly designed wrappers, or several competing ones for the same task.

So, maybe there can exist some common foundation to DRY up the task?..

## Showcase of TLAW: API wrapper for GIPHY

**Preparation:**

* Open GIPHY API [docs](https://developers.giphy.com/docs/) (obviously);
* `gem install tlaw` ([git repo](https://github.com/molybdenum-99/tlaw), [docs](http://www.rubydoc.info/gems/tlaw)).

Create API wrapper class:

```ruby
class Giphy < TLAW::API
  define do
    # all DSL definition will go to this block,
    # in order not to pollute resulting class with DSL methods
  end
end
```

Documentation says, that all API endpoints start with `http://api.giphy.com/v1/`, and have mandatory
`api_key` param (Note, that GIPHY [policies](https://developers.giphy.com/docs/#api-keys) require you
to use different API keys for development and production.)

```ruby
class Giphy < TLAW::API
  define do
    base 'http://api.giphy.com/v1'

    param :api_key, required: true
  end
end
```

There are several [endpoints](https://developers.giphy.com/docs/#technical-documentation), basically
grouped into work with **gifs** and **stickers** (transparent gifs suitable for chats):

```ruby
class Giphy < TLAW::API
  define do
    # ...
    namespace :gifs do # Now API object will have a #gifs method for accessing the namespaces
                       # URL prefix also automatically deduced to be '/gifs'
    end

    namespace :stickers, '/stickers' do # ...but can be rewritten, if you want method name to be different
                                        # from URL prefix.
    end
  end
end
```

For the sake of simplicity, let's focus on implementing only [/gifs/search](https://developers.giphy.com/docs/#operation--gifs-search-get)
endpoint (fortunately, GIPHY's API is pretty regular, so this example will be enough to continue
the work).

```ruby
class Giphy < TLAW::API
  define do
    # ...
    namespace :gifs do
      endpoint :search do # method #search, URL deduced to be '/gifs/search' and can be rewritten, also
      end
```

The documentation says it has 7 params: `api_key`, `q` (search query), `limit`, `offset`, `rating` (MPAA
rating of the GIF), `lang`, `fmt`. Of those seven, `api_key` is already whole API parameter, and for
`lang` it is also reasonable to be so. And `fmt` parameter is probably useless. So...

```ruby
class Giphy < TLAW::API
  define do
    base 'http://api.giphy.com/v1'

    param :api_key, required: true
    param :lang

    namespace :gifs do
      endpoint :search do
        param :q
        param :limit
        param :offset
        param :rating
      end
```

We can immediately investigate our object in IRB/Pry:

```ruby
> Giphy
# => #<Giphy: call-sequence: Giphy.new(api_key:, lang: nil); namespaces: gifs; docs: .describe>

> Giphy.describe # as hint above suggests
# Giphy.new(api_key:, lang: nil)
#   @param api_key
#   @param lang
#
#   Namespaces:
#
#   .gifs()

> Giphy.namespaces[:gifs]
# => #<Giphy::Gifs: call-sequence: gifs(); endpoints: search; docs: .describe>

> Giphy.namespaces[:gifs].describe
# .gifs()
#
#   Endpoints:
#
#   .search(q: nil, limit: nil, offset: nil, rating: nil)
```

...and really use it:
```ruby
giphy = Giphy.new(api_key: '<YOUR API KEY>')
# => #<Giphy.new(api_key: "856a5bc1a2304900bf4d60bc7e4dfea6", lang: nil) namespaces: gifs; docs: .describe>

giphy.gifs.search(q: 'tardis')
# => hooray! search results!
```

In 16 lines of code, we have working API wrapper. It also provides you with:

* on-the-fly docs, as shown above;
* errors handling with meaningful reporting;
* [some response post-processing](https://github.com/molybdenum-99/tlaw#response-processing) (out of
  the scope of this short showcase, but really useful).

There are also several other helpful methods in [DSL](http://www.rubydoc.info/gems/tlaw/0.0.2/TLAW/DSL),
let's look at some of them as a bonus.

Note the method signature for `#search` in on-the-fly docs:

```ruby
search(q: nil, limit: nil, offset: nil, rating: nil)
```

In fact, we can do it this way:
```ruby
endpoint :search do
  desc 'Search all GIFs by word or phrase.' # short docs for endpoint

  param :query,             # give param mor meaningful name
    field: :q,              # ...while preserving q in query string
    required: true,         # it should not have default value
    keyword: false,         # ...and probably should not be keyword argument, being ovious
    desc: 'Search string'   # ...and can have docs
  param :limit,
    :to_i,                  # ...passed value should be converted with value.to_i method
    desc: 'Max number of results to return'
  param :offset, :to_i, desc: 'Results offset'
  param :rating,
    enum: %w[y g pg pg-13 r unrated nsfw],  # only one of those is accepted, otherwise error thrown
    desc: 'Parental advisory rating'
end
```
...resulting in:
```ruby
> Giphy.namespaces[:gifs].describe
# .gifs()
#
#   Endpoints:
#
#   .search(query, limit: nil, offset: nil, rating: nil)
#     Search all GIFs by word or phrase.

> Giphy.namespaces[:gifs].endpoints[:search].describe
# .search(query, limit: nil, offset: nil, rating: nil)
#   Search all GIFs by word or phrase.
#
#   @param query Search string
#   @param limit [#to_i] Max number of results to return
#   @param offset [#to_i] Results offset
#   @param rating Parental advisory rating
#     Possible values: "y", "g", "pg", "pg-13", "r", "unrated", "nsfw"
```

You can now review "full" (with docs, some output post-processing and all endpoints implemented) [Giphy
wrapper](https://github.com/molybdenum-99/tlaw/blob/master/examples/giphy.rb) in tlaw's repo (it takes
75 LoC).

But one of the points of TLAW is: in many cases, you don't need the full wrapper. For example,
using GIPHY's [translate](https://developers.giphy.com/docs/#operation--gifs-translate-get)
functionality in chat-bot, all you need is a very simplistic client, but
still a lot of boilerplate code (errors processing, response parsing, data extraction). What TLAW
does is providing this "boilerplate foundation", so you can get straight to business.
