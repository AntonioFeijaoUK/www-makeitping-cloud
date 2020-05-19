---
layout: default
title: Learning by doing! Hands-on AWS exercises with Antonio Feijao UK at AntonioAWS.com
---

Learning by doing! Hands-on [AWS](https://aws.amazon.com) exercises with [Antonio Feijao UK](https://www.antoniocloud.com) on this personal website [www.AntonioAWS.com](https://www.antonioaws.com)

{% avatar AntonioFeijaoUK size=500 %}

---

## Posts 

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
