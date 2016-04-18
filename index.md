---
layout: default
title: Home
---

<hr />

{% for post in site.posts %}
   ## {{ post.title }} ##
   <span>{{ post.date | date_to_string }}</span>
   {{ post.content }}
{% endfor %}

<h2>Open Source Projects</h2>
[lxc-java](https://github.com/waseemh/lxc-java) - Java API for LXC (Linux Containers)

[webdriver-assert](https://github.com/waseemh/webdriver-assert) - JUnit assertions library for Selenium WebDriver 2.0
