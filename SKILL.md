---
name: seo-master
description: SEO audits, CTR optimization, and indexing-error resolution for web projects — meta tags, canonicals, structured data (JSON-LD), sitemaps, robots.txt, Core Web Vitals, GSC-data-driven title/description rewrites, and fixes across Google, Bing (incl. Copilot/GEO + IndexNow), Yandex, Apple and regional engines. Use when the user mentions SEO, rankings, CTR, indexing, Search Console errors, structured data, sitemaps, or search visibility.
---

# SEO Master

Comprehensive SEO auditing, optimization, and indexing error resolution for web projects across all major search engines.

## Quick Start Workflow

Copy this checklist and track progress:

```
SEO Audit Progress:
- [ ] Step 1: Discover project structure and tech stack
- [ ] Step 2: Audit HTML head tags (title, meta, canonical, OG)
- [ ] Step 2b: Audit and optimize click-through rate (CTR)
- [ ] Step 3: Audit robots.txt and sitemap.xml
- [ ] Step 4: Audit structured data (JSON-LD / schema.org)
- [ ] Step 5: Audit internal linking and URL structure
- [ ] Step 6: Audit Core Web Vitals and performance signals
- [ ] Step 7: Audit JavaScript SEO (if SPA/SSR)
- [ ] Step 8: Audit multi-engine compatibility (Bing/Copilot, Yandex, Apple, Baidu, Naver, Seznam)
- [ ] Step 9: Fix all discovered issues
- [ ] Step 10: Validate fixes
```

## Step 1: Discover Project Structure

Identify the tech stack and page generation method:

- **Static HTML**: Check each `.html` file directly
- **SSR frameworks** (Next.js, Nuxt, SvelteKit, Astro, Remix): Check layout files, head components, metadata config
- **SPA** (React, Vue, Angular): Check for SSR/prerendering setup — SPAs without prerendering have major SEO issues
- **CMS** (WordPress, etc.): Check theme templates, SEO plugin config

Search for:
```
# Find HTML files
**/*.html

# Find head/meta configuration
grep -r "<title" "<meta" "canonical" "og:" "json-ld" "schema.org"

# Find robots.txt and sitemap
robots.txt, sitemap.xml, sitemap*.xml

# Find layout/head files (framework-specific)
_app.*, layout.*, _document.*, app.html, +layout.*, head.*
```

## Step 2: Audit HTML Head Tags

Every indexable page MUST have these elements in `<head>`:

### Required Tags

```html
<!-- Unique, descriptive title (50-60 chars) -->
<title>Primary Keyword - Secondary Keyword | Brand Name</title>

<!-- Unique meta description (150-160 chars) -->
<meta name="description" content="Compelling description with primary keyword that encourages clicks.">

<!-- Self-referencing canonical URL (absolute, HTTPS) -->
<link rel="canonical" href="https://example.com/current-page/">

<!-- Viewport for mobile -->
<meta name="viewport" content="width=device-width, initial-scale=1">

<!-- Charset -->
<meta charset="utf-8">
```

### Open Graph (Facebook, LinkedIn, Discord, etc.)

```html
<meta property="og:title" content="Page Title">
<meta property="og:description" content="Page description">
<meta property="og:image" content="https://example.com/image.jpg">
<meta property="og:url" content="https://example.com/page/">
<meta property="og:type" content="website">
<meta property="og:site_name" content="Brand Name">
```

### Twitter Card

```html
<meta name="twitter:card" content="summary_large_image">
<meta name="twitter:title" content="Page Title">
<meta name="twitter:description" content="Page description">
<meta name="twitter:image" content="https://example.com/image.jpg">
```

### Common Mistakes to Fix

| Issue | Impact | Fix |
|-------|--------|-----|
| Missing `<title>` | Critical — no ranking signal | Add unique title per page |
| Duplicate titles across pages | Dilutes rankings | Make each title unique |
| Missing canonical | Duplicate content risk | Add self-referencing canonical |
| Relative canonical URL | Google may ignore it | Use absolute HTTPS URL |
| Missing meta description | Lower CTR | Add compelling description |
| Missing viewport meta | Mobile ranking penalty | Add viewport tag |
| Multiple `<h1>` tags | Confuses hierarchy | Use single `<h1>` per page |

## Step 2b: Click-Through Rate (CTR) Optimization

Even well-ranked pages waste visibility if titles/descriptions don't earn the click. Full playbook: [ctr-optimization.md](ctr-optimization.md). Summary:

1. **Get GSC Performance data** (queries/pages with impressions, clicks, CTR, position) and flag pages below the position-CTR benchmarks in the playbook
2. **Rewrite titles** — front-load the keyword, add a number/data point, add the current date for time-sensitive queries, keep the RENDERED tag at 55-60 chars (account for framework title templates)
3. **Rewrite descriptions** — 150-160 chars, start with the answer, include a CTA and differentiator
4. **Use dynamic metadata** for live-data pages (`generateMetadata()` pattern) so titles always match date-stamped searches
5. **Fix www/non-www duplication** — 301 to one canonical domain or clicks split across URLs
6. **Add structured data** for eligible rich results (Breadcrumb, Article dates, Product ratings — note FAQ/HowTo rich-result restrictions in the playbook)
7. **Monitor after 2-4 weeks** in GSC

## Step 3: Audit robots.txt and Sitemap

### robots.txt

Must exist at site root. Check for:

```
# Good baseline
User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /private/

# Point to sitemap
Sitemap: https://example.com/sitemap.xml
```

**Deprecated directives — do not add:** `Crawl-delay` (Google never supported it; Bing moved it to Webmaster Tools crawl settings; Yandex dropped it in 2018) and Yandex's `Host` (dropped 2018 — use 301 redirects to your canonical domain instead).

**Critical rules:**
- NEVER block CSS/JS resources Google needs to render pages
- `Disallow` does NOT prevent indexing — use `noindex` meta tag instead
- Verify in Search Console: Settings > robots.txt report (the standalone robots.txt Tester tool was retired)
- **AI crawlers**: Block training bots (`GPTBot`, `Google-Extended`, `ClaudeBot`, `CCBot`, etc.) while allowing AI search bots (`OAI-SearchBot`, `PerplexityBot`) that cite sources with links back. See [technical-seo.md → AI Crawler Management](technical-seo.md#ai-crawler-management-training-vs-search) for the full bot reference table and templates.
- **Disallow and structured data must agree**: anything blocked here must also be absent from `SiteNavigationElement`/`BreadcrumbList` URLs and from the sitemap — never advertise to a crawler a URL this file tells it to skip (see Step 4).

### XML Sitemap

Must include all canonical, indexable pages:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<urlset xmlns="http://www.sitemaps.org/schemas/sitemap/0.9">
  <url>
    <loc>https://example.com/page/</loc>
    <lastmod>2026-01-15</lastmod>
  </url>
</urlset>
```

**Rules:**
- Max 50,000 URLs or 50MB per sitemap file
- Use sitemap index for larger sites
- Only include canonical URLs (not redirects, not noindexed pages)
- `<loc>` must use absolute HTTPS URLs matching canonicals
- `<lastmod>` must reflect actual content change dates — Google uses it (if consistently truthful); `<priority>` and `<changefreq>` are IGNORED by Google, so skip them
- Reference sitemap in robots.txt
- Submit to Google Search Console, Bing Webmaster Tools, and Yandex Webmaster

### Bing-Specific: IndexNow

For instant indexing on Bing, Yandex, DuckDuckGo:

```bash
# Notify search engines of new/updated URLs instantly
curl "https://api.indexnow.org/indexnow?url=https://example.com/page/&key=YOUR_API_KEY"
```

Create a key file at `https://example.com/YOUR_API_KEY.txt` containing the key.

## Step 4: Audit Structured Data

Use JSON-LD format (Google recommended). Place in `<head>` or `<body>`.

### Website + Organization (homepage)

```json
{
  "@context": "https://schema.org",
  "@type": "WebSite",
  "name": "Site Name",
  "url": "https://example.com",
  "potentialAction": {
    "@type": "SearchAction",
    "target": "https://example.com/search?q={search_term_string}",
    "query-input": "required name=search_term_string"
  }
}
```

### Breadcrumbs (all inner pages)

```json
{
  "@context": "https://schema.org",
  "@type": "BreadcrumbList",
  "itemListElement": [
    {"@type": "ListItem", "position": 1, "name": "Home", "item": "https://example.com/"},
    {"@type": "ListItem", "position": 2, "name": "Category", "item": "https://example.com/category/"}
  ]
}
```

### Common Schema Types

Choose based on content type. For detailed implementation, see [structured-data.md](structured-data.md).

| Content Type | Schema Type |
|-------------|-------------|
| Articles/Blog | `Article`, `BlogPosting` |
| Products | `Product` with `Offer` |
| Local Business | `LocalBusiness` |
| FAQ pages | `FAQPage` with `Question` |
| How-to guides | `HowTo` |
| Events | `Event` |
| Recipes | `Recipe` |
| Videos | `VideoObject` |
| Job postings / careers pages | `JobPosting` |
| Reviews | `Review`, `AggregateRating` |
| Software/App | `SoftwareApplication` |

**Validation:** Test with Google Rich Results Test and Schema.org Validator.

**Navigation schema must not name a URL robots.txt blocks.** A navbar link can legitimately point at a path you `Disallow` (a plan picker under `/auth/`, an internal tool), but that link must then stay **out** of `SiteNavigationElement` and `BreadcrumbList` — cross-check every schema URL against `robots.txt`, because markup that advertises a disallowed URL works against your own crawl rules. The schema list mirrors the *crawlable* navigation, not the visible navbar. And markup cannot rescue a page that needs to rank: if the only pricing/plans page lives under a disallowed path, give it a crawlable URL. See [structured-data.md → SiteNavigationElement](structured-data.md#sitenavigationelement-homepage).

### FAQPage — Visible-Content Requirement (Critical)

Note: Google restricted FAQ rich results to authoritative gov/health sites (Aug 2023) — for most sites this markup now targets Bing rich results and AI answer grounding, not Google snippets. Where it applies, engines only grant FAQ rich results when the question/answer text is **visibly rendered on the page** and matches the JSON-LD. Schema-only FAQ (markup with no on-page Q&A) violates Google's structured-data policy — it won't earn the rich result and can trigger a manual action. So adding `FAQPage` schema is a TWO-part change: visible `<dl>`/accordion **and** matching markup.

**Single-source pattern** — define the Q&A once, render both the visible section and the schema from it so they can never drift out of sync (drift silently disqualifies the rich result):

```tsx
const FAQS: { q: string; a: string }[] = [
  { q: 'What is X?', a: 'X is …' },
  // …
];

// 1) Visible section
<dl>{FAQS.map(f => (<div key={f.q}><dt>{f.q}</dt><dd>{f.a}</dd></div>))}</dl>

// 2) Schema built from the SAME array
const faqStructuredData = {
  '@context': 'https://schema.org',
  '@type': 'FAQPage',
  mainEntity: FAQS.map(f => ({
    '@type': 'Question',
    name: f.q,
    acceptedAnswer: { '@type': 'Answer', text: f.a },
  })),
};
```

When a site has multiple pages eligible for FAQ (e.g. a data page and an education page), write **distinct, intent-matched questions per page** — duplicate FAQ blocks across URLs dilute relevance and can suppress the snippet on all of them.

## Step 5: Audit Internal Linking and URL Structure

- Use descriptive, keyword-rich anchor text (not "click here")
- Ensure flat URL hierarchy (max 3 levels deep from root)
- Use hyphens in URLs, not underscores
- Lowercase URLs only
- Avoid URL parameters when possible; use clean paths
- Every important page should be reachable within 3 clicks from homepage
- Add breadcrumb navigation matching structured data
- Check for orphan pages (no internal links pointing to them)
- Check for naming that only the developer understands (`/referral` for a public jobs page) — it is cheap to fix **before** the URL is indexed and expensive afterwards

### Renaming or moving an indexed URL

The file move is the easy half. If the old URL is indexed, in the sitemap, or linked from
anywhere, the rename is a migration and ships as one atomic change:

1. **Permanent redirect** (301/308) for each old path → the new one, one hop, no chains,
   no mass redirect to the homepage. In Next.js `redirects()`, `permanent: true` emits
   **308** and `permanent: false` emits 307 — never ship a temporary redirect for a rename.
2. **Keep the old path crawlable** — do not add it to `robots.txt`; a blocked URL's
   redirect is never fetched.
3. **Swap the sitemap** to the new URLs in the same commit (sitemaps list canonical URLs
   only — a redirecting URL in a sitemap is self-inflicted "Page with redirect").
4. **Update every internal link** — nav, footer, breadcrumbs, in-body links.
5. **Re-point the canonical on each moved page.** Hard-coded absolute canonicals do not
   follow a directory move; a moved page still canonicalising to the old URL is a circular
   canonical and can de-index.
6. **Leave API/backend paths alone.** Public page slugs are a contract with search engines;
   `/api/...` paths are a contract with your own in-flight form posts. Renaming the page
   does not require renaming the endpoint.
7. **Keep the redirect forever** — it is what keeps other people's links working.

Afterwards, expect the old URLs in GSC's **Page with redirect** bucket (valid, healthy —
not an error to drive to zero) and request indexing for the **new** URLs. Per-path renames
have no Change of Address tool; the redirect, sitemap and internal links are the whole
signal. Full checklist and case study: [url-migration.md](url-migration.md).

### Job and careers pages

A public hiring page is a content page with its own rules: it needs `JobPosting` markup
(required `title`, `description`, `datePosted`, `hiringOrganization`, and either
`jobLocation` or `jobLocationType: "TELECOMMUTE"` + `applicantLocationRequirements`),
one posting per URL, and a `validThrough` so a filled role stops advertising itself.
Commission-only roles omit `baseSalary` rather than marking up a figure that is not
guaranteed. See [structured-data.md](structured-data.md) → JobPosting.

## Step 6: Core Web Vitals

Check and optimize these metrics:

| Metric | Target | What It Measures |
|--------|--------|-----------------|
| LCP | < 2.5s | Largest Contentful Paint (loading speed) |
| INP | < 200ms | Interaction to Next Paint (responsiveness) |
| CLS | < 0.1 | Cumulative Layout Shift (visual stability) |

**Common fixes:**
- LCP: Optimize images (WebP/AVIF, lazy-load below fold, preload hero image), minimize render-blocking CSS/JS
- INP: Break up long tasks, use `requestIdleCallback`, defer non-critical JS
- CLS: Set explicit `width`/`height` on images/videos, avoid injecting content above existing content

## Step 7: JavaScript SEO (SPA/SSR)

If the project uses client-side rendering:

1. **Verify SSR/SSG is enabled** — Pure client-side rendering is an SEO blocker
2. **Check that meta tags render server-side** — Not just injected by JS after load
3. **Use History API for routing** — Not hash fragments (`#/page`)
4. **Ensure canonical tags are in initial HTML** — Not dynamically added
5. **Return proper HTTP status codes** — 404 for missing pages, not soft 404
6. **Test with "View Page Source"** — If content isn't in source, Google may not see it

For detailed JS SEO guidance, see [technical-seo.md](technical-seo.md).

## Step 8: Multi-Engine Compatibility

### Bing + Copilot (GEO)

Bing now powers both traditional search and AI-generated Copilot answers. SEO and GEO (Generative Engine Optimization) share the same crawling/indexing foundation.

- Submit sitemap at bing.com/webmaster
- Add `<meta name="msvalidate.01" content="VERIFICATION_CODE">` for ownership
- **Configure IndexNow** — Bing prefers streaming submissions over batch for faster indexing
- **AI/Copilot control directives**:
  - `noarchive` — blocks content from Copilot answers entirely
  - `nocache` — limits Copilot to URL/title/snippet only
  - `data-nosnippet` — excludes specific HTML elements from snippets and Copilot
  - `data-snippet` — specifies text Bing may quote in Copilot citations
- **302 redirects** — Bing says use only for changes lasting <2 days (stricter than Google)
- **Prompt injection** — Hidden text designed to manipulate AI models results in delisting
- For full Bing/Copilot reference, see [bing-copilot-seo.md](bing-copilot-seo.md)

### Yandex Webmaster

- Submit sitemap at webmaster.yandex.com
- Add `<meta name="yandex-verification" content="VERIFICATION_CODE">`
- `Host` directive in robots.txt for preferred domain (Yandex-only feature)
- `Crawl-delay` in robots.txt for Yandex bot
- **`keywords` meta tag matters** — Yandex actually considers it for relevance (Google ignores it)
- **Allow directives take priority** over deny when conflicting — opposite of Google
- **robots.txt overrides meta tags** — if robots.txt blocks a page, `noindex` is ignored
- **10 MB document size limit** (Google is 15 MB)
- **Original Texts tool** — submit original content before publication to establish authorship
- **Indexes iframe/frame content** — unlike most engines
- For Yandex-specific error types, see [multi-engine-errors.md](multi-engine-errors.md)

### Apple (Applebot)

Powers Spotlight, Siri, and Safari search across all Apple devices:

- **Two user agents** with distinct purposes:
  - `Applebot` — search crawling (allow this for search visibility)
  - `Applebot-Extended` — AI training data; block to opt out without losing search
- **Falls back to Googlebot rules** if no Applebot section in robots.txt
- Renders pages in a browser — don't block CSS/JS
- Supports `noindex`, `nosnippet`, `nofollow`, `none`, `all` directives
- Verify traffic via reverse DNS (`*.applebot.apple.com`) or IP list at `search.developer.apple.com/applebot.json`

```
# robots.txt — allow search, block AI training
User-agent: Applebot
Allow: /

User-agent: Applebot-Extended
Disallow: /
```

### DuckDuckGo

- Sources results from Bing — optimizing for Bing covers DuckDuckGo
- Supports IndexNow protocol
- Respects standard robots.txt and meta robots tags

### Regional Engines (Baidu, Naver, Seznam.cz)

Critical for projects targeting specific markets. Key gotchas:

- **Baidu (China)** — Cannot render JavaScript; does NOT support `noindex` meta tag; requires ICP License and mainland China hosting; .CN domain preferred
- **Naver (South Korea)** — Registration-first discovery; indexing takes up to 14 days; supports IndexNow
- **Seznam.cz (Czech Republic)** — Crawls much slower than Google; prefers server-rendered HTML; 100/day manual URL submission limit

For full regional engine requirements, see [regional-engines.md](regional-engines.md).

### IndexNow Protocol (Multi-Engine Instant Notification)

Single submission notifies all participating engines: Bing, Yandex, Naver, Seznam.cz, Yep.

```bash
# Submit URL change (shared to all IndexNow engines)
curl "https://api.indexnow.org/indexnow?url=https://example.com/page/&key=YOUR_API_KEY"
```

**Not supported by**: Google, Baidu, Apple — use their native submission tools.

**Operational notes (field-tested):**

- **Pass `keyLocation` explicitly** — `…?url=<URL>&key=<KEY>&keyLocation=https://host/<KEY>.txt`. Required when the host serves multiple key files or the key isn't at the document root. A `200` from `api.indexnow.org` means the submission was accepted.
- **Key file behind a WAF / bot shield is a false alarm.** Many sites front their `public/<key>.txt` with a scrape shield that returns `403`/`401` to a plain `curl` user-agent. IndexNow's own servers fetch the key file server-side and still validate it — so you can get `403` spot-checking the `.txt` yourself while the API returns `200` and accepts the URL. Do NOT treat your own keyfile `403` as failure; trust the API status code.
- **Protected app endpoint vs. direct API.** Projects often wrap IndexNow in an authenticated route (e.g. `POST /api/indexnow` gated by a `CRON_SECRET`/bearer). If you don't hold that secret locally, skip the wrapper and submit straight to `https://api.indexnow.org/indexnow?url=…&key=…&keyLocation=…`. Any `public/*.txt` file whose **contents equal its filename** is a valid IndexNow key you can use directly.
- **What to submit.** Ping the exact URLs whose content or metadata changed — including pages whose visible data changed even if the route is unchanged (e.g. a dashboard that now renders backfilled history). Engines only recrawl what you name.

### Engine-Specific Gotchas

| Gotcha | Engine | Impact |
|--------|--------|--------|
| `noindex` meta tag ignored | Baidu | Must use robots.txt to block pages |
| No JavaScript rendering | Baidu | All content must be server-rendered |
| `keywords` meta tag used for ranking | Yandex | Add relevant keywords (others ignore it) |
| Allow overrides Deny in meta directives | Yandex | Opposite of Google's "restrictive wins" |
| Falls back to Googlebot rules | Applebot | Your Google robots.txt rules affect Apple |
| `noarchive` blocks AI answers | Bing | Prevents content from appearing in Copilot |
| 302 must be <2 days | Bing | Stricter than Google's 302 guidance |
| Indexing takes up to 14 days | Naver | Much slower than Google |
| ICP License legally required | Baidu | Cannot host in China without it |

### General Multi-Engine Rules

- Use standard HTML meta tags (all engines understand them)
- Structured data benefits all engines — validate with multiple tools
- Valid XML sitemap works across all engines
- Clean, semantic HTML is universally beneficial
- Server-side rendering is mandatory for Baidu, strongly preferred by Seznam.cz
- Configure IndexNow alongside Google's native submission tools

## Step 9: Fixing Issues

When issues are found, fix them in priority order:

1. **Critical** (blocks indexing): Missing pages, server errors, redirect loops, noindex on important pages, blocked by robots.txt
2. **High** (hurts rankings): Missing titles/descriptions, duplicate content, broken canonical tags, missing structured data
3. **Medium** (limits potential): Poor Core Web Vitals, missing Open Graph, weak internal linking
4. **Low** (polish): Missing alt text on non-critical images, minor schema improvements

For Google Search Console indexing error resolution, see [indexing-errors.md](indexing-errors.md).
For Bing and Yandex indexing errors, see [multi-engine-errors.md](multi-engine-errors.md).

## Step 10: Validate Fixes

After applying fixes:

1. Check HTML validity with W3C validator
2. Test structured data with Google Rich Results Test, Bing Markup Validator, and Schema.org Validator
3. Check mobile usability via Lighthouse or Chrome DevTools device mode (Google's standalone Mobile-Friendly Test was retired Dec 2023)
4. Verify robots.txt in Search Console: Settings > robots.txt report
5. Validate sitemap XML structure
6. Use Google Search Console URL Inspection tool to request re-indexing
7. Use Bing URL Inspection tool to request re-indexing and check SEO card
8. Submit updated sitemap to all search engine webmaster tools (Google, Bing, Yandex, Baidu, Naver)
9. Submit URL changes via IndexNow for Bing, Yandex, Naver, Seznam.cz
10. Verify Applebot can access pages (check robots.txt fallback to Googlebot rules)
11. For a renamed URL, `curl -I` the **old** path against production and confirm a permanent status (301/308) with the right `Location` — a local check does not prove the deployed redirect config or middleware

## Step 11: Post-Deploy Manual Actions

After pushing code fixes, the user/administrator MUST complete these steps manually in each search engine console. **Always output this checklist to the user after making SEO changes**, customized to the specific fixes that were applied.

### Google Search Console (search.google.com/search-console)

1. **Check robots.txt report** (if robots.txt was changed)
   - Settings > robots.txt > confirm latest fetch parsed without errors
   - Verify blocked/allowed paths are correct

2. **Resubmit sitemap** (if sitemap was changed)
   - Indexing > Sitemaps > enter `sitemap.xml` > Submit
   - This clears stale entries (e.g. removed URLs) from Google's copy

3. **Request re-indexing of changed pages** (URL Inspection tool)
   - Paste each changed URL into the search bar, click "Request Indexing"
   - Prioritize: pages with new structured data, new/changed canonical, new meta tags
   - For a **renamed** page, request the new URL only — the old URL's job is to redirect, and it will legitimately appear under "Page with redirect"
   - Rate limit: ~10 URLs per day max

4. **Check Page Indexing report for existing errors**
   - Indexing > Pages > review "Why pages aren't indexed" table
   - Look for errors that should now be fixed (e.g. "Submitted URL blocked by robots.txt" after removing contradictory sitemap entries)
   - Click "Validate Fix" on any error categories that were addressed

5. **Validate structured data**
   - Enhancements > check for new rich result types
   - Or use Rich Results Test (search.google.com/test/rich-results) to test individual URLs

6. **Verify site ownership** (if verification meta tag was added/changed)
   - Settings > Ownership verification > confirm HTML tag method shows "Verified"

### Bing Webmaster Tools (bing.com/webmaster)

7. **Resubmit sitemap** (if sitemap changed)
   - Sitemaps > resubmit URL

8. **Submit URLs via IndexNow** (fastest method for Bing/Yandex/DuckDuckGo)
   - If the project has an IndexNow endpoint, trigger it for all changed URLs
   - Otherwise: `curl "https://api.indexnow.org/indexnow?url=<URL>&key=<KEY>"`

9. **Check URL Inspection** for changed pages
   - Enter changed URLs > review SEO tab for warnings

### Yandex Webmaster (webmaster.yandex.com)

10. **Set up Yandex Webmaster** (if not already done)
    - Add site, verify via HTML meta tag (add `yandex-verification` to layout verification config)
    - Submit sitemap

11. **Recheck indexing** (if robots.txt or sitemap changed)
    - Indexing > Sitemap files > resubmit
    - Check "Pages in search" for error counts

### Template for User Output

When presenting post-deploy steps to the user, customize this template based on what was actually changed:

```
## After You Push — Manual Steps

### Google Search Console
- [ ] Resubmit sitemap (Indexing > Sitemaps > Submit)
- [ ] Request re-indexing for: [list specific changed URLs]
- [ ] Validate fix for: [list specific GSC error categories that were fixed]
- [ ] Test structured data on: [list pages with new/changed JSON-LD]

### Bing Webmaster Tools
- [ ] Resubmit sitemap
- [ ] Submit changed URLs via IndexNow

### Yandex Webmaster
- [ ] Resubmit sitemap (if set up)
```

## Scripts (deterministic checks — prefer these over manual re-derivation)

- `scripts/audit_pages.py <URL>...` — rendered-HTML audit per URL: title/description lengths, canonical, robots noindex, OG/Twitter, JSON-LD validity, h1 count. Exit 1 on CRITICAL.
- `scripts/gsc_ctr.py <gsc-export.csv>` — flags pages/queries below the position-CTR benchmarks from a GSC Performance export.
- `scripts/indexnow_submit.sh <KEY> <URL>... | --file urls.txt` — IndexNow submission with explicit keyLocation.

## Additional Resources

- For all GSC indexing error types and fixes: [indexing-errors.md](indexing-errors.md)
- For Bing and Yandex indexing errors: [multi-engine-errors.md](multi-engine-errors.md)
- For Bing Copilot/AI grounding and GEO: [bing-copilot-seo.md](bing-copilot-seo.md)
- For Baidu, Naver, and Seznam.cz: [regional-engines.md](regional-engines.md)
- For technical crawling/rendering details: [technical-seo.md](technical-seo.md)
- For CTR benchmarks, title formulas and rich-result guidance: [ctr-optimization.md](ctr-optimization.md)
- For structured data implementation: [structured-data.md](structured-data.md)
- For renaming or moving an indexed URL: [url-migration.md](url-migration.md)
- For full audit checklist: [audit-checklist.md](audit-checklist.md)
