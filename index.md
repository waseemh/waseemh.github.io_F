---
layout: default
title: Home
---
<h3>About</h3>
<p> this is about me </p>

<h3>Papers</h3>
{% for post in site.posts %}
  * {{ post.date | date_to_string }} &raquo; [ {{ post.title }} ]({{ post.url }})
{% endfor %}

<h3>Open source</h3>
##[lxc-java](https://github.com/waseemh/lxc-java)
Java API for LXC (Linux Containers).

##[webdriver-assert](https://github.com/waseemh/webdriver-assert)
Assertions library for Selenium WebDriver 2.0, based on JUnit.