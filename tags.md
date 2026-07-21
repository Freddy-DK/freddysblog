---
layout: page
title: Tags
permalink: /tags/
---

{% assign tags = site.tags | sort %}
<ul>
  {% for tag in tags %}
    <li>
      <a href="{{ tag[0] | slugify | prepend: '/tag/' | append: '/' | relative_url }}">{{ tag[0] }}</a>
      ({{ tag[1] | size }})
    </li>
  {% endfor %}
</ul>
