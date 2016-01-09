---
layout: page
title: Archive
permalink: /archive_cat/
---

<h3><a href="/archive/">Posts</a> &bull; Categories &bull; <a href="/archive_tag/">Tags</a></h3>

{% assign categories = site.categories | sort %}
{% for cat in categories %}
 <p id="site-tag">
    <a id="tag-link" href="/blog/category/{{ cat | first | slugify }}/"
        style="font-size: {{ cat | last | size  |  times: 4 | plus: 90  }}%">
            {{ cat[0] | replace:'-', ' ' }} ({{ cat | last | size }})
    </a>
</p>
{% endfor %}