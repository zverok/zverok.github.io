Fantastict Wikipedia Infoboxes and How To Parse Them

WHY SHOULD I READ THIS → look into infoboxes parsing in the FIRST paragraph

I might be repeating myself, but Wikipedia will never stop to fascinate me. The abundance of openly-accessible, coherent, cross-linked human data emerged during couple of decades. I can still remember past life without it, but it is hard to imagine the future with no Wikipedia.

> example: when it was created?

And it is a living thing, which is another fact to marvel. Like a living thing, it grows, changes, evolves. Like a living thing, it is never complete, frequently self-contradictory, and inconsistent and irregular in a myriad of small different ways.

As an engineer, I should be irritated by any data irregularity. But as a human being, I tend to find the irregularity natural and aesthetically pleasing; and as a programmer with a focus on expressiveness and "solve the impossible tasks" attitude, I am up for a challenge!

The most _seemingly_ formal part of the Wikipedia articles are "infoboxes"—it is a term to designate those parts of the page:


That's the part of Wikipedia that creates a temptation to parse—and that's how I was hooked up too, many years ago. When I imagined that API like `City('Kharkiv').area` should be available in open programming languages, using an open data sources, wikipedia infoboxes were my very first guess.

That's why my very first project—Wikipedia parser, still alive and quite powerful—was called [Infoboxer](https://github.com/molybdenum-99/infoboxer). Its scope quickly grew up much further than this part of the page, but the name stuck.

> A small historical digression: Wikidata discovery; but still useful perspective

And so, I am returning to where it all started: look into Wikipedia infoboxes parsing for WikipediaQL query language, and trying to design parsing primitives for it—in a slow realtime, while writing this article.


They are still evading strict formal form, as we'll see later.

## Just give me the data already

Typical infobox looks like this (Chrome inspect on the right shows the internal structure)

![](image00.png)

So, it is actually just a table, with a more or less clear structure: `th`'s in each row give field name, `td` gives its value. There are also subheaders, images and stuff, but at a first glance, most of the data should be extractable with `table-data` I designed for Wikidata tables. Lets check!

```
$ wikipedia_ql --page "Tilda Swinton" 'table.infobox >> table-data >> tr >> td >> { @row as "name"; text as "value" }'

- value: Swinton at the 2018 Vienna International Film Festival
- value: Swinton at the 2018 Vienna International Film Festival
- name: Born
  value: "Katherine Matilda Swinton\n\n (1960-11-05) 5 November 1960 (age 61)\nLondon,\
    \ England"
- name: Education
  value: New Hall, Cambridge (BA)
- name: Occupation
  value: Actress
- name: Years active
  value: 1984–present
- name: Works
  value: Full list
- name: Partner(s)
  value: 'John Byrne (1989–2003)

    Sandro Kopp (since 2004)'
- name: Children
  value: 2, including Honor Swinton Byrne
- name: Parent(s)
  value: 'John Swinton

    Judith Balfour'
- name: Family
  value: Swinton
```

That's surprisingly coherent (save for two nameless duplicates at the beginning—that's what `table-data` does with `colspan=2` cell with a photo; we'll get to it later).

The fact that it works as expected makes it possible to just fetch some values:

```
$ wikipedia_ql --page "Tilda Swinton" 'table.infobox >> table-data >> { td[row^="Born"] as "born"; td[row^="Partner"] as "partner" }'

- born: "Katherine Matilda Swinton\n\n (1960-11-05) 5 November 1960 (age 61)\nLondon,\
    \ England"
  partner: 'John Byrne (1989–2003)

    Sandro Kopp (since 2004)'
```
....which hopefully works with most of the similar pages?

```
$ wikipedia_ql --page "Keanu Reeves" 'table.infobox >> table-data >> { td[row^="Born"] as "born"; td[row^="Partner"] as "partner" }'

- born: "Keanu Charles Reeves\n\n (1964-09-02) September 2, 1964 (age 57)\nBeirut,\
    \ Lebanon"
  partner: "Jennifer Syme\n(1998–2001; her death) \n Alexandra Grant\n(c. 2018–present)"
```

(We'll need to use `row^="Spouse"` for some other people; which suggest some possibilities to improval of query language: either allow regexps, or we need "either" operator... but that's relatively minor and can be solved by postprocessing with, say, `jq`)

But maybe I've simplified the task too much, choosing a page with a small-ish infobox. Let's take something more hairy, like a country.


Hmm, a large part looks OK, but at the end, there is a huge bunch of `name`-less rows.

Looking at the page? Ooops. That's because there are two "infoboxes" there, the second one is in "Economy" section:

We could've guessed with a script, doing it this way:

```
wikipedia_ql --page "Thailand" 'table.infobox as "infobox" >> table-data >> { @title as "title"; tr >> td >> { @row as "name"; text as "value" } }'

- infobox:
  - title: Kingdom of Thailandราชอาณาจักรไทย (Thai)Ratcha-anachak Thai
  # ... normal data

- infobox:
  - title: Economic indicators
  # ... value-only data
```

If we'll take just the first one:

```
$ wikipedia_ql --page "Thailand" 'table.infobox:first >> table-data >> tr >> td >> { @row as "name"; text as "value" }'

...
NotImplementedError: ':first' pseudo-class is not implemented at this time
```

Ugh. That's not me, that's Python's soupsieve CSS selectors library. Which is totally awesome, but probably wasn't targeting cases as complicated as ours :(

Well... At least we could

```
$ wikipedia_ql --page "Thailand" 'section:first >> table.infobox >> table-data >> tr >> td >> { @row as "name"; text as "value" }'
```

...which fails too, but that's one is on me (`:first` **was** not implemented for `section`, so I am implementing it now)

```
$ wikipedia_ql --page "Thailand" 'section:first >> table.infobox >> table-data >> tr >> td >> { @row as "name"; text as "value" }'
```
To handle the second infobox (that in "Economy" section) we also have a trick up our sleeve. When I developed `table-data` pseudo-selector and decided on its usecases, I noticed that some tables in Wikipedia don't have row-level `th`-s, even if logically first cell of the row _is_ its header. So there is a "function" `:force-row-headers(<num_of_cells>)` for that:

```
$ wikipedia_ql --page "Thailand" 'section[heading="Economy"] >> table.infobox >> table-data:force-row-headers(1) >> tr >> td:first-child >> { @row as "name"; text as "value" }'

- name: Nominal GDP
  value: ฿14.53 trillion  (2016)
- name: GDP growth
  value: 3.9%  (2017)
- name: Inflation • Headline • Core
  value: '0.7%  (2017)

    0.6%  (2017)'
- name: Employment-to-population ratio
  value: 68.0%  (2017)
- name: Unemployment
  value: 1.2%  (2017)
- name: Total public debt
  value: ฿6.37 trillion  (Dec. 2017)
- name: Poverty
  value: 8.61%  (2016)
- name: Net household worth
  value: ฿20.34 trillion  (2010)
```

That will also help us when we meet Titanic (not like iceberg met it!)

```
$ wikipedia_ql --page "Titanic" 'table.infobox >> table-data:force-row-headers(1) >> tr >> td >> { @row as "name"; text as "value" }'
```

(Nice presentation dilemma)

### Infobox which was not

We fail miserably with "World War II"; and that's a time to stop and think.

Many levels of nesting.
(Template is simpler but)
What IS the right thing here?

Note that Wikidata [fails to structure/contain this information either](https://www.wikidata.org/wiki/Q362).
But **if** we'll forget the attempt to parse it as a regular infobox, we can think of something...

There is one more trick of `table-data` I invented but haven't implemented yet, and now it is time to do so!

One more infobox I randomly checked is https://en.wikipedia.org/wiki/Mars

## Conclusions: Embracing the uncertainty

Let me be open here: I opened this weeks document/plan in expectations to design some neat selector/set of selectors like `infobox`/`infobox-value`....

It turned out that with the approach of WikipediaQL, there is nothing so special about infoboxes



### What's next?

I am not sure. Designing the thing that has so many cool potential purposes

I'll try to look the new direction opened: maybe the generic-from-the-beginning is actually better? There are several equally useful albeit less known wikis. My favorite is Wikivoyage (not to confuse with Wikitravel) which grew into a go-to resource.

_Also, I always liked the Fandom (nee Wikia) rich set of cause-dedicated wikis; but unsure whether I want to get involved with them for experiments and demos. For one, the site is so ad-ridden now my Chrome almost dying, even with AdBlock. Another cause is that they seem to not advertise MediaWiki API, and the half of it isn't even available._

So, next time I might
