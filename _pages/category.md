---
layout: page
title: Category
permalink: /category/
---

{% for category in site.categories %}
<ul>
    <li><a href="{{ site.baseurl}}/categories/{{ category.first }}/">{{category.first}}（{{category.last.size}}）</a></li>
</ul>
{% endfor %}
