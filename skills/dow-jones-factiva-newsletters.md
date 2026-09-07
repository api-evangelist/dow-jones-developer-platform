---
name: dow-jones-factiva-newsletters
description: Walk the Factiva Newsletters API from newsletter list to a specific edition's content.
api: DJ Factiva Newsletters API
operations:
  - NewslettersRead
  - NewsletterReadById
  - EditionsRead
  - EditionReadById
generated: '2026-09-07'
method: generated
source: openapi/dow-jones-developer-platform-factiva-newsletters-api-openapi.yml
---

# Read Factiva newsletters and editions

Base URL `https://api.dowjones.com`. Responses are **JSON:API** (`application/vnd.api+json`) — this is
the one Dow Jones contract that declares the media type explicitly. Authenticate first — see
`dow-jones-authenticate`.

## Steps

1. `GET /newsletters` (`NewslettersRead`) — the newsletters the account is entitled to.
2. `GET /newsletters/{id}` (`NewsletterReadById`) — one newsletter's metadata.
3. `GET /newsletters/{id}/editions` (`EditionsRead`) — that newsletter's editions.
4. `GET /newsletters/{id}/editions/{editionId}` (`EditionReadById`) — one edition.

Binary assets referenced from an edition are fetched from the Factiva Content API, e.g.
`GET /newsletters/binary/{id}?type=NewsletterImage`.

## Rules

- Read the JSON:API `data`/`included` split rather than assuming a flat object.
- Paginate with the `links` object (`self`, `prev`, `next`, `first`, `last`) when present; fall back to
  `offset`/`limit` when it is not.
- Declared failures are `400`, `404`, `500`, `503`. This API declares **no `401`** — a bad token still
  fails, it is simply undocumented here, so do not treat the absence as "no auth required".
- Read-only surface: no writes, no idempotency concern, nothing to reverse.
