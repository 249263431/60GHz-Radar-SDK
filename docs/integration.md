---
layout: default
title: Integrations
permalink: /integration/
description: Connect your 60GHz mmWave radar to Home Assistant, MQTT brokers, and dashboards.
---

# Integrations

<p><a href="{{ '/archive.html' | relative_url }}">← All articles</a></p>

<ul class="post-list">
{% for post in site.categories.integration %}
  <li>
    <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a><span class="post-date">{{ post.date | date: "%Y-%m-%d" }}</span>
    <p>{{ post.excerpt | strip_html | truncate: 160 }}</p>
  </li>
{% endfor %}
</ul>
