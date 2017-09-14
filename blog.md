---
layout: page
title: blog
permalink: /blog/
---

#### Coding tutorials
<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">
        {{ post.title }}
      </a>[  {% for cat in post.categories %}
          {{cat}} {%if forloop.last==false %} > {% endif %}
        {% endfor %}]
      - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time>
    </li>
  {% endfor %}
</ul>
