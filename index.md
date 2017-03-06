---
layout: default
title: 日月言己
category: index
---
# 先来只宠物

![](img/other/mao.gif)

# 博客列表

{% for post in site.posts %}
 [{{ post.title }}]({{ site.baseurl }}{{ post.url }})——{{ post.date | date_to_string }}
  {{ post.excerpt}}

---

{% endfor %}




