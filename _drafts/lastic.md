# On Implementing Good Elastic Search DSL in Ruby

ElasticSearch Query DSL is implemented as an AST (abstract syntax tree).
What's wrong with it:

* verbosity and "unnaturality", goind directly from its AST nature;
* some pretty unusual constructs, which are complex to comprehend.

**NB**: This is neither critics of ElasticSearch (which is awesome),
nor of its approach to query language (which is reasonable). What I'm
trying to think about is

## ElasticSearch Basics

### Queries and filters

### Combining statements

### Putting everything in order

## Solving the "make DSL puzzle"

**Prerequizites**:

* as Ruby-natural as possible;
* as less "magic" as possible;
* as thin wrapper as possible.

**Ruby-natural**

**Less magic**

**Thin wrapper**
