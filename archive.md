---
layout: page
title: Blog
---

## Posts

{% for post in site.posts %}
	<li><span></span></li>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ post.url }}">{{ post.title }}</a></li>
{% endfor %}