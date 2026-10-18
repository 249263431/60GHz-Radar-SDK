---
layout: default
title: Products
permalink: /product/
description: Product guides for fall detection, presence, and sleep monitoring radars.
---

# Products

<p><a href="{{ '/archive.html' | relative_url }}">← All articles</a></p>

<ul class="post-list">
{% for post in site.categories.product %}
  <li>
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
  </li>
{% endfor %}
</ul>
