---
layout: post
title:  "Fun with Ruby method argument defaults"
date:   2020-11-19
categories: ruby weird
comments: true
---

Just yesterday I [suddenly understood](https://twitter.com/zverok/status/1329100996312698882) that there is a small neat trick, allowing to provide friendly error messages for missing method parameters:

```ruby
# default way to do things:
def read_data(file)
  # ...
end

read_data('1.txt') # => works
read_data
# wrong number of arguments (given 0, expected 1) (ArgumentError)
# ^^ not really friendly

def read_data(file:)
  # ...
end

read_data
# missing keyword: file (ArgumentError)
# ^^ at least some hint of what was expected

# But how about this?
def read_data(file = raise(ArgumentError, '#read_data requires path to .txt file with data in proper format'))
  # ...
end

read_data
# #read_data requires path to .txt file with data in proper format (ArgumentError)
# ^^ isn't it nice?..
```

Of course, it is not _that_ useful with a simple method with one argument, but for complicated APIs with several keyword args, it might be of some use. But what I was pleasantly surprised with is how simple it is—and that it works.

## How it works?

The `argument = raise(...)` is not some separate Ruby feature, but it is a natural consequence of two facts:

1. You can put any Ruby expression as the argument's default value, and it will be evaluated in the same context as the method's body, on each method call (when the argument is not provided)
2. `raise` is [just a method](https://docs.ruby-lang.org/en/2.7.0/Kernel.html#method-i-raise), not some special syntax, and like any other method call, it is an expression and can be put as an argument's default value.

## "Any expression"? Really?

Yep.

You can do even this (though you probably shouldn't!):

```ruby
def read_data(file = begin
                       puts "Using default argument"
                       if Time.now.hour < 12
                        'morning.txt'
                       else
                        'evening.txt'
                       end
                     end)
  puts "Reading #{file}"
end

read_data
# Prints:
#  Using default argument
#  Reading evening.txt
```

As was already said above, the context of evaluation is the same as for method body, and all default values are evaluated sequentially, so you can do this (and probably shouldn't!):

```ruby
class ArgsTracker
  attr_reader :args

  def initialize
    @args = []
  end

  def track(
    a: begin; args << :a; 100 end,
    b: begin; puts "a was #{a}"; args << :b end)
  end
end

tracker = ArgsTracker.new

tracker.track
# Prints: "a was 100", and adds [:a, :b] to tracker

tracker.track(a: 5)
# Prints: "a was 5", and adds only [:b] (which was not provided) to tracker

tracker.args # => [:a, :b, :b]
```

Cool. Ugly, but cool.

## How is this useful?

The fact that default values are calculated on each call, and in the context of called class, have some simple and useful consequences. Probably you already have seen and used some of them:

```ruby
def log(something, at: Time.now) # will be calculated at each call of log, when alternative at: is not provided
  #...
end

def setup_output(out: $stdout, err: $stderr, warn: out) # default output device for warn would be always
                                                        # the same as `out`
  # ...
end

class A
  def process(order: default_order) # will call the same object's method to calculate default
  end

  private

  def default_order
    # some complicated calculation, depending on the object's state
  end
end
```

## More advanced usage

Besides the example from which we have started, one might think about other _relatively sane_ but not very simple usages of the on-the-fly calculation, for example, tracking of default values usage (might be useful on legacy refactoring, when we aren't sure whether defaults are used at all, but can't allow ourselves to just break the codebase):

```ruby
def log_default(name, value)
  # or logger.debug
  puts "#{caller.first}: default value for #{name} was invoked from #{caller[2]}"
  value
end

# Now change this:
def some_method(factor: 100)
end
#...to this:
def some_method(factor: log_default(:factor, 100))
end

# ...and...
some_method
# Logs:
#   ...in `some_method': default value for factor was invoked from `some_other_method'
```

One might also imagine my initial example (with `fail`) extended for some _very friendly_ API to use like `fun(arg: friendly_fail(:arg))`, which fetches large explanatory string from constant/`i18n` config, enriches it with calling context (like, "if caller contains _this_, we are saying `this shouldn't be called from <framework>`") and raises Very Friendly Exception.

Not you _should_ do something like this anytime soon, but rather "it is interesting that you can, and probably someday you'd like to try".

Have fun!
