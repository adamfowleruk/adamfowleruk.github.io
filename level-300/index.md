---
layout: page
title:  Level 300 Introduction
categories: main
---

# What is Level 300?

Level 300 posts are a detailed project series
linking together concepts from level 200 posts to work
on a real-world (or as close as I can make it!) application.

## Current projects

<ul>
{% for related in site.categories["level-300"] %}
  <li><a href="{{related.permalink}}">{{ related.title }}</a></li>
{% endfor %}
</ul>