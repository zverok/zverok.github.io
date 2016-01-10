---
layout: post
title:  "On DataFrame datatype in Ruby"
date:   2016-01-10 22:21:30
categories: ruby sciruby rants
comments: true
---

**TL;DR**: We need DataFrame as a data structure in Ruby. There are several
promising candidates but no one with good usability. Some considerations
on requirements to _good_ DataFrame library are proposed, alongside with
some rants on using Ruby for science.

## Intro: what is DataFrame and why we should has itz?

As programming languages evolve, our notion of "basic necessary" data
types for high-level language also evolves. So, nowadays in modern languages
we have different kinds of numbers for different tasks; we have arrays
(which are not "just pointers to memory area"); we have strings (which are
not "just arrays of characters"), hashes/dictionaries (which are not
"some specialized algorithmic concept, available through separate library"),
regexps, ranges and so on.

Having complex types in a language core and having literals for them has
a great value: programmers don't need to reinvent something already considered
as a wheel. (If you tried to use together several C++ libraries, each having
different classes for strings and arrays, you'll understand.)

Now, I'm asking you to consider this thought:

> **We need new "standard" type in Ruby: DataFrame.**

("Standard" here does not mean adding to core language or even standard
library—for now—but just well-designed, highly usable and widely
aknowledged gem.)

**What is DataFrame**? It is currently widespread datatype you can find
in many data processing languages and libraries, like R, python+pandas,
Wolfram Mathematica, Julia and so on, for storing _tabular data_.
It can be described this way:

* like 2-dimensional array (columns × rows), but...
* ...columns are named,
* ...and rows are "indexed": each row has corresponding label,
  which can be just sequental number (simplest and default case), or
  timestamp, or something else;
* each column has data of only one type inside it (different columns
  can have different data types);
  * though, any column can have empty places (`nil` values) instead of
    data;
* typical easy/cheap operations on DataFrame:
  * change data values (without changing row count);
  * including: create a "view" on some part of DataFrame and modify
    data inside it in one clean line of code (like "replace all `nil`
    values in column Salary with 0.0");
  * add/remove/switch columns;
  * calculate new column on base of existing ones;
  * select some rows/columns to another DataFrame;
* typically, DataFrame provides methods, or supplied with libraries,
  for performing   stats, summaries, and groupings on data;
* often, DataFrame supports complex columns and complex indexes, like
  results of "pivotal table" operation (monthly income, grouped by department
  AND by manager inside department).

**Why we need DataFrame**? Because, it's, like, 2016? Because data processing
is everywhere: small data, big data, open data, internal data, frightening
amounts of data. And "tabular data" is a common humus for experimenting,
prototyping, developing new approaches and understanding the world—tasks
Ruby once was prominent for. You know, before it fell into "the thing under
Rails" tar-pit.

## Ruby-ish DataFrame, its qualities and responsibilities

What does Ruby DataFrame need to have from the beginning?

* Good Ruby DataFrame class should, of course, correspond to all expectations
  of modern data processing, coming from other languages and tools (look
  above for the list of expectations);
* initialization of DataFrame should look as close to "literal", as possible
  (for futher integration in libraries and practices);
* public interface should be clear, terse and unambiguous: for ex., access
  to rows should be clearly distinguished from access to columns;
* public interface should be as close as possible to Ruby's best practices:
  see `Array`, and `Hash`, and `Enumerable`; it should not resemble
  DataFrames of other languages and tools;
* column **is** an object, row **is not** (it is rather slice across all
  the columns)—_column object even have a name in other DataFrame-y solutions:
  either Series or Vector_—and this difference should be clearly visible
  from interface;
* complex columns and complex indexes should ideally be supported;
* as DataFrame is frequently used for experiments and prototypes, it
  should be pretty-printed by design (and this pretty-printing should by
  design consider "very large" data sets).

What Ruby DataFrame **need not** be at the beginning?

* Import and export from tons of file formats and datasources: whether
  we'll have good usable data structure, it will not be hard to add this
  functionality;
* Plotting: the same as above;
* All stats methods and algorithms somebody may ever need: they may be
  in mixins, some (or most) of them in different gems; main DataFrame
  responsibility is to hold data, you see? and make it easy to process;
* You'll laugh, but even performance topics are NOT as important as
  good API and universal aknowledgement of new DataFrame datatype; of
  course, performance _does_ matter, but fast-and-ugly library is destined
  to have very limited usage, and pretty-yet-slow one can at least become
  popular for moderate-sized, "toy" tasks. After that, it becames widespread
  and somebody optimizes it, and—voila!—it is performant AND pretty.

## Existing solution(s)?

There were [several](https://github.com/willpearse/data_frame)
[attempts](https://github.com/davidrichards/data_frame)
[to create](https://github.com/domitry/mikon) DataFrame gem for Ruby
(the latter even had been sponsored with Ruby Association's grant, but
there is no commits since grant program finish). Most
of those attempts are just silently dying, used by no one except original
author (if anyone at all).

And it's just a standalone gems, not to mention Table-y and DataFrame-y
solutions inside bigger libraries, using them for
[plotting](http://www.rubydoc.info/github/domitry/nyaplot/Nyaplot/DataFrame)
or [console output](https://github.com/tj/terminal-table). 

But lately, there is one candidate, receiving significant amount of
attention inside [SciRuby](http://sciruby.com/) community: it is called
[DaRu](https://github.com/v0dro/daru), which stands for Data Analysis
in RUby. You can look at pretty detailed showcase of it
[here](http://nbviewer.ipython.org/github/SciRuby/sciruby-notebooks/blob/master/Data%20Analysis/Daru%20Demo.ipynb).

First and really important: I should say DaRu is definitely a great effort
and it definitely should be highly appreciated.

But the fact it became (relatively) popular recently, makes me rather
sad than happy.

Why? For two (connected) reasons:

* despite being useful, having many features and incorporating large amount
  of work, DaRu fails to be good Ruby library;
* which, for me, is one of signs of bad communication between general
  Ruby community and scientific Ruby community.

About first part of that rant ("fails to be good Ruby library"): just
look at [DaRu](https://github.com/v0dro/daru) README and examples, and
try to use it by yourself. It contains many useful statistical features,
integrates with [IRuby](https://github.com/SciRuby/iruby) scientific
notebook, but...

* it provides no examples in README, except for links to those notebooks
  (like, it from the beginning says "if you don't know our staff,
  you'll not need our libraries!");
* examples in notebooks use old wordy syntax for hashes, like
  `DataFrame.new({:column => values})` instead of
  `DataFrame.new(column: values)`—not a sin on its own, but tends to
  show an attitude;
* it has no reasonable methods like `columns` (they are `vectors`, very
  scientific, wow) or `rows`;
* it doesn't respond to `first` and `last` like Ruby collections (they are
  `head` and `tail`!);
* it has no `select` and `reject`, but has `filter` and `where`;
* it has no `slice`, but has overloaded `[]` for all kinds of data slicing
  (just like Pythonists accustomed to do!), just try yourself to guess
  what `data_frame[1]` should do or "how to extract first two rows".

You can say "OK, it is not shiniest API possible, but heck with that!
DaRu works and it is only alive and working Ruby's DataFrame!"

And here **the big problem of Ruby+Science** comes.

Ruby has brevity, flexibility, consistency and deserves to be language
for data processing and scientific experiments. It is very clear and
readable for writing reference implementations for state-of-the-art algorithms.
Yet, we all know, it is not the case in today's world.

There's only one way to fix this: have a great tools for scientists. And
**"port Pythons tools" is not the way to achieve this goal**. (Those using
Python for science would not migrate to Ruby "because it has almost the
same tools").

Great tools can be made by _positive feedback loop_:

* provide something good for existing rubyists (even if they are not
  scientists);
* receive their feedback and support, enchance the tools, make them
  integrated into Ruby ecosystem and widely accepted inside it;
* ...make users outside the Ruby community envious.

(Or, if you feel really brave, there's also another way, the way Rails
were made: _invent new, ingenious approach to known problems, approach
that only Ruby makes possible_.)

Currently, [SciRuby](http://sciruby.com/) (part of which DaRu is), being
solid and honorable initiative, seems to go through _negative feedback
loop_:

* not much of generic Ruby community uses those libraries: they are
  "not for us" (all examples are in IRuby, for artificial scientific
  tasks);
* not much of scientists use Ruby/SciRuby—it is still far from maturity,
  and, faithfully, SciRuby does not feel like "Ruby will open new horizons
  for you!", it is rather like "Ruby also can do something (almost) like
  Python/pandas";
* almost nobody uses libraries → almost no support for further development
  → things don't become better → ....

I think, we should at least **start to try** to go out of this "scientific
ghetto" by clean, easy-to-use, Ruby-ish DataFrame concept.

## Conclusions

* We need DataFrame in Ruby for many kind of tasks;
* It requires solid community effort for planning, testing and inventing
  use cases; natural API is the most important thing to achieve (even
  **before** good performance for large datasets!);
* DaRu may be, or may not be, a good starting point for Good DataFrame,
  but its current state is very far from what we need;
* Would be great if scientific-oriented projects in Ruby come out from
  their ghetto and try to play with other guys.

Unnecessary addition: [here](https://gist.github.com/zverok/5438eb69da9ac34d791c)
is some rough draft of "how Good DataFrame could work in Ruby". Really
rough and underthougt, though.
