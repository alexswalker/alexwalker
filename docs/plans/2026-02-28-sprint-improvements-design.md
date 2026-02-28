# Sprint: Site Improvements Design

**Date:** 2026-02-28
**Scope:** 8 items from future-sprint-backlog.md

## 1. WebP headshot with JPEG fallback

- Convert `photo.jpg` to `photo.webp` using `cwebp`
- Replace `<img>` with `<picture>` element, WebP source first, JPEG fallback
- Keep `width="640" height="640" loading="lazy"` and alt text on the `<img>`

## 2. ProfilePage schema

- Wrap the existing Person schema inside a `ProfilePage` schema
- Add `dateCreated`, `dateModified`, `mainEntity` pointing to the Person
- Single JSON-LD block in `<head>`

## 3. Article schema for authored pieces

- Identify press items where Alex is the **author** (The Media Leader, The Grocer, The Drum opinion pieces)
- Add an `Article` JSON-LD block with `author` referencing the Person entity by `@id`
- One JSON-LD script block containing an array of articles

## 4. Print stylesheet

- Add `@media print` block to the inline `<style>`
- Hide: header nav, contact section links, skip-link
- Clean typography: black on white, no background colours
- Ensure headshot prints, press links show URLs

## 5. Self-host Google Fonts

- Download DM Sans (300, 400, 500) and DM Serif Display (regular, italic) as WOFF2
- Subset to Latin characters
- Place in a `fonts/` directory
- Replace Google Fonts `<link>` tags with inline `@font-face` declarations using `font-display: swap`
- Remove the `preconnect` hints to `fonts.googleapis.com`

## 6. WCAG contrast audit

- Calculate contrast ratio of `--mid` (#6b6560) on `--paper` (#f5f2ed)
- If below 4.5:1, darken `--mid` minimally to pass
- Check all other colour combinations too (`--ink` on `--paper`, `--accent` on `--paper`, etc.)
- Update the CSS variable

## 7. security.txt

- Create `.well-known/security.txt` with standard fields (Contact, Preferred-Languages, Canonical, Expires)
- Contact points to LinkedIn profile (no public email)

## 8. Ongoing maintenance

- Update footer "Last updated" to current date
- Sync `llms.txt` with any HTML changes
- Update `sitemap.xml` lastmod date

## Decisions

- **Fonts inline:** `@font-face` declarations go in the existing inline `<style>` block (no separate CSS file)
- **Skipped:** Speaking section (not enough engagements yet), Expand Perspectives (no new content)
- **Contrast:** Auto-fix `--mid` if it fails WCAG AA 4.5:1 — darken minimally
