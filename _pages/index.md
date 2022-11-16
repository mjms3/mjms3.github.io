---
layout: defaults/page
permalink: index.html
narrow: true
title: Welcome to Max's Site
---

This site covers things that are of interest to me:
- Coding
- Climbing
- Cooking

Apparently, I like things that begin with 'C'.

### Recent Posts

{% for post in site.posts limit:3 %}
{% include components/post-card.html %}
{% endfor %}


