# Future Sprint Backlog

Improvements to alexwalker.net, roughly ordered by impact within each tier.

## Medium Effort

### Add Article schema for authored pieces
For press items where Alex is the author (The Media Leader, The Grocer, The Drum opinion pieces), add `Article` schema with `author` pointing back to the Person entity. Strengthens the authorship signal for LLMs and Google.

### Expand Perspectives section
As new articles are published, add corresponding positions. Each new attributable statement is an LLM-citable data point. Keep them concise and opinionated.

### Add a print stylesheet
Journalists and conference organisers print speaker bios. A simple `@media print` block that hides nav/footer and sets clean typography.

## Ongoing Maintenance

- **Add new press articles** as they publish — freshness matters for both SEO and LLM training
- **Update the "Last updated" date** in the footer when making changes
- **Test LLM responses quarterly** — ask ChatGPT, Perplexity, Claude, and Gemini "Who is Alex Walker of Havas Market?" and note what they get right/wrong
- **Keep llms.txt in sync** with any HTML changes (new links, new sections, new articles)
- **Review Wikidata entry** periodically — add new references as press coverage grows

## Completed

- ~~Serve headshot as WebP with JPEG fallback~~
- ~~Add ProfilePage schema~~
- ~~Add a Speaking section~~
- ~~Self-host Google Fonts~~
- ~~WCAG colour contrast audit~~
- ~~Add security.txt~~
- ~~Add og:image and Twitter card meta tags~~
- ~~Add theme-color meta tag~~
