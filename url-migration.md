# Renaming or Moving an Indexed URL

A URL rename looks like a file rename and is not one. The filesystem move is the trivial
half; the half that decides whether the page keeps its rankings is everything that
referenced the old path — the redirect, the sitemap, the canonical, the navigation, the
inbound links you don't control, and the API endpoints that must **not** move.

Get it wrong and Google reports the old URL as a 404, drops the page from the index, and
the replacement starts from zero with no link equity and no history. Get it right and the
new URL inherits the old one's standing within a crawl or two.

## Case study: `/referral` → `/jobs` (Next.js App Router)

A commission-only sales role had been published at `/referral` — jargon that no candidate
searches for — and was indexed, in the sitemap, and linked from the main navigation. The
rename to `/jobs` touched five things, and they had to ship **together**:

| Change | File | Why |
|---|---|---|
| 308 permanent redirects for all three old paths | `next.config.ts` | `/referral` was indexed; a deletion would 404 every inbound link |
| Sitemap URLs swapped `/referral*` → `/jobs*` | `app/sitemap.ts` | A sitemap that lists a redirecting URL is the "Page with redirect" report, self-inflicted |
| Nav link updated | `components/Navbar.tsx` | Internal links must point at the destination, not bounce through the redirect |
| Per-page canonicals re-pointed | `app/jobs/page.tsx`, `app/jobs/apply/page.tsx`, `app/jobs/agreement/page.tsx` | Each page carries an **absolute** canonical; a moved page that still canonicalises to the old URL de-indexes itself |
| Page directory moved, API route left alone | `app/jobs/*`, `app/api/referral/apply/route.ts` | `/api/referral/apply` is a backend endpoint, not a public URL — see below |

The redirects were declared as three explicit paths rather than one wildcard, because the
three pages were the entire public surface and an explicit list is auditable:

```ts
// next.config.ts — Next.js serves `permanent: true` as HTTP 308
{
  source: '/referral',
  destination: '/jobs',
  permanent: true,
},
{
  source: '/referral/apply',
  destination: '/jobs/apply',
  permanent: true,
},
{
  source: '/referral/agreement',
  destination: '/jobs/agreement',
  permanent: true,
},
```

**Next.js detail worth knowing:** in `redirects()`, `permanent: true` emits **308**, not
301, and `permanent: false` emits **307**. Both 308 and 301 are treated by Google as
permanent and pass signals; 308 additionally preserves the request method. What is *not*
acceptable is a 302/307 for a permanent rename — Google reads a temporary redirect as
"this will come back", keeps the old URL as canonical, and never consolidates.

## The rename checklist

1. **Add the permanent redirect — before or in the same deploy as the move.** One hop,
   old path → final path. Never chain (`/a → /b`, then later `/b → /c`) and never blanket
   the old section to the homepage: a mass redirect to an unrelated page is a soft 404.
2. **Keep the old path out of `robots.txt`.** A disallowed URL can't be fetched, so Google
   never sees its redirect and never learns about the move. Redirect + allow is the
   correct pair.
3. **Swap the sitemap in the same commit.** The sitemap lists canonical URLs only. Leaving
   the old URL in means submitting a redirect; leaving the new one out means the
   replacement waits for a crawl it didn't have to wait for.
4. **Update every internal link** — nav, footer, breadcrumbs, in-body links, related
   content. Internal links are the strongest, fastest signal you control, and pointing
   them through a redirect wastes it.
5. **Re-point the canonical on each moved page.** In framework code, canonicals are
   usually hard-coded absolute strings, so a directory move silently leaves them behind —
   the page then declares itself a duplicate of a URL that now redirects to it, which is a
   circular canonical and often ends in de-indexing.
6. **Leave backend/API paths where they are.** `/api/referral/apply` kept its path through
   the rename. The public page slug is a contract with search engines; an internal API
   path is a contract with your own in-flight requests, open form tabs, and any client
   already posting to it. Renaming it for cosmetic consistency buys nothing and breaks
   submissions.
7. **Keep the redirect indefinitely.** A redirect is not a migration step to be cleaned up
   later; it is what makes other people's links keep working. Removing it in a "tidy-up"
   re-breaks every inbound link and every bookmark.
8. **Keep it out of the build's caching.** Verify against production, not a local server:
   the old URL must answer with the redirect status in the deployed environment, where
   middleware, CDN and framework config actually run.

## What to expect in Search Console afterward

- The old URLs move into **Page with redirect**, which is a *valid*, healthy bucket — it
  means Google followed the redirect and consolidated on the destination. It is not an
  error to "fix", and it does not need to reach zero.
- The new URLs initially appear under **Discovered/Crawled – currently not indexed** while
  they earn their first render.
- Request indexing for the **new** URLs (URL Inspection → Request Indexing), not the old
  ones — the old URLs' job is only to redirect.
- There is no path-level "Change of Address" tool. Google's Change of Address is for
  domain/site moves; a path rename is communicated by the redirect itself plus the
  updated sitemap and internal links.
- If a "Not found (404)" row appears for an old URL weeks later, the redirect is missing,
  mis-scoped (trailing slash, case, `.html`), or blocked — check that before assuming
  Google ignored it.

## Choosing the new slug

Rename for a reason, not for taste: the URL had jargon (`/referral`), a typo, an
underscore, a migration artifact, or it targeted the wrong intent. A rename now costs a
redirect that must live forever, so spend it on a slug that reads as what a person would
search: `/jobs` over `/referral`, `/silver-inventory` over `/silv-inv`.

## Common mistakes

| Mistake | Impact | Fix |
|---|---|---|
| Old URL deleted with no redirect | 404s, all inbound equity lost | Permanent redirect (301/308) per old path |
| 302/307 used for a permanent rename | Google keeps the old URL as canonical; no consolidation | Use 301/308 |
| Old URL still in the sitemap | Submits a redirect; wastes crawl budget | Sitemap lists canonical URLs only |
| Canonical still pointing at the old URL | Circular canonical, possible de-indexing | Re-point every moved page's canonical |
| Old URL blocked in robots.txt | Redirect is unfetchable, move never discovered | Redirect + allow crawling |
| Internal links left on the old path | Slowest possible consolidation | Update nav, footer, and in-body links |
| Redirect chain or mass redirect to homepage | Diluted signals, soft 404 | One hop to the closest equivalent page |
| Redirect removed after a few months | Inbound links break again | Keep it permanently |

## Related

- Per-page canonical and metadata handling: [technical-seo.md](technical-seo.md)
- GSC "Page with redirect" and 404 buckets: [indexing-errors.md](indexing-errors.md)
- Post-deploy resubmission steps: [SKILL.md](SKILL.md) → Step 11
- Sitemap rules: [audit-checklist.md](audit-checklist.md) → §2, §7
