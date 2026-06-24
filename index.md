---
layout: home
title: "LTX-2 技术指南"
---

{{ site.description }}

---

{% assign ai_anime = site["ai-anime"] | sort: "chapter_number" %}
{% assign total_ai = ai_anime | size %}

## AI 动漫

从零开始的 AI 动漫创作完整教程。共 {{ total_ai }} 章 · [查看完整目录]({{ '/books/' | relative_url }}#ai-动漫)

{% for chapter in ai_anime limit: 6 %}
{% unless chapter.chapter_number == 0 %}
- [{{ chapter.title }}]({{ chapter.url | relative_url }})
{% endunless %}{% endfor %}
{% if total_ai > 6 %}... 等 {{ total_ai | minus: 1 }} 章{% endif %}

---

{% assign ltx_sdk = site["ltx-sdk"] | sort: "chapter_number" %}
{% assign total_sdk = ltx_sdk | size %}

## LTX-2 SDK 文档

LTX-2 Python SDK 技术参考与使用指南。共 {{ total_sdk }} 章 · [查看完整目录]({{ '/books/' | relative_url }}#ltx-2-sdk-文档)

{% for chapter in ltx_sdk limit: 6 %}
{% unless chapter.chapter_number == 0 %}
- [{{ chapter.title }}]({{ chapter.url | relative_url }})
{% endunless %}{% endfor %}
{% if total_sdk > 6 %}... 等 {{ total_sdk | minus: 1 }} 章{% endif %}
