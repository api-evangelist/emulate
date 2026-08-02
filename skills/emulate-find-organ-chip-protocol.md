---
name: Find an Emulate Organ-Chip protocol or user guide
description: >-
  Locate the right Emulate protocol or user guide for a given Organ-Chip model or
  laboratory procedure using the anonymous Emulate Content REST API, then link the user
  to the human-readable page.
api: openapi/emulate-content-api-openapi.yml
operations:
  - listSupportContent
generated: '2026-08-01'
method: generated
source: openapi/emulate-content-api-openapi.yml
---

# Find an Emulate Organ-Chip protocol or user guide

Use this when someone asks how to run an Emulate Organ-Chip procedure — seeding a
Liver-Chip, using the Compound Distribution Kit, a Zoe-CM2 culture step — and you need
the authoritative Emulate document rather than a summary.

## Before you start

- **No credentials.** The Emulate Content REST API is anonymous. Do not ask the user for
  an API key; none exists. See `authentication/emulate-authentication.yml`.
- **No search parameter.** The route takes no arguments. You fetch the whole collection
  once and filter locally.
- **Base URL:** `https://emulatebio.com/wp-json`

## Steps

1. Call **`listSupportContent`** — `GET /emulate-support/v2/support`.

2. Parse the two-element envelope. This is the shape that trips up naive clients:
   - `response[0].posts` is the array of support documents (26 observed on 2026-08-01).
   - `response[1]` is the taxonomy map, with `models` (7 terms) and `protocols` (6 terms).

   Do not assume element 0 is the only thing in the array, and do not assume the response
   is a bare array — the blog and forms routes use a bare array, these do not. See
   `conventions/emulate-conventions.yml`.

3. Filter `posts` locally on `post_title` and `post_content` against the organ model
   (Liver, Kidney, Brain, Lung, Intestine, Bone Marrow, Lymphoid, Vagina) and the
   procedure the user named. Use the `models` and `protocols` vocabularies from
   `response[1]` as your controlled term list so you match Emulate's own naming.

4. Return the human page, not the raw payload. Build the link from `post_name` (the slug)
   under `https://emulatebio.com/support/<post_name>/`, or use `media_url` when the
   document is a downloadable PDF.

5. Quote protocol text from `post_content` only with the source link attached. This is
   laboratory procedure content — never paraphrase a step, a concentration or a timing
   without pointing the user at the original.

## Rules

- `post_content` is WordPress block markup (`<!-- wp:paragraph -->`). Strip block comments
  before showing text to a user.
- Check `post_status` is `publish` before surfacing anything.
- If nothing matches, say so and route the user to
  <https://emulatebio.com/contact-support/> rather than guessing at a procedure.
- There is no pagination and no rate-limit contract. Fetch once and cache; do not poll.

## Errors

Failure on this surface is not signalled by HTTP status — see
`errors/emulate-problem-types.yml`. Always inspect the parsed body: an object carrying
`{"status": …, "error": …}` is a failure even when the transport returned 200.
