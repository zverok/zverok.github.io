---
layout: post
title:  "Implicit vs explicit type conversions in Ruby (to_h/to_hash and others)"
date:   2016-01-18 18:25:00
categories: ruby cheatsheets
comments: true
---

Just a quick reference post.

There are several pairs of type coercions methods in Ruby:

* `to_i` and `to_int`;
* `to_s` and `to_str`;
* `to_a` and `to_ary`;
* `to_h` and `to_hash`.

**What's the difference** and when to use/implement each of pair?

Shorter version is **explicit** conversion:

* your type should implement if it can be somehow converted into target
  type (integer, string, array and so on);
* you should call those method on incoming data, if you expect it to
  have _some_ means of converting to target type.

Longer version is **implicit** conversion:

* your type should implement it only if it sees itself as "kind of" target
  type ("better" or "specialized" number, string, array, hash);
* you should call those methods on incoming data, if you have _strict_
  requirement for it being of target type, but want to check it in Ruby
  way, with duck typing, not class comparison;
* you **should not** implement and use this methods in other cases!

Most of Ruby operators and core methods use _implicit_ conversions on
data:

* `"string" + other` will call `#to_str` on `other`, `[1,2,3] + other` will
  call `#to_ary` and so on (and will raise `TypeError: no implicit conversion`
  if object not responds to this methods);
* `"string" * other` and `[1,2,3] * other` will call `#to_int` on `other`;
* `a, b, c = *other` will call `other#to_ary` (and, therefore, can be
  used to "unpack" custom collection objects—but also can cause
  unintended behavior, if `other` implements `to_ary` but was not thought
  as a collection);
* `some_method(*other)`—same as above, uses `other#to_ary`;
* `some_method(**other)`—new in Ruby 2, uses `other#to_hash`.

I should repeat: **never** implement implicit conversion methods unless
you sure know what you are doing! It is widely seen, for example, the
`#to_hash` method being implemented (maybe because of "prettier name"
than `#to_h`?) and causing strangest effects.

Practical example:

{% highlight ruby %}
class Dog
  attr_reader :name, :age
  def initialize(name, age)
    @name, @age = name, age
  end

  # erroneously defined for serialization instead of to_h
  def to_hash
    {species: 'Dog', name: name, age: age}
  end
end

def set_options(**some_options)
  p some_options
end

dog = Dog.new('Rex', 5)

set_options(foo: 'bar')
# ok, prints {foo: 'bar'}

set_options(1)
# can't be called this way, raises helpful ArgumentError

set_options(dog)
# ooops, treats our dog as an options hash
# and prints {species: 'Dog', name: name, age: age}


require 'awesome_print' # pretty popular pretty printing gem
ap dog
# ooops, uses Hash pretty-printing method
# instead of Dog#inspect
{% endhighlight %}

See also:

* [to_i Vs to_int](http://www.rubyfleebie.com/to_i-vs-to_int/);
* [Ruby: to_s vs. to_str](http://briancarper.net/blog/98.html);
* [StackOverflow discussion](http://stackoverflow.com/questions/11182052/to-s-vs-to-str-and-to-i-to-a-to-h-vs-to-int-to-ary-to-hash-in-ruby).
