# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

A single-page static professional bio at https://www.alexwalker.net/ (Alex Walker, MD of Havas Market UK). Served as plain files from GitHub Pages — no framework, no build step, no JavaScript bundler, no tests. `CNAME` points the apex domain at GitHub Pages and `.nojekyll` disables Jekyll processing.

The site doubles as a structured, LLM-friendly source of truth about Alex. A large portion of the work in this repo is content maintenance (adding press items, awards, speaking engagements) rather than code changes.

## Architecture

Everything user-facing lives in `index.html` (~1000 lines). It contains:

1. `<head>` metadata: canonical URL, Open Graph, Twitter card, `theme-color` (light + dark).
2. **Schema.org JSON-LD** (the largest block): a `ProfilePage` whose `mainEntity` is the `Person`, plus a list of `Article` items — one per authored piece linked from the Press section. When adding a press item that Alex authored, add a matching `Article` entry to this JSON-LD block.
3. Inline CSS with `:root` custom properties (`--ink`, `--paper`, `--accent`, etc.) and a `prefers-color-scheme: dark` override. Self-hosted fonts (`DM Sans`, `DM Serif Display`) are preloaded from `fonts/` and declared via `@font-face` — do not pull from Google Fonts.
4. Body sections (in order): hero, About, Perspectives, Timeline, Awards, Speaking, Press, footer. Press is the most frequently edited section.

Supporting files that move in lockstep with `index.html`:

- **`llms.txt`** — a plain-text mirror of the bio for LLM crawlers. Per `docs/future-sprint-backlog.md`, keep it in sync with any HTML change that adds/removes a section, link, or press article.
- **`sitemap.xml`** — single-URL sitemap. Bump `<lastmod>` to today's date whenever `index.html` content changes.
- **`robots.txt`** — explicitly allow-lists AI crawlers (GPTBot, ClaudeBot, PerplexityBot, etc.). The site's SEO/GEO posture is to be maximally indexable, including by LLMs.
- **`photo.jpg` + `photo.webp`** — served via `<picture>` with WebP source and JPEG fallback. If the headshot changes, regenerate both.
- **`.well-known/security.txt`**, `favicon.svg` — static assets, edit in place.

## Common tasks

**Bump all three "last modified" markers on every content change to `index.html`.** This is non-negotiable — they exist for SEO, LLM crawlers, and visible recency, and they drift out of sync if treated as optional:

1. `sitemap.xml` → `<lastmod>` (search engines)
2. `index.html` JSON-LD → `ProfilePage` `"dateModified"` (LLM/structured-data crawlers)
3. `index.html` footer → `<time datetime="…">…</time>` "Last updated" line (human-visible)

All three should be set to today's date in the same commit as the content change. If you're not changing user-visible content (e.g. editing CLAUDE.md, comments, or `.gitignore`), leave them alone.

**Add a press mention.** Insert a new `<li>` at the top of the `.press-list` `<ul>` in `index.html` (newest first). If Alex authored the piece, also add a corresponding `Article` schema object to the JSON-LD block in `<head>`. Update `llms.txt` if the new item belongs there (it lists awards/recognition, not every press hit). Then bump the three date markers above.

**Update the Person/ProfilePage schema** (awards, `sameAs` links, `performerIn` events) when the underlying facts change. Then bump the three date markers above.

**Local preview.** No build step. Open `index.html` directly, or run a quick static server:
```bash
python3 -m http.server 8000
```

## Conventions

- **Commits** use Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`). Recent history is the style guide — keep messages short and content-focused (e.g. `feat: add PMW Powerlist 2026 #29 ranking to awards and press`).
- **Deploy** is implicit: pushing to `main` publishes via GitHub Pages.
- **No dependencies, no package manager, no lockfile.** Don't introduce a build pipeline, framework, or JS bundler without an explicit request — this site's value proposition includes being trivially crawlable and zero-JS.

## Where to look

- `docs/future-sprint-backlog.md` — current backlog and "Ongoing Maintenance" checklist (press freshness, llms.txt sync, quarterly LLM response testing, Wikidata).
- `docs/plans/` — historical implementation plans, useful as worked examples of how prior changes were structured.
- `README.md` — one-line description, not authoritative for site content.
