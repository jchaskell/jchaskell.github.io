---
layout: page
title: Posts
---



{% for post in site.posts %}
	<a href="{{ post.url }}">{{ post.title }}</a><abbr{{ post.date | date_to_string }}</abbr>
{% endfor %}