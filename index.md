---
layout: default
title: Home
---

{% for post in site.posts %}
   <h3> <a href="{{ post.url }}">{{ post.title }}</a> </h3> 
        <div class="entry">
        {{ post.excerpt }}
      </div>
   <span class="post-date">({{ post.date | date_to_string }})</span>
{% endfor %}
