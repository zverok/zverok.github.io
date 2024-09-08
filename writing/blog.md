---
layout: page
title: On Software Development
permalink: /writing/blog/
breadcrumbs: /writing/
---

<div class="callout" markdown="1">
  My articles on various aspects of software development, mostly Ruby, its style, evolution, and pragmatic approaches; though from time to time I get interested in various topics like [spellcheckers](/blog/2021-05-06-how-to-spellcheck.html) or [generative art](/blog/2021-12-28-grok-shan-shui.html). Also available at [Substack](https://zverok.substack.com/).
</div>

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
