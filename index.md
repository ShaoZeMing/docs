---
layout: default
---

{% for category in site.categories %}
<h2>{{ category | first }}</h2>
</span>{{ category | last | size }}</span>
<ul class="arc-list">
    {% for post in category.last %}
        <li>{{ post.date | date:"%d/%m/%Y"}}<a href="{{ post.url }}">{{ post.title }}</a></li>
    {% endfor %}
</ul>
{% endfor %}



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

{% for post in site.md_pages %}
-  {{ post.url }} {{ post.title }}
{{ post.excerpt | remove: 'test' }}
{% endfor %}

## moods

{% for mood in site.html_pages %}
-   {{ mood.url }} {{ mood.title }}
{{ mood.excerpt | remove: 'test' }}
{% endfor %}


## posts

{% for post in site.posts %}
 - {{ post.date | date_to_string }} <a href="{{ site.baseurl }}{{ post.url }}">{{ post.title }}</a>
{% endfor %}

## blog

{% for blog in site.site.blogs_data%}
 - {{ blog.date | date_to_string }} <a href="{{ site.baseurl }}{{ blog.url }}">{{ blog.title }}</a>
{% endfor %}

## 心情


