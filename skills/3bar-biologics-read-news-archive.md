---
name: 3bar-biologics-read-news-archive
description: >-
  Read and filter the 3Bar Biologics (3BarBio) news, press-release and insights archive through the
  company's public WordPress content API. Use when you need what 3Bar Biologics has publicly
  announced — funding, partnerships, product launches, LiveMicrobe technology posts — rather than
  what its website looks like.
api: 3bar-biologics:3bar-biologics-posts-api
operations:
  - listPosts
  - getPost
  - listCategories
  - listTags
  - listUsers
---

# Read the 3Bar Biologics news archive

Base URL: `https://www.3barbiologics.com/wp-json`
No credential. No key, no token, no account. Every request below is anonymous.

51 posts at last capture (2026-09-05), most recent 2026-08-18.

## 1. Decide what you actually need

Records are large — roughly 16KB each with rendered HTML. Always pass `_fields` unless you need
the body:

```
GET /wp/v2/posts?per_page=20&_fields=id,date,slug,link,title,excerpt
```

Read `X-WP-Total` and `X-WP-TotalPages` from the response headers to plan pagination.

## 2. Filter by topic

The archive is segmented by category, not by tag — the tag vocabulary has three terms applied to one
post each and is effectively unused. Fetch the categories first rather than hard-coding ids, since
they are ordinary auto-increment integers that carry no meaning:

```
GET /wp/v2/categories?_fields=id,slug,name,count
```

At capture: `8` In the News (22), `17` Insights (19), `7` Press Release (19), `6` News (15),
`1` Uncategorized (4).

```
GET /wp/v2/posts?categories=7&per_page=20&_fields=id,date,title,link
```

Free-text search is a parameter on the same operation:

```
GET /wp/v2/posts?search=LiveMicrobe&_fields=id,date,title,link
```

## 3. Read one post fully

```
GET /wp/v2/posts/{id}?_embed
```

`_embed` inlines the author, featured image and terms under `_embedded`, which saves three follow-up
requests. The body is in `content.rendered` as HTML — it is rendered WordPress output, so expect
Elementor wrapper markup around the prose.

## 4. Page the whole archive

`per_page` maxes at 100; 101 returns `400 rest_invalid_param`.

```
GET /wp/v2/posts?per_page=100&page=1
```

**Stop when the `Link` header stops carrying `rel="next"`.** Do not increment `page` until you get an
error — page 999 returns `400 rest_post_invalid_page_number`, and treating that as your terminator
means every complete run ends on a failed request.

## Rules

- **Match errors on `code`, never on `message`.** The envelope is `{code, message, data:{status}}` —
  the WordPress shape, not RFC 9457 problem+json. `message` is localized and unstable.
- **A 404 does not prove non-existence.** `rest_post_invalid_id` is also returned for a post that
  exists but is not published.
- **There are no rate-limit headers.** Nothing tells you when to back off, so self-throttle.
  Responses are cached for 7 days at the edge, so repeated reads are cheap for the origin anyway.
- **Reads may be up to a week stale.** `Cache-Control: public, max-age=604800` behind Varnish/Fastly.
  If you need to know whether something is current, this API cannot tell you.
- **This API is undocumented and unsupported.** 3Bar Biologics publishes no versioning policy, no
  deprecation policy, no status page and no developer support channel. Nothing here is promised to
  you. Handle failure rather than assuming shape.
