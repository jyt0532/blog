---
layout: page
title: Search
---

<div id="search-searchbar"></div>

<div class="post-list" id="search-hits">
{% for post in site.posts %}
<div class="post-item">

<h2>
<a class="post-link" href="{{ post.url }}">
{{ post.title | escape }}
</a>
</h2>

{% assign date_format = site.minima.date_format | default: "%b %-d, %Y" %}
<span class="post-date">{{ post.date | date: date_format }}</span>

<div class="post-snippet">{{ post.excerpt }}</div>
</div>
{% endfor %}
</div>

{% include algolia.html %}
