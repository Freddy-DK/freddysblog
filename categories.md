---
layout: page
title: Categories
permalink: /categories/
---

{% assign cats = site.categories | sort %}
<ul>
{% for cat in cats %}
  <li>
    <a href="{{ cat[0] | slugify | prepend: '/category/' | append: '/' | relative_url }}">{{ cat[0] }}</a>
    ({{ cat[1] | size }})
  </li>
{% endfor %}
</ul>
