---
layout: default
title: Blog
permalink: /blog/
---

<ul>
  {% for post in site.posts %}
    <li>
      {{ post.date | date: "%d %b, %Y" }} – <a href="{{ post.url | relative_url }}">{{ post.title }}</a>
    </li>
  {% endfor %}
</ul>
