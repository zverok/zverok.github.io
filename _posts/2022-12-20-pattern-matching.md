---
layout: post
title:  What not to forget when implementing a pattern-matching in Ruby for custom objects
date:   2022-12-20
description: "Trivial yet easy to mess up a set of things to consider, demonstrated on the latest core classes Time/Date."
categories: ruby new-features
comments: true
image: https://zverok.space/img/2022-12-20/code-sample.png
---

Hi there.

> I am Ukrainian Rubyist from Kharkiv. There is still war in my country: a full-scale invasion that Russia started on Feb 24, continuing its 8-year-long hybrid war. My city is further from the frontline now than it was a few months ago, and it is less direct threat, but the total blackout due to russian shelling was as recently as last Friday. (See also my two [March](2022-03-03-WAR.html) [posts](2022-03-15-STILL-WAR.html).)

Nevertheless, in the wake of Ruby 3.2 upcoming release, between my daywork, family, and volunteering, I have some time now to slowly return to writing about Ruby. So you might want to follow me on [Substack](https://zverok.substack.com/).

Here is a small(ish) practical article about some considerations for implementing proper pattern-matching for your classes.

## What are we doing

Ruby [introduced](https://rubyreferences.github.io/rubychanges/2.7.html#pattern-matching) proper powerful pattern-matching as an experimental feature at 2.7 and polished it through [3.0](https://rubyreferences.github.io/rubychanges/3.0.html#pattern-matching)-[3.1](https://rubyreferences.github.io/rubychanges/3.1.html#pattern-matching). It is a feature that allows branching by the structure of the data:

```ruby
case user
in role: 'admin', name:
  puts "Hello, mighty #{name}!"
in role: 'user', registered_at: ...Date.new(2022)
  puts "Hi, old friend"
else
  puts "I don't know you well yet"
end
```

Or check data structure:
```ruby
if post in status: 'pending', author: {role: 'editor'}
  # ...
```

Or unpack data with known structure:
```ruby
config => {db: {name:, user:, password:}, logger: {target:}}
# Here, `name`, `user`, `password` and `target` variables are available, set
# to particular values from config; or, if config didn't match the expected
# structure, NoMatchingPattern is raised.
```

The best thing is, not only naked arrays and hashes can be matched this way, but any object if it implements `#deconstruct` and/or `#deconstruct_keys` method.

For Ruby 3.2 (upcoming on Dec 24!) I've added the pattern-matching deconstruction to Ruby's [`Time`](https://github.com/ruby/ruby/pull/6594), [`Date`, and `DateTime`](https://github.com/ruby/date/pull/80).

I want to share some generic observations which might be helpful if you want to do it for your own objects.

Basically, you can now do, say, this (with `ruby-head` or with code we'll write during this article):
```ruby
require 'date'
case Date.today
in year: ...2022
  puts "It was a long time ago..."
in month: 6..8, day:
  puts "It was a nice day #{day} of a beautiful summer month..."
in wday: 3
  puts "It is Wednesday, my dudes!"
# ...etc.
```

So, here are a few simple rules to go by:

## Choose good keys

When some object is matched by a hash pattern (by `case` branch, `in` or `=>`), Ruby checks if the object responds to `#deconstruct_keys` method and calls it with keys the pattern contains. E.g., when we do this:
```ruby
# only match if the year is 2022; unpack month and day into local vars
Date.today in year: 2022, month:, day:
```
...ruby calls `Date#deconstruct_keys` with keys `[:year, :month, :day]`, and expects that it will return a hash with those keys and corresponding values.

Implementation is trivial:

```ruby
class Date
  # There are MANY ways to write this method, I am deliberately choosing
  # a very old-school one
  def deconstruct_keys(keys)
    res = {}
    keys.each do |k|
      case k
      when :year then res[k] = year
      when :month then res[k] = month
      when :day then res[k] = day
      end
    end
    res
  end
end
```

You can try this in Ruby 3.1, say (which doesn't have native `Date#deconstruct_keys`), and make sure this will work:
```ruby
if Date.today in year: 2022, month: 12, day:
  p "Dec #{day}!"
end
```
Actually, you can try it in 3.0 too, but then the brackets around the pattern were mandatory:
```ruby
if Date.today in {year: 2022, month: 12, day:}
  p "Dec #{day}!"
end
```

Is it good enough? One might say so: after all, we unpack all components of the date, what else can one need? But the good deconstruction is not necessarily minimal, so if we'll support `wday` (which Ruby 3.2 implementation does)...

```ruby
class Date
  def deconstruct_keys(keys)
    res = {}
    keys.each do |k|
      case k
      when :year then res[k] = year
      when :month then res[k] = month
      when :day then res[k] = day
      when :wday then res[k] = wday
      end
    end
    res
  end
end
```

...then we can do nice things like this "every first Thursday of the month in 2022" one-liner:
```ruby
# starting from the year beginning, produce an infinite sequence of + 1 day
Enumerator
  .produce(Date.new(2022)) { _1 + 1 }
  .lazy
  .select { _1 in day: ..7, wday: 4 } # take only those where we are in the
                                      # first 7 days
                                      # of the month, and it is Thursday
  .take_while { _1 < Date.today }
  .to_a
[#<Date: 2022-01-06>,
 #<Date: 2022-02-03>,
 #<Date: 2022-03-03>,
 # ...
```

## Don't fail on unknown keys

When we know there is only a limited set of valid input (`Date#deconstruct_keys` only supports certain keys), it is a big temptation to protect yourself from errors by raising an exception on unsupported input. In the case of `#deconstruct_keys` it would be wrong. Imagine we added this to the `case` above:
```ruby
      # ...
      when :day then res[k] = day
      when :wday then res[k] = wday
      else raise ArgumentError, "Unsupported key #{k}"
```

Now, if we have some complicated processing of a mixed set of times and dates in various formats and try to do something like
```ruby
case creation_mark
in year:,month:,day:,hour:
  # ...
in year:,month:,day:
  # ...
in String
  # ...
end
```
...then implementation that just ignores the unknown key would be skipped by the first branch (rightfully), while implementation that raises will break the whole matching with `Unsupported key hour (ArgumentError)`. The user's expectation most probably was, "you don't match, you just skip this branch!"

Another way to be caught up by this problem is trying to be smart with metaprogramming: it is so easy to implement the thing smartly:
```ruby
class Date
  # Super-duper modern Ruby (one-line method and implicit block argument)
  # + old good metaprogramming
  def deconstruct_keys(keys) = keys.to_h { [_1, public_send(_1)] }
end
```
It will work:
```ruby
Date.today in year: 2022 # OK
```
...but not for long
```ruby
timestamp = Date.today
# ...
timestamp in hour:, min: # Confusing NoMethodError
```

BTW, the interesting thing about the unrecognized key is that if it is present, the pattern would definitely not match, so it is OK to return no data at all. So doing this is OK:
```ruby
      # ...
      when :day then res[k] = day
      when :wday then res[k] = wday
      else return {} # return immediately, don't try any more keys
```


> Aside note: if somebody to really mix Date and Time in pattern-matching, Ruby provide a neat way to clarify what class is expected in addition to deconstruction (works out of the box, you don't need to implement anything else to support this):

  ```ruby
  case creation_mark
  in Time(year:,month:,day:,hour:)
    # ...
  in Date(year:,month:,day:)
    # ...
  in String
    # ...
  end
  ```

## ...but also don't support unknown keys accidentally

While raising on keys your object doesn't know anything about is a bad idea, just spitting them back is equally bad. The "cool metaprogrammed version" above one might want to fix like this:
```ruby
class Date
  def deconstruct_keys(keys)
    keys.to_h { [_1, public_send(_1) if respond_to?(_1)] }
  end
end
# So..
Date.today.deconstruct_keys(%i[year month day hour])
#=> {:year=>2022, :month=>12, :day=>19, :hour=>nil}
```

The problem with this implementation is that structural pattern matching frequently asks "if the data corresponds to the structure expected" (think duck typing on steroids), and by returning the key Date knows nothing about, we provide a false promise. Think something like this:

```ruby
def print_time(tm)
  tm => hour:, min:
  puts "%02i:%02i" % [hour, min]
end

print_time(Time.now)
# prints "22:01".
# But only on Ruby 3.2, before that, Time couldn't deconstruct :)
```

The first line of this method has an interesting effect: it both checks the contract of `tm` (it has `hour` and `min`, `NoMatchingPattern` raised immediately if the contract is not matched), and puts them into local vars for convenience.

By our flawed implementation that returns `nil` for any key provided, we'll break it in a hardly debuggable way:
```ruby
print_time(Date.today) # in `%': can't convert nil into Integer (TypeError)
```
...while the proper implementation (which doesn't return back unknown keys), also raises an error, but the one method's author expected and quite informative:
```
in `print_time': #<Date: 2022-12-19 ...>: key not found: :hour (NoMatchingPatternKeyError)
```

> There are, in fact, many problems with the "smart" implementations (like, it would respond to the weirdest things, like `d in iso8601:`), but that's only tangentially related.

## Don't forget about the "any key" option

The pattern matching clause can specify "put the rest of the hash in this variable":
```ruby
Date.today in year: 2022, **rest
```
In this case, the `#deconstruct_keys` method receives `nil`, and is expected to return all keys it supports, so the "right" implementation is actually this:

```ruby
class Date
  def deconstruct_keys(keys)
    if keys
      res = {}
      keys.each do |k|
        case k
        when :year then res[k] = year
        when :month then res[k] = month
        when :day then res[k] = day
        when :wday then res[k] = wday
        end
      end
      res
    else
      {year:, month:, day:, wday:}
    end
  end
end

Date.today in year: 2022, **rest
pp rest #=> {:month=>12, :day=>19, :wday=>1}
```

The interesting thing to note here is that actually `keys` argument is introduced for optimization: if it is known that the pattern only includes a couple of keys, your object is given a _possibility_ (not an obligation) to limit what it returns. But it also can return all known keys always: it wouldn't be an error!

It is mostly necessary when the calculation of some values is expensive, but in the case of an object as simple as `Date`, this optimization doesn't actually bring much. So, would I implement `#deconstruct_keys` in Ruby for a small custom `Date`-like class, I'd just go with simple
```ruby
class Date
  def deconstruct_keys(*) = {year:, month:, day:, wday:}
end
```

## Don't overdo it

As implementing `#deconstruct_keys` is not that hard, and pattern-matching looks nice and fancy, there sometimes would be a temptation to go the extra mile for nicety. Say, weekday numbers are confusing (why 0 is Sunday?!), so we could probably just do this:
```ruby
class Date
  def deconstruct_keys(*)
    {year:, month:, day:, wday:, wday_name: strftime('%A')}
  end
end
```
...and then have ourselves a nice cool easy to remember
```ruby
Date.today in wday_name: 'Monday' #=> true
```

While it might feel cool and expressive in some contexts, pattern-matching shines best on processing a lot of data, and we just now introduced, probably, a performance penalty for _every_ `Date` matching for the sake of "a bit nicer" statement in one or two places.

(While `strftime` is not the _slowest_ method on Earth, it is definitely slower than simple `Date` getters; and what I am trying to do here is to show the principle: think twice before `deconstruct`-ing extra value that might need to be calculated or even fetched from DB.)

## Deconstructing into an array

So far, we talked about `#deconstruct_keys`, the method that handles matching to hash patterns: anything looking like `key:`, or `key: pattern`. There is another method that _might_ be implemented: array deconstructor `#deconstruct`. Its protocol is simple: just return the object representation as an array, in some logical order, if one can be defined.

Like this:

```ruby
class Date
  def deconstruct = [year, month, day]
end
```

Then this would be possible:
```ruby
if Date.today in [2022, month, *]
  puts "#{month} month of this year"
end
# => prints 12 month of this year
```

Note a few things here:

* Unlike hash patterns, array patterns only match the whole return value of `deconstruct`, so this would NOT match:
  ```ruby
  Date.today in 2022, month
  #=> false, because nothing is matching what's after the month
  ```
* Array patterns only make sense when there is an obvious logical ordering corresponding to the whole object (so in our sample implementation, there is "nowhere" to put weekday, it would be confusing).

When we implemented pattern matching for `Time` and `Date` for Ruby 3.2, we started with `Time`, and it was [Matz's decision](https://bugs.ruby-lang.org/issues/19071#note-2) that the order/set of all components of Time is not that obvious, so `#deconstruct` is NOT present in these classes of Ruby.

But it might make sense for your class!

## Closing words

You know how some authors mention that "this article was only possible due to" some company, or organization, or foundation?

Well, in my case, me sitting comfortably in my Kharkiv home and having some free time and electricity to write this (and also implement those changes in Ruby core) is only possible due to:

* Armed Forces of Ukraine;
* International support with weapons, donations, and publicity.

**Please [donate](https://war.ukraine.ua/donate/), and don't be silent!**

Слава Україні! 🇺🇦

<iframe src="https://zverok.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
