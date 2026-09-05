---
name: 3bar-biologics-map-site-content
description: >-
  Build a complete structured map of the 3Bar Biologics (3BarBio) website — every published page and
  post, its URL, and its structured-data graph — through the public content API, without scraping
  HTML. Use when you need the company's full public surface as data rather than one article.
api: 3bar-biologics:3bar-biologics-discovery-api
operations:
  - getApiRoot
  - listTypes
  - listPages
  - listPosts
  - searchContent
  - getSeoHead
  - getOembed
---

# Map the 3Bar Biologics site as structured data

Base URL: `https://www.3barbiologics.com/wp-json`
No credential required.

This site is small and completely enumerable: 14 pages and 51 posts. You can hold the entire public
surface in memory, so do that once rather than crawling.

## 1. Confirm what is actually published

The API is self-describing. Read the registry before assuming a resource exists:

```
GET /wp/v2/types
```

14 post types are registered, but only `post`, `page` and `attachment` are publicly readable — the
rest are WordPress core and Elementor plugin internals that return 401 anonymously.

**There is no custom post type on this site.** No product, strain, batch, formulation or customer
resource exists. If you are looking for 3Bar's agricultural biologicals data, it is not here; the
words LiveMicrobe, Iso-Pak and Re-Pak appear only inside page HTML.

## 2. Pull every page and post in two requests

```
GET /wp/v2/pages?per_page=100&_fields=id,slug,link,title,parent,menu_order
GET /wp/v2/posts?per_page=100&_fields=id,date,slug,link,title,categories
```

The 14 pages at capture: `livemicrobe-products`, `insights`, `develop-2`, `deliver-2`, `design`,
`design-develop-deliver`, `case-studies`, `news-insights`, `careers`, `why-3bar`,
`innovative-biomanufacturing`, `contact-us`, `home`, `privacy-policy`.

## 3. Get structured data without parsing HTML

For any URL on the site, the Yoast endpoint returns the rendered head **and** its parsed schema.org
JSON-LD graph:

```
GET /yoast/v1/get_head?url=https%3A%2F%2Fwww.3barbiologics.com%2Fwhy-3bar%2F
```

Read `json.@graph` for the structured entities. This is strictly better than fetching the page and
parsing `<script type="application/ld+json">`.

**This endpoint returns 404 with a fully populated, parseable body** when the URL does not resolve —
it hands you the head of the 404 page. Branch on the HTTP status code, never on whether the body
parsed.

## 4. Get embed metadata

```
GET /oembed/1.0/embed?url=<url-encoded site URL>
```

Returns `provider_name` `3BarBio`, author, title, dimensions and iframe HTML. Returns
`404 oembed_invalid_url` for URLs not on this site.

## Traps on this specific deployment

These are verified, reproducible, and will silently corrupt a site map if you do not handle them:

- **Do not sort the media collection ascending.** `GET /wp/v2/media?order=asc` returns **HTTP 500**
  with an **HTML** body — not a JSON error envelope, so a client that parses every error as JSON
  throws here. It reproduces for `orderby=date`, `id` and `author`. Use the default descending order,
  or sort ascending by `title` or `slug`, both of which work. Posts and pages are unaffected.
- **Do not trust `X-WP-Total` on the media collection.** It advertises 560 while anonymous pages
  return 0-5 records with HTTP 200. **An empty 200 is not the end of the collection.** If you stop
  paging on the first empty page you will conclude the media library is empty. Page until the `Link`
  header stops carrying `rel="next"`.
- **The Careers page is unreachable in a browser.** It is readable through the Pages API (id 134),
  but `https://www.3barbiologics.com/careers/` is an infinite redirect loop
  (`/careers/` → `/about-us/careers/` → `/careers/`). Do not present its `link` field as a working URL.
- **Always use the `www.` host.** `https://3barbiologics.com/` serves a certificate that does not
  cover the name and fails the TLS handshake; plain HTTP on the apex returns 404 without redirecting.
- **Some links in the site's own HTML point at pre-production hosts.** The production homepage
  hard-codes URLs on `dev-3bar-biologics.pantheonsite.io` and `live-3bar-biologics.pantheonsite.io`,
  including its only Privacy Policy link. Canonicalize everything to `www.3barbiologics.com`; the
  canonical `/privacy-policy/` does exist and returns 200.

## Rules

- Cache aggressively and re-read rarely. Responses declare `max-age=604800` and the whole site is
  65 records.
- Every response carries `X-Robots-Tag: noindex`. The data is public and machine-readable, but the
  provider signals it should not be indexed as content. Respect that in anything you publish.
- This surface is undocumented, unsupported and unversioned. It is a marketing site's CMS API, not a
  product. Nothing here is promised to you.
