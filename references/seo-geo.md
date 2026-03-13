# SEO & GEO (Generative Engine Optimization) Checklist

*Only applicable if the site has public-facing pages that should be discoverable via search engines or AI assistants.*

---

## Technical SEO

### Crawlability & Indexing
- Every public page is rendered server-side (SSR) or statically generated (SSG). Client-side-only rendered pages are invisible to most crawlers.
- `robots.txt` exists at site root. Allows indexing of public pages, blocks private/admin/API routes, points to sitemap.
- XML sitemap exists, auto-generated, includes all public URLs with `<lastmod>` dates. Submitted to Google Search Console and Bing Webmaster Tools.
- No orphan pages (pages with no internal links pointing to them). Every important page reachable within 3 clicks from the homepage.
- Proper HTTP status codes: 200 for live pages, 301 for permanent redirects (never redirect chains), 404 for genuinely removed content, 410 for permanently deleted content. No soft 404s (200 status on empty/error pages).
- Pagination uses `rel="next"` / `rel="prev"` or infinite scroll with crawlable fallback (paginated URLs).
- No duplicate content. If the same content exists at multiple URLs, canonical tags point to the primary version.
- JavaScript-rendered content is verified as crawlable: test with Google's URL Inspection tool or by viewing the page with JS disabled.

### URL Structure
- Clean, human-readable URLs using slugs: `/blog/how-to-optimize-performance` not `/page?id=a8f3b2`.
- Consistent URL patterns across the site. No mixed conventions (some with trailing slash, some without).
- URLs are lowercase. No spaces, underscores, or special characters (hyphens only for word separation).
- URL depth is shallow: 3-4 levels max. Deep nesting signals low importance to crawlers.
- HTTPS-only. HTTP → HTTPS 301 redirect. No mixed content warnings.

### Page-Level Meta
- Every page has a unique `<title>` tag (50-60 characters). Includes the primary keyword naturally. Most important words first.
- Every page has a unique `<meta name="description">` (150-160 characters). Compelling, includes primary keyword, acts as ad copy in search results.
- `<link rel="canonical" href="...">` on every page, pointing to itself (or the canonical version if duplicated).
- `<html lang="en">` (or appropriate language code) set on the root element.
- No `<meta name="robots" content="noindex">` on pages that should be indexed (verify this isn't accidentally set by a framework default).

### Performance (SEO-Specific)
- Core Web Vitals passing: LCP < 2.5s, INP < 200ms, CLS < 0.1. Google uses these as a ranking signal.
- Mobile-friendly (responsive design). Google uses mobile-first indexing — the mobile version is what gets ranked.
- Page load time < 3 seconds on 3G. Slow pages get deprioritized.
- No render-blocking resources in the critical path. CSS inlined or loaded async. JS deferred.
- Images: compressed, correctly sized, lazy-loaded below fold, `width` and `height` attributes set (prevents CLS).

---

## Content & On-Page SEO

### Heading Hierarchy
- One `<h1>` per page containing the primary keyword. Describes what the page is about.
- `<h2>` tags for major sections. `<h3>` for subsections. Sequential, no skipping levels.
- Headings are descriptive and keyword-relevant, not generic ("Our Services" → "Web Development Services for Startups").

### Content Quality Signals
- Every important page has substantial, unique content (not thin pages with 50 words and a form).
- Content answers the user's intent: informational pages inform, transactional pages enable transactions, navigational pages help users find what they need.
- Internal linking: every important page is linked to from relevant contextual content (not just the nav). Use descriptive anchor text (not "click here").
- Images have descriptive `alt` text that includes relevant keywords naturally (not keyword-stuffed).
- No hidden text, cloaking, or other deceptive practices.

### Structured Data (JSON-LD)
- Add JSON-LD structured data where applicable. Validate with Google's Rich Results Test:
  - **Organization:** Company name, logo, social profiles, contact info.
  - **WebSite:** Name, URL, search action (for sitelinks searchbox).
  - **BreadcrumbList:** Breadcrumb navigation on every page with hierarchy.
  - **Article / BlogPosting:** For blog content — headline, author, datePublished, dateModified, image.
  - **Product:** For product pages — name, description, price, availability, reviews, SKU.
  - **FAQ:** For FAQ sections — question and answer pairs. High GEO value.
  - **HowTo:** For tutorial/guide content — step-by-step instructions with names and text.
  - **LocalBusiness:** If applicable — address, hours, phone, geo coordinates.
  - **Review / AggregateRating:** For products or services with ratings.
- Structured data is accurate and matches visible page content (no schema markup for content not on the page).

### Internal Linking Architecture
- Important pages receive the most internal links. Link equity flows from high-authority pages (homepage, popular content) to pages that need ranking support.
- Contextual links within body content (not just nav/footer) pass the most value.
- Anchor text is descriptive and varied — not the same keyword phrase every time.
- No broken internal links (404s). Crawl the site to find and fix.
- Flat site architecture: important pages within 3 clicks of homepage.

---

## Social & Sharing (Open Graph / Twitter Cards)

- Every shareable page has:
  - `og:title` — compelling, may differ slightly from `<title>` for social context
  - `og:description` — social-optimized summary
  - `og:image` — 1200×630px minimum, absolute URL, visually compelling
  - `og:url` — canonical URL
  - `og:type` — `website`, `article`, `product` as appropriate
  - `og:site_name` — brand name
- Twitter Card tags: `twitter:card` (`summary_large_image`), `twitter:title`, `twitter:description`, `twitter:image`
- Test by pasting URLs into Twitter, LinkedIn, Slack, WhatsApp, iMessage. Preview must render correctly.
- `og:image` is not a generic logo on every page — it's contextual (article image, product image, etc.).

---

## GEO — Generative Engine Optimization

GEO optimizes content for AI systems (ChatGPT, Claude, Perplexity, Google AI Overviews) that synthesize answers from web content. This is distinct from traditional SEO and is increasingly important for discoverability.

### Content Structure for AI Citability
- Write in clear, direct prose that answers questions definitively. AI systems prefer content that makes specific, quotable claims over vague or hedging language.
- Use a question-and-answer format where natural. AI assistants often pull from content that directly answers the query. FAQ sections are high-value.
- Define entities and terms explicitly. If your page is about a concept, product, or service, state clearly what it is in the first paragraph: "[Product] is a [category] that [does what]."
- Include statistics, data points, and specific numbers. AI systems cite quantified claims more often than qualitative ones.
- Attribution and sourcing: reference credible sources where relevant. AI systems preferentially cite content that itself cites authoritative sources.

### Topic Authority & Comprehensiveness
- Cover topics comprehensively. AI systems prefer authoritative, in-depth content over thin pages. A single comprehensive guide outranks five thin pages on subtopics.
- Build topic clusters: a pillar page covering the broad topic, linked to detailed pages on subtopics, all interlinked. This signals topical authority to both search engines and AI systems.
- Include "People Also Ask" type content: anticipate follow-up questions and answer them on the same page or linked pages.
- Update content regularly. AI systems weight recency — include `dateModified` in schema and actually update content.

### Technical GEO
- **Structured data is critical for GEO.** FAQ, HowTo, Article, Product, and Organization schema give AI systems structured information to cite. This is the single highest-leverage GEO action.
- **Clean HTML semantics.** AI crawlers parse the DOM — semantic HTML (`<article>`, `<section>`, `<h1>`-`<h6>`, `<p>`, `<ul>`, `<table>`) is parsed more accurately than `<div>` soup.
- **Content in HTML, not images/PDFs.** Key information must be in crawlable text, not locked in images, infographics, or downloadable files.
- **Fast, accessible pages.** AI crawlers respect robots.txt and have limited crawl budgets. Pages that load slowly or block bots are skipped.
- **Author and entity markup.** Add author information with schema (`Person` linked to author pages). AI systems are increasingly evaluating E-E-A-T (Experience, Expertise, Authoritativeness, Trustworthiness) signals.

### What NOT to Do for GEO
- Don't create content solely for AI extraction — it must serve human visitors first.
- Don't stuff keywords unnaturally. AI systems are trained on natural language and detect keyword stuffing.
- Don't block AI crawlers (Googlebot, GPTBot, ClaudeBot, PerplexityBot) in robots.txt unless you explicitly don't want AI systems referencing your content.
- Don't duplicate content across pages hoping different pages get cited — AI systems deduplicate.

---

## Monitoring & Verification

- Verify pages are indexed: Google Search Console → URL Inspection for key pages.
- Monitor search performance: impressions, clicks, CTR, position by page and query.
- Check for crawl errors weekly (Google Search Console → Pages report).
- Test structured data after every deploy: Rich Results Test or Schema.org validator.
- Monitor social sharing previews after changes — OG images and descriptions can break silently.
- Track AI citation: search for your brand/product in ChatGPT, Claude, Perplexity periodically to verify your content appears in AI-generated answers.
