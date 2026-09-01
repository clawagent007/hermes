---
layout: home
title: Hermes Agent 설치 가이드
---

F-Droid Termux에서 Python 3.13과 uv를 사용해 Hermes Agent를 설치하고 검증하는 재현 가능한 절차입니다.

## 글 목록

{% for post in site.posts %}
- [{{ post.title }}]({{ post.url | relative_url }}) — {{ post.date | date: "%Y-%m-%d" }}
{% endfor %}
