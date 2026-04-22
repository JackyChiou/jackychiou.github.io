---
layout: default
title: Blog
nav_order: 2
has_children: false
permalink: /blog/
---

# 所有文章

{% assign posts_by_year = site.posts | group_by_exp: "post", "post.date | date: '%Y'" %}
{% for year in posts_by_year %}

## {{ year.name }}

{% for post in year.items %}
- **[{{ post.title }}]({{ post.url | relative_url }})** <span class="text-grey-dk-000 fs-3">— {{ post.date | date: "%m-%d" }}</span>
  <br>{% for cat in post.categories %}<span class="label label-green">{{ cat }}</span> {% endfor %}
{% endfor %}

{% endfor %}
