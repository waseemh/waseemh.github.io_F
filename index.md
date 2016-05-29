---
layout: default
title: Home
---

{% for post in site.posts %}
   <h3> <a href="{{ post.url }}">{{ post.title }}</a> </h3> 
   <span style="display: inline" class="post-date">{{ post.date | date_to_string }}</span>
{% endfor %}

<h2>Open Source Projects</h2>
[lxc-java](https://github.com/waseemh/lxc-java) - Java API for LXC (Linux Containers)

[webdriver-assert](https://github.com/waseemh/webdriver-assert) - JUnit assertions library for Selenium WebDriver 2.0
