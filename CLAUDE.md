# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Hugo static blog called "The Sloth" (緩慢前進的懶惰書店), deployed to GitHub Pages at `https://dennytsai-arch.github.io/the-sloth/`. Posts are written in Traditional Chinese. The repo doubles as an Obsidian vault for drafting.

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

- **`content/posts/`** — blog posts as Markdown files (Chinese filenames are fine); `Template.md` here is an Obsidian drafting template (`draft: true`), never published
- **`static/images/`** — images served at `/images/`; this is where cover images live
- **`static/icons/`** — SVG icons for the four category nav items
- **`layouts/partials/`** — overrides of PaperMod partials: `cover.html` and `home_info.html`
- **`layouts/_default/single.html`** — full single-post layout override (not just a partial); adds category badge above title and strips Obsidian wikilinks from body at render time
- **`layouts/_default/list.html`** — full list/home layout override; adds category badge to post cards
- **`assets/css/extended/custom.css`** — all custom styling; PaperMod auto-merges this
- **`themes/PaperMod/`** — git submodule; do not edit directly
- **`design.md`** — Obsidian design note at the repo root; Hugo ignores it (not inside `content/`)

## Post frontmatter template

```yaml
---
title: Title
date: 2026-04-19
draft: true
slug: english-slug-here
tags:
categories:
  - Review
cover:
  image: /images/filename.png
  alt: Description
  hiddenInList: false
---
```

Valid `categories` values (single-select, drives the nav menu): `Book`, `Exhibition`, `Review`, `Design`.

## Slug rule

Hugo derives the URL from the filename. Chinese filenames produce percent-encoded URLs (e.g. `/posts/ai%E4%BA%BA%E6%89%8D.../`) that are ugly and fragile. **Always add a `slug` field whenever the post title contains any Chinese characters.** The slug must be lowercase ASCII with hyphens — a short English translation of the title is ideal (e.g. `slug: from-taste-to-personality`). Never omit the slug for Chinese-titled posts, even for drafts, so it is set before first publish.

## Obsidian wikilink images

Posts drafted in Obsidian may contain `![[filename.jpg]]` inline image syntax. `layouts/_default/single.html` strips these at render time via regex, so they don't appear on the live site — but clean them from the source anyway. Post images belong in the `cover:` frontmatter block, not in the body.

## Files to never commit

- `無題のファイル.md` — Obsidian's auto-created untitled scratch file, always lives at the repo root untracked. It is **not** gitignored; never stage or commit it.
- `archetypes/default.md` — do not use this as a post template; it is outdated and lacks `slug`, `categories`, and `cover` fields. Use `content/posts/Template.md` as the reference instead.

## Image path gotcha

The site lives at a subdirectory (`/the-sloth/`). Hugo's `relURL` resolves cover image paths correctly at build time, so always write cover paths as `/images/filename` — not as full URLs and not with `/the-sloth/` prepended.

Exception: `layouts/partials/home_info.html` hardcodes the hero banner path as `/the-sloth/images/hero.png` because it bypasses Hugo's URL processing. If the `baseURL` subdirectory ever changes, update that partial manually.

## Customisation points

- **Theme config**: `hugo.toml` — PaperMod params, menu, outputs
- **Styling**: `assets/css/extended/custom.css` — Inter font, light/dark CSS variables, card grid layout, post page layout
- **Home page**: `layouts/partials/home_info.html` — hero banner + Facebook follow button
- **Cover images**: `layouts/partials/cover.html` — responsive srcset logic with fallback for external URLs
