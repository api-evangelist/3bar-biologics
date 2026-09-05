# 3Bar Biologics

<!-- API-EVANGELIST-PROVENANCE:BEGIN -->
> ### About this repository
>
> **This is not our API.** This repository is an independent, third-party profile of a company's
> **publicly available** API surface, maintained by [API Evangelist](https://apievangelist.com).
> API Evangelist does not operate, host, resell, or support this company's APIs, and is not
> affiliated with or endorsed by the company unless stated on the profile.
>
> **Where the information came from.** Everything here is assembled from material a member of the
> public can reach with a browser and no credentials — the company's own website, developer portal
> and documentation, the specifications it publishes for public use (OpenAPI, AsyncAPI, JSON Schema,
> `apis.json`, `llms.txt` and similar), its public repositories, and its public status, pricing and
> changelog pages. **Nothing here is obtained by breaching a system, defeating an access control, or
> using credentials of any kind.**
>
> **The rating is an independent assessment.** The Kin Score and Agent Readiness rating are
> independently calculated scores of a company's *public* API artifacts, produced by API Evangelist
> against a published rubric. They are not certifications, endorsements, security assessments, or
> audits, and they score published artifacts — not the quality, safety, or security of the software.
>
> **Corrections, re-scores, and removal are free.** No partnership, contract, or purchase is
> required, and you do not need to justify the request.
>
> - **Something wrong?** Open an issue on this repository, or email
>   [info@apievangelist.com](mailto:info@apievangelist.com).
> - **Published something new?** Ask for a re-score and we will re-run the rating.
> - **Want the listing taken down?** Say so and we will honor it. The profile is reduced to your
>   company name, a factual description, and a link to your own site, and the company is recorded as
>   **unrated** — never scored zero for having asked.
>
> **Response times.** Acknowledgement within **one business day**; removal or restriction within
> **two business days**; corrections and re-scores within **five business days**.
>
> **Not from the company, and here with a question?** You are welcome here — we would rather be the
> front line and point you the right way than have a good report go nowhere. What this repository
> can answer is narrow, though, so it is worth knowing who you are actually looking for:
>
> - **A question about how the API works, an account, billing, or a bug in the service** — that is
>   the company's own support, not us. We profile this API; we do not operate it and cannot see
>   your account.
> - **A bug in an open-source project we only catalog** — file it on that project's own repository.
>   This has happened with a real and correct bug report that reached us instead of the people who
>   could fix it, which helped nobody.
> - **Anything about this listing itself** — the description, the tags, the rating, a missing or
>   wrong artifact — is ours. Open an issue here.
> - **Not sure, or something general about API Evangelist or APIs.io** — open an issue on the
>   [APIs.io Inbox](https://github.com/api-search/inbox) and we will route it.
>
> **This repository contains no software, and we will never ask you to download anything.** There is
> no build, release, installer, or binary here — only text and machine-readable API descriptions, so
> there is nothing here that can be "corrupt" or need "repairing". Any issue, comment, or email
> claiming otherwise and offering a download link is not from us and is hostile. Do not follow the
> link; it is a lure. Report it to GitHub and, if you like, tell us at
> [info@apievangelist.com](mailto:info@apievangelist.com) so we can take it down.
>
> **On a security or compliance team?** Email
> [info@apievangelist.com](mailto:info@apievangelist.com) with *security* in the subject line and
> you will get a person, not a form. We will tell you exactly which public URLs this profile was
> built from so your team can see the same surface we did, and we will take the listing down on
> request while you work through it.
>
> Full detail: **[Where this data comes from](https://apievangelist.com/about/where-our-data-comes-from)**
<!-- API-EVANGELIST-PROVENANCE:END -->

3Bar Biologics (3BarBio) is an agricultural biotechnology company in Columbus, Ohio, spun out of
research at The Ohio State University and operating as the first contract development and
manufacturing organization (CDMO) dedicated to agricultural biologicals. Its patented LiveMicrobe
platform — including the Iso-Pak and Re-Pak delivery systems — keeps beneficial bacteria viable
through storage and distribution, the problem that has historically limited adoption of microbial
crop inputs.

## What this profile found

**3Bar Biologics is not a software vendor and publishes no developer program.** There is no
developer portal, no API documentation, no SDK, no client library in any package registry, no GitHub
organization and no pricing for any programmatic product.

The only machine-readable interface the company exposes is the **WordPress REST content API** behind
its corporate website at `www.3barbiologics.com`. It is anonymously readable, read-only, and
incidental to the website rather than offered as a product. It carries the company's news archive and
marketing pages — nothing about strains, formulations, batches, viability testing or customers.

That surface is nonetheless real, self-describing and worth cataloging, so it is documented here in
full: **9 APIs, 19 operations**, every one derived from the server's own published route index
(`GET /wp-json/`, 360 routes across 17 namespaces) and its per-route JSON Schemas (`HTTP OPTIONS`).
No part of any specification in `openapi/` was authored from documentation or inference.

## Defects verified on this deployment (2026-09-05)

These were reproduced live, not inferred. They are recorded in `errors/`, `lifecycle/`, `security/`
and at the operation level in `overlays/`.

| # | Surface | Finding |
|---|---|---|
| 1 | `GET /wp/v2/media` | Returns **HTTP 500** for any ascending numeric or date sort (`order=asc`, `orderby=date\|id\|author`). The body is HTML, not the JSON error envelope. `orderby=title\|slug` ascending works; descending always works. Posts and pages are unaffected. |
| 2 | `GET /wp/v2/media` | Advertises `X-WP-Total: 560` while anonymous pages return 0–5 records with **HTTP 200**. Fails silently — an empty 200 is indistinguishable from an exhausted collection. |
| 3 | `GET /yoast/v1/get_head` | Returns **404 with a populated, parseable body** (the head of the site's 404 page). Branch on status, not body shape. |
| 4 | `https://www.3barbiologics.com/careers/` | **Infinite redirect loop** — `/careers/` → `/about-us/careers/` → `/careers/`. The page is readable through the Pages API but unreachable in a browser. |
| 5 | `https://3barbiologics.com/` | **Apex domain serves a certificate that does not cover it** (`CN=pantheonsite.io`). HTTPS fails; plain HTTP returns 404 without redirecting. Only the `www.` host works. |
| 6 | `https://www.3barbiologics.com/` | **79 absolute URLs to Pantheon pre-production hostnames** in the production HTML, including the site's only Privacy Policy link, which points at `dev-3bar-biologics.pantheonsite.io`. That dev environment is publicly reachable and serves the same REST API. |

## Contents

| Path | What is there |
|---|---|
| `openapi/` | 9 OpenAPI 3.1 documents, 19 operations, derived from the server's published schemas |
| `overlays/` | Per-spec Overlay 1.0.0 documents carrying provenance, stability posture and the defects above |
| `authentication/` | Anonymous read; application passwords gate the unreachable write surface |
| `conventions/` | Pagination, sparse fields, caching, CORS, tracing. Idempotency and reversibility are `na` — read-only surface |
| `errors/` | 13 problem types and 2 silent failures, every one observed live |
| `data-model/` | 10 entities and their relationships — and an explicit note on the domain entities that do **not** exist |
| `lifecycle/` | No versioning policy, no deprecation policy, no SLA, no status page, no changelog. Graded `unmanaged` |
| `conformance/` | 16 standards checked; domain standards (ADAPT, ISOBUS, GS1) recorded as inapplicable, not failed |
| `security/` | Domain security probe with 5 findings, including the apex TLS mismatch and the dev-host leak |
| `well-known/` | 18 probes across 2 hosts, 0 documents found — a recorded absence |
| `rate-limits/`, `plans/` | Honest zeros: no limits published, no pricing published |
| `packages/` | 0 packages. Every major registry queried; no first-party library exists |
| `mcp/` | No server exists. A candidate tool list only, `mode: none` |
| `skills/` | 2 packaged Agent Skills grounded in verified operationIds |
| `examples/` | 11 working requests and 2 counter-examples, all executed successfully |
| `agentic-access/` | Per-operation access contract. Entire surface classified `safe-read-only` |

## Company

- **Website** — https://www.3barbiologics.com/ (use the `www.` host)
- **Contact** — Sales@3BarBiologics.com · 1-877-3BAR-BIO · 1275 Kinnear Road, Columbus, OH 43212
- **LinkedIn** — https://www.linkedin.com/company/3bar-biologics-inc-/
