---
layout: default
title: 系列书目
permalink: /books/
---

<div class="books-index">

{{ site.description }}

{% assign books = site.chapters | group_by: "book" %}

{% for book in books %}
{% assign book_chapters = book.items | sort: "chapter_number" %}

## {{ book.name }}

| 章节 | 内容 |
|------|------|
{% for chapter in book_chapters %}| [{{ chapter.title }}]({{ chapter.url | relative_url }}) | {% if chapter.subtitle %}{{ chapter.subtitle }}{% endif %} |
{% endfor %}

{% endfor %}

</div>
