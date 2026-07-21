---
layout: page
title: Tags
permalink: /tags/
---

<div class="tag-cloud">
{% assign tags = site.tags | sort %}
{% for tag in tags %}
  {% assign pct = tag[1].size | times: 8 | plus: 85 | at_most: 250 %}
  <a style="font-size: {{ pct }}%" href="{{ tag[0] | slugify | prepend: '/tag/' | append: '/' | relative_url }}">{{ tag[0] }}</a>
{% endfor %}
</div>
