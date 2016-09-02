---
layout: post
title:  "Tricks with RSpec components outside RSpec"
date:   2016-09-02 18:44
categories: ruby cheatsheets
comments: true
---

## What?

In this short post I'd like to show how some of RSpec components (matchers
and expectations) can be used for a greater good outside your tests. Like
in your normal everyday scripts.

## Why?

Trying to be expressive, concise and readable, RSpec have developed a great
number of idioms, mechanisms and approaches. They are reliable, most of
Rubyists familiar with them, and code becames pretty expressive.

While I don't recommend <s>trying this at home</s> using the tricks
shown below in your Big Enterprise Production core, you may like those
things for short scripts, experiments, insights, and just for fun.

## How?

### Argument matchers for checking values

```ruby
require 'rspec'

include RSpec::Mocks::ArgumentMatchers

hash_including(:foo) === {foo: 1}
# => true
hash_including(:foo) === {bar: 1}
# => false
```

Those [argument matchers](https://relishapp.com/rspec/rspec-mocks/v/3-5/docs/setting-constraints/matching-arguments)
just define case equality operator, making themselves really good for
(kind of) pattern matching. Basically, for `case` (it is "case equality
operator", all in all!) and `grep`:

```ruby
require 'rspec'

include RSpec::Mocks::ArgumentMatchers

case arg
when hash_including(:foo)
# ...
when hash_excluding(:bar)
# ...
when array_including(1, 2, 3)
# ...
when duck_type(:to_xyz) # just an object having this particular method
# ...
#
# standard RSpec matchers also work,
# you just need to include RSpec::Matchers too,
# which can have consequences - see below.
when be_within(0.001).of(28.3)
# ...
when have_attributes(class: String, length: 5)
# remember standard RSpec matchers are composable:
when have_attributes(class: String, length: 4).or(start_with('f'))
# ...and so on...
end

# another usage:
[1, %w[a b c], :foo].grep(duck_type(:each))
# => [["a", "b", "c"]]
```

### Partial stubs to alter behavior quickly

```ruby
require 'net/http'
require 'rspec'

include RSpec::Mocks::ExampleMethods
RSpec::Mocks.setup # <= will not work without this line :(

allow(Net::HTTP).to receive(:get).and_return('foo')

Net::HTTP.get('https://google.com')
# => "foo"

allow_any_instance_of(String).to receive(:size).and_return(8)
'foo'.size
# => 8

RSpec::Mocks.space.reset_all
'foo'.size
# => 3
```

### Probe for unknown code

Not _that_ useful, but you can easily drop it into unknown code (especially
if it wants something complex or sensible) and see what will be done with
it:

```ruby
def test(database)
  database.drop_everything_forever
end

d = double.as_null_object

test(d)

# Not that pretty... But works
p ::RSpec::Mocks.space.proxy_for(d)
  .instance_variable_get('@messages_received')
# => [[:drop_everything_forever, [], nil]]
#      method                   args block
```

### `expect` as Ye Olde Good Assert

**CAUTION**: HIGHLY QUESTIONABLE!

```ruby
include RSpec::Matchers

def my_method_vith_validations(str)
  expect(str).to have_attributes(size: 4)

  puts str
end

my_method_with_validations('test') # => ok
my_method_with_validations('foo')
# RSpec::Expectations::ExpectationNotMetError: expected "foo" to have attributes {:size => 4} but had attributes {:size => 3}
# Diff:
# @@ -1,2 +1,2 @@
# -:size => 4,
# +:size => 3,
```

The error is pretty informative. So, you can use it like "oldschool"
assert (for checking pre-, post- and intermediate conditions), just don't
forget to read "Don't do a mess" section!

### Other unholy tricks

```ruby
require 'rspec'
include RSpec::Mocks::ExampleMethods
RSpec::Mocks.setup

stub_const('WHATEVER', 8)
WHATEVER
# => 8
stub_const('WHATEVER', 9)
WHATEVER
# => 9
RSpec::Mocks.space.reset_all
WHATEVER
# NameError: uninitialized constant WHATEVER
```

### Don't do a mess

If you really happy about the tricks above and want to move them into
_real_ code, I'd really suggest to evaluate _how strong_ is your willing
to pollute your namespaces with RSpec stuff. For example, "innocent" line

```ruby
include RSpec::Matchers
```

...will "enrich" your current context with ton of methods like `eq`, `be`
`all`, and `method_missing` that would treat any `be_wtf` as conversion
to matcher for checking whether it's argument responds to `wtf?` method
and returns truthy value.

Maybe more sane approach would including/extending necessary RSpec features
into isolated module, like this:

```ruby
module M
  extend RSpec::Matchers
  extend RSpec::Mocks::ArgumentMatchers
end

some_huge_json.grep(M.hash_including(:foo))
```

...or even stricter tricks with delegating only really necessary things:

```ruby
module M
  module Internals
    extend RSpec::Matchers
    extend RSpec::Mocks::ArgumentMatchers
  end

  class << self
    extend Forwardable

    def internals
      Internals
    end

    # OK, it is officially cumbersome now... But still useful!

    def_delegator :internals, :hash_including
  end
end

M.hash_including(:foo)
# => #<RSpec::Mocks::ArgumentMatchers::HashIncludingMatcher:0x9a2b440 @expected={:foo=>#<RSpec::Mocks::ArgumentMatchers::AnyArgMatcher:0x9995a58>}>
M.array_including(1)
# undefined method `array_including' for M:Module (NoMethodError)
```

But, all in all, for experimenting and prototyping, most of shown techinques
are rather helpful than destructive. Just be cautious moving your experiments
to production.

And be curious.

And be happy.
