---
layout: default
title: Home
---
<h2>About</h2>
<p> this is about me </p>

<h2>Papers</h2>
{% for post in site.posts %}
   [ {{ post.title }} ]({{ post.url }}) ({{ post.date | date_to_string }})
{% endfor %}

<h2>Open source</h2>
[lxc-java](https://github.com/waseemh/lxc-java) - Java API for LXC (Linux Containers)

[webdriver-assert](https://github.com/waseemh/webdriver-assert) - JUnit assertions library for Selenium WebDriver 2.0