---
layout: default
title: tags
permalink: /tags/
---

{% for tags in site.tags %}
  <h3>{{ tags[0] }}</h3>
  <ul>
    {% for post in tags[1] %}
      <li><a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
  </ul>
{% endfor %}
