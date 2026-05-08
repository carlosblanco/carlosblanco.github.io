# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
bundle install              # Install Ruby dependencies
bundle exec jekyll serve    # Start local dev server at http://localhost:4000
bundle exec jekyll build    # Build static site into _site/
```

## Architecture

This is a Jekyll blog hosted on GitHub Pages. Key configuration lives in `_config.yml` (site metadata, theme, plugins). The theme is Minima 2.5.1 with custom overrides.

**Content**: Blog posts go in `_posts/` as Markdown files named `YYYY-MM-DD-title.md`. Supported frontmatter fields:

```yaml
layout: article
title: "Post Title"
date: 2024-01-01
author: Carlos Blanco
categories: [category]
tags: [tag1, tag2]
original_site: "Medium"        # optional: for cross-posts
original_url: "https://..."    # optional: source URL for cross-posts
canonical_url: "https://..."   # optional: canonical URL for SEO
```

**Layouts & Includes**: `_layouts/article.html` extends Minima's default layout and adds support for `original_site`/`original_url` frontmatter (shows attribution for cross-posted articles). `_includes/header.html` overrides Minima's header to add the site logo.

**Styles**: `assets/main.scss` imports Minima then adds custom rules — sticky glassmorphism header, Source Sans 3 typography, and logo sizing.

## Deployment

Pushing to `master` triggers automatic deployment via GitHub Pages. No CI configuration is needed. The `_site/` directory is generated locally (gitignored) and rebuilt by GitHub Pages on push.
