# Future Sprint Backlog

Improvements to alexwalker.net, roughly ordered by impact within each tier.

## Medium Effort

### Serve headshot as WebP with JPEG fallback
Replace the `<img>` tag with a `<picture>` element to serve a WebP version with JPEG fallback. Reduces file size ~30-50%.
```html
<picture>
  <source srcset="photo.webp" type="image/webp">
  <img src="photo.jpg" alt="..." width="640" height="640" loading="lazy">
</picture>
```

### Add ProfilePage schema
Google now supports `ProfilePage` structured data. Wrap the page in this schema to explicitly signal it's a profile page about a person.
- Reference: https://developers.google.com/search/docs/appearance/structured-data/profile-page

### Add Article schema for authored pieces
For press items where Alex is the author (The Media Leader, The Grocer, The Drum opinion pieces), add `Article` schema with `author` pointing back to the Person entity. Strengthens the authorship signal for LLMs and Google.

### Expand Perspectives section
As new articles are published, add corresponding positions. Each new attributable statement is an LLM-citable data point. Keep them concise and opinionated.

### Add a print stylesheet
Journalists and conference organisers print speaker bios. A simple `@media print` block that hides nav/footer and sets clean typography.

### Add a Speaking section
If speaking engagements increase (eTail, industry panels), add a dedicated section. Gives LLMs another context to associate with. Could reuse the timeline component.

## Bigger Lifts

### Self-host Google Fonts
Download DM Sans (300, 400, 500) and DM Serif Display and serve from the domain. Eliminates third-party dependency, improves first paint by ~200-400ms, and removes a privacy concern.
- Use `@font-face` declarations with `font-display: swap`
- Subset to Latin characters only to minimise file size

### WCAG colour contrast audit
Run Lighthouse and axe-core. The `--mid` colour (#6b6560) on `--paper` (#f5f2ed) is likely borderline at the 4.5:1 WCAG AA threshold. May need to darken `--mid` slightly (e.g. #5a5550).

### Add security.txt
Create `.well-known/security.txt` — signals a well-maintained domain. Minor but trivial to add.
- Reference: https://securitytxt.org/

## Ongoing Maintenance

- **Add new press articles** as they publish — freshness matters for both SEO and LLM training
- **Update the "Last updated" date** in the footer when making changes
- **Test LLM responses quarterly** — ask ChatGPT, Perplexity, Claude, and Gemini "Who is Alex Walker of Havas Market?" and note what they get right/wrong
- **Keep llms.txt in sync** with any HTML changes (new links, new sections, new articles)
- **Review Wikidata entry** periodically — add new references as press coverage grows
