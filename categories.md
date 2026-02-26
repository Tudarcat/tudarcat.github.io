---
layout: page
title: 分类
---
{% assign categories = site.categories | sort %}
<div class="categories">
  {% for category in categories %}
    <h2 id="{{ category[0] | slugify }}">{{ category[0] }}</h2>
    <ul class="post-list">
      {% for post in category[1] %}
        <li>
          <span class="post-meta">{{ post.date | date: "%Y-%m-%d" }}</span>
          <a href="{{ post.url | relative_url }}">{{ post.title | escape }}</a>
        </li>
      {% endfor %}
    </ul>
  {% endfor %}
</div>