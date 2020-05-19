---
layout: default
title: Learning by doing! Hands on AWS exercises with Antonio AWS
---

{% avatar AntonioFeijaoUK size=100 %}

Learning by doing! Hands on AWS exercises with [Antonio AWS](https://www.AntonioAWS.com)

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
