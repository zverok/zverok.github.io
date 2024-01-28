---
layout: page
title: Writing
permalink: /writing/
---

<div class="callout" markdown="1">
  There is a [war](https://war.ukraine.ua/) in Ukraine. I am still trying to write about Ruby, and coding, and such things, but my Twitter is now mostly about the war.
</div>

<br/>

_**Before the war, the text was:** I write a lot but quite irregularly. Fiction, poetry, magazine articles, blog posts, <a href="https://twitter.com/zverok">Twitter</a>. The book and <a href="https://zverok.substack.com">Substack</a>/blog are my current main focuses._

## Fiction/poetry

* [Mine](/writing/fiction/) (mostly in Ukrainian)
* [My translations of modern Ukrainian Poetry into English](https://7uapoems.substack.com/)

<!--

## Books

* **41 Ruby Intuitions (Gaining architectural insight from looking at code as text)**: the book on Ruby I wanted to write for many years, and finally planned to start in 2022... Then the full-scale Russian invasion started, and I just don't have resource to proceed with it. I am still planning to do it, but after the war. There is an (empty) [Substack](https://rubyintuitions.substack.com/) dedicated to it; I'll also write on my [primary one](https://zverok.substack.com) when the project would move forward.
* **Ruby Evolution**:

## Books

Yeah, plural. There are two to talk about:

* **41 Ruby Intuitions (Gaining architectural insight from looking at code as text)**: the book on Ruby I wanted to write for many years, and finally planned to start in 2022... Then the full-scale Russian invasion started, and I just don't have resource to proceed with it. I am still planning to do it, but after the war. There is an (empty) [Substack](https://rubyintuitions.substack.com/) dedicated to it; I'll also write on my [primary one](https://zverok.substack.com) when the project would move forward.
* **Going to Yalta After the Apocalypse (a novel in 213 lost photographs)**: A post-apocalyptic road novel I also planned to write this year... And this is the one I was able to proceed with. It is written and [published on Substack](https://goingtoyalta.substack.com/) in everyday small (\~300-something words) chunks since May 1st, and is planned to be finished at December 1st. You can read more about the intention and writing process in the [author's intro](https://goingtoyalta.substack.com/p/pre-intro). **UPD:** The book was finished as scheduled, on December 1st. Author's outro and link to full unedited EPUB [are available here](https://goingtoyalta.substack.com/p/outro). I plan to remove it from the Internet in a month before editing and looking for a publisher.

## "41 Ruby Intuitions"—book in progress {#ruby-intuitions}

**Gaining architectural insight from looking at code as text**

The main topic of the book is looking into the ways of code writing that allow creating lucid code. Lucid code conveys its intention and means in the most humane ways possible. I'll be exploring the ways of thinking that Ruby programming language, "designed for developer's happiness" teaches, and how paying attention to singular (code) phrases allow one to write better.

<iframe src="https://rubyintuitions.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>
-->

## Blog {#blog}

**Note:** Most of my writing is currently first published at [Substack](https://zverok.substack.com). You can subscribe by email or read it on the web.

{% for post in site.posts%}
<div class="post postContent">
  <!--
  <div class="postTag">
    {{post.tag}}
  </div>
-->
  <br>
  <div class="postTitle">
  <a class='postLink' href="{{site.url}}{{site.baseurl}}{{post.url}}">{{post.title}}</a>
  </div>
  <div  class="postDate"><time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %-d, %Y" }}</time>
  </div>
  <div class="postExt">
    {% if post.description %}
      {{ post.description }}
    {% else %}
      {{ post.content | strip_html | truncate:200}}
    {% endif %}
  </div>
</div>

{% endfor %}
