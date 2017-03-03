---
layout: default
title: 日月言己
category: index
---

# 博客列表

{% for post in site.posts %}
{{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a> 
  {{ post.excerpt}}

---

{% endfor %}



![](img/other/mao.gif)


