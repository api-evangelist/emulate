---
name: Search the Emulate resource library and newsroom
description: >-
  Find Emulate application notes, posters, webinars, publications, blog posts and press
  coverage for a given organ model, application area or product, using the anonymous
  Emulate Content REST API.
api: openapi/emulate-content-api-openapi.yml
operations:
  - listResources
  - listPosts
  - listNews
generated: '2026-08-01'
method: generated
source: openapi/emulate-content-api-openapi.yml
---

# Search the Emulate resource library and newsroom

Use this to answer "what has Emulate published about X" — where X is an organ model
(Liver-Chip, Brain-Chip), an application area (toxicology, cell therapy, immunology), or
a product (Chip-R1, Zoe-CM2, Ava).

## Before you start

- **No credentials, no query parameters.** Every route below is an anonymous GET that
  returns the full collection. Filtering is your job.
- **Watch the payload size.** `listResources` returned roughly 1.03 MB with 299 records on
  2026-08-01. Fetch it once, cache it, and filter in memory — do not re-fetch per query.
- **Base URL:** `https://emulatebio.com/wp-json`

## Steps

1. **Resource library — `listResources`** (`GET /emulate-resources/v2/resources`).
   Two-element envelope: `response[0].posts` is the asset list; `response[1]` carries four
   taxonomies you should treat as the controlled vocabulary for the user's question:
   - `application` (12 terms) — research area
   - `organ-models` (11 terms) — which Organ-Chip
   - `product` (12 terms) — which instrument or chip
   - `resource_type` (8 terms) — poster, application note, webinar, and so on

   Map the user's phrasing onto these terms before filtering, so "gut" resolves to
   Emulate's own model name rather than a substring miss.

2. **Blog — `listPosts`** (`GET /emulate-posts/v2/posts`). Different shape: a **bare
   array** of 62 records, no envelope. Useful fields: `post_title`, `post_excerpt`,
   `display_date`, `link` (already an absolute URL — use it directly), `media_url`,
   `cats`.

3. **Newsroom — `listNews`** (`GET /emulate-in-the-news/v2/news`). Back to the two-element
   envelope; `response[0].posts` (114 records) each carry `post_link` pointing at the
   external publication, and `response[1].news_category` is the category vocabulary.

4. Merge and rank by `post_date_gmt` descending. Label each result with which surface it
   came from — a peer-reviewed poster and a press mention are not the same evidence.

5. Return links, and prefer Emulate's own field: `link` on blog posts, `post_link` on news
   items, `media_url` for downloadable assets, and
   `https://emulatebio.com/resources/<post_name>/` for library items.

## Rules

- Three routes, two response shapes. Branch on the shape explicitly rather than assuming.
- `post_content` is WordPress block markup — strip `<!-- wp:… -->` comments before display.
- Gated assets are fronted by forms (`listForms`). If a resource has no direct `media_url`,
  tell the user it is gated and link the page instead of implying an open download.
- Cite Emulate as the source. Peer-reviewed claims belong to the underlying publication —
  follow through to <https://emulatebio.com/publications/> when the user needs the primary
  source.

## Errors

No RFC 9457, and at least one route reports failure in-body over HTTP 200. Inspect the
parsed body for `{"status": …, "error": …}` before treating a 200 as success. See
`errors/emulate-problem-types.yml`.
