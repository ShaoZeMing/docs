---
layout: default
title: 日月言己
category: blog
---

###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |


![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)



![](assets/image/ps001.jpg)



### Definition lists can be used with HTML syntax.

## 测试

{% for post in site.md_pages %}
-  {{ post.url }} {{ post.title }}
{{ post.excerpt | remove: 'test' }}
{% endfor %}

## moods

{% for mood in site.pages %}
### [{{ mood.title }}]({{ mood.url }})   
  *{{ mood.excerpt | remove: 'test' }}*
{% endfor %}






