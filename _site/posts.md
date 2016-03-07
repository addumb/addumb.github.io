---
layout: default
title: Addumb.com posts
---
Here are like all of my shitty blog posts:
{% for post in site.posts %}
 * [{{ post.title }}]({{ post.url }}) posted on {{post.date | date_to_long_string}}
{% endfor %}
