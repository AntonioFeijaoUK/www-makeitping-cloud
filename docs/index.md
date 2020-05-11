---
layout: default
title: Learning by doing! Hands on AWS exercises with Antonio AWS
---

Learning by doing! Hands on AWS exercises with [Antonio AWS](https://www.AntonioAWS.com)

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
