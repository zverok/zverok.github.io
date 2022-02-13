---
layout: post
title:  On sustainable testing
date:   2017-11-07 18:30:00
categories: ruby rspec rants
comments: true
---

A few days ago, I've posted on /r/ruby [announce](https://www.reddit.com/r/ruby/comments/7b7bmy/saharspec_003_new_experimental_rspec_components/) of some RSpec matchers/plugins library, and it met with a (not very heated, TBH) discussion that I've expected, seen several times, and now want to focus on. The discussion is between two opposite approaches:

**Concise and expressive specs without textual descriptions and lot of custom matchers.
<br/>
vs.
<br/>
Formal and structured specs with textual descriptions and no special attention to laconism.**

Here is a showcase from Reddit thread (imagine we are testing `Array#[]` behavior):

```ruby
# approach 1
describe '#[]' do
  subject { [1, 2, 3].method(:[]) }

  its_call(0) { is_expected.to ret 1 }
  its_call(1..-1) { is_expected.to ret [2, 3] }
  its_call(:b) { is_expected.to raise_error TypeError }
end

# vs approach 2
describe '#[]' do
  subject { [1, 2, 3] }

  it 'returns the value at a given index' do
    expect(subject[0]).to eq 1
  end
  it 'returns a list of values between a range on indexes' do
    expect(subject[1..-1]).to eq [2, 3]
  end
  it 'raises TypeError when passed a symbol' do
    expect { subject[:b] }.to raise_error TypeError
  end
end
```
The discussion is pretty old. Current RSpec maintainers themselves consider the second approach "right", which is demonstrated by their banning of `its(:attr)` syntax in core and [this prominent explanation](https://gist.github.com/myronmarston/4503509).

Now, let me tell you something.

I adore RSpec. I firmly believe that RSpec itself, and the concept of "writing specs with behavior-driven development" (as opposed to tests and TDD). While working in Toptal, I saw the development culture which allowed 200+ developers being able to work simultaneously and effectively on a large codebase, written in the dynamic language (Ruby), because they have everything covered with specs.

Yet, considering all of the above, I need to say this:

**Currently preferred (by RSpec maintainers and significant part of the community) "verbose" specs style is not sustainable.**

What are sustainable tests (or specs)? They are tests that _would not be abandoned_. Which means, your colleague or future self, when adding or changing behavior, would most probably add/change specs.

As simple as that.

Now, what will make programmers write specs? (What? "Professionalism and responsibility"? Bullshit. Those two only can make programmer feel guilty for not having enough tests.)

Let's rephrase.

What makes programmers write code? Most of the time (at least that time when we are satisfied with ourselves) we do it because we love to, and want to. But not just _any_ code (please, kind sir, just let me write this cycle 100 more times!), we enjoy the feeling of solving the problem in the most minimal, readable, elegant and DRY way. That's what we do (_especially_ in Ruby, "optimized for developer's happiness").

And then, back to our tests/specs.

**Sustainable tests** is tests that are enjoyable to write. (Looks at the audience. Silence.)

Tests are just code. So we want them short. Elegant. DRY. We want to define higher-level abstractions, and we want code to speak for itself.

Let's review it again:

```ruby
# Currently forced approach
describe '#[]' do
  subject { [1, 2, 3] }

  it 'returns the value at a given index' do
    expect(subject[0]).to eq 1
  end
  it 'returns a list of values between a range on indexes' do
    expect(subject[1..-1]).to eq [2, 3]
  end
  it 'raises TypeError when passed a symbol' do
    expect { subject[:b] }.to raise_error TypeError
  end
end
```

Imagine it is just your regular app code (not "some special thing, not exactly code, it is tests, they are a COMPLETELY different thing"). Nothing surprises you in this code, everything is OK? Let's focus on some random piece:

```ruby
it 'raises TypeError when passed a symbol' do
  expect { subject[:b] }.to raise_error TypeError
end
```

The same thing, repeated twice, first in words, second in code. Then, there are those `subject[]` repeated and repeated everywhere, instead of saying "we want to test `[]`" once.

Here is, again, alternative version (NB: I am not saying that's the best syntax/RSpec additions possible, I am discussing the idea of cleaning up the code):

```ruby
describe '#[]' do
  subject { [1, 2, 3].method(:[]) }

  its_call(0) { is_expected.to ret 1 }
  its_call(1..-1) { is_expected.to ret [2, 3] }
  its_call(:b) { is_expected.to raise_error TypeError }
end
```

It is readable, and, most important, _declarative_: the DSL helps to structure code/and show how we want it to be structured.

What I hope, is the next developer adding a method to the same class will follow the DSL — just because it is more convenient than not to. And most probably he will not be too lazy to add new test when `[]`'s behavior will change: just because it is _even easier_ than test the new behavior manually (which TDD/BDD denialist probably will do).

Everything said above is not new. There are two main counter-arguments I am aware of: (1) descriptionless RSpec is badly documented one, and (2) tests are not code.

About (1), it goes like this. Let's compare two of our spec versions with `rspec format --doc`:

"Proper" version output:
```
#[]
  returns the value at a given index
  returns a list of values between a range on indexes
  raises TypeError when passed a symbol
```

"Concise" version ouput:
```
#[]
  (0)
    should return 1
  (1..-1)
    should return [2, 3]
  (:b)
    should raise TypeError
```

For me, both are informative enough. The "proper" one just outputs descriptions somebody wrote (IF they were well written... and NOT, for example, copied from neighbor group of tests, which is frequently the case). The second one shows pretty readable examples, auto-generated from code (and therefore always in sync with the actual test).

Proponents of "proper" version, though, will argue that `(1..-1) ... returns [2, 3]` is "misleading", saying that "method always returns `[2, 3]`".

Decide yourself!

The second counter-argument, "tests are not code", usually goes like this: Testing phase is _not_ for writing elegant code, it is boring formal activity, like proper documentation, or, IDK, manual testing of deployment on 100 target platforms. So it should be really simple, with all attempts to cut off the path explicitly banned.

The problem with this point of view is that unlike documentation (which can be delegated to those who really like to do it, me, for example; or postponed till "future release") testing is _everyday_ activity, and if you want it to be as formal as possible... Well, you would not have tests. It is not a coincidence that the same people frequently seen to argue about "test should not contain any 'magic'", and "all in all, unit-tests are not that necessary". Of course, you'd be arguing it is not necessary if it is boring, but who made it boring in the first place?
