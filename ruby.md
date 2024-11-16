---
layout: page
title: Contributions to Ruby
permalink: /ruby.html
---

List of my ([zverok aka Victor Shepelev](https://zverok.space)) contributions to discussing/changing of the Ruby programming language. It links to [Ruby bug tracker discussions](https://bugs.ruby-lang.org/issues?utf8=%E2%9C%93&set_filter=1&sort=priority%3Adesc%2Cupdated_on%3Adesc&f%5B%5D=status_id&op%5Bstatus_id%5D=*&f%5B%5D=author_id&op%5Bauthor_id%5D=%3D&v%5Bauthor_id%5D%5B%5D=710&f%5B%5D=&c%5B%5D=project&c%5B%5D=tracker&c%5B%5D=status&c%5B%5D=subject&c%5B%5D=assigned_to&c%5B%5D=updated_on&group_by=&t%5B%5D=) and [GitHub PRs](https://github.com/ruby/ruby/pulls?q=is%3Apr+author%3Azverok+) of things I've contributed and proposed and their current status. Rejected/ignored proposals are eventually removed from the list.

## Ruby features

### Accepted

* **Ruby 3.2**:
  * [`Data` class](https://bugs.ruby-lang.org/issues/16122)
  * [`Time#deconstruct_keys`](https://bugs.ruby-lang.org/issues/19071), also [`Date` and `DateTime`](https://github.com/ruby/date/pull/80)
* **Ruby 3.1**:
  * [`Enumerable#compact` and `Enumerator::Lazy#compact`](https://bugs.ruby-lang.org/issues/17312)
* **Ruby 3.0**:
  * [`Symbol#to_proc` lambdiness](https://bugs.ruby-lang.org/issues/16260)
  * [Array slicing with `ArithmeticSequence`](https://bugs.ruby-lang.org/issues/16812)
* **Ruby 2.7**:
  * [`Comparambe#clamp` with a range](https://bugs.ruby-lang.org/issues/14784): `1.clamp(0..100)`, `1.clamp(5..)`, `1.clamp(..18)`
  * [`Enumerator#produce`](https://bugs.ruby-lang.org/issues/14781)
  * [Beginless range](https://bugs.ruby-lang.org/issues/14799)
  * [`Method#inspect`](https://bugs.ruby-lang.org/issues/14145)
  * ~[`Date#inspect` simplified](https://github.com/ruby/date/pull/12)~ (reverted)
* **Ruby 2.6**:
  * [`Kernel#then`](https://bugs.ruby-lang.org/issues/14594) as a synonym (better name) for `#yield_self` ([context](https://zverok.space/blog/2018-03-23-yield_self2.html))
  * [`Enumerable#chain`](https://bugs.ruby-lang.org/issues/15144)
  * [`Range#===`](https://bugs.ruby-lang.org/issues/14575) to use `cover?` instead of `include?`
  * [`CSV::Row#each_pair`](https://github.com/ruby/csv/pull/33) for compatibility with `OpenStruct`

### Pending

* [`Hash#merge`: smarter protocol depending on passed block arity](https://bugs.ruby-lang.org/issues/19272)
* [Make a concept of "consuming enumerator" explicit](https://bugs.ruby-lang.org/issues/19061)
* [Update of semantics for `Range#step`](https://bugs.ruby-lang.org/issues/18368)
* [`Enumerator::Lazy#partition`](https://bugs.ruby-lang.org/issues/18262)
* [`Enumerable#take_while_after`](https://bugs.ruby-lang.org/issues/18136)
* [`Dir#empty?` and `File#empty?`](https://bugs.ruby-lang.org/issues/16249)
* [`Object#non`](https://bugs.ruby-lang.org/issues/17330)

### Contributed to discussions

(The cases where I was not an author of initial proposal, but believe my contribution/pushing for solution was significant enough)

* **3.1**: `IO::Buffer` [reviews/proposals](https://bugs.ruby-lang.org/issues/18417)
* **3.0**: [`Hash#except`](https://bugs.ruby-lang.org/issues/15822)
* ~~**2.7**: [Syntax sugar for method reference](https://bugs.ruby-lang.org/issues/13581): seems `.:` is finally accepted for 2.7~~ (ugh, [reverted](https://bugs.ruby-lang.org/issues/16275))
* **2.5**: [`Kernel#yield_self`](https://bugs.ruby-lang.org/issues/6721) -- I don't like the name, but for several months pushed for "we should have this method, whatever name you choose" :) Then we renamed it to `#then` in 2.6

## Minor clarifications and bugs

* **3.2** [`Range#include?` inconsistency for beginless `String` ranges](https://bugs.ruby-lang.org/issues/18580)
* **3.1** [Necessity of `require 'fiber'`](https://bugs.ruby-lang.org/issues/17407)
* **3.0** [`Object.clone(freeze: true)`](https://bugs.ruby-lang.org/issues/16175)
* **2.7** [Deprecate](https://bugs.ruby-lang.org/issues/15893) `Kernel#open` provided by `open-uri` in favor of more explicit `URI.open`
* **2.7** [`Time#dst?` bug for time with real timezone](https://bugs.ruby-lang.org/issues/15988)
* **2.7** [`IO#set_encoding_by_bom` raising on already set encoding](https://bugs.ruby-lang.org/issues/16422)
* [`StringIO#internal_encoding` is broken](https://bugs.ruby-lang.org/issues/16497)

## Documenting Ruby

### Merged

* **Ruby 3.2**:
  * [Improve `TracePoint#allow_reentry` docs](https://github.com/ruby/ruby/pull/5380)
  * [Improve `Random::Formatter` docs](https://github.com/ruby/ruby/pull/5434)
  * [Clarify `Class#subclases` behavior quirks](https://github.com/ruby/ruby/pull/5480)
  * [Document `AST.parse`'s keyword arguments](https://github.com/ruby/ruby/pull/6996)
  * [Group of pre-3.2 fixes for docs](https://github.com/ruby/ruby/pull/6985)
  * [Document new methods of `IO::Buffer` and `Fiber::Scheduler`](https://github.com/ruby/ruby/pull/7016)
* **Ruby 3.1**:
  * [Add documentation for new hash value omission syntax](https://github.com/ruby/ruby/pull/5244)
  * [Add documentation for new `Refinement` class](https://github.com/ruby/ruby/pull/5272)
  * [Improve `Thread::Queue.new` docs](https://github.com/ruby/ruby/pull/5273)
  * [Fix `StructClass::` class method docs](https://github.com/ruby/ruby/pull/5279)
  * [`Fiber::SchedulerInterface#io_read` and `#io_write`](https://github.com/ruby/ruby/pull/5280)
  * `IO::Buffer`: [1](https://github.com/ruby/ruby/pull/5302), [2](https://github.com/ruby/ruby/pull/5374)
  * [`Thread::Backtrace.limit`](https://github.com/ruby/ruby/pull/5305)
  * [Fix `String#unpack` and `#unpack1` docs](https://github.com/ruby/ruby/pull/5331)
  * [Document `Marshal#load` parameter `freeze:`](https://github.com/ruby/ruby/pull/5332)
* **Ruby 3.0**:
  * [Full docs for non-blocking Fibers and scheduler](https://github.com/ruby/ruby/pull/3891)
  * [New docs for `Fiber.transfer`](https://github.com/ruby/ruby/pull/3981)
  * [Full class- and method-level docs for Ractors](https://github.com/ruby/ruby/pull/3895)
  * [Update method definition docs to include `...` and one-line methods](https://github.com/ruby/ruby/pull/3997)
  * [group of small fixes in 3.0 core docs](https://github.com/ruby/ruby/pull/3997)
  * [JSON](https://github.com/flori/json/pull/349): hide irrelevant parts of docs
  * [JSON: enhance generic `JSON` and `#generate` docs](https://github.com/flori/json/pull/347)
* **Ruby 2.7**:
  * [`Kernel#system(exception: true)`](https://bugs.ruby-lang.org/issues/15480)
  * [`NoMethodError`/`NameError` new arguments](https://bugs.ruby-lang.org/issues/15481)
  * [`Kernel#BigDecimal(exception: false)`](https://github.com/ruby/bigdecimal/pull/117)
  * [`BigDecimal::Jacobian` -- fix invisible docs](https://github.com/ruby/bigdecimal/pull/130)
  * [`TracePoint#enable`](https://bugs.ruby-lang.org/issues/15484)
  * [`Enumerator::Lazy`](https://bugs.ruby-lang.org/issues/15529)
  * [fileutils](https://github.com/ruby/fileutils/pull/33): facelift module docs and fix some bugs;
  * [group of small fixes of formatting in core docs](https://bugs.ruby-lang.org/issues/16126)
  * [group of larger fixes](https://github.com/ruby/ruby/pull/2612): toplevel `return`, full comments syntax explanation, `rescue` in blocks, better docs for `chomp:` option, `Object#to_enum`, `Proc#>>` and `#<<`, `Process` module
  * [Matrix: slightly enhance docs](https://github.com/ruby/matrix/pull/11)
  * [group of stdlib documentation fixes](https://github.com/ruby/ruby/pull/2615): ERB, StringIO, IRB, Net::FTP, open-uri, OptionParser, Net::HTTP
  * [`Bundler` entire stdlib](https://github.com/bundler/bundler/pull/7394)
  * [Webrick: document Proc body for Response](https://github.com/ruby/webrick/pull/35)
  * [Numbered block parameters](https://github.com/ruby/ruby/pull/2767)
  * [Small final pre-2.7 changes](https://github.com/ruby/ruby/pull/2768)
  * [`Module#const_source_location`](https://github.com/ruby/ruby/pull/2750)
  * ~[`RubyVM.resolve_feature_path`](https://bugs.ruby-lang.org/issues/15482)~ (the feature is moved to `$LOAD_PATH`)
  * [Pattern matching](https://github.com/ruby/ruby/pull/2786)
* **Ruby 2.6**:
  * [`&.`](https://bugs.ruby-lang.org/issues/15109)
  * [`Kernel#yield_self`](https://bugs.ruby-lang.org/issues/1443)
  * [`Method`](https://bugs.ruby-lang.org/issues/14483)
  * [`YAML`](https://bugs.ruby-lang.org/issues/14567) (explanation of aliasing to `Psych`)
  * [`MatchData`](https://bugs.ruby-lang.org/issues/14450)
  * [`Proc`](https://bugs.ruby-lang.org/issues/14610)
  * [`Endless range`](https://bugs.ruby-lang.org/issues/15405)
  * [`Integer(exception: false)` and others](https://bugs.ruby-lang.org/issues/15452)
  * [`Tempfile`](https://bugs.ruby-lang.org/issues/15411)
  * [`Psych#dump`](https://github.com/ruby/psych/pull/351)
  * [`CSV`: redesign main class docs](https://github.com/ruby/csv/pull/32)

### Pending

* [Update docs on sharing module instance variables in Ractors](https://github.com/ruby/ruby/pull/5437)

