# Sprint Improvements Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Implement 8 improvements to alexwalker.net — WebP image, ProfilePage + Article schema, print stylesheet, self-hosted fonts, WCAG contrast audit, security.txt, and maintenance sync.

**Architecture:** Pure static site. Single `index.html` with inline CSS, no JS, no build step. All changes are file edits and new static assets.

**Tech Stack:** HTML, CSS, cwebp (image conversion), curl (font download)

---

### Task 1: Convert headshot to WebP and add `<picture>` element

**Files:**
- Create: `photo.webp` (via cwebp conversion)
- Modify: `index.html:536-538` (headshot `<img>` tag)

**Step 1: Convert photo.jpg to photo.webp**

Run:
```bash
cwebp -q 80 photo.jpg -o photo.webp
```

If cwebp not installed:
```bash
sudo apt-get update && sudo apt-get install -y webp
cwebp -q 80 photo.jpg -o photo.webp
```

Expected: `photo.webp` created, significantly smaller than 59KB.

**Step 2: Replace `<img>` with `<picture>` element**

In `index.html`, replace line 537:
```html
        <img src="photo.jpg" alt="Alex Walker, Managing Director of Havas Market UK" width="640" height="640" loading="lazy">
```

With:
```html
        <picture>
          <source srcset="photo.webp" type="image/webp">
          <img src="photo.jpg" alt="Alex Walker, Managing Director of Havas Market UK" width="640" height="640" loading="lazy">
        </picture>
```

**Step 3: Update CSS selector for picture element**

In `index.html`, the CSS rule at line 212 `.headshot-wrap img` will still work because `<img>` is inside `<picture>`. No CSS change needed.

**Step 4: Commit**

```bash
git add photo.webp index.html
git commit -m "feat: serve headshot as WebP with JPEG fallback"
```

---

### Task 2: Add ProfilePage schema

**Files:**
- Modify: `index.html:31-107` (existing Person JSON-LD block)

**Step 1: Wrap Person inside ProfilePage**

Replace the existing JSON-LD block (lines 31-107) with a ProfilePage that contains the Person as `mainEntity`. Add `@id` to the Person for cross-referencing from Article schema later.

```json
{
  "@context": "https://schema.org",
  "@type": "ProfilePage",
  "dateCreated": "2025-01-01",
  "dateModified": "2026-02-28",
  "mainEntity": {
    "@type": "Person",
    "@id": "https://www.alexwalker.net/#person",
    "name": "Alex Walker",
    ... (rest of existing Person schema unchanged)
  }
}
```

Key changes:
- Outer type becomes `ProfilePage`
- Person moves inside `mainEntity`
- Person gets `@id: "https://www.alexwalker.net/#person"`
- Add `dateCreated` and `dateModified`

**Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add ProfilePage schema wrapping Person entity"
```

---

### Task 3: Add Article schema for authored pieces

**Files:**
- Modify: `index.html` (add new JSON-LD script block before closing `</head>`)

**Step 1: Identify authored articles**

These press items are byline/opinion pieces authored by Alex (not articles where he's quoted):

1. "Loyalty That Works" — Retail Gazette, Feb 2026
2. "Search Was Always a Vanity Metric" — The Media Leader, Oct 2025
3. "No More Death by a Thousand Cuts" — Marketing Procurement iQ, Apr 2025
4. "Lidl's TikTok Shop Success Is a Bellwether for FMCG Brands" — The Grocer, Mar 2025
5. "Forget the FOMO, E-commerce Isn't an 'All or Nothing' Challenge" — The Drum, Apr 2024

**Step 2: Add Article JSON-LD block**

Insert a new `<script type="application/ld+json">` block after the ProfilePage block (before the Google Fonts links). Contains an `@graph` array of Article objects, each with `author` referencing `"@id": "https://www.alexwalker.net/#person"`.

```json
{
  "@context": "https://schema.org",
  "@graph": [
    {
      "@type": "Article",
      "headline": "Loyalty That Works",
      "url": "https://www.retailgazette.co.uk/blog/2026/02/loyalty-that-works/",
      "datePublished": "2026-02",
      "publisher": { "@type": "Organization", "name": "Retail Gazette" },
      "author": { "@id": "https://www.alexwalker.net/#person" }
    },
    ... (repeat for each authored article)
  ]
}
```

**Step 3: Commit**

```bash
git add index.html
git commit -m "feat: add Article schema for authored press pieces"
```

---

### Task 4: Add print stylesheet

**Files:**
- Modify: `index.html` (add `@media print` block inside `<style>`, before closing `</style>` tag at line 522)

**Step 1: Add print styles**

Insert before the closing `</style>` tag:

```css
/* ── PRINT ── */
@media print {
  *, *::before, *::after { animation: none !important; }
  body { background: #fff; color: #000; font-size: 12pt; }
  .skip-link, nav, #contact .contact-links, .eyebrow { display: none; }
  header { border-bottom: 1px solid #ccc; }
  a { color: #000; text-decoration: none; }
  .press-item::after { content: " (" attr(href) ")"; font-size: 8pt; color: #666; }
  footer { border-top: 1px solid #ccc; }
  h1, h2 { color: #000; }
  .headshot-wrap img { border: 1px solid #ccc; box-shadow: none; }
  .tag { border-color: #ccc; color: #333; background: #fff; }
  section { break-inside: avoid; padding: 1.5rem 0; }
}
```

Key decisions:
- Hide nav, skip-link, contact links (not useful on paper)
- Show press URLs via `::after` pseudo-element with `attr(href)`
- Remove animations
- Black on white, no background colours
- Keep headshot visible
- Avoid page breaks inside sections

**Step 2: Commit**

```bash
git add index.html
git commit -m "feat: add print stylesheet for clean printed output"
```

---

### Task 5: Self-host Google Fonts

**Files:**
- Create: `fonts/` directory with WOFF2 files
- Modify: `index.html:109-112` (remove Google Fonts links)
- Modify: `index.html` inline `<style>` (add `@font-face` declarations)

**Step 1: Download font files from Google Fonts**

```bash
mkdir -p fonts

# Download DM Sans (300, 400, 500) - Latin subset
# Get the CSS to find WOFF2 URLs
curl -s -H "User-Agent: Mozilla/5.0" "https://fonts.googleapis.com/css2?family=DM+Sans:wght@300;400;500&display=swap" | grep -oP 'url\(\K[^)]+\.woff2' | while read url; do
  filename=$(echo "$url" | grep -oP '[^/]+$')
  curl -s -o "fonts/$filename" "$url"
done

# Download DM Serif Display (regular + italic)
curl -s -H "User-Agent: Mozilla/5.0" "https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&display=swap" | grep -oP 'url\(\K[^)]+\.woff2' | while read url; do
  filename=$(echo "$url" | grep -oP '[^/]+$')
  curl -s -o "fonts/$filename" "$url"
done
```

Note: Google Fonts returns different URLs for different subsets. We only need Latin. The User-Agent header ensures we get WOFF2 format.

**Step 2: Identify the Latin subset files**

The downloaded files will include multiple subsets (latin, latin-ext, etc.). We need to inspect the CSS to match filenames to subsets and keep only the Latin ones. Parse the CSS output to identify which WOFF2 URL corresponds to `/* latin */`.

**Step 3: Remove Google Fonts link tags**

Delete lines 109-112 from `index.html`:
```html
  <link rel="preconnect" href="https://fonts.googleapis.com">
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
  <link rel="preload" as="style" href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:wght@300;400;500&display=swap">
  <link href="https://fonts.googleapis.com/css2?family=DM+Serif+Display:ital@0;1&family=DM+Sans:wght@300;400;500&display=swap" rel="stylesheet">
```

**Step 4: Add @font-face declarations**

Add at the top of the `<style>` block (before `:root`):

```css
/* ── FONTS ── */
@font-face {
  font-family: 'DM Sans';
  font-style: normal;
  font-weight: 300;
  font-display: swap;
  src: url('fonts/dm-sans-v15-latin-300.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
@font-face {
  font-family: 'DM Sans';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url('fonts/dm-sans-v15-latin-400.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
@font-face {
  font-family: 'DM Sans';
  font-style: normal;
  font-weight: 500;
  font-display: swap;
  src: url('fonts/dm-sans-v15-latin-500.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
@font-face {
  font-family: 'DM Serif Display';
  font-style: normal;
  font-weight: 400;
  font-display: swap;
  src: url('fonts/dm-serif-display-v15-latin-regular.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
@font-face {
  font-family: 'DM Serif Display';
  font-style: italic;
  font-weight: 400;
  font-display: swap;
  src: url('fonts/dm-serif-display-v15-latin-italic.woff2') format('woff2');
  unicode-range: U+0000-00FF, U+0131, U+0152-0153, U+02BB-02BC, U+02C6, U+02DA, U+02DC, U+0304, U+0308, U+0329, U+2000-206F, U+20AC, U+2122, U+2191, U+2193, U+2212, U+2215, U+FEFF, U+FFFD;
}
```

Note: Actual filenames will depend on what Google Fonts serves. The implementer should name files sensibly after download (e.g. `dm-sans-latin-300.woff2`).

**Step 5: Commit**

```bash
git add fonts/ index.html
git commit -m "feat: self-host Google Fonts as WOFF2 (Latin subset)"
```

---

### Task 6: WCAG colour contrast audit

**Files:**
- Modify: `index.html:119` (`--mid` CSS variable, if needed)

**Step 1: Calculate contrast ratios**

Programmatically verify these colour pairs against WCAG AA 4.5:1 for normal text:

| Foreground | Background | Usage |
|-----------|-----------|-------|
| `--mid` #6b6560 | `--paper` #f5f2ed | Nav, dates, publications, timeline orgs |
| `--mid` #6b6560 | `--white` #ffffff | Tags |
| `--ink` #0f0f0f | `--paper` #f5f2ed | Headings |
| `#2a2825` | `--paper` #f5f2ed | Body text |
| `--accent` #c8402a | `--paper` #f5f2ed | Eyebrow, section labels |

Run a quick Node.js or Python script to calculate WCAG contrast ratios.

My calculation: `--mid` on `--paper` ≈ 5.14:1 (passes). But verify programmatically.

**Step 2: If any pair fails, adjust the variable**

If `--mid` fails, darken it minimally. Try `#5f5955` or similar — keep it warm grey, just darker. The goal is the minimum change to pass 4.5:1.

**Step 3: Commit (only if changes needed)**

```bash
git add index.html
git commit -m "fix: adjust --mid colour for WCAG AA contrast compliance"
```

---

### Task 7: Add security.txt

**Files:**
- Create: `.well-known/security.txt`

**Step 1: Create .well-known directory and security.txt**

```bash
mkdir -p .well-known
```

Write `.well-known/security.txt`:
```
Contact: https://www.linkedin.com/in/indigo/
Expires: 2027-02-28T00:00:00.000Z
Preferred-Languages: en
Canonical: https://www.alexwalker.net/.well-known/security.txt
```

**Step 2: Commit**

```bash
git add .well-known/security.txt
git commit -m "feat: add security.txt"
```

---

### Task 8: Ongoing maintenance sync

**Files:**
- Modify: `index.html:921` (footer date)
- Modify: `sitemap.xml:5` (lastmod date)
- Modify: `llms.txt` (sync any new content)

**Step 1: Update footer date**

Line 921: Change "Last updated: February 2026" to "Last updated: February 2026" (already correct — but if any other tasks change the date to reflect the current work, update it).

Actually, since we're making changes today (2026-02-28), the footer already says "February 2026" which is correct. No change needed unless we want to be more specific.

**Step 2: Update sitemap.xml lastmod**

Change lastmod to `2026-02-28` (today's date, reflecting the changes made in this sprint).

**Step 3: Sync llms.txt**

Review llms.txt against the HTML. Currently it's in good sync. No new sections were added to HTML in this sprint (we skipped Perspectives expansion and Speaking section). If Article schema or ProfilePage changes add any new visible content, mirror it in llms.txt. In practice, no llms.txt update needed this sprint since schema changes are machine-readable only.

**Step 4: Commit**

```bash
git add sitemap.xml llms.txt index.html
git commit -m "chore: update sitemap lastmod and sync maintenance files"
```
