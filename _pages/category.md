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

<ul>
    //这里使用了 jekyll 语法，会被编译，所以加多个"\"
    {% for category in site.categories %}
    <li><a href="/categories/{{ category | first }}/" title="view all
posts">{{ category | first }} {{ category | last | size }}</a>
    </li>
    {% endfor %}
</ul>