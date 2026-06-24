---
layout: home
title: "LTX-2 技术指南"
---

{{ site.description }}

---

{% assign book = site["ai-anime"] | sort: "chapter_number" %}
{% assign total = book | size %}

## AI 动漫

共 {{ total }} 章 · [查看完整目录]({{ '/books/' | relative_url }})

{% for chapter in book limit: 7 %}
{% unless chapter.chapter_number == 0 %}
- [{{ chapter.title }}]({{ chapter.url | relative_url }})
{% endunless %}{% endfor %}
{% if total > 7 %}... 等 {{ total | minus: 1 }} 章{% endif %}
