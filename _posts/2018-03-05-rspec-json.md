---
layout: post
title:  'Embracing composability: be_json RSpec matcher'
date:   2018-03-02
categories: ruby rspec
comments: true
---

There is one small yet tricky thing when testing some HTTP APIs with RSpec: how to properly and idiomatically test response's JSON? That's where you typically start:

```ruby
let(:response) { call_my_api }
subject { JSON.parse(response.body) }

it { is_expected.to ... }
```

But it quickly turns out to be really boring, especially when testing lots of small endpoints, and testing for several response properties at once, like:

```ruby
subject { response }

its(:status) { is_expected.to eq 201 }
its(:headers) { are_expected.to include('Location') }
its(:body) { is_expected.to ... what?.. }
```
> **Note:** If you feel uneasy or unfamiliar with this style of RSpec (exactly one statement per test being expectation, description-less `it`, use of `rspec-its`), you can read my [theoretical rant](https://zverok.github.io/blog/2017-11-07-on-culture-of-bdd.html) about why it could be preferred. If you are familiar with it and prefer your test verbose and "magic-less" (duh), probably this article would not be that interesting for you.

There are suprisingly lot of attempts to find the "holy grail" of this simple json matcher ([1](https://github.com/waterlink/rspec-json_expectations), [2](https://github.com/collectiveidea/json_spec), [3](https://github.com/brooklynDev/airborne), [4](https://github.com/paul/rspec-resembles_json_matchers), [5](https://github.com/brigade/rspec_json_matchers)), and I've used one or another of them eventually, yet it always was a mixing feeling like "OK, it works, but not the way I'd like it to".

_(I by no means want to offend any of authors of the gems listed above, I am just investigating my concerns about designing tests.)_

Finally, [this gem](https://github.com/PikachuEXE/rspec-json_matchers) felt _almost right_, and after a short discussion with its author, I came with my own solution that finally feels "right."

In fact, the problem is painfully simple: once you start writing things like `expect(something).to match_json(value)`, it turns out that the thing is not as easy as just testing against static values: it is rather something "in this context, `meta.nextPage` should contain URL ending with `offset=100`". At this point, all of the gems listed above tend to jump into inventing their own conventions and DSLs of matching inside JSON, matching despite the array order, make sure that specified path exists and so on and so on. What bothered me with those approaches that it somehow felt absolutely unrelated to JSON, like generic values/structure testing, which RSpec probably already implemented?..

And then it clicked. The key is _matchers composability_, which is RSpec's natural feature emerged in RSpec 3:

```ruby
expect(response).to be_json('meta' => {})
expect(response).to be_json include('meta' => include('next' => end_with('offset=200')))
expect(response).to be_json hash_excluding('total')
```

This allows us to have the matcher itself really minimal ([read](https://github.com/zverok/saharspec/blob/master/lib/saharspec/matchers/be_json.rb) the full definition) while staying powerful for the most of real cases:

* we can use simple values;
* we can use RSpec matchers and combine them to any depth:
```ruby
include('items' => contain_exactly(String, String, kind_of(Hash).or(nil)))
```
* we also can use RSpec argument matchers from rspec mocks and combine them with other matchers:
```ruby
include(hash_including(anything => duck_type(:each)))
```
* it is pretty easy to achieve reasonable matcher output both for success (in `--format doc` mode) and fail (what exactly failed, and diffs of expected and actual).

All of this is possible because of RSpec matchers **composability** (they are designed to play well when combined):

1. They provide generic "case equality"/pattern matching operator `===` (which is one of nicest and most underused Ruby features, even advised against by some tutorials);
2. They use `===` to match all values inside (like hash keys and values), so anything that defines this operator in any way is suitable. Meaning you can mix matchers with simple values, as well as regexps, ranges, classes and even lambdas (though the latter may not be a useful idea):

```ruby
expect(response).to be_json(
  "meta" => hash_including("next" => 1..4),
  "result" => /success/i,
  "items" => Array,
  "total" => ->(t) { (t % 1000) * 18 < 46 }
)
```

That's the lesson of composability for the greater good!

### Use it

`be_json` matcher, described above is included in the next version of my [saharspec](https://github.com/zverok/saharspec) RSpec addons library, alongside with its brother `be_json_sym` (name is meh, have any idea of better?) which does the same but parses JSON with `symbolize_names: true`, which allows to embrace Ruby's short Hash keys syntax sugar:

```ruby
expect(response).to be_json_sym(
  meta: hash_including(next: 1..4),
  result: /success/i,
  ...
)
```

### Quirks

Mixing of RSpec matchers and argument matchers from rspec-mocks has its limitations: argument matchers don't have such a pretty `#describe`, and are not composable with `.and`. But the [reliable source](https://github.com/rspec/rspec-expectations/issues/1049#issuecomment-370048322) says the inconsistency will be fixed in a future RSpec versions with the unification of matchers.

### Bonus: cheap matchers composability

Sometimes, while testing complicated data structures, you can find yourself using some complicated statements like `be_an(Array).and(all(be_an(Integer)))` or `eq({next: 3}).or be_nil`. There is pretty terse syntax trick allowing to extract this kind of composition into single matchers:

```ruby
RSpec::Matchers.define :be_array_of do |item|
  match_unless_raises do |actual|
    expect(actual).to be_an(Array).and(all(match(item)))
  end
end

# Now you can use
expect(%w[foo bar]).to be_array_of(Integer)
# expected ["foo", "bar"] to be array of Integer

RSpec::Matchers.define :be_nullable_of do |matcher|
  match_unless_raises do |actual|
    expect(actual).to match(matcher).or be_nil
  end
end

expect('test').to be_nullable_of(Integer)
# expected "test" to be nullable of Integer

# In composition:
expect(response).to be_json(
  "meta" => {"next" => be_nullable_of(Integer)},
  "items" => be_array_of(Hash)
)
```