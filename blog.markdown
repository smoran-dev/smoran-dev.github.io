---
layout: default
title: Blog
permalink: /blog/
---

<h1>Blog</h1>
<ul class="post-list">
{% for post in site.posts %}
  <li>
    <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    <span class="post-date">{{ post.date | date: "%b %-d, %Y" }}</span>
    <p>{{ post.excerpt | strip_html | truncate: 180 }}</p>
  </li>
{% endfor %}
</ul>
