---
layout: default
title: 系列书目
permalink: /books/
---

{{ site.description }}

---

{% assign ai_anime = site["ai-anime"] | sort: "chapter_number" %}

## AI 动漫

从零开始的 AI 动漫创作完整教程。

| 章节 | 内容 |
|------|------|
{% for chapter in ai_anime %}| [{{ chapter.title }}]({{ chapter.url | relative_url }}) | {% if chapter.subtitle %}{{ chapter.subtitle }}{% endif %} |
{% endfor %}

---

{% assign ltx_sdk = site["ltx-sdk"] | sort: "chapter_number" %}

## LTX-2 SDK 文档

LTX-2 Python SDK（ltx-core、ltx-pipelines、ltx-trainer）技术参考与使用指南。

| 章节 | 内容 |
|------|------|
{% for chapter in ltx_sdk %}| [{{ chapter.title }}]({{ chapter.url | relative_url }}) | {% if chapter.subtitle %}{{ chapter.subtitle }}{% endif %} |
{% endfor %}
