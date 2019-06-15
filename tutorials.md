---
layout: page
title: blog
category: blog
permalink: /blog/
---

#### Dev Tips
<ul>
  {% for post in site.categories.devtips %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### Computer Science
<ul>
  {% for post in site.categories.compsci %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### ES6 Morsels
<ul>
  {% for post in site.categories.ES6 %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### JS Fundamentals
<ul>
  {% for post in site.categories.fundamentals %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### Basic Algos
<ul>
  {% for post in site.categories.algorithms %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### Fun Projects
<ul>
  {% for post in site.categories.projects %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### Collaborative Magnetic Poetry (Attractio)
<ul>
  {% for post in site.categories.attractio %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a></li>
  {% endfor %}
</ul>

#### Cryptopals 1-14
<ul>
  {% for post in site.categories.cryptopals %}
    <li><a href="{{ site.baseurl }}{{ post.url }}">{{post.title}}</a> </li>
  {% endfor %}
</ul>
