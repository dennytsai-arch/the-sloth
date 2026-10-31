# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A Hugo static blog called "The Sloth" (緩慢前進的懶惰書店), deployed to GitHub Pages at `https://dennytsai-arch.github.io/the-sloth/`. Posts are written in Traditional Chinese. The repo doubles as an Obsidian vault for drafting.

## Commands

```bash
# First-time clone: PaperMod is a git submodule and won't be present otherwise
git submodule update --init --recursive

# Local dev server (includes draft posts)
hugo server -D

# Build for production
hugo --minify

# Build with explicit baseURL (matches what CI does)
hugo --minify --baseURL "https://dennytsai-arch.github.io/the-sloth/"
```

There are no tests or linters. A clean `hugo --minify` (no errors or warnings) is the only verification step — run it before every push, especially after changing tags, categories, or layouts.

Deployment is fully automated: pushing to `main` triggers `.github/workflows/hugo.yml`, which builds and deploys to GitHub Pages (CI passes `--baseURL` from `actions/configure-pages`, overriding `hugo.toml`'s `baseURL`). The `public/` directory is gitignored — never commit it. CI installs Hugo extended at `hugo-version: "latest"`, so it can be ahead of the local Homebrew Hugo; if CI fails but a local build passes, check for a Hugo version difference first.

`git add -A` / `git add .` is never safe here — the working tree routinely holds Obsidian leftovers (see "Files to never commit"). Always stage specific paths.

## Project structure

- **`content/posts/`** — blog posts as Markdown files (Chinese filenames are fine); `Template.md` is the drafting template (`draft: true`), never published. Organized into subfolders by category: `content/posts/books/` (category `Book`), `content/posts/design/` (category `Design`), `content/posts/exhibitions/` (category `Exhibition`). Posts with category `Post` (the generic default) stay directly at the `content/posts/` root — no `post/` subfolder, since that would be redundant with the section name itself. When creating or moving a post, put it in the subfolder matching its `categories:` value (create the subfolder if it doesn't exist yet).
- **`content/books/`** — headless book entries, currently unused (the post-page BOOKS section matches Book-category posts instead — see "BOOKS section on posts" below); `Template.md` is the drafting template; `_index.md` sets `build.render: never` / `list: never` so the section itself has no public page
- **`content/about.md`** / **`content/archives.md`** — standalone pages; edit directly if needed
- **`temp/`** — staging area for posts being prepared before moving to `content/posts/`; not gitignored, so never commit its contents unless a post is ready
- **`static/images/`** — images served at `/images/`; this is where cover images live
- **`static/images/books/`** — cover images for book-recommendation entries
- **`static/icons/`** — SVG icons for the category nav items (Exhibition's icon is unused since it was removed from the nav menu, but the `Exhibition` category and its posts still work; `review.svg` is an unused leftover with no matching category)
- **`layouts/partials/`** — overrides of PaperMod partials: `cover.html` and `home_info.html`; also `recommend-book.html` (post-page BOOKS section), `featured-posts.html` + `books-carousel.html` (home page sections), and `post_nav_links.html` (prev/next image cards)
- **`layouts/_default/single.html`** — full single-post layout override (not just a partial); adds category badge above title, strips Obsidian wikilinks from body at render time, and renders the tag-matched BOOKS section after the tags
- **`layouts/_default/list.html`** — full list/home layout override; adds category badge to post cards
- **`assets/css/extended/custom.css`** — all custom styling; PaperMod auto-merges this
- **`themes/PaperMod/`** — git submodule; do not edit directly
- **`design.md`** — Obsidian design note at the repo root; Hugo ignores it (not inside `content/`) and it is gitignored

## Post frontmatter template

```yaml
---
title: Title
date: 2026-04-19
draft: true
slug: english-slug-here
description:
tags:
categories:
  - Post
cover:
  image: /images/filename.png
  alt: Description
  hiddenInList: false
---
```

Valid `categories` values (single-select): `Book`, `Exhibition`, `Post`, `Design`. `Book`, `Post`, and `Design` each have a nav menu entry in `hugo.toml`; `Exhibition` does not (removed from the nav, but the category and its subfolder/posts still work — see `static/icons/` note above).

## Slug rule

Hugo derives the URL from the filename. Chinese filenames produce percent-encoded URLs (e.g. `/posts/ai%E4%BA%BA%E6%89%8D.../`) that are ugly and fragile. **Always add a `slug` field whenever the post title contains any Chinese characters.** The slug must be lowercase ASCII with hyphens — a short English translation of the title is ideal (e.g. `slug: from-taste-to-personality`). Never omit the slug for Chinese-titled posts, even for drafts, so it is set before first publish.

## Publishing a new post

When the user says "update and push new post", the workflow is:

1. `git status` — find the new untracked file under `content/posts/`
2. Read the post — it will have template placeholder values that need filling:
   - `title:` → use the filename (minus `.md`) as the title
   - `slug:` → short English translation with hyphens
   - `date:` → use the UTC-safe date (see UTC gotcha below); if the post date is today in Taiwan, use yesterday to be safe
   - `cover.image:` → match the `![[filename.jpg]]` wikilink in the body; that image will be untracked in `static/images/` (Obsidian sometimes drops it at the repo root instead — if so, `mv` it into `static/images/` first)
   - `cover.alt:` → brief descriptive alt text in Chinese
   - `description:` → one or two Chinese sentences (~80–120 characters) summarising the post. This becomes the meta description / search-result snippet; without it Hugo falls back to the first paragraph, which reads poorly in Google.
3. Remove the `![[filename.jpg]]` line from the body
4. `tags:` → read the post and assign 2–4 semantic tags based on its actual content. Reuse existing tags from the vocabulary below whenever one fits; only introduce a new tag when nothing in the list covers the topic. Never use a tag that matches an existing category name (`book`, `exhibition`, `post`, `design`) — Hugo's nav menu does `site.GetPage` on those exact names, and a same-named tag term makes it ambiguous and breaks the build (this happened once — see "Tag vocabulary" below).
5. Move the file into the subfolder matching its `categories:` value if it's not already there — `content/posts/books/`, `content/posts/design/`, or `content/posts/exhibitions/` (create the subfolder if it doesn't exist yet). Posts with category `Post` stay at the `content/posts/` root. Moving between subfolders never changes the live URL — see "Post subfolder / permalink gotcha" below.
6. Stage the post file and its cover image (`git add <post> static/images/<cover>`)
7. Commit and push

## Tag vocabulary

Tags drive the post-page BOOKS recommendation (see below), so consistency matters more than precision — reuse an existing tag rather than coining a near-synonym. Current vocabulary, grouped loosely by theme:

- **Tech/AI**: `ai`, `technology`, `apple`, `startup`, `accessibility`
- **Design**: `design-practice` (applied/product design — *not* `design`, which collides with the `Design` category), `design-theory`
- **Business/econ**: `brand`, `finance`, `future-of-work`, `management`, `retail`, `geopolitics`, `globalization`
- **Culture/society**: `culture`, `identity`, `family`, `sports`, `music`, `film`, `art`, `literature`
- **Place**: `japan`, `taiwan`
- **Other**: `wellness`, `sustainability`, `space`, `education`

When a post doesn't fit anything above, add a new lowercase-hyphenated English tag — but first double-check it doesn't equal `book`, `exhibition`, `post`, or `design` (the four category names), and add the new tag to this list.

## BOOKS section on posts

Every post page shows a "BOOKS" section (after the tags, before the prev/next cards) recommending one **Book-category post** whose `tags` overlap the current post's. Driven by `layouts/partials/recommend-book.html`, called from `layouts/_default/single.html`.

- **Matching logic**: compares the post's `tags` against every Book-category post's `tags` (excluding the post itself) and picks the one with the most overlap; ties go to the newest. If the post has no tags, or no Book post shares a tag, the section is omitted entirely — nothing renders.
- So a new Book post only gets recommended if its tags overlap other posts' tags — another reason to reuse tags from the vocabulary above.
- The card shows the Book post's cover, title, `description` (if set) and date. Styling: `.recommend-book*` rules in `assets/css/extended/custom.css`.
- `content/books/` (headless book entries) is **no longer used** by this section; it's a leftover from the previous "你也許會喜歡" card.

## Obsidian wikilink images

Posts drafted in Obsidian may contain `![[filename.jpg]]` inline image syntax. `layouts/_default/single.html` strips these at render time via regex, so they don't appear on the live site — but clean them from the source anyway. Post images belong in the `cover:` frontmatter block, not in the body.

## Files to never commit

- `無題のファイル.md` (and numbered variants like `無題のファイル 1.md`, `無題のファイル 2.md`) — Obsidian's auto-created untitled scratch files, always at the repo root untracked. They are **not** gitignored; never stage or commit them.
- Any other stray `.md` files at the repo root (e.g. date-named files like `2026-04-19.md`) — Obsidian sometimes creates these; never stage or commit them. Posts belong under `content/posts/`.
- `archetypes/default.md` — do not use this as a post template; it is outdated and lacks `slug`, `categories`, and `cover` fields. Use `content/posts/Template.md` instead.
- `*.base` files (e.g. `無題のファイル.base`) — Obsidian internal files; never stage or commit them.
- `www.youtube.com/` and similar URL-named directories — Obsidian sometimes creates these when saving web clips; never stage or commit them.
- Stray image files at the repo root (e.g. `kv_pc_02.webp`, a pasted screenshot, or a long auto-named `....jpg.webp`) — leftovers from Obsidian web clips or pastes; never stage or commit them. Post/book cover images belong under `static/images/`.
- Unreferenced media inside `static/images/` — Obsidian also drops pasted files there, including duplicates with a ` 1` suffix (e.g. `HKGKywIbAAA7cig 1.jpeg`) and videos (`.mp4`). When publishing, stage only the image the post's `cover.image` actually points to, never everything untracked in `static/images/`.

## Post subfolder / permalink gotcha

`hugo.toml` sets `[permalinks] posts = "/posts/:slug/"`. This exists because posts live in category subfolders (`content/posts/books/`, `content/posts/design/`, `content/posts/exhibitions/`) — without this override, Hugo would fold the subfolder name into the URL (e.g. `/posts/books/foo/` instead of `/posts/foo/`), silently breaking every existing shared link and search index entry for posts in a subfolder. Do not remove this permalinks override, and do not add further nested subfolders without first confirming the URL stays flat (`hugo --minify` then check `public/posts/<slug>/`).

## Post date and UTC gotcha

CI builds run in UTC. If a post's `date:` is set to a date that is still in the future relative to UTC at build time, Hugo silently excludes it — no error, the post just won't appear. Taiwan is UTC+8, so this happens whenever a post is published after **8 PM Taiwan time**: the local date has already rolled over to the next day, but UTC hasn't. Always use the UTC calendar date in `date:` frontmatter. When in doubt, use yesterday's date.

## Language / SEO settings

`hugo.toml` sets `languageCode = "zh-tw"`, `defaultContentLanguage = "zh-tw"`, and `hasCJKLanguage = true`. PaperMod reads the page language (`<html lang>`, `hreflang`, JSON-LD `inLanguage`) from `defaultContentLanguage`, not `languageCode`, so both must stay `zh-tw` — otherwise every page tells search engines it's English. `hasCJKLanguage` makes word counts, reading time and auto-summaries correct for Chinese text. With a single language, `defaultContentLanguage` adds no URL prefix; it also switches PaperMod's UI strings (pagination, reading time) to `themes/PaperMod/i18n/zh-tw.yaml`.

## Image path gotcha

The site lives at a subdirectory (`/the-sloth/`). Hugo's `relURL` resolves cover image paths correctly at build time, so always write cover paths as `/images/filename` — not as full URLs and not with `/the-sloth/` prepended.

Exception: `layouts/partials/home_info.html` hardcodes the hero banner path as `/the-sloth/images/hero.png` because it bypasses Hugo's URL processing. If the `baseURL` subdirectory ever changes, update that partial manually.

## OG / Twitter card image URLs

PaperMod's default `opengraph.html` and `twitter_cards.html` use `absURL` directly on the cover image path. When the path starts with `/` (e.g. `/images/foo.jpg`), Hugo's `absURL` drops the subdirectory, producing `https://dennytsai-arch.github.io/images/foo.jpg` instead of the correct `…/the-sloth/images/foo.jpg`. This breaks link preview thumbnails in messaging apps and social media.

Both templates are overridden in `layouts/partials/templates/` using `strings.TrimLeft "/" | absURL` instead of bare `absURL`. Do not revert this. If PaperMod is ever updated and the upstream templates change, re-apply this fix to the local overrides.

## Customisation points

- **Theme config**: `hugo.toml` — PaperMod params, menu, outputs
- **Styling**: `assets/css/extended/custom.css` — Inter font, light/dark CSS variables, card grid layout, post page layout
- **Home page**: `layouts/partials/home_info.html` — hero banner + Facebook follow button
- **Cover images**: `layouts/partials/cover.html` — responsive srcset logic with fallback for external URLs
