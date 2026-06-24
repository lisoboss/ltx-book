---
layout: default
title: 系列书目
permalink: /books/
---

<div class="books-index">

{{ site.description }}

{% assign book = site["ai-anime"] | sort: "chapter_number" %}

## AI 动漫

| 章节 | 内容 |
|------|------|
{% for chapter in book %}| [{{ chapter.title }}]({{ chapter.url | relative_url }}) | {% if chapter.subtitle %}{{ chapter.subtitle }}{% endif %} |
{% endfor %}

</div>
