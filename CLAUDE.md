# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ltx-book** is a series of Chinese-language technical ebooks about AI video generation using **LTX-2**, the open-source DiT-based audio-video foundation model from Lightricks. Content is authored in Markdown and published via **GitHub Pages + Jekyll**.

## Repository Structure

```
ltx-book/
├── _ai-anime/                 # "AI 动漫" book (collection: ai-anime)
├── _layouts/                  # Jekyll layouts (default, chapter, home)
├── _includes/                 # Jekyll includes (head, header, footer, chapter-nav)
├── assets/css/                # SCSS styles (CJK typography customizations)
├── .github/workflows/         # GitHub Actions Pages deployment
├── _config.yml                # Jekyll site configuration
├── Gemfile                    # Ruby dependencies (github-pages gem)
├── index.md                   # Home landing page
├── books.md                   # Books index (series overview with chapter listing)
└── .pre-commit-config.yaml    # Pre-commit hook: validates Jekyll build
```

## Book Content

Each book is a Jekyll collection at `_<slug>/` in the repo root. Chapters are `.md` files with YAML frontmatter.

### Chapter frontmatter

```yaml
---
layout: chapter
title: "章节标题"
chapter_number: N
subtitle: "可选的副标题"
permalink: /books/collection-name/slug/
---
```

The `_config.yml` `defaults` section auto-sets `layout: chapter` for all collection items, so the `layout` field can be omitted in individual files.

### Adding a new book

1. Create `_new-book/` with chapter `.md` files including frontmatter
2. Add the collection to `_config.yml`:
   ```yaml
   collections:
     ai-anime:
       output: true
       permalink: /books/:collection/:name/
     new-book:
       output: true
       permalink: /books/:collection/:name/
   ```
3. Add a `defaults` scope for the new collection type
4. Add the book to `index.md` and `books.md` Liquid templates (reference via `site["collection-name"]`)

### Adding a new chapter

1. Create `_collection-name/NN-Title.md` with frontmatter
2. That's it — no duplication needed. The file is both source and Jekyll content.

## Writing Conventions

- All content is in Chinese with English technical terms preserved inline
- Code blocks use `bash`, `python`, and `yaml` language tags
- Pipeline and command references should be accurate (verify against the `LTX-2/` git submodule)
- Chapter 00 is the foreword (excluded from preview on the home page via `chapter_number == 0`)

## GitHub Pages

The site is built with **Jekyll + minima theme** and deployed via GitHub Actions from `main`.

- **Collections**: Each book is a Jekyll collection at `_<slug>/` in the repo root
- **Theme**: minima (official GitHub Pages theme)
- **Fonts**: Noto Sans SC (body), Noto Sans Mono (code)
- **CJK typography**: paragraph indent (2em), justified text, line-height 1.9
- **Navigation**: chapter-nav uses Liquid to compute prev/next from collection sorted by `chapter_number`
- **Pre-commit**: validates Jekyll build on every commit

### Local preview

```bash
bundle install
bundle exec jekyll serve
```

## LTX-2 Submodule

A git submodule at `LTX-2/` tracks the upstream [Lightricks/LTX-2](https://github.com/Lightricks/LTX-2) repository. It is excluded from Jekyll builds and CI checkout (`submodules: false`). Used only as a reference for verifying pipeline names, commands, and API usage in book content.
