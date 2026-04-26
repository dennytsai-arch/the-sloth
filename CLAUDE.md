# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Hugo static blog called "The Sloth" (慢速前進的懶惰書店), deployed to GitHub Pages at `https://dennytsai-arch.github.io/the-sloth/`. Posts are written in Traditional Chinese. The repo doubles as an Obsidian vault for drafting.

## Commands

```bash
# Local dev server (includes draft posts)
hugo server -D

# Build for production
hugo --minify

# Build with explicit baseURL (matches what CI does)
hugo --minify --baseURL "https://dennytsai-arch.github.io/the-sloth/"
```

Deployment is fully automated: pushing to `main` triggers `.github/workflows/hugo.yml`, which builds and deploys to GitHub Pages. The `public/` directory is gitignored — never commit it.

## Project structure

- **`content/posts/`** — blog posts as Markdown files (Chinese filenames are fine)
- **`static/images/`** — images served at `/images/`; this is where cover images live
- **`layouts/partials/`** — overrides of PaperMod partials: `cover.html` and `home_info.html`
- **`assets/css/extended/custom.css`** — all custom styling; PaperMod auto-merges this
- **`themes/PaperMod/`** — git submodule; do not edit directly

## Post frontmatter template

```yaml
---
title: Title
date: 2026-04-19
draft: true
slug: english-slug-here
tags:
cover:
  image: /images/filename.png
  alt: Description
  hiddenInList: false
---
```

## Slug rule

Hugo derives the URL from the filename. Chinese filenames produce percent-encoded URLs (e.g. `/posts/ai%E4%BA%BA%E6%89%8D.../`) that are ugly and fragile. **Always add a `slug` field whenever the post title contains any Chinese characters.** The slug must be lowercase ASCII with hyphens — a short English translation of the title is ideal (e.g. `slug: from-taste-to-personality`). Never omit the slug for Chinese-titled posts, even for drafts, so it is set before first publish.

## Image path gotcha

The site lives at a subdirectory (`/the-sloth/`). Hugo's `relURL` resolves cover image paths correctly at build time, so always write cover paths as `/images/filename` — not as full URLs and not with `/the-sloth/` prepended.

Exception: `layouts/partials/home_info.html` hardcodes the hero banner path as `/the-sloth/images/hero.png` because it bypasses Hugo's URL processing. If the `baseURL` subdirectory ever changes, update that partial manually.

## Customisation points

- **Theme config**: `hugo.toml` — PaperMod params, menu, outputs
- **Styling**: `assets/css/extended/custom.css` — Inter font, light/dark CSS variables, card grid layout, post page layout
- **Home page**: `layouts/partials/home_info.html` — hero banner + Facebook follow button
- **Cover images**: `layouts/partials/cover.html` — responsive srcset logic with fallback for external URLs
