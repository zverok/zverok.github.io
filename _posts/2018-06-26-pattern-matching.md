---
layout: post
title:  "Quest for Ruby Pattern Matching"
date:   2018-06-26
categories: ruby
comments: true
---

It happened so that this blog was started almost 3 years ago with [post](https://zverok.github.io/blog/2015-07-18-matchish.html) about exercise in imitating pattern matching as a Ruby library. That was fun, but ended in a sad realization:

> **All in all, powerful pattern matching need to be core language feature**.

Three years forward, the discussion about how pattern matching in _Ruby language core_ may look was [raised](https://bugs.ruby-lang.org/issues/14709), ending with Matz's statement:

> _If_ we were going to add pattern matching in Ruby, we should add it with **better syntax**.

Now, this post is **an invitation to discuss** possible pattern matching (or its elements) in future Ruby versions, and what could be introduced with minimal and reasonable changes to the language.

> **Note**: It is not some kind of "official" invitation from Ruby maintainers, I represent only myself here. But as recent events have shown, "opportunity window" for new Ruby features is pretty open currently, and Ruby maintainers seem to listen carefully to community's proposals.

## Quick recap

[Wikipedia's article](https://en.wikipedia.org/wiki/Pattern_matching) on pattern matching is rather descriptive than definitive, but let's say that for the practical purposes that pattern matching can be defined as _checking data structures for correspondence to some patterns_ plus _destructuring_ (binding to local variables) of parts of matched structures; it can be used to define polymorphic methods, or can be not. <small>Yeah, I know it is totally ad-hoc and corresponds to no textbook, but clear enough (for me).</small>

Ruby _has_ some elements of those in different places, which we'll investigate a bit later.

There are several attempts to implement pattern matching as a library, some of them are referenced from my [previous post](https://zverok.github.io/blog/2015-07-18-matchish.html), and to that we now can add:

* [Qo](https://github.com/baweaver/qo) library by Brandon Weaver aka [@keystonelemur](https://twitter.com/keystonelemur), which includes some interesting experiments and theoretical discussion of the subject;
* Yuki Torii's [prototype](https://github.com/yakitorii/pattern-match-ruby) ([Ruby Kaigi presentation](http://rubykaigi.org/2017/presentations/yotii23.html)) of how _core language_ implementation may look.

## Options and ideas for implementation

There is three options to introduce/enchance pattern-matching capabilities in Ruby:

1. New syntax for method definition, polymorphic methods:
```ruby
# Something like this...
defm method(Integer, nil) { |i| ... }
defm method(0..20, Hash) { |i, h| ... }
# ...or this
defm method(Integer i, nil)
  # ...
end
defm method(0..20 i, Hash h)
  # ...
end
# ...or this: also "compiles" to several different method bodies, chosen in call-time by arguments
defm method
  match(Integer, nil) { |i| ... }
  match(0..20, Hash) { |i, h| ... }
end
```
2. Some new construct used inside method's code, like:
```ruby
def method(*args)
  match args do
    on Integer, nil ...
  end
end
```
3. Try to do "cosmetic" changes to existing Ruby constructs and conventions to achieve more powerful constructs evolutionary.

Of those three, I believe (2) is too boring (e.g. you can always glue totally new syntax into any language, and it very rarely makes languages better), and (1) is too radical (it requires redesign of the whole message passing/method calling idea of Ruby OO, which I don't expect to happen in near future anyway). Which leaves us with "evolutionary changes" option.

Now, if we want to introduce some new elements in language core, there are two important limitations to follow:

1. Unlike library solution, we **can't introduce a lot of DSL**/"sublanguage", existing constructs should be reused as much as possible;
2. New constructs (if any) or new usage of known constructs should be **gracefully incompatible** with previous versions of Ruby. This means they should be just incorrect code in the previous version of Ruby, not correct code with a different meaning.

Then, let's look what we already have in different areas of the language specification, that resembles the two main goals of pattern matching: deep and rich checking against complex values, and destructuring of those values.

```ruby
# Obvious example: #case or #grep and #=== operator
case value
when 1..10
  # ...
when Regexp
  # ...
when ->(x) { (x % 10).zero? }
  # ...
end

[1, 'test', 1/3r].grep(Numeric) # => [1, (1/3)]

# Other case: Destructuring when calling proc:
{a: 'foo', b: 'bar'}.map.with_index { |(k, v), i| [k, v, i] }
#                                      ^^^^^^^^^
# => [[:a, "foo", 0], [:b, "bar", 1]]

# Works with methods, too:
def test((k, v), i)
  [k, v, i]
end
{a: 'foo', b: 'bar'}.map.with_index(&method(:test))
# => [[:a, "foo", 0], [:b, "bar", 1]]

# In fact, this "destructuring" (of arguments structure) could be pretty fancy:
def test((a, b), c, *d, e:, **f)
  puts "a=#{a}, b=#{b}, c=#{c}, d=#{d}, e=#{e}, f=#{f}"
end

data = [[1, 2], 3, 4, 5, {e: 6, g: 7, h: 8}]
test(*data)
# a=1, b=2, c=3, d=[4, 5], e=6, f={:g=>7, :h=>8}

# Finally, there is one case in Ruby where testing AND local variable assignment happens simultaneously:
rescue ArgumentError => e
#      ^^^^^^^^^^^^^    ^^
#      type to check    local variable to put matching value
```

With this in mind, let's see how far can we stretch our habits and intuitions without breaking them!

> **Note**: In examples below, I assume the most useful case for pattern matching is dynamic dispatch by some kind of input: either from outside world (parsers, flexible form handlers and so on) or just polymorphic method of some service class, like a custom collection.

> **One more note**: There are many ways to refactor code samples below. But the scope of the current article is just an attempt to think on a _gentle_ ways to make Ruby's existing pattern matching more featureful.

Let's start with something simple. Sometimes we can see this pattern:

```ruby
def process_coord(lat, lng = nil)
  case lat
  when String
    Geo::Coord.parse(lat)
  when Geo::Coord
    lat.dup
  when Numeric
    case lng
    when Numeric
      Geo::Coord.new(lat, lng)
    when nil
      Geo::Coord.new(lat, lat)
    when Array
      if lng.last.is_a?(Hash) # yeah, with keyword args options hashes are almost never necessary, but...
        options = lng.pop
        ....
      end
    else
      fail ArgumentError
    end
  else
    fail ArgumentError
  end
```

...e.g., "`case` by the first argument, inside that `case` by next argument (and probably some more `case`s)". I always wondered, couldn't we do it this way?

```ruby
case (lat, *lng)
when (String) # Only one argument is passed
  Geo::Coord.parse(lat)
when (Numeric, Numeric)
  Geo::Coord.new(lat, lng)
when (Numeric, Numeric, Hash)
  # options hash was passed...
```

This seems to be:

* pretty obvious;
* incorrect syntax currently (see "gentle incompatibility" requirement above);
* no conflict with existing syntax ("or"-ing several conditions in one `when` is done outside any parentheses, matching of a list is inside them)
* resembling "deconstruction" on method/proc arguments definition.

Once we started, it is pretty easy to "stretch" the approach to most of the things that "method param deconstruction" is willing to accept:

```ruby
when ((Numeric, Numeric), Hash) # nested sequences
  # call-sequence may have been `parse_coordinates([57.0, 32.0], strict: true)`
when (:skip, _, _, Numeric)     # as in method args, _ has a special meaning of "ignore this"/match anything
when (*Numeric)                 # array of any size, but all numerics
when (*/\d+(\.\d+)?/)           # array of number-alike strings, of any size
when (Numeric, Numeric, radius: Numeric, **) # are we too far yet?..
```

Our tastes and feelings of "what is good for the language" could diverge seriously at this point, but _for my eye_ this code still looks like something that could easily exist in Ruby for ages (just note that `*` here probably should be rather special syntax, like in method definition, not call of `#to_a`, like everywhere else: having `#to_a` defined in all objects of the world returning something "pattern-y" would be definitely bad).

Finally, let's look closer here:

```ruby
case (lat, *lng)
when (Numeric, Numeric, Hash)
  # options hash was passed...
```

...what should be written next, how we split `lng` into longitude itself and options hash? With `.pop` again? How about that?

```ruby
case (lat, *lng)
when (Numeric, Numeric => lng, Hash => opts)
  # options hash was passed...
```

Looks familiar, right? That's the same syntax we use in `rescue`, to _check type AND bind variable_ at the same time. I am not sure about the consequences (all in all, sometimes we probably would want a regular hash in this context...), but the idea still seems surprisingly appealing, honestly.

### Play and discuss

[Here](https://github.com/zverok/pattern-matching-prototype) is a small quazi-gem (yet installable as a `pattern-matching-prototype` gem) which imitates the proposed changes as close as possible, trading it for some pretty nasty tricks:

* You should use `M()` instead of just `()` to create a pattern for several items;
* Binding of local variables done with the help of `binding_of_caller` gem, and requires variables to be _already defined_;
* Against what've been said above about `#to_a`, it was used to imitate `*Numeric` feature;
* Global `_` method is not probably what you want to repeat at home.

Remember it is just a _way to try_ new syntax ideas.

Discussions (hopefully) will be here:

* [Twitter](https://twitter.com/zverok/status/1011704387096465410)
* [Reddit](https://reddit.com/r/ruby)
* (just here in comments, I read them too, but maybe less visible to the community)

Let me know what do you think of it!