---
layout: post
title:  yield_self is more awesome than you could think
date:   2018-01-24 12:40:00
categories: ruby rants
comments: true
---

**...the name still sucks, tho**

As you probably have been told, Ruby 2.5 has introduced `Kernel#yield_self` method, now available in any object. On the first sight, it does nothing fancy: just, as the name suggests, yields object it is applied to and returns the result:

```ruby
1.yield_self { |i| i + 2 } # => 3
```

Despite of how simple it may look (almost foolishly so), the method is a great addition for cleaner chains of values processing:

```ruby
# before
url = construct_url
response = Faraday.get(url)
data = JSON.parse(response.body)
id = data.dig('object', 'id') || '<undefined>'
return "server:#{id}"
# or, short yet less readable form:
return "server:" +
  (JSON.parse(Faraday.get(construct_url)).dig('object', 'id') || '<undefined>')

# after:
construct_url
  .yield_self { |url| Faraday.get(url) }
  .yield_self { |response| JSON.parse(response) }
  .dig('object', 'id').yield_self { |id| id || '<undefined>' }
  .yield_self { |id| "server:#{id}" }

# with method()
construct_url
  .yield_self(&Faraday.method(:get))
  .body
  .yield_self(&JSON.method(:parse))
  .dig('object', 'id').yield_self { |id| id || '<undefined>' }
  .yield_self { |id| "server:#{id}" }
```

There always will be voices about the last form being "obscure" and "unreadable for novices", but for most of us it is pretty idiomatic and play well with habits and intuitions we developed while processing collections in the same way (chaining the processing blocks with `map`, `select`, `reject` and others).

> **Note**: A lot of people were seen to prefer Elixir-y `|>` pipe operator instead of the method. What the pipe operator can't do, is play well with existing syntax and ecosystem, which is already seen in this small example: how chaining something AFTER the pipe operator should look? `body |> JSON.method(:parse).dig(...)`? `(body |> JSON.method(:parse)).dig(...)`? Where the parentheses should start, then?..

> **Another note**: `method(:name)` notation is useful and idiomatic, yet obviously longer than most of us would like. [Here](https://bugs.ruby-lang.org/issues/13581) is the ongoing discussion in ruby-core about possible syntax shorthand.

So far, nothing new.

But the fact that `yield_self` is designed to play well with the ecosystem have some interesting consequences. Let's look at them.

> **One more note**: Depending on your mindset, you may adore or loathe the following examples. I am **not** saying it is the way to do things, but the fact that it all works and looks logically, sometimes make my mind ticklish in a pleasant way.

Let's start with something simple.

```ruby
"Ruby".yield_self # => #<Enumerator: "Ruby":yield_self>
```

Hm, that's already interesting. Where can it lead us?

```ruby
"Ruby".yield_self.to_a # => ["Ruby"]
```

Not that fancy, yet imagine it is the last statement of 3-5 methods chain? Probably if you need an array in the end (for further processing by methods consuming only arrays), it could be the cleanest way to convert.

But that's not the end; we are just warming up.

Imagine farther processing requires a pair of values, like `(id, human title)`, but this particular method fetches just an id. And you want to return `[id, id]`

```ruby
# normal way
id = really.long.chain.of.processing
[id, id]

# wicked way
"Ruby".yield_self.cycle.take(2)
# => ["Ruby", "Ruby"]
```

Or imagine some page parser fetches page's paragraphs, and in the end, you want to make them into pairs `[page_title, paragraph]`:

```ruby
# normal way
paragraphs.map { |para| [page_title, para] }

# wicked way
%w[2.2 2.3 2.4].zip('Ruby'.yield_self.cycle).map(&:reverse)
# => [["Ruby", "2.2"], ["Ruby", "2.3"], ["Ruby", "2.4"]]
```

BTW, if it is still not enough wickedness for you, you may be interested in trying this example and understanding how it works:

```ruby
def fmt((software, version))
  puts "#{software}, v#{version}"
end
%w[2.2 2.3 2.4].zip('Ruby'.yield_self.cycle).map(&:reverse).each(&method(:fmt))
```

Another one, using `Enumerator::Lazy` for postponed computation:

```ruby
require 'open-uri'
postponed = 'http://ruby-lang.org'
  .yield_self.lazy.map((&method(:open)).map(&method(:read)).map(&:length))
# => #<Enumerator::Lazy: #<Enumerator::Lazy: #<Enumerator::Lazy: #<Enumerator::Lazy: #<Enumerator: "http://ruby-lang.org":yield_self>>:map>:map>:map>
# Nothing is performed still!

postponed.first
# => 1002
```

But the most useful example I can think about is "nullifying wrong results" one. Like returning `nil` on server errors. (I _know_ about "`nil` considered harmful" objection, OK? Just don't start.)

```ruby
# before
def fetch_something(url)
  response = Faraday.get(url)
  response.success? ? response.body : nil
end

# after
def fetch_something(url)
  Faraday.get(url).yield_self.detect(&:success?)&.body
end
```

More complex examples:

```ruby
# before
response = Faraday.get(url)
return nil unless response.success?
data = JSON.parse(response.body)
...

# after
Faraday.get(url)
  .yield_self.select(&:success?)
  .map(&JSON.method(:parse)) # It is now like ye olde functional "map" here

# before
value.match(DATE_PATTERN) ? Date.parse(value) : nil
# after
value.yield_self.grep(DATE_PATTERN).map(&Date.method(:parse)).first
```

## Don't try this at home!

**...Or do?**

To be completely honest, I still have a mixed feeling (to say the least) of those examples (the last one, with result filtering, looking the most pragmatic). But the fact that it is logically possible entertains me a lot. And I wanted to entertain you, too.

PS: Still hate the name. It is like having `enumerate_and_return_values` instead of `map`.
