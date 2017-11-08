---
layout: post
title:  Please stop calling it "magic"
date:   2017-10-22 18:30:00
categories: ruby rants
comments: true
---

> **[Japanese translation](https://techracho.bpsinc.jp/hachi8833/2017_11_08/47766) of this post is available!**

Recently I've read (informative and well-written) article called
[_7 Gems Which Will Make Your Rails Code Look Awesome_](https://blog.rubyroidlabs.com/2017/10/7-gems-rails-code/),
and my eye was caught with this phrase in the description of [decent_exposure](https://github.com/hashrocket/decent_exposure)
gem:

> If you are not a big fan of using **magic** – this library is not for you. (emphasis is mine)

This "m-word" was used to describe library that uses _metaprogramming_: adds `expose` method to
Rails controller class, that generates trivial `#index`/`#show`/`#new` methods for simple CRUD
situations.

I honestly don't want to offend the article's author, it is just a cause to say some important things.
Lately, it seems to be pretty spread in Ruby/Rails communities: to use "magic" and "metaprogramming"
as a kind of wicked synonyms. So, I just want to ask politely yet firmly:

**Please stop calling it "magic"**

Why? Read below.

> **Disclaimer**: for the sake of discussion, I am using made-up examples, not particularly based on
  any existing library. I just don't want my points to be lost in "Oh, he is just blaming %libraryname%",
  like it already [happened in the past](http://zverok.github.io/blog/2016-10-09-minitest.html).

So...

## What is magic?

According to Wikipedia,

> **Magic** ... is the use of rituals, symbols, actions, gestures, and language with the aim of
  utilizing **supernatural** forces.[→](https://en.wikipedia.org/wiki/Magic_%28paranormal%29)

and

> The **supernatural** ... includes all that cannot be explained by the laws of nature.[→](https://en.wikipedia.org/wiki/Supernatural)

Applying this to programming, we can formulate that:

> **Magic** in code is the situation when some behavior emerges, whose cause, even for experienced
  developer on target language is hard/impossible to trace.

This is not a perfect definition, so let's clarify it with some semi-theoretical examples.

```ruby
class LandCruiserJ200 < Car
  def type
    :suv
  end

  def engine
    '4.5 L'
  end
end

LandCruiserJ200.new.drive! # "wroom! wrrroooom!"
```

"Is it magic? I don't see any `#drive!` method here, but it _magically_ works???"

Any sane developer would laugh.

No, silly, it is just inheritance. We can guess that `#drive!` method
defined in parent `Car` class, and can easily find this class' and method's code (and/or its docs,
including the case when it is from some external gem).

OK, one more.

```ruby
class LandCruiserJ200
  include Drivable

  def type
    :suv
  end

  def engine
    '4.5 L'
  end
end

LandCruiserJ200.new.drive! # "wroom! wrrroooom!"
```

Oh, how weird! No inheritance here, where did `#drive!` method came from? Magic!

... Or just normal mixin, find `Drivable`'s docs and it probably will explain how and why it is used,
and what `#drive!` method does.

One more?

```ruby
class LandCruiserJ200
  drivable type: :suv, engine: '4.5 L'
end

LandCruiserJ200.new.drive! # "wroom! wrrroooom!"
```

Oh, that's definitely a case of unexplainable forbidden witchcraft, mere human will never understand
that, right?..

... Or probably `drivable` is a method, defined for a `Class` directly or by extending with some
module (which is not the best practice possible, yet still pretty easy to guess for any non-junior
Rubyist), and you probably can:

```ruby
LandCruiserJ200.method(:drivable) # => #<Method: Class(Drivable)#drivable>
LandCruiserJ200.method(:drivable).source_location # => /some/gem/drivable/lib/drivable.rb:123
```

### So, what is magic?

I'll show you:

```ruby
class LandCruiserJ200Car
end

car = LandCruiserJ200Car.new
# => #<Car model="Land Cruiser J200">
car.drive! # "wroom! wrrroooom!"
```

Imagine the code above works. Why?

If I need to deal with the fact, OK, I can _guess_ that probably
in current scope _something_ provides a convention by which... hm... Everything named `...Car` is
treated specially? Or everything from folder `app/cars/`?.. Or, there is some YAML-config, where all
car classes are listed?

What exactly is responsible for attaching the behavior and where can I find
its docs, if I need them? How do I debug it, if it fails to behave the way I expect?..

That's an example of "supernatural" behavior: e.g. when something happens not being obviously related
to _natural_ forces of the language. ...And is bad _exactly_ because of this fact: your _natural_
tools and intuitions stop working, you need just to remember entire spellbook to deal with magical code.

## What's not magic

### What junior does not understand, is not magic

It is common to see nowadays statements like "this is too magical for less experienced developers to
understand". As Matz says (reciting from memory), "Ruby is complex language to make developer's life
simple", not the other way round.

Your car's engine and your computer's processor are pretty complicated, too, but that doesn't mean
that they are "magical". Neither that you shouldn't use them and stick to "less magical" solutions.

### Monkey-patching is not magic

Yep, monkey-patching is questionable. Monkey-patching of core classes is considered deadly sin by some.
And so on.

But, when some people coming from another language, see `5.days` and call that "magic", why
should we repeat their opinion?

It may be counter-intuitive for other language developers, but for
Rubyists it is pretty obvious what's going on, and how to understand where method came from and look
for its docs/implementations.

> And, as a side note, open classes (even core ones) is significant feature for Ruby evolution,
  allowing both experiment with new concepts and provide backward compatibility by demand, with
  gems like [backports](https://github.com/marcandre/backports) and
  [polyfill](https://github.com/AaronLasseigne/polyfill).

### Metaprogramming is not magic

And now, back to the beginning of this article :)

"Metaprogramming", e.g. generation of code objects during code execution, is one of the most powerful
Ruby features, and it is so because how natural it is for the language. And due to this naturality,
it is one of the first and the best guesses for code design in most cases.

Your class needs to have several math operators, just delegating to internal values? You do

```ruby
%i[+ - * /].each { |op| define_method(op) { ...
```

Several similar classes need some "settings" that declaratively define their behavior? You add simple
DSL of class methods. And so on.

Yes, later you may redecide this due to performance reasons, or for the sake of documentability (though
nowadays we have pretty decent tools for documenting metaprogrammed stuff), or because your homogenous
code objects became not that homogenous...

But metaprogramming is just one of the tools in your drawer and pretty useful one.

It **does not become unnatural** just because somebody has called it so 100 times.

## Conclusion

Why am I writing this?

Because "m-word" is _harmful_ for the community. It becomes a bad label that used
with too much freedom both from outside and inside the Ruby world, being attached randomly to
practices and features one just doesn't like or doesn't understand. This situation leads to decay
of ideas, when perfectly well-designed and natural libraries and language features become almost
"prohibited".

So, simply put:

**Dear Rubyists! Please stop use "magic" as a synonym of "metaprogramming"!**
