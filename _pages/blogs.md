---
title:  "Blogs"
layout: archive
permalink: /blogs/
author_profile: true
comments: false
---

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>


