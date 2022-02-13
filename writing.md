---
layout: page
title: Writing
permalink: /writing/
---

<div class="callout">I write a lot but quite irregularly. Fiction, poetry, magazine articles, blog posts, <a href="https://twitter.com/zverok">Twitter</a>. The book and <a href="https://zverok.substack.com">Substack</a>/blog are my current main focuses.</div>

> You can also check [some of my old fiction/poetry](/writing/old/) (most of it is in Ukrainian and Russian, though).

## "41 Ruby Intuitions"—book in progress {#ruby-intuitions}

**Gaining architectural insight from looking at code as text**

The main topic of the book is looking into the ways of code writing that allow creating lucid code. Lucid code conveys its intention and means in the most humane ways possible. I’ll be exploring the ways of thinking that Ruby programming language, “designed for developer’s happiness” teaches, and how paying attention to singular (code) phrases allow one to write better.

<iframe src="https://rubyintuitions.substack.com/embed" width="480" height="320" style="border:1px solid #EEE; background:white;" frameborder="0" scrolling="no"></iframe>

## Blog {#blog}

**Note:** Most of my writing is currently first published at [Substack](https://zverok.substack.com). You can subscribe by email or read it on the web.

{% for post in site.posts%}
<div class="post postContent">
  <div  class="postDate"><time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %-d, %Y" }}</time>
  </div>
  <div class="postTag">
    {{post.tag}}
  </div>
  <br>
  <div class="postTitle">
  <a class='postLink' href="{{site.url}}{{site.baseurl}}{{post.url}}">{{post.title}}</a>
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
