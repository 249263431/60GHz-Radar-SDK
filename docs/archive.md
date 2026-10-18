---
layout: listing
title: Articles
permalink: /archive.html
description: All articles published on the 60GHz mmWave Radar SDK site, grouped by category.
---

# Articles

Browse every article we've published, grouped by category. You can also [subscribe via RSS]({{ '/feed.xml' | relative_url }}).

{% assign cats = "fundamentals,product,guide,integration" | split: "," %}
{% assign names = "Fundamentals,Products,Guides,Integrations" | split: "," %}
{% for cat in cats %}
{% assign posts = site.categories[cat] %}
{% if posts.size > 0 %}

## {{ names[forloop.index0] }} <small>({{ posts.size }})</small>

<ul class="post-list">
{% for post in posts %}
  <li class="post-item" data-date="{{ post.date | date: '%Y-%m' }}" data-category="{{ post.categories | join: ' ' }}">
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
  </li>
{% endfor %}
</ul>
{% endif %}
{% endfor %}
