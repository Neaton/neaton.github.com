---
layout: page
title: ArchFan
tagline: 关注高性能网站架构
---
{% include JB/setup %}

### Blog Posts

<ul class="posts">
  {% for post in site.posts %}
    <li><span>{{ post.date | date_to_string }}</span> &raquo; <a href="{{ BASE_PATH }}{{ post.url }}">{{ post.title }}</a></li>
  {% endfor %}
</ul>

### Projects

[Erlang入门指南](http://learnerlang.archfan.com):Learn you some Erlang for Great Good 中文版


