---
layout: default
title: Learning by doing! AWS Hands-on networking workshops with makeitping.cloud - Make it ping!
---

Learning by doing! Hands-on [AWS](https://aws.amazon.com) networking workshops.

**Disclaimer**: The opinions expressed are my own and do not necessarily represent those of current or past employers. The content in this website is intended for educational use. Use at Your Own Risk Own.

[Makeitping.cloud](https://www.makeitping.cloud) is a personal website maintained by [Antonio Feijao UK](https://www.antoniocloud.com). 

{% avatar AntonioFeijaoUK size=200 %}

---

## AWS networking exercises workshops

<ul>
  {% for post in site.posts %}
    <li>
      <h2><a href="{{ post.url }}">{{ post.title }}</a></h2>
      {{ post.excerpt }}
    </li>
  {% endfor %}
</ul>
