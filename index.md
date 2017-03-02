---
layout: default
---


###### [](#header-6)Header 6

| head1        | head two          | three |
|:-------------|:------------------|:------|
| ok           | good swedish fish | nice  |
| out of stock | good and plenty   | nice  |
| ok           | good `oreos`      | hmm   |
| ok           | good `zoute` drop | yumm  |

### 这是一条水平线

---

![](https://assets-cdn.github.com/images/icons/emoji/octocat.png)

### Large image

![](https://guides.github.com/activities/hello-world/branching.png)

- /assets/image/ps001.jpg


![](assets/image/ps001.jpg)



### Definition lists can be used with HTML syntax.

## 测试

{% for post in site.posts %}
	{{ post.url }} {{ post.title }}
	{{ post.excerpt | remove: 'test' }}
{% endfor %}

## posts

{% for post in site.posts %}
 - {{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
{% endfor %}

## blog

{% for blog in site.blogs %}
 - {{ blog.date | date_to_string }} <a href="{{ site.baseurl }}{{ blog.url }}">{{ blog.title }}</a>
{% endfor %}

## 心情


