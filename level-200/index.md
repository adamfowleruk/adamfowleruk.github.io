---
layout: page
title:  Level 200 Introduction
categories: main
---

# What is Level 200?

Level 200 posts show how to do a useful thing in isolation.
They're ideal for refreshing your brain or doing something
for the first time.

I don't do 'hello world' style posts, so these will
actually mention real-world issues of using a particular
approach, and mention best practice rather than just
'get it working' style instructions.

## Current pages

<ul>
{% for related in site.categories["level-200"] %}
  <li><a href="{{related.permalink}}">{{ related.title }}</a></li>
{% endfor %}
</ul>