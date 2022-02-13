---
layout: post
title:  "Fun with each_with_object and other Enumerator adventures"
date:   2019-10-18
categories: ruby
comments: true
---

A short preface: I am writing in Ruby since 2004. For last 5 years I am also participating in language development, documenting new and old features, proposing some (several got accepted) and diving deep into weirdest mysteries of the language. So, I was pretty confident I've seen every technique for clean and DRY code with functional flavour. That was until today, my friend and fellow Rubyist [Vladimir Ermilov](https://github.com/adworse) discovered this:

<blockquote class="twitter-tweet"><p lang="en" dir="ltr">I just discovered you can do this in Ruby <a href="https://t.co/iK029ozz7Y">pic.twitter.com/iK029ozz7Y</a></p>&mdash; Lionka Panteleev (@adworse) <a href="https://twitter.com/adworse/status/1185090585650221057?ref_src=twsrc%5Etfw">October 18, 2019</a></blockquote> <script async src="https://platform.twitter.com/widgets.js" charset="utf-8"></script>

Yup, this is a short, DRY, no-block-arg-names-repetition way to do this:

```ruby
[1, 2, 3].zip([4, 5, 6]).map { |ary| ary.join(' ') }
```
....e.g. to _pass additional argument_ to `Symbol#to_proc`. Works with any enumerable and any proc (including those produced by `Symbol#to_proc` and method references):

```ruby
content = <<~DATA
name|John
age|30
city|Nashville
DATA

content.each_line(chomp: true).each_with_object('|').map(&:split).to_h
# => {"name"=>"John", "age"=>"30", "city"=>"Nashville"}

jsons = ['{"name":"John"}', '{"name":"Amy"}', '{"name":"Rajesh"}']
# Using Ruby 2.7's method references:
jsons.map(&JSON.:parse)
# => [{"name"=>"John"}, {"name"=>"Amy"}, {"name"=>"Rajesh"}]
jsons.each_with_object(symbolize_names: true).map(&JSON.:parse)
# => [{:name=>"John"}, {:name=>"Amy"}, {:name=>"Rajesh"}]
```

## Why is it cool?

I am unusually excited about this: there is a long-standing quest for combining implicit-argument blocks (`&:sym` and `obj.:method`) with additional details, e.g. to say something like:
```ruby
numbers.map { |num| num % 10 }
```
...without repetition of `|num|`/`num`. This quest became somewhat of a holy grail for searching for new syntaxes, and led to proposing (and rejecting) things like
```ruby
lines.map(&:split.('|')) # It could work by redefining Symbol#call, and using .call() → .() shortcut
```
...and eventually to Ruby 2.7's "numbered block args":
```ruby
numbers.map { _1 % 10 }
```
...which, for me, always felt like an "orphan" feature, not leading to any updating of our understanding of language's semantics and possibilities, just "being short" (some may say that's the plus, though).

But it turns out you always could
```ruby
numbers.each_with_object(10).map(&:%)
```
...which could be neglected as being "longer" by character-count, yet somehow struck me as conceptually right.

## How it works, and when it does not

Being curious, you may compare those two pieces of code:
```ruby
[1, 2, 3].each_with_object(4).map(&:+)  # => [5, 6, 7]
[1, 2, 3].zip([4, 4, 4]).map(&:+)       # ArgumentError (wrong number of arguments (given 0, expected 1))
```

What's the difference, why one works and another not? Checked naively, they seem to produce exact same sequences:
```ruby
[1, 2, 3].each_with_object(4).to_a # => [[1, 4], [2, 4], [3, 4]]
[1, 2, 3].zip([4, 4, 4]).to_a # => [[1, 4], [2, 4], [3, 4]]
```

But the thing is, after `each_with_object` you have an `Enumerator`, which yields to the block _two separate arguments_, while result of `zip` yields _two-element arrays as one argument_. This is invisible in "non-lambda" procs (which unpack single array argument into multiple), but visible in lambda:
```ruby
[1, 2, 3].each_with_object(4).map(&->(x, y) { }) # works
[1, 2, 3].zip([4, 4, 4]).map(&->(x, y) { }) # ArgumentError (wrong number of arguments (given 1, expected 2))
```
...and result of `Symbol#to_proc` behaves like a lambda (no argument conversion):
```ruby
:+.to_proc.call(1, 2) # => 3
:+.to_proc.call([1, 2]) # ArgumentError

# Though, it doesn't aknowledge it is:
:+.to_proc.lambda? # => false
```

The only other methods that produce yielding-multiple-args `Enumerator` are, obviously `Enumerable#each_with_index` and `Enumerable#reduce`.

But...

## Fixing Ruby

But we can do this:

```ruby
module Enumerable
  def each_tuple
    return to_enum(__method__) unless block_given?
    each { |item| yield(*item) } # unpacking possible array into several args
  end
end
```

...and, suddenly, everything illuminated:

```ruby
[1, 2, 3].zip([4, 4, 4]).each_tuple.map(&:+) # => [5, 6, 7]
list_of_files.zip(list_of_contents).each_tuple(&File.:write)
# calls File.write(filename, content) for each pair
```
