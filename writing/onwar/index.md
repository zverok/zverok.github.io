---
layout: page
title: On War
permalink: /writing/onwar/
breadcrumbs: /writing/
---

<div class="callout" markdown="1">
  A set of texts—mostly long Twitter threads put together and edited for ease of reading—that I am writing during the Russia’s full-scale invasion of Ukraine since 2022. Mourning friends, personal stories, and some political/sociological analysis on Russian and Western ways of about the genocidal invasion.
</div>

{% assign posts = site.onwar | reverse %}
{% for post in posts %}
<div class="post postContent">
  <div  class="postDate"><time datetime="{{ post.date | date_to_xmlschema }}" itemprop="datePublished">{{ post.date | date: "%b %-d, %Y" }}</time>
  </div>
  <br>
  <div class="postTitle">
  <a class='postLink' href="{{site.url}}{{site.baseurl}}{{post.url}}">{{post.title}}</a>
  </div>
  <div class="postExt">
    {{post.description}}
  </div>
</div>

{% endfor %}
