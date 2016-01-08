---
layout: page
title: Archive
permalink: /archive/
---

<h3>Posts &bull; <a href="/archive_cat/">Categories</a> &bull; <a href="/archive_tag/">Tags</a></h3>

{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

