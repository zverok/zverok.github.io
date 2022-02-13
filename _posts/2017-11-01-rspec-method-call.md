---
layout: post
title:  Useful RSpec trick for testing method with arguments
date:   2017-11-01 16:45:00
categories: ruby rants tricks
comments: true
---

**What's inside:** A useful rspec/rspec-its trick for testing methods with arguments + philosophical explanations why I consider such tricks a good thing.

## The task

Currently we are working hard on [daru](https://github.com/SciRuby/daru)'s next version, and part of this work is refactoring specs. Most of them are pretty old and written by Google Summer of Code students, which sometimes lead to not ideal coverage, and almost always—to very "wordy" specs.

As daru is algorithmically complex (trying to provide "natural" interface for Rubyists to simple-to-explain-yet-powerful concepts like dataframes and series), those specs refactoring provides a lot of funny challenges, and I'd like to share one of them.

So, we have `Index` class, which is, basically, an ordered set of unique labels, used as an axis labels for `Vector` (1-D array with labels) and `DataFrame` (2-D array, think of it as of spreadsheet).

One of the most important Index's methods is `pos` method, which basically works like this:

```ruby
index = Daru::Index.new %w[Kharkiv Kyiv Odesa Dnipro Lviv]
# => #<Daru::Index(5): {Kharkiv, Kyiv, Odesa, Dnipro, Lviv}>

# by label
index.pos('Kharkiv') # => 0
index.pos('Kharkiv'..'Odesa') # => [0, 1, 2]
index.pos('Kharkiv', 'Dnipro', 'Lviv') # => [0, 3, 4]

# ...or by number
index.pos(0) # => 0
index.pos(1..-1) # => [1, 2, 3, 4]
```

This method is used by methods like `Vector#[]`, allowing some vector with population-by-city values to be queried in a Hash-like and Array-like manner:

```ruby
populations = Daru::Vector.new([1439036, 2900920, 1016515, 979046, 727968], index: index)
# => #<Daru::Vector(5)>
# Kharkiv 1439036
#    Kyiv 2900920
#   Odesa 1016515
#  Dnipro  979046
#    Lviv  727968

populations['Kharkiv'] # calls Index#pos internally
# => 1439036
populations['Kharkiv'..'Odesa']
# => #<Daru::Vector(3)>
# Kharkiv 1439036
#    Kyiv 2900920
#   Odesa 1016515
populations[1]
# => 2900920
```

The behavior is both "natural" for Ruby (`Array#[]` behaves the same way), but requires several tests for different cases (one label, list of labels, a range of labels, one position, ..... and so on).

How'd you write tests for this?

## The trick

My preconditions:
* I am a real fan of one-assertion-per-test approach;
* I believe in (most of) [BetterSpecs](http://www.betterspecs.org/) rules;
* I like to have my tests as concise as possible (it is the only way you will not get boring while writing them);
* I use [rspec-its](https://github.com/rspec/rspec-its) (and also some [custom advanced things](https://github.com/zverok/saharspec) inspired by it).

So, here is the code:

```ruby
RSpec.describe Daru::Index do
  let(:index) { described_class.new [:a, :b, :c, :d] }

  describe '#pos' do
    subject { index.method(:pos) }

    context 'by label' do
      its([:a]) { is_expected.to eq 0 }
      its([:a, :c]) { is_expected.to eq [0, 2] }
      its([:b..:d]) { is_expected.to eq [1, 2, 3] }
  # .. and so on
```

`rspec --format doc` output:

```
Daru::Index
  #pos
    by label
      [:a]
        should eq 0
      [:a, :c]
        should eq [0, 2]
      [:b..:d]
        should eq [1, 2, 3]
```

_Surprisingly_ readable (for me!): method `#pos` (call result), with `[:a]` should be equal to `0`, with `[:a, :c]` to `[0, 2]` and so on.

How is it works?.. Pretty logically, in fact:
* `object.method(:name)` returns callable lambda, corresponding to `object.send(:name, *args)`
* `some_lambda[*args]` is a synonym for `some_lambda.call(*args)`;
* rspec-its defines the `its` method so, that if it is passed with array, it calls subject's `[]` method with this array.

That's it!

## Why is it a good thing?

Let's look at some of the alternative approaches.

### Impatient approach

```ruby
describe '#pos' do
  it 'works' do
    expect(index.pos(:a)).to eq 0
    expect(index.pos(:a, :c)).to eq [0, 2]
    expect(index.pos(:b..:d)).to eq [1, 2, 3]
    #...
```

* **Pro:** Short! Simple! No tricks!
* **Contra:** "One-assertion-per-test" is not somebody's evil invention to make your life harder. It is a way of looking on your test and structuring them. Once you make a step from this road (especially in a team of various backgrounds), you are lost: you'll see all kind of `if`s, and enumerations inside tests. You will never know if some test works because of the new/refactored method works, or because of side-effects of 20 previous lines in the same test. You can never run only one test case in isolation and debug it.

### Proper formal RSpec approach

```ruby
describe '#pos' do
  subject { vector.pos(*arguments) }

  context 'when one argument' do
    let(:arguments) { :a }

    it { is_expected.to eq 0 }
  end

  context 'when list of arguments' do
    let(:arguments) { [:a, :c] }

    it { is_expected.to eq [0, 2] }
  end

  #...
```

Or even **The most and the only right RSpec approach**

```ruby
describe '#pos' do
  subject { vector.pos(*arguments) }

  context 'when one argument' do
    let(:arguments) { :a }

    it 'should return singular value' do
      is_expected.to eq 0
    end
  end
  #...
```

> NB: I am not making it up on the fly. That's why they [have dropped `its` from the core](https://gist.github.com/myronmarston/4503509).

* **Pro:** It is proper. And formal. And documented as hell.
* **Contra:** Only a small one: nobody will write tests unless you'll bite them really hard. They will brag about "too much boilerplate", "I need to write 10 lines of tests per each 1 line of code, I don't have time", "my code is too trivial to write all this stuff". Then, they'll either sneak into "impatient" approach shown above or just stop to write (enough) tests.

### The solution, again:

```ruby
RSpec.describe Daru::Index do
  let(:index) { described_class.new [:a, :b, :c, :d] }

  describe '#pos' do
    subject { index.method(:pos) }

    context 'by label' do
      # we test method #pos call results with different arguments
      its([:a]) { is_expected.to eq 0 }
      its([:a, :c]) { is_expected.to eq [0, 2] }
      its([:b..:d]) { is_expected.to eq [1, 2, 3] }
```

It is:
* one assertion per test;
* tests are almost as short as just describing what it should do;
* pretty readable (the only thing I dislike is "unnecessary" `[]` inside `its()`);
* reasonably low learning curve (new programmer working with this code probably will be surprised at first several minutes, but it is pretty easy to deduce what's going on, and pretty hard to forget it, once grasped).

### Possible enchancements

For a large class with a lot of methods (like our `Daru::Index`), it probably can make sense to add a feature to RSpec config to produce subject automatically:

```ruby
subject { Daru::Index.new [...] }

describe '#pos', :method do # subject is deduced from parent context's subject and description string
  its([:a]) { is_expected.to eq 0 }
```

Also, I am considering this extension to [saharspec](https://github.com/zverok/saharspec)'s `its_call` method:

```ruby
its_call(:a) { is_expected.to ret 0 } # short for `return` which can't be used being a keyword
its_call(:a, :c) { is_expected.to ret [0, 1] }
its_call(:d) { is_expected.to raise_error }
```
...which probably shows intentions better (and can support expecting-to-raise on incorrect arguments).

## Conclusions

The good testing framework allows you to write _concise_, _robust_ and _elegant_ tests. That's the only way to be sure good programmers would write them.

And any small helper that allows doing so is good for you.
