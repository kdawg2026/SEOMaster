# SEO Audit Checklist

Comprehensive checklist for auditing a web project. Work through each section systematically.

## Pre-Audit: Project Discovery

```
- [ ] Identify tech stack (static, SSR, SPA, CMS)
- [ ] Identify build/deploy pipeline
- [ ] Locate HTML templates / layout files
- [ ] Locate existing SEO configuration (meta tags, sitemap, robots.txt)
- [ ] Identify all page types (homepage, listing pages, detail pages, blog, etc.)
- [ ] Check for existing structured data
- [ ] Check for existing analytics (GA4, Search Console)
```

## 1. Crawlability

```
- [ ] robots.txt exists at site root
- [ ] robots.txt does not block important pages
- [ ] robots.txt does not block CSS/JS resources
- [ ] robots.txt includes Sitemap directive
- [ ] No URL is both `Disallow`ed and named in structured data / navigation schema
- [ ] Googlebot can access all important pages (no auth/firewall blocks)
- [ ] Server responds within 200ms for most pages
- [ ] No redirect loops or long redirect chains
- [ ] HTTPS is enforced (HTTP -> HTTPS redirect)
- [ ] www/non-www is redirected (pick one)
- [ ] Trailing slash preference is consistent
```

## 2. XML Sitemap

```
- [ ] sitemap.xml exists and is valid XML
- [ ] Only canonical URLs included (no redirects, no noindex pages)
- [ ] After a URL rename, old URLs removed and new URLs added in the same commit
- [ ] All important pages are included
- [ ] <loc> uses absolute HTTPS URLs
- [ ] <lastmod> dates reflect actual content changes
- [ ] Sitemap has < 50,000 URLs and < 50MB per file
- [ ] Sitemap index used for larger sites
- [ ] Sitemap submitted to Google Search Console
- [ ] Sitemap submitted to Bing Webmaster Tools
- [ ] Sitemap submitted to Yandex Webmaster
- [ ] Sitemap referenced in robots.txt
- [ ] Sitemap and navigation schema list the same crawlable URLs — nothing disallowed appears in either
```

## 3. On-Page SEO (per page type)

```
- [ ] Unique <title> tag (50-60 chars, primary keyword first)
- [ ] Unique <meta name="description"> (150-160 chars, compelling)
- [ ] <link rel="canonical"> present (absolute HTTPS, self-referencing)
- [ ] Single <h1> tag containing primary keyword
- [ ] Logical heading hierarchy (h1 > h2 > h3, no skips)
- [ ] <meta name="viewport"> present
- [ ] <meta charset="utf-8"> present
- [ ] <html lang="xx"> attribute set correctly
- [ ] Images have descriptive alt attributes
- [ ] Images use modern formats (WebP/AVIF) with fallbacks
- [ ] Images have explicit width/height attributes (prevents CLS)
- [ ] Job/careers pages: one posting per URL, visible location + compensation, `validThrough` maintained
- [ ] Application/form pages: indexation decided deliberately (indexable unique content, or `noindex` **and** out of the sitemap)
```

## 4. Open Graph & Social

```
- [ ] og:title present on all pages
- [ ] og:description present on all pages
- [ ] og:image present (min 1200x630px recommended)
- [ ] og:url present (matches canonical)
- [ ] og:type present (website, article, product, etc.)
- [ ] og:site_name present
- [ ] twitter:card present (summary_large_image)
- [ ] twitter:title present
- [ ] twitter:description present
- [ ] twitter:image present
```

## 5. Structured Data

```
- [ ] JSON-LD format used (not Microdata/RDFa)
- [ ] WebSite schema on homepage
- [ ] Organization schema on homepage (or about page)
- [ ] BreadcrumbList on all inner pages
- [ ] Content-specific schema per page type (Article, Product, FAQ, etc.)
- [ ] `JobPosting` on careers pages (title, description, datePosted, hiringOrganization, and jobLocation or TELECOMMUTE + applicantLocationRequirements)
- [ ] `JobPosting` has `validThrough`; closed roles are updated or removed
- [ ] All required properties present (no Rich Results Test errors)
- [ ] Recommended properties added where possible
- [ ] Structured data matches visible page content
- [ ] Every `url` in `SiteNavigationElement`/`BreadcrumbList` is allowed by robots.txt (human-only, disallowed links omitted)
- [ ] Tested with Google Rich Results Test
- [ ] Tested with Schema.org Validator
```

## 6. Internal Linking

```
- [ ] Every important page reachable within 3 clicks from homepage
- [ ] No orphan pages (pages with zero internal links)
- [ ] Descriptive anchor text (not "click here" or "read more")
- [ ] Standard <a href> tags used (not JS click handlers)
- [ ] No broken internal links (404s)
- [ ] Breadcrumb navigation present and matches structured data
- [ ] Footer/header navigation covers key pages
- [ ] Related content links on detail pages
```

## 7. URL Structure

```
- [ ] All URLs are lowercase
- [ ] Hyphens used between words (not underscores)
- [ ] URLs are descriptive and keyword-relevant (no developer-facing jargon in public slugs)
- [ ] Max 3 directory levels from root
- [ ] No unnecessary URL parameters
- [ ] Consistent trailing slash convention
- [ ] No duplicate content from URL variations (params, www, trailing slash)
```

### Renamed / moved URLs

```
- [ ] Permanent redirect (301/308) exists for every old path — not 302/307
- [ ] One hop only (no redirect chains); destination is the closest equivalent page, not the homepage
- [ ] Old path is NOT blocked in robots.txt (a blocked URL's redirect is never fetched)
- [ ] Old URLs removed from the sitemap; new URLs added
- [ ] Canonical on each moved page re-pointed to the new URL (circular canonicals de-index)
- [ ] Internal links (nav, footer, breadcrumbs, in-body) updated to the new path
- [ ] Backend/API paths deliberately left unchanged
- [ ] Old path verified in production (curl -I) returns the permanent status and correct Location
- [ ] Redirect is permanent, not scheduled for cleanup
```

## 8. Performance (Core Web Vitals)

```
- [ ] LCP < 2.5 seconds
- [ ] INP < 200 milliseconds
- [ ] CLS < 0.1
- [ ] Hero/above-fold images preloaded
- [ ] Below-fold images lazy-loaded
- [ ] CSS/JS minified and bundled
- [ ] Render-blocking resources minimized
- [ ] Font loading optimized (font-display: swap)
- [ ] Compression enabled (gzip/brotli)
- [ ] CDN used for static assets
```

## 9. JavaScript SEO (if SPA/SSR)

```
- [ ] SSR or SSG is enabled (not pure client-side rendering)
- [ ] <title>, <meta description>, <canonical> in server-rendered HTML
- [ ] Main content present in server-rendered HTML (View Source test)
- [ ] Structured data in server-rendered HTML
- [ ] History API routing (not hash-based)
- [ ] Proper HTTP status codes (real 404s, not soft 404s)
- [ ] Links are <a href> tags, not JS-only navigation
- [ ] Dynamic rendering configured if full SSR isn't possible
```

## 10. Security & HTTPS

```
- [ ] Valid SSL certificate on all domains/subdomains
- [ ] HTTP to HTTPS redirect (301)
- [ ] No mixed content (HTTP resources on HTTPS pages)
- [ ] HSTS header present
- [ ] Analytics/conversion tag hosts allowed in CSP `connect-src` as well as `script-src` (a load-only allow-list blocks the beacon silently)
- [ ] Secure cookies (Secure, HttpOnly, SameSite flags)
```

## 11. Multi-Engine Optimization

### Core Engines (All Projects)

```
- [ ] Google Search Console set up and verified
- [ ] Bing Webmaster Tools set up and verified
- [ ] Yandex Webmaster set up and verified
- [ ] IndexNow API key configured and hosted (covers Bing, Yandex, Naver, Seznam.cz, Yep)
- [ ] Bing meta verification tag present (<meta name="msvalidate.01">)
- [ ] Yandex meta verification tag present (<meta name="yandex-verification">)
- [ ] Crawl-delay set for Bing/Yandex in robots.txt (if needed)
- [ ] Host directive in robots.txt for Yandex (if needed)
```

### Bing Copilot / AI Grounding

```
- [ ] AI/Copilot control directives reviewed and set (noarchive / nocache as appropriate)
- [ ] data-nosnippet used on sensitive content sections
- [ ] data-snippet used on preferred citation text
- [ ] Content is clear, verifiable, and self-contained (improves grounding eligibility)
- [ ] Each URL focused on a single topic
- [ ] Key information placed early on page (not buried below long intros)
- [ ] Entities named clearly and consistently (people, orgs, products)
- [ ] No prompt injection or AI manipulation in page content
- [ ] Bing AI Dashboard monitored for grounding citations
```

### Apple (Applebot)

```
- [ ] Applebot not unintentionally blocked (check Googlebot fallback behavior)
- [ ] Explicit User-agent: Applebot section in robots.txt (or permissive Googlebot rules)
- [ ] Applebot-Extended controlled if opting out of AI training
- [ ] CSS/JS not blocked from Applebot (needed for rendering)
- [ ] Meta robots directives compatible with Applebot (noindex, nosnippet, nofollow)
```

### Baidu (China Market)

```
- [ ] ICP License obtained (legally required for mainland China hosting)
- [ ] Website hosted in mainland China (not HK/Macau/Taiwan)
- [ ] .CN domain used (strong ranking signal)
- [ ] Chinese mobile number registered for Baidu Webmaster Tools
- [ ] Site verified in Baidu Webmaster Tools (ziyuan.baidu.com)
- [ ] Sitemap submitted to Baidu (XML or TXT format)
- [ ] ALL content server-rendered (Baiduspider cannot execute JavaScript)
- [ ] noindex meta tag NOT relied upon (Baidu ignores it — use robots.txt)
- [ ] Dead links proactively submitted to Baidu
- [ ] robots.txt configured for Baiduspider
- [ ] Page load time ≤2 seconds
- [ ] Mobile-optimized design
```

### Naver (South Korea Market)

```
- [ ] Site registered in Naver Search Advisor (searchadvisor.naver.com)
- [ ] HTML verification meta tag added (<meta name="naver-site-verification">)
- [ ] Sitemap submitted to Naver
- [ ] RSS feed submitted to Naver (if applicable)
- [ ] Structured data for Naver-supported types (FAQ, breadcrumbs, recipes, jobs, etc.)
- [ ] IndexNow configured (Naver participates)
- [ ] Crawl requests submitted for key pages
- [ ] Korean language content present for Korean market
```

### Seznam.cz (Czech Market)

```
- [ ] Site verified in Seznam Webmaster Tools (reporter.seznam.cz/wm/)
- [ ] Sitemap submitted to Seznam
- [ ] IndexNow configured (Seznam participates)
- [ ] Content server-rendered (SeznamBot prefers static HTML)
- [ ] SeznamBot not blocked in robots.txt
- [ ] Czech language content present for Czech market
```

## 12. Content Quality

```
- [ ] No thin content pages (pages with < 300 words of unique content)
- [ ] No duplicate content across pages
- [ ] Content is original and provides unique value
- [ ] Spelling and grammar are correct
- [ ] Content addresses user search intent
- [ ] Author information present where relevant (E-E-A-T)
- [ ] Fresh content (recent dates, updated information)
- [ ] User-visible freshness claims ("last updated", date ranges, row counts) served live or via a single revalidation layer — stacked caches (ISR + unstable_cache on serverless/CDN) can pin them weeks stale
- [ ] Dataset/source expansions reflected in a dated changelog or about entry (new source names are long-tail keywords; dated entries are freshness + E-E-A-T signals); resubmit sitemap / ping IndexNow after
- [ ] Every advertised data date range is reachable from a named, documented endpoint; ranges spanning multiple sources carry their caveat in the data model, not only in the page copy (so an AI assistant quoting the same rows cannot overstate it)
```

## 13. Accessibility (SEO-relevant)

```
- [ ] Semantic HTML elements used (nav, main, article, section, aside)
- [ ] All images have alt attributes
- [ ] Skip navigation links present
- [ ] Proper ARIA labels where needed
- [ ] Color contrast meets WCAG AA
- [ ] Keyboard navigable
```

## Post-Audit: Monitoring

```
- [ ] Google Search Console Page Indexing report reviewed
- [ ] Bing Webmaster Tools URL Inspection and SEO Reports reviewed
- [ ] Yandex Webmaster Indexing Status and Diagnostics reviewed
- [ ] All indexing errors documented and prioritized (across all engines)
- [ ] Fix plan created with priority order (Critical > High > Medium > Low)
- [ ] Re-indexing requested via Google URL Inspection, Bing URL Inspection, and Yandex Reindex tool
- [ ] URL changes submitted via IndexNow (covers Bing, Yandex, Naver, Seznam)
- [ ] Bing AI Dashboard reviewed for Copilot grounding/citation metrics
- [ ] Monitoring schedule set (weekly check across GSC, Bing WMT, Yandex WM)
- [ ] Performance baseline recorded for comparison
```

## Priority Order for Fixes

1. **Critical** — Anything blocking indexing of important pages
2. **High** — Missing/broken meta tags, duplicate content, broken structured data
3. **Medium** — Performance issues, weak internal linking, missing social tags
4. **Low** — Minor schema improvements, content enhancements, polish
