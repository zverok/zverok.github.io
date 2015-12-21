---
layout: post
title:  "On DataFrame datatype in Ruby"
date:   2015-09-22 22:21:30
categories: ruby sciruby rants
comments: true
---

**TL;DR**: We need DataFrame as a data structure in Ruby. There is several
promising candidates but no with good usability. Potential API and quality
considerations for implementation have been proposed.

## Intro: what is DataFrame and why we should has itz?

As programming languages evolve, our notion of "basic necessary" data
types for high-level language also evolve. So, nowadays in modern languages
we have different kinds of numbers for different tasks, arrays (which are
not "just pointers to memory area"), strings (which are not "just arrays
of characters"), hashes/dictionaries (which are not "some specialized
algorithmic concept, available through separate library"), regexps and so on.

Having complex types in a language core and having literals for them has
a great value: programmers don't need to reinvent something already considered
as a wheel. (If you tried to use together several C++ libraries, each having
different classes for strings and arrays, you'll understand.)

Now, I'm asking you to consider this thought:

> We need new "standard" type in Ruby: DataFrame.

("Standard" here not means adding to core language or even standard
library—for now—but just well-designed, highly usable and widely
aknowledged gem.)

**What is DataFrame**? It is currently widespread datatype you can find
in many data processing languages and libraries, like R, python+pandas,
Wolfram Mathematica, Julia and so on for storing _tabular data_.
It can be described this way:

* like 2-dimensional array (columns × rows), but...
* ...columns are named,
* ...and rows are "indexed": each row has corresponding label,
  which can be just sequental number (simplest and default case), or
  timestamp, or something else;
* each column have data of only one type inside it (different columns
  can have different data types);
* typical easy/cheap operations on DataFrame:
  * change data values (without changing row count);
  * including: create a "view" on some part of DataFrame and modify
    data inside it in one clean line of code (like "replace all `nil`
    values in column Salary with 0.0");
  * add/remove/switch columns;
  * calculate new column on base of existing ones;
  * select some rows/columns to another DataFrame;
* typically, DataFrame provides or supplied with methods for performing
  stats, summaries, and groupings on data;
* oftenly, DataFrame supports complex columns and complex indexes, like
  results of "pivotal table" operation (monthly income, grouped by department
  AND by manager inside department).

**Why we need DataFrame**? Because, it's, like, 2015? Because data processing
is everywhere: small data, big data, open data, internal data, frightening
amounts of data. And "tabular data" is a common humus for experimenting,
prototyping, developing new approaches and understanding the world—tasks
Ruby once was prominent for. You know, before it fell into "the thing under
Rails" tar-pit.

## Ruby-ish DataFrame, its qualities and responsibilities

What Ruby DataFrame need to have from the beginning?

* Good Ruby DataFrame class should, of course, correspond to all expectations
  of modern data processing, coming from other languages and tools (see
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
* Plotting: same as above;
* All stats methods and algorithms somebody may ever need: they may be
  in mixins, some (or most) of them in different gems; main DataFrame
  responsibility is to hold data, you see? and make it easy to process;
* You'll laugh, but even performance topics are NOT as important as
  good API and universal aknowledgement of new DataFrame datatype; of
  course, performance _does_ matter, but fast-and-ugly library is destined
  to have very limited use, and pretty-but-slow one can at least become
  popular for moderate-sized, "toy" task, became widespread and then be
  optimized and became performant AND pretty.

## Existing solutions and their brief critique

There were [several](https://github.com/willpearse/data_frame)
[attempts](https://github.com/davidrichards/data_frame)
[to create](https://github.com/domitry/mikon) DataFrame gem for Ruby
(the latter even had been sponsored with Ruby Association's grant, but
there is no commits since grant program finish). Most
of those attempts are just silently dying, used by no one except original
author (if anyone at all).

And it's just a standalone gems, not to mention Table-y and DataFrame-y
solutions inside bigger libraries, needing them for
[plotting](http://www.rubydoc.info/github/domitry/nyaplot/Nyaplot/DataFrame)
or [console output](https://github.com/tj/terminal-table). 

But lately, there is one candidate, receiving significant amount of
attention inside [SciRuby](http://sciruby.com/) community: it is called
[DaRu](https://github.com/v0dro/daru), which stands for Data Analysis
in RUby. You can look at pretty detailed showcase of it
[here](http://nbviewer.ipython.org/github/SciRuby/sciruby-notebooks/blob/master/Data%20Analysis/Daru%20Demo.ipynb).

First and really important: I should say DaRu is definitely a great effort
and it definitely should be highly appreciated.

What makes me feel sad about it is a scent of "ghetto", scientific software
ghetto in this case. DaRu feels for me (alongside with some other scientific
Ruby projects, I should admit) not like "enriching Ruby with concepts it
lacks", but rather mechanical translation of something to preserve familiar
to some scientists Python-like (or R-like) look. This way it will be hardly
accepted by wider Ruby community, and this acceptance is an only way for
library to live and prosper, not becoming "another curious attempt to
chase after Python".



What makes me feel sad

Head and tail???


> **Rant on science projects.**

## Show. Me. The. Code.

Here we go!

## Some final thoughts

### On Performance Considerations

### On Infrastructure Considerations


