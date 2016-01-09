---
layout: page
title: Archive
permalink: /archive_tag/
---

<h3><a href="/archive/">Posts</a> &bull; <a href="/archive_cat/">Categories</a> &bull; Tags</h3>

{% assign tags = site.tags | sort %}
{% for tag in tags %}
 <p id="site-tag">
    <a id="tag-link" href="/blog/tag/{{ tag | first | slugify }}/"
        style="font-size: {{ tag | last | size  |  times: 4 | plus: 90  }}%">
            {{ tag[0] | replace:'-', ' ' }} ({{ tag | last | size }})
    </a>
</p>
{% endfor %}