---
layout: main
title: Blog Archive
aside: Shocking headline to generate interest!
---

{% for post in site.posts %}
* <a href="{{post.url}}">{{post.title}}</a>
{% if post.aside %}{{ post.aside }}{% endif%}
{% endfor %}