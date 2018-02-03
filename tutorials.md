---
layout: page
title: tutorials
category: tutorials
permalink: /blog/
---

#### Computer Science
<ul>
  {% for post in site.categories.compsci %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></li>
  {% endfor %}
</ul>

#### JS Fundamentals
<ul>
  {% for post in site.categories.fundamentals %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></li>
  {% endfor %}
</ul>

#### Basic Algos
<ul>
  {% for post in site.categories.algorithms %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></li>
  {% endfor %}
</ul>

#### Fun Projects
<ul>
  {% for post in site.categories.projects %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></li>
  {% endfor %}
</ul>

#### Collaborative Magnetic Poetry (Attractio)
<ul>
  {% for post in site.categories.attractio %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></li>
  {% endfor %}
</ul>

#### Cryptopals 1-14
<ul>
  {% for post in site.categories.cryptopals %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> - <time datetime="{{ post.date | date: "%Y-%m-%d" }}">{{ post.date | date_to_long_string }}</time></li>
  {% endfor %}
</ul>
