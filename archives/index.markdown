---
layout: main
title: Blog Archive
---

<ul>{% for post in site.posts %}
<li><a href="{{post.url}}">{{post.title}}</a>
{% if post.aside %}<br>{{ post.aside }}{% endif%}</li>
{% endfor %}</ul>