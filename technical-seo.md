# Technical SEO Reference

Deep reference for crawling, rendering, indexing, and server-side SEO configuration.

## How Google Crawls and Indexes

### Three-Phase Process

1. **Crawling** — Googlebot fetches URLs, parses HTML, discovers links
2. **Rendering** — JavaScript pages queued for headless Chromium execution
3. **Indexing** — Rendered content analyzed, stored in search index

**Key facts:**
- Googlebot processes first **15MB** of any page
- Crawls primarily from **US IP addresses**
- Supports HTTP/1.1 and HTTP/2
- Supports gzip, deflate, and brotli compression
- Pages with non-200 status codes may skip rendering

### Crawl Budget Optimization

Crawl budget = how many pages Google will crawl on your site in a given time period.

**Increase crawl budget:**
- Fast server response times (< 200ms)
- No server errors (5xx)
- Avoid duplicate content (wastes crawl budget on duplicates)
- Use XML sitemap to prioritize important pages
- Strong internal linking structure
- Remove low-value pages (thin content, parameter variations)

**Reduce crawl waste:**
- Block crawling of faceted navigation / filter URLs with robots.txt
- Use `noindex` on paginated archive pages (or `rel="next"`/`rel="prev"`)
- Canonicalize parameter variations
- Return 404/410 for truly removed pages (not soft 404)

### HTTP Caching for Crawlers

Implement caching headers to optimize Googlebot crawl efficiency:

```
# Preferred: ETag-based caching
ETag: "abc123"
# Googlebot sends: If-None-Match: "abc123"
# Server returns 304 Not Modified if unchanged

# Alternative: Date-based caching
Last-Modified: Wed, 15 Jan 2026 10:00:00 GMT
# Googlebot sends: If-Modified-Since: Wed, 15 Jan 2026 10:00:00 GMT
```

Google recommends ETag over Last-Modified to avoid date formatting issues.

## robots.txt Deep Dive

### Syntax Reference

```
# Comment
User-agent: Googlebot
Disallow: /private/
Allow: /private/public-page.html

User-agent: *
Disallow: /api/
Disallow: /admin/
Disallow: /tmp/
Disallow: /*?sort=
Disallow: /*&filter=

Sitemap: https://example.com/sitemap.xml
```

### Pattern Matching

| Pattern | Matches | Example |
|---------|---------|---------|
| `/path/` | Exact directory | `/path/anything` |
| `/path` | Prefix match | `/path`, `/pathway` |
| `/*.pdf$` | Files ending in .pdf | `/docs/file.pdf` |
| `/*?` | URLs with parameters | `/page?id=1` |
| `/path/*/file` | Wildcard directory | `/path/sub/file` |

### Crawl-delay Directive

Not supported by Google, but respected by Bing and Yandex:

```
User-agent: Bingbot
Crawl-delay: 1

User-agent: Yandex
Crawl-delay: 2
```

For Google, adjust crawl rate in Search Console settings.

### Common robots.txt Mistakes

| Mistake | Problem | Fix |
|---------|---------|-----|
| `Disallow: /` for all agents | Blocks entire site | Be specific about what to block |
| Blocking CSS/JS files | Google can't render pages | Allow CSS/JS resources |
| Using robots.txt to hide pages | Pages can still appear in index (without snippet) | Use `noindex` instead |
| Forgetting trailing slash | `/admin` matches `/administrator` too | Use `/admin/` for directories |
| No Sitemap directive | Misses easy discovery signal | Add `Sitemap:` line |
| A disallowed path also named in structured data or the sitemap | Crawler is given opposite instructions; the URL surfaces as "Indexed, though blocked by robots.txt" | Keep `Disallow`ed URLs out of `SiteNavigationElement`/`BreadcrumbList` and out of the sitemap |

### robots.txt Redirect Handling

Google fetches `robots.txt` before crawling any page. Redirect behavior directly affects whether your crawl rules are enforced.

**Google's robots.txt fetch rules:**
- Follows up to **5 redirects** for robots.txt
- If > 5 redirects: treats robots.txt as unreachable → **assumes all crawling allowed** (silently bypasses all `Disallow` rules)
- If robots.txt returns 4xx: assumes no crawl restrictions (open crawl)
- If robots.txt returns 5xx: **assumes entire site is restricted** (conservative — blocks crawl)

**Requirements:**
- `https://yourdomain.com/robots.txt` must return **200 directly** — no redirect at the canonical URL
- HTTP → HTTPS redirect for robots.txt is acceptable (1 hop, within the 5-hop limit)
- If www and non-www both resolve independently, serve **identical** robots.txt on both — Google may fetch either depending on which version it discovers first; divergent rules cause unpredictable crawl behavior
- A `www/robots.txt` that 301s to non-www in a single hop is also acceptable

**Diagnose all four robots.txt variants:**
```bash
curl -sI http://example.com/robots.txt   | grep -E "^HTTP|^Location"
curl -sI https://example.com/robots.txt  | grep -E "^HTTP|^Location"
curl -sI http://www.example.com/robots.txt  | grep -E "^HTTP|^Location"
curl -sI https://www.example.com/robots.txt | grep -E "^HTTP|^Location"

# Verify www and non-www serve identical content
diff <(curl -s https://example.com/robots.txt) <(curl -s https://www.example.com/robots.txt)
# Empty output = consistent. Any diff = fix required.
```

**www/non-www and the redirect chain problem:**

A common multi-hop trap for robots.txt:
```
http://www.example.com/robots.txt
  → 301 https://www.example.com/robots.txt   (hop 1: HTTP→HTTPS)
  → 301 https://example.com/robots.txt       (hop 2: www→non-www)
```
Two hops is still within Google's limit but wastes crawl setup time. Worse: if either intermediate hop 404s or 403s, Google falls back to "assume open crawl." Fix: configure www to directly serve robots.txt (200) with identical content, or ensure the www→non-www redirect fires in the same response as HTTP→HTTPS (one 301, not two sequential ones).

## Meta Robots Tags

### Supported Directives

```html
<!-- Block indexing -->
<meta name="robots" content="noindex">

<!-- Block link following -->
<meta name="robots" content="nofollow">

<!-- Block both -->
<meta name="robots" content="noindex, nofollow">

<!-- Block caching/archiving -->
<meta name="robots" content="noarchive">

<!-- Block snippet in search results -->
<meta name="robots" content="nosnippet">

<!-- Limit snippet length -->
<meta name="robots" content="max-snippet:150">

<!-- Limit image preview size -->
<meta name="robots" content="max-image-preview:large">

<!-- Limit video preview -->
<meta name="robots" content="max-video-preview:30">

<!-- Prevent indexing after a date -->
<meta name="robots" content="unavailable_after: 2026-12-31T23:59:59+00:00">
```

### X-Robots-Tag HTTP Header

Same directives as meta tags but via HTTP headers (useful for PDFs, images, non-HTML):

```
X-Robots-Tag: noindex
X-Robots-Tag: noindex, nofollow
X-Robots-Tag: googlebot: noindex
```

### Directive Precedence

When conflicting directives exist, **the more restrictive one wins** (Google behavior):
- `index` + `noindex` = `noindex`
- `follow` + `nofollow` = `nofollow`

**Warning:** Yandex handles this differently — see Multi-Engine Directive Differences below.

### Multi-Engine Meta Directive Differences

Each search engine handles meta directives differently. These differences can cause a page to be indexed on one engine but blocked on another.

#### Directive Support Matrix

| Directive | Google | Bing | Yandex | Baidu | Applebot |
|-----------|--------|------|--------|-------|----------|
| `noindex` | Yes | Yes | Yes | **NO** | Yes |
| `nofollow` | Yes | Yes | Yes | Yes | Yes |
| `noarchive` | Yes | Yes (+ blocks Copilot) | Yes | Yes | No |
| `nocache` | No | **Yes** (Copilot: URL/title/snippet only) | No | No | No |
| `nosnippet` | Yes | Yes | No | No | Yes |
| `max-snippet` | Yes | Yes | No | No | No |
| `max-image-preview` | Yes | Yes | No | No | No |
| `max-video-preview` | Yes | Yes | No | No | No |
| `unavailable_after` | Yes | No | No | No | No |
| `data-nosnippet` attr | Yes | Yes | No | No | No |
| `data-snippet` attr | No | **Yes** | No | No | No |

#### Engine-Specific Targeting

Use `name` attribute to target specific engines:

```html
<!-- All engines -->
<meta name="robots" content="noindex">

<!-- Google only -->
<meta name="googlebot" content="noindex">

<!-- Bing only -->
<meta name="bingbot" content="noarchive">

<!-- Yandex only -->
<meta name="yandex" content="nofollow">

<!-- Baidu only (noindex NOT supported) -->
<meta name="baiduspider" content="noarchive">

<!-- Apple only -->
<meta name="applebot" content="nosnippet">
```

#### Critical Behavioral Differences

| Behavior | Google | Bing | Yandex | Baidu |
|----------|--------|------|--------|-------|
| Conflicting allow/deny | Restrictive wins | Restrictive wins | **Allow wins** | N/A |
| robots.txt vs meta tag | Independent | Independent | **robots.txt overrides meta** | robots.txt only for blocking |
| `keywords` meta tag | Ignored | Ignored | **Used for relevance** | Used (limited) |
| Applebot fallback | N/A | N/A | N/A | N/A |

**Applebot fallback behavior:** If robots.txt has no `User-agent: Applebot` section but has `User-agent: Googlebot`, Applebot follows Googlebot's rules.

#### Bing AI/Copilot-Specific Directives

Bing has unique directives for controlling content in AI-generated answers:

```html
<!-- Block content from Copilot answers AND AI training -->
<meta name="robots" content="noarchive">

<!-- Limit Copilot to URL/title/snippet only; limit AI training similarly -->
<meta name="robots" content="nocache">

<!-- Exclude specific elements from Copilot citations -->
<div data-nosnippet>Sensitive content here</div>

<!-- Specify preferred text for Copilot to cite -->
<div data-snippet>This is the text Bing should quote.</div>
```

See [bing-copilot-seo.md](bing-copilot-seo.md) for the complete AI/Copilot impact matrix.

#### Baidu: The noindex Problem

Baidu does not support `noindex` in meta tags or HTTP headers. The ONLY way to prevent Baidu from indexing a page:

```
# robots.txt — the only way to block Baidu indexing
User-agent: Baiduspider
Disallow: /private-page/
```

This means a page with `<meta name="robots" content="noindex">` will still be indexed by Baidu unless also blocked in robots.txt.

See [regional-engines.md](regional-engines.md) for full Baidu requirements.

#### Recommended Multi-Engine robots.txt Template

```
User-agent: *
Allow: /
Disallow: /api/
Disallow: /admin/
Disallow: /private/

# Google: adjust crawl rate in Search Console (Crawl-delay not supported)
User-agent: Googlebot
Allow: /

# Bing: Crawl-delay supported
User-agent: Bingbot
Crawl-delay: 1

# Yandex: Crawl-delay and Host supported
User-agent: Yandex
Crawl-delay: 2
Host: https://example.com

# Apple: allow search, block AI training
User-agent: Applebot
Allow: /

User-agent: Applebot-Extended
Disallow: /

# Baidu: remember noindex doesn't work — use Disallow for pages to hide
User-agent: Baiduspider
Allow: /
Disallow: /pages-to-hide-from-baidu/

Sitemap: https://example.com/sitemap.xml
```

## AI Crawler Management: Training vs Search

Most AI companies now separate their crawlers into two distinct types:

1. **AI Search bots** — Power AI-assisted search results (ChatGPT Search, Perplexity, etc.). They index your pages and cite them with a link back. Blocking these removes you from AI search results.
2. **AI Training bots** — Scrape content to train large language models. They consume your content without linking back. Blocking these has zero effect on search visibility.

This means you can **block AI training while keeping your site indexed in AI search products**.

### AI Crawler Reference Table

| Bot | Company | Purpose | Recommendation |
|-----|---------|---------|----------------|
| `Googlebot` | Google | Search indexing (including AI Overviews) | **Allow** |
| `Google-Extended` | Google | Gemini / AI training only | **Block** |
| `GoogleOther` | Google | R&D / non-search products | **Block** |
| `Bingbot` | Microsoft | Bing Search + Copilot indexing | **Allow** |
| `OAI-SearchBot` | OpenAI | ChatGPT Search results (cites sources) | **Allow** |
| `GPTBot` | OpenAI | Model training data collection | **Block** |
| `ChatGPT-User` | OpenAI | Live browsing when users ask ChatGPT to visit a URL | **Block** (scrapes on demand, not indexing) |
| `PerplexityBot` | Perplexity | AI search engine (cites sources) | **Allow** |
| `Applebot` | Apple | Siri, Spotlight, Safari search | **Allow** |
| `Applebot-Extended` | Apple | Apple Intelligence / AI training | **Block** |
| `anthropic-ai` / `ClaudeBot` / `Claude-Web` | Anthropic | Model training | **Block** |
| `CCBot` | Common Crawl | Open dataset used by many AI labs | **Block** |
| `Bytespider` | ByteDance | TikTok / AI training | **Block** |
| `meta-externalagent` / `FacebookBot` | Meta | AI training (Llama models) | **Block** |
| `Amazonbot` | Amazon | Alexa / AI training | **Block** |
| `cohere-ai` | Cohere | Model training | **Block** |
| `Diffbot` | Diffbot | Web scraping / knowledge graph | **Block** |
| `YouBot` | You.com | AI search + training | **Block** (combined training) |
| `img2dataset` | LAION | Image dataset scraping | **Block** |
| `PetalBot` | Huawei | Search / AI training | **Block** |
| `Omgilibot` | Omgili | Discussion scraping | **Block** |

### Key Distinctions

**Google**: `Googlebot` handles all search indexing including AI Overviews (formerly SGE). Blocking `Google-Extended` opts you out of Gemini training but keeps you fully indexed in Google Search and AI Overviews. These are independent controls.

**OpenAI**: `OAI-SearchBot` powers ChatGPT Search — when users search the web through ChatGPT, this bot fetches pages and cites them with links back. `GPTBot` is strictly for training data. Blocking `GPTBot` while allowing `OAI-SearchBot` keeps you in ChatGPT search results without contributing to training.

**Apple**: `Applebot` crawls for Siri, Spotlight, and Safari Suggestions. `Applebot-Extended` collects data for Apple Intelligence features. Block Extended, keep the base bot. Note: if no `Applebot` rule exists in robots.txt, Applebot falls back to `Googlebot` rules.

**Bing/Copilot**: There is no separate Copilot training bot. `Bingbot` handles both Bing Search and Copilot answer indexing. To control what Copilot can quote, use `noarchive`, `nocache`, `data-nosnippet`, and `data-snippet` meta directives instead of robots.txt. See [bing-copilot-seo.md](bing-copilot-seo.md).

### robots.txt Template: Block Training, Allow Search

```
# ── Search engine bots (allow) ─────────────────────────────
User-agent: Googlebot
Allow: /

User-agent: Bingbot
Allow: /

User-agent: YandexBot
Allow: /

User-agent: DuckDuckBot
Allow: /

User-agent: Baiduspider
Allow: /

User-agent: Applebot
Allow: /

# ── AI search bots (allow — they cite sources with links) ──
User-agent: OAI-SearchBot
Allow: /
Disallow: /api/

User-agent: PerplexityBot
Allow: /
Disallow: /api/

# ── AI training crawlers (block) ───────────────────────────
User-agent: GPTBot
Disallow: /

User-agent: ChatGPT-User
Disallow: /

User-agent: Google-Extended
Disallow: /

User-agent: GoogleOther
Disallow: /

User-agent: CCBot
Disallow: /

User-agent: anthropic-ai
Disallow: /

User-agent: ClaudeBot
Disallow: /

User-agent: Claude-Web
Disallow: /

User-agent: Bytespider
Disallow: /

User-agent: Diffbot
Disallow: /

User-agent: FacebookBot
Disallow: /

User-agent: meta-externalagent
Disallow: /

User-agent: Applebot-Extended
Disallow: /

User-agent: Amazonbot
Disallow: /

User-agent: cohere-ai
Disallow: /

User-agent: YouBot
Disallow: /

User-agent: img2dataset
Disallow: /

User-agent: PetalBot
Disallow: /

User-agent: Omgilibot
Disallow: /

Sitemap: https://example.com/sitemap.xml
```

### Next.js Implementation

For Next.js App Router projects using `app/robots.ts`:

```typescript
import { MetadataRoute } from 'next';

export default function robots(): MetadataRoute.Robots {
  const commonDisallow = ['/api/', '/auth/', '/admin/'];

  return {
    rules: [
      { userAgent: '*', allow: '/', disallow: ['/private/', '/_next/', ...commonDisallow] },
      { userAgent: 'Googlebot', allow: '/', disallow: commonDisallow },
      { userAgent: 'Bingbot', allow: '/', disallow: commonDisallow },

      // AI search bots — index for AI-powered search, cite with links
      ...['PerplexityBot', 'OAI-SearchBot'].map(bot => ({
        userAgent: bot,
        allow: '/' as const,
        disallow: ['/api/'],
      })),

      // AI training crawlers — block scraping for model training
      ...[
        'GPTBot', 'ChatGPT-User', 'CCBot', 'anthropic-ai', 'Claude-Web',
        'ClaudeBot', 'Bytespider', 'Diffbot', 'FacebookBot', 'Google-Extended',
        'GoogleOther', 'Applebot-Extended', 'Amazonbot', 'cohere-ai',
        'meta-externalagent', 'img2dataset', 'PetalBot', 'YouBot', 'Omgilibot',
      ].map(bot => ({ userAgent: bot, disallow: '/' as const })),
    ],
    sitemap: 'https://example.com/sitemap.xml',
  };
}
```

### Verifying Your Setup

After deploying, confirm the generated robots.txt is correct:

```bash
curl -s https://example.com/robots.txt | head -80
```

Check that:
- `GPTBot`, `Google-Extended`, `ClaudeBot`, etc. show `Disallow: /`
- `OAI-SearchBot`, `PerplexityBot` show `Allow: /`
- `Googlebot`, `Bingbot` show `Allow: /`

### Staying Current

AI companies frequently launch new crawlers. Monitor your server logs for unfamiliar bot user-agents and check announcements from OpenAI, Google, Anthropic, Meta, and Apple for new bot names. When a new training crawler appears, add it to the block list. When a new AI search bot appears that cites sources, add it to the allow list.

## Canonical URL Implementation

### Methods (strongest to weakest)

1. **301 Redirect** — Strongest signal, use when removing a duplicate permanently
2. **`rel="canonical"` HTML tag** — Strong signal, use when both URLs must remain accessible
3. **`Link:` HTTP header** — Same as HTML tag, for non-HTML resources
4. **Sitemap inclusion** — Weak signal, supplementary only

### Implementation Rules

```html
<!-- CORRECT: Absolute HTTPS URL -->
<link rel="canonical" href="https://example.com/page/">

<!-- WRONG: Relative URL -->
<link rel="canonical" href="/page/">

<!-- WRONG: HTTP instead of HTTPS -->
<link rel="canonical" href="http://example.com/page/">
```

**Rules:**
- Always use absolute URLs with HTTPS
- Every page should have a self-referencing canonical (even if not a duplicate)
- Canonical target must return 200 status
- Canonical target should not be `noindex`
- Canonical target should not redirect
- Only one canonical tag per page
- Place in `<head>`, not in `<body>`
- If using JavaScript to inject canonical, ensure it's in initial server-rendered HTML

### Common Canonical Conflicts to Resolve

| Conflict | Resolution |
|----------|------------|
| `www` vs `non-www` | 301 redirect one to the other, canonical on preferred |
| `http` vs `https` | 301 redirect HTTP to HTTPS, canonical on HTTPS |
| Trailing slash vs no slash | Pick one, redirect the other, canonical on preferred |
| URL parameters (`?ref=`, `?utm_`) | Canonical to clean URL without parameters |
| Pagination (`/page/2/`) | Each page canonicalizes to itself (NOT to page 1) |
| AMP vs regular | AMP canonicalizes to regular version |

## JavaScript SEO

### Server-Side Rendering (SSR) Checklist

For frameworks like Next.js, Nuxt, SvelteKit, Astro, Remix:

1. `<title>` and `<meta name="description">` present in initial server HTML
2. `<link rel="canonical">` present in initial server HTML
3. Structured data (JSON-LD) present in initial server HTML
4. Main content text present in initial server HTML
5. Internal links are real `<a href="...">` tags (not JS click handlers)
6. Proper HTTP status codes (404 for missing pages, not client-side redirect)
7. `<meta name="robots">` directives in initial server HTML

### Dynamic Rendering

If full SSR isn't possible, use dynamic rendering to serve pre-rendered HTML to bots:

- Detect Googlebot user-agent at the server/CDN level
- Serve pre-rendered static HTML to crawlers
- Serve normal SPA to users
- Google officially supports this as a workaround (not cloaking)
- Tools: Rendertron, Prerender.io, Puppeteer-based solutions

### Common JS SEO Failures

| Problem | Symptom | Fix |
|---------|---------|-----|
| Client-only rendering | "Discovered not indexed" or "Indexed without content" | Implement SSR/SSG |
| Hash routing (`#/page`) | Only homepage indexed | Use History API routing |
| Lazy-loaded content | Content not in Google's index | Ensure content loads without user interaction |
| JS-injected canonical | Google ignores it | Put canonical in server-rendered HTML |
| Click-based navigation | Google can't discover pages | Use standard `<a href>` links |
| Infinite scroll only | Deep content never crawled | Add pagination with `<a>` links |

## International SEO (hreflang)

For multilingual or multi-regional sites:

```html
<link rel="alternate" hreflang="en" href="https://example.com/page/">
<link rel="alternate" hreflang="es" href="https://example.com/es/page/">
<link rel="alternate" hreflang="fr" href="https://example.com/fr/page/">
<link rel="alternate" hreflang="x-default" href="https://example.com/page/">
```

**Rules:**
- Every page must reference all alternate versions including itself
- `x-default` specifies the fallback for unmatched languages
- Use ISO 639-1 language codes, optionally with ISO 3166-1 region codes
- Can be implemented via HTML `<link>`, HTTP headers, or sitemap
- Canonical URL should be within the same language group

## HTTPS and Security

- All pages must be served over HTTPS (ranking signal)
- HTTP URLs should 301 redirect to HTTPS
- Use HSTS headers: `Strict-Transport-Security: max-age=31536000; includeSubDomains`
- Ensure SSL certificate is valid and covers all subdomains
- Mixed content (HTTP resources on HTTPS pages) can cause rendering issues

## URL Structure Best Practices

```
GOOD:
https://example.com/blog/seo-guide/
https://example.com/products/blue-widget/

BAD:
https://example.com/page?id=123&cat=4
https://example.com/blog/2026/01/15/post-title (unnecessary date in URL)
https://EXAMPLE.COM/Blog/SEO-Guide/ (mixed case)
```

**Rules:**
- Lowercase only
- Hyphens between words (not underscores)
- No unnecessary parameters
- Keep URLs short and descriptive
- Max 3 directory levels from root
- Trailing slash: pick one convention, be consistent
- Use HTTPS always

## Stale Metadata Audit

Metadata written once tends to drift from reality as the site evolves. Run this audit whenever content changes significantly (new evidence added, records updated, archives added).

### Fields That Go Stale

| Field | Where it lives | Drift cause |
|-------|---------------|-------------|
| `dateModified` / `SITE_CONTENT_UPDATED` | layout.tsx, JSON-LD | Not bumped after content updates |
| File/record counts in titles | page metadata | Data grows but title string is hardcoded |
| Archive counts ("18 sealed archives") | vault layout | New ZIPs added without updating copy |
| Buyer/record counts ("1,200+ buyers") | wall-of-shame metadata | Registry grows but metadata isn't updated |
| "X+ files verified" in vault title | vault layout | Evidence vault grows over time |

### Audit Process

1. **Find all hardcoded numbers in metadata** — grep titles and descriptions for digits:
   ```bash
   grep -rn "title:\|description:" src/app --include="*.tsx" --include="*.ts" | grep -E '[0-9]+'
   ```

2. **Cross-reference against actual data**:
   - File counts → count actual files in the directory
   - Record counts → `JSON.parse(fs.readFileSync(...)).length`
   - Archive counts → count ZIPs in evidence directory
   - Dates → compare to last git commit date or last data update

3. **Check `dateModified` in JSON-LD** — it should match the most recent substantive content change, not the initial launch date

### Next.js App Router Pattern — Centralized Content Date

Avoid scattered date literals. Define a single constant and bump it when content changes:

```typescript
// app/layout.tsx
/** Bump when substantive content changes (SEO freshness signal + JSON-LD dateModified). */
const SITE_CONTENT_UPDATED = "2026-04-23T00:00:00.000Z";

export const metadata: Metadata = {
  other: {
    "article:modified_time": SITE_CONTENT_UPDATED,
  },
};

const jsonLd = {
  "@type": "NewsArticle",
  dateModified: SITE_CONTENT_UPDATED,
  // ...
};
```

One variable to update, propagates everywhere. Failure to bump this means Google sees a stale `dateModified` and may deprioritize the page in freshness-sensitive queries.

### Next.js App Router — Client vs Server Component Metadata Constraint

`export const metadata` can only appear in **server components**. This affects where you place page-level SEO:

| Component type | Has `"use client"`? | Where to put metadata |
|---------------|--------------------|-----------------------|
| Server page | No | Directly in `page.tsx` |
| Client page | Yes | In a sibling `layout.tsx` (server component) |
| Mixed (server shell + client child) | Child has `"use client"` | In parent server component |

**Pattern for client-component pages:**

```
app/
  wall-of-shame/
    layout.tsx   ← export const metadata here (server component)
    page.tsx     ← "use client" here, no metadata export
```

```typescript
// app/wall-of-shame/layout.tsx  (server component)
import type { Metadata } from "next";

export const metadata: Metadata = {
  title: "Wall of Shame — Sex Buyer Registry",
  description: "...",
  alternates: { canonical: "https://example.com/wall-of-shame" },
};

export default function Layout({ children }: { children: React.ReactNode }) {
  return children;
}
```

**Quick check:** If a page file starts with `"use client"` and also tries to `export const metadata`, Next.js will silently ignore the metadata — no build error, just no SEO. Always verify with View Page Source that meta tags appear in the initial HTML.

## Analytics & Conversion Tags Under a Strict CSP

A Content-Security-Policy has **two** directives that matter for every tracking tag, and whitelisting a host in only one of them breaks measurement silently:

| Directive | Allows | Symptom when missing |
|-----------|--------|----------------------|
| `script-src` | the tag library itself to load | tag never appears — obvious in a network log |
| `connect-src` | the tag to **send** its beacon (fetch/XHR/`sendBeacon`) | tag loads, reports healthy, data never arrives, nothing on the page explains it |

Google Ads, GA4, Meta and X pixels all post to hosts that belong in `connect-src` — `googleadservices.com`, `www.google-analytics.com` / `region1.google-analytics.com`, `connect.facebook.net`, `analytics.twitter.com`. Listing them in `script-src` only is the common miss.

**Checklist:**
- Add each tag host to **both** `script-src` and `connect-src` (and `img-src` for image-pixel fallbacks such as Meta's).
- One sale, one conversion. If GA4 `purchase` *and* a separate Ads conversion action are both configured, either fire the Ads action on the event the action is defined on and demote the GA4-imported action to secondary, or expect every sale counted twice. Share one `transaction_id` across destinations so a duplicate is discarded rather than double-counted.
- The conversion event name belongs to the ad platform, not to you. A typo does not raise — it leaves the action at "no recent conversions" while the tag reports healthy, so keep the name in one constant carrying that warning.
- Verify in a **real browser against a production build**: the served tag must carry the destination id, exactly one conversion must fire per purchase, and a refresh must not repeat it. Any synthetic purchase from a CI/verification script needs a flag that keeps it out of the live ad account.

