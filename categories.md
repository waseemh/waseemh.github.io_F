---
layout: page
title: Categories
---

{% for category in site.categories %}
  <h2><a name="{{ category | first }}">{{ category | first }}</h2>
    <ul>
    {% for posts in category %}
      {% for post in posts %}
        <li><a href="{{ post.url }}">{{ post.title }}</a></li>
      {% endfor %}
    {% endfor %}
    </ul>
{% endfor %}
