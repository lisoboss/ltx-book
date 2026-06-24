---
layout: home
title: "LTX-2 AI 动漫创作指南"
---

{{ site.description }}

---

{% assign books = site.chapters | group_by: "book" %}

{% for book in books %}
{% assign book_chapters = book.items | sort: "chapter_number" %}
{% assign total = book_chapters | size %}

## {{ book.name }}

共 {{ total }} 章 · [查看全部]({{ '/books/' | relative_url }})

{% assign first_few = book_chapters | slice: 0, 6 %}
{% for chapter in first_few %}
{% unless chapter.chapter_number == 0 %}
- [{{ chapter.title }}]({{ chapter.url | relative_url }})
{% endunless %}{% endfor %}
{% if total > 6 %}... 等 {{ total | minus: 1 }} 章{% endif %}

{% endfor %}
