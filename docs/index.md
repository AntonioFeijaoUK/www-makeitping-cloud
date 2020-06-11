---
layout: default
title: Learning by doing! Hands-on networking workshops on AWS with makeitping.cloud - Make it ping!
---

Learning by doing! Hands-on networking workshops on [AWS](https://aws.amazon.com) .

**Disclaimer**: [Makeitping.cloud](https://www.makeitping.cloud) is a personal website maintained by [Antonio Feijao UK](https://www.antoniocloud.com). The opinions expressed are my own and do not necessarily represent those of current or past employers. The content in this website is intended for educational use. Use at Your Own Risk Own.


{% avatar AntonioFeijaoUK size=100 %}

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
