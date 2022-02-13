---
layout: post
title:  "Good Ruby Idiom: and/or operators"
date:   2016-02-23 18:45:00
categories: ruby rants
comments: true
---

Any tutorial and book will teach you there are two sets of similar
operators in Ruby: `&&`/`||` vs `and`/`or` (and also `&`/`|` for bit
operations, but that's not the case today). But typical tutorial will not
provide further explanation **why** we need both of those pairs.

Some of tutorials and blog posts about this matter even go as far as
explicit recommendations _never_ use `and`/`or`, because `&&` and `||`
is enough for all reasonable cases, and because `and`/`or` provide only
confusion. They would typically [warn](http://stackoverflow.com/questions/1426826/difference-between-and-and-in-ruby)
you about `and`'s lower precedence than `=`, but nobody will tell you
**how** this can and should be useful.

Unfortunately, even respectable (and forced by Rubocop) 
[Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide) explicitly
[bans](https://github.com/bbatsov/ruby-style-guide#no-and-or-or) those
operators and embraces usage of `&&` and `||` even for control flow.
(Faithfully, this is one of Rubocop checks I turn off immediately after
adding Rubocop to every new project of mine.) 

But `and` and `or` are you friends, which allow to write very clean and
DRY and readable code: exactly the thing we love Ruby for.

**They are control flow operators.**

Example:

{% highlight ruby %}
File.exists? or raise "No config file!"
document.ready? and return
{% endhighlight %}

This idiom is one of (numerous) _good_ parts of Perl heritage in Ruby,
it is very terse and readable and demonstrate intentions well.

Somebody may argue `&&` and `||` can be used for both examples? Or, to
heck with it, why we should do control flow this way, we have our `if`
and `unless`, even modyfying ones?..

So, let's look at some examples.

**Case 1.** Task: receive some data from method; if data received, process
it further, if not—return (or fail) immediately. For example, complex
Nokogiri document parsing:

{% highlight ruby %}
def extract_actors(movie_doc)
  list = movie_doc.at('ul#actorsList')
  # now, if no list in doc, we should return, otherwise do lots of stuff
{% endhighlight %}

What options do we have?

{% highlight ruby %}
# option 1
list = movie_doc.at('ul#actorsList')
return unless list # extra independent statement

# option 2
if (list = movie_doc.at('ul#actorsList'))
  # ^ extra parentheses for distinguishing from ==
end

# option 3..20
# ...your choice!

# using control flow operator:
list = movie_doc.at('ul#actorsList') or return
{% endhighlight %}

Which is cleaner declaration of intent?.. For me—definitely the last one.
And you can't replace it with `||`, exactly because of operator precedence.

**Case 2.** List of checks: before some complex processing you want to
check your arguments and maybe state, fail if something is not OK, and
only then do the processing.

I do it like this:

{% highlight ruby %}
File.exists?(config) or raise "Configuration file #{config} not found"
x.is_a?(Numeric) && y.is_a?(Numeric) or
  raise "Only numeric values expected"
halted? || waits_for_halt? and raise "Process already halted"

# ...now to your normal processing code...
{% endhighlight %}

The same code with modyfying if/unless is not that clear:

{% highlight ruby %}
raise "Configuration file #{config} not found" unless File.exists?(config)
raise "Only numeric values expected" unless x.is_a?(Numeric) && y.is_a?(Numeric)
raise "Process already halted" if halted? || waits_for_halt?
{% endhighlight %}

Code with control flow operators reads like
`IMPORTANT_CONDITION or raise ...something...`. Code with modifying statements
reads like `raise THIS LONG VALUEABLE STRING unless .... something`.

And again, you will not want to use `&&` and `||` here because of operator
precedence (and some complications with parentheses, also).

I like this (a bit abstruse) definition:

> Logical operators (`&&` and `||`) operate on values; control flow operators
(`and` and `or`) operate on side effects.

PS: Of course, this is not first blog post with such kind of rant/explanation,
[there](https://www.tinfoilsecurity.com/blog/ruby-demystified-and-vs)
[are](http://devblog.avdi.org/2014/08/26/how-to-use-rubys-english-andor-operators-without-going-nuts/)
[plenty](http://www.prestonlee.com/2010/08/04/ruby-on-the-perl-origins-of-and-versus-and-and-or/)
of them. Just wanted to make [another] clear point about that.

