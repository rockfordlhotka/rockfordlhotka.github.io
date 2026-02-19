# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Jekyll-powered technical blog for Rockford Lhotka, hosted on GitHub Pages at `blog.lhotka.net`. Content focuses on CSLA .NET, Blazor, AI/LLMs, MCP, and software architecture.

## Build & Deploy

There is no local build system (no Gemfile, package.json, or Rakefile). GitHub Pages automatically builds the site with Jekyll on push to `main`.

To serve locally (requires Ruby + Jekyll installed):
```bash
jekyll serve
# Site at http://localhost:4000
```

## Architecture

- **Jekyll static site** using `jekyll-theme-slate` with kramdown markdown processor
- **Layouts**: `_layouts/default.html` (base with nav, GA, footer) → `_layouts/post.html` (title + date wrapper)
- **Includes**: `_includes/post-featured-image.html`, `_includes/disqus.html`, `_includes/adsense.html`
- **Permalink pattern**: `/:year/:month/:day/:title` (configured in `_config.yml`)
- **Plugins**: `jekyll-sitemap`, `jekyll-seo-tag`
- **RSS feed**: `atom.xml` (Atom format with MediaRSS for featured images)

## Creating a New Blog Post

**File naming**: `_posts/YYYY-MM-DD-Title-With-Hyphens.md`

**Front matter template**:
```yaml
---
layout: post
title: Post Title Here
postDate: 2026-01-11T18:02:53-06:00
categories: []
tags: []
published: true
permalink:
image: /assets/2026-01-11-Post-Title/featured-image.png
---
```

- `postDate` uses ISO 8601 with timezone offset
- `image` is optional; used as the featured image in the RSS feed and rendered via `post-featured-image.html`
- `categories` and `tags` are kept as empty arrays

## Image Conventions

- Store images in `/assets/YYYY-MM-DD-Post-Title/` (one directory per post, matching the post filename)
- Reference in markdown: `![alt text](/assets/YYYY-MM-DD-Post-Title/image-name.png)`
- Featured image goes in the `image:` front matter field
