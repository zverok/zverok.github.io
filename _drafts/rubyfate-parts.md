Now, here is the idea of RSpec: using methods-with-blocks and small self-sufficient objects, to write _only what's necesary_ in expressive manner. Like this:

```ruby
subject { 2 + other }

context 'with numerics' do
  let(:other) { 3 }
  it { is_expected.to eq(5) }
#...
```

The "DSL" has surprising low numer of building blocks, and the most powerful ones are perfectly in line with Ruby's qualities: small matchers objects (`eq(5)`), with new custom ones being easily produced, methods-with-blocks allowing to structure names, descriptions, and examples in a regular way. The best thing about the system it is fully hackable and discoverable: methods like `it` and `let` defined in predictable places and have predictable calling context, allowing natural mix with "just Ruby":

```ruby
[1, 2, 3, 4].each do |test_value|
  context "when rendered with #{test_value}" do
    let(:value) { test_value }
    it { is_expected.to .... }
  end
end
```

Now, the basic idea of MiniTest is this (quoted from its README):

```ruby
class TestMeme < Minitest::Test
  def setup
    @meme = Meme.new
  end

  def test_that_kitty_can_eat
    assert_equal "OHAI!", @meme.i_can_has_cheezburger?
  end
```

Looks familiar? It definitely should! MiniTest follows the well-known design of "XUnit" library, looking mostly the same in... every mainstream language. So, yeah, yay for "mainstream quasilanguage" approach, which nowadays considered "more Rubyish" (till you make a typo in `test_` prefix, or try to hack into assertion system).

> The interesting nuance of this story is that current RSpec maintainers seem to befall under "let's don't do magic" spell themselves. Currently encouraged style—by most of modern guides, excluding some features like `its()` API and linters seems (to me!) to going against the "spirit" of RSpec, with one-line write-as-much-as necessary approach. Some time ago I covered those topics in more detail in a pair of blog-posts: [on MiniTest/RSpec](https://zverok.github.io/blog/2016-10-09-minitest.html) and on [tests expressivenes](https://zverok.github.io/blog/2017-11-07-on-culture-of-bdd.html).

For the

>>>>>> This fact frequently plays _against_ the changes importanse in public opinion, like "yawn, they've just added some more methods... language bloated ALREADY!" (Oh, and when the change _is_ the syntax one, like beginless/endless ranges, it is met with equal disgust "now they not only adding methods, but changed the syntax! WHY?")
