---
layout: default
title: Fundamentals
permalink: /fundamentals/
description: How mmWave radar works — concepts, comparisons, and the physics behind the product.
---

# Fundamentals

<p><a href="{{ '/archive.html' | relative_url }}">← All articles</a></p>

<ul class="post-list">
{% for post in site.categories.fundamentals %}
  <li>
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
  </li>
{% endfor %}
</ul>
