---
layout: default
---

# 임시 블로그 홈페이지

<ul>
  {% for post in site.posts %}
    <li>
      <a href="{{ post.url }}">{{ post.title }}</a> | {{ post.date | YYYY-MM-DD }}
    </li>
  {% endfor %}
</ul>