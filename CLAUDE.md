# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ltx-book** is a series of Chinese-language technical ebooks about AI video generation using **LTX-2**, the open-source DiT-based audio-video foundation model from Lightricks. Content is authored in Markdown and published via **GitHub Pages + Jekyll**.

## Repository Structure

```
ltx-book/
├── _chapters/                 # Jekyll collection: chapter files with frontmatter
├── _layouts/                  # Jekyll layouts (default, chapter, home)
├── _includes/                 # Jekyll includes (head, header, footer, chapter-nav)
├── assets/css/                # SCSS styles (CJK typography customizations)
├── books/                     # Original source chapters (excluded from Jekyll build)
├── .github/workflows/         # GitHub Actions Pages deployment
├── _config.yml                # Jekyll site configuration
├── Gemfile                    # Ruby dependencies (github-pages gem)
├── index.md                   # Home landing page
├── books.md                   # Books index (series overview with chapter listing)
└── .gitignore                 # Standard Jekyll/Pages gitignore
```

## Book Content

Each book lives in its own subdirectory under `books/`. Chapters are standalone `.md` files numbered `NN-Title.md`. The `_chapters/` directory mirrors this content as a Jekyll collection with YAML frontmatter added.

### Frontmatter per chapter

```yaml
---
layout: chapter
title: "章节标题"
book: "Book Display Name"
chapter_number: N
subtitle: "可选的副标题"
permalink: /chapters/slug/
---
```

The `book` field groups chapters into book series. Both `index.md` and `books.md` use Liquid's `group_by: "book"` to auto-organize chapters per book.

### Adding a new book

1. Create `books/New Book/` with chapter `.md` files
2. Create matching chapter files in `_chapters/` with frontmatter (`book: "New Book"`)
3. The books index and home page auto-detect the new book via `group_by`

### Adding a new chapter

1. Write the chapter in `books/Book Name/NN-Title.md`
2. Copy to `_chapters/NN-Title.md` with frontmatter added
3. Update both locations when editing

## Writing Conventions

- All content is in Chinese with English technical terms preserved inline
- Code blocks use `bash`, `python`, and `yaml` language tags
- Pipeline and command references should be accurate (verify against the `LTX-2/` git submodule)
- Chapter 00 is the foreword (not listed in per-book previews on the home page)

## GitHub Pages

The site is built with **Jekyll + minima theme** and deployed via GitHub Actions from `main`.

- **Theme**: minima (official GitHub Pages theme)
- **Fonts**: Noto Sans SC (body), Noto Sans Mono (code)
- **CJK typography**: paragraph indent (2em), justified text, line-height 1.9
- **Navigation**: chapter-nav uses Liquid to compute prev/next from `site.chapters` sorted by `chapter_number`
- **Excluded from build**: `books/`, `LTX-2/`, `CLAUDE.md`, `.claude/`

The `jekyll-relative-links` plugin converts `.md` links in chapter content to correct output URLs, so existing "下一章" links at the bottom of each chapter work automatically.

### Local preview

```bash
bundle install
bundle exec jekyll serve
```

## LTX-2 Submodule

A git submodule at `LTX-2/` tracks the upstream [Lightricks/LTX-2](https://github.com/Lightricks/LTX-2) repository. It is excluded from Jekyll builds and CI checkout (`submodules: false`). The submodule serves as a reference for verifying pipeline names, commands, and API usage in book content. Its own CLAUDE.md at `LTX-2/packages/ltx-trainer/AGENTS.md` is authoritative for training internals.
