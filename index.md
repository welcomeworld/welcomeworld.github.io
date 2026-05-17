---
title: 首页
---

# 欢迎来到我的博客

这里会分享技术、生活和思考。

## 最近更新
{% for post in site.posts limit:5 %}
- [{{ post.title }}]({{ post.url }}) - {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
