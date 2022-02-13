---
layout: post
title:  "Small nice feature that emerged in Ruby 3.1... But has a nasty quirk"
date:   2021-12-08
categories: ruby
comments: true
---

A few days ago, I stumbled upon [an article about small JS debugging techniques](https://christianheilmann.com/2021/11/01/developer-tools-secrets-that-shouldnt-be-secrets/)—right when I was in the middle of debugging my JS hobby project. There is a lot of cool stuff there, but one that struck me as "brilliant, why didn't I think about it myself?.." was this:
```js
x = 42
console.log(x)
// "classic" way I used, logs 42
console.log({x})
// the way article suggested, logs {x: 42} --- variable name AND value
```

This is invaluable when debugging small algorithms with a lot of intermediate values: just a few more characters, and instead of a bunch of values thrown into the console, you see a clear picture of "this variable was this, that variable was that".

It is possible due to "shortcut" hash syntax, `{x}` is the same as `{x: x}`.

Then, I remembered that in my primary and beloved language's new version (3.1, coming December), hash omission syntax was recently introduced:

```ruby
p RUBY_VERSION
# => "3.1.0"
x = 42
p x:
# => {:x => 42}
```

Tada! Our life will be so much better now! I immediately posted a happy [tweet](https://twitter.com/zverok/status/1468240457826193416), and happy [announce to `/r/ruby`](https://www.reddit.com/r/ruby/comments/rb4rez/with_ruby_31_this_is_now_posible/), feeling a bit like Discoverer Of Things.

But, in a few hours, I saw [a new ticket](https://bugs.ruby-lang.org/issues/18396) in Ruby Bug Tracker (probably caused by my tweet? Or maybe I am delusional) and understood my joy was premature.

Let's try a new and shiny "debugging feature" in a more realistic program:

```ruby
def hypotenuse(a, b)
  p a:, b:
  Math.sqrt(a**2 + b**2)
end

p hypotenuse(10, 15)
```
This is expected to print `{:a => 10, :b => 15}`, and then hypotenuse value `18.027...`.

But, to my _mild_ horror (I already knew what would go wrong from the above ticket), the actual result was:
```
{:a=>10, :b=>18.027756377319946}
{:a=>10, :b=>18.027756377319946}
```
(yes, twice).

What's happening?..

Backward compatibility is happening!

In Ruby 3.0, this (with only one key) would be a valid code:
```ruby
p b:
Math.sqrt(a**2 + b**2)
# It is actually parsed as:
p(b: Math.sqrt(a**2 + b**2))
```
Therefore line ending with `foo:` treats the next line as the value for that key. Therefore, our method is actually saying
```ruby
def hypotenuse(a, b)
  p(a:, b: Math.sqrt(a**2 + b**2)) # calculate hash, then `p` it and return the result of `p` (hash itself)
end
```

Damn.

Thus, any parenthesis-less method ending in omitted keyword argument will actually treat the next line as the value—if not, too much code would be broken by a sudden "value omission feature" being turned on in the middle of the long expression broken in lines for formatting reasons.

This would work as expected (though mandating parenthesis is less appealing for debug):

```ruby
def hypotenuse(a, b)
  p(a:, b:)              # prints {:a=>10, :b=>15}
  Math.sqrt(a**2 + b**2)
end

p hypotenuse(10, 15) # prints 18.027756377319946
```

So, the sad outtakes!

* **You shouldn't actually do `p variable:` for debugging** (but still can `p(variable:)`, which is just a bit more typing... but you should always remember!)
* **You should be very careful with parenthesis omission now AND keyword argument values omission** in the same method call.

Stay safe.
