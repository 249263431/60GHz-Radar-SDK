---
layout: listing
title: Guides
permalink: /guide/
description: Step-by-step setup, installation, and configuration guides.
---

# Guides

<p><a href="{{ '/archive.html' | relative_url }}">← All articles</a></p>

<ul class="post-list">
{% for post in site.categories.guide %}
  <li class="post-item" data-date="{{ post.date | date: '%Y-%m' }}" data-category="{{ post.categories | join: ' ' }}">
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
  </li>
{% endfor %}
</ul>
