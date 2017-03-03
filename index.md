---
layout: default
title: 日月言己
category: index
---

# 博客列表

{% for post in site.posts %}
 [{{ post.title }}]({{ site.baseurl }}{{ post.url }})——{{ post.date | date_to_string }}
  {{ post.excerpt}}

---

{% endfor %}



![](img/other/mao.gif)


