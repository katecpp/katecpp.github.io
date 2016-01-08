---
layout: page
title: Archive
permalink: /archive_cat/
---

<h3><a href="/archive/">Posts</a> &bull; Categories &bull; <a href="/archive_tag/">Tags</a></h3>

<ul type="circle">
{% for category in site.data.categories %}
   <li><a id="category-link" href="/blog/category/{{ category.slug }}">{{ category.name }}</a></li>
{% endfor %}
</ul>

{% comment %}


Todo: posts by category list
## Post by category

{% for category in site.categories %}
  <li><a name="{{ category | first }}">{{ category | first }}</a>
    <ul>
    {% for posts in category %}
      {% for post in posts %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    {% endfor %}
    </ul>
  </li>
{% endfor %}

{% endcomment %}
