---
name: dow-jones-calendar-events-search
description: Search Dow Jones Calendar Live economic, corporate and IPO events and resolve their taxonomy codes.
api: Calendar Live API
operations:
  - GET /calendar-events/search
  - POST /calendar-events/search
  - GET /calendar-events/{id}
  - GET /calendar-ipo-events/search
  - GET /taxonomy/calendar-events/eventcodes
  - GET /taxonomy/calendar-events/eventclass
  - GET /taxonomy/calendar-events/countries
  - GET /taxonomy/calendar-events/regions
generated: '2026-09-07'
method: generated
source: openapi/dow-jones-developer-platform-calendar-live-api-openapi.yml
---

# Search Calendar Live events

Base URL `https://api.dowjones.com/`. Calendar Live API v1.1.9, 29 operations, bearer JWT.
Authenticate first — see `dow-jones-authenticate`.

## Steps

1. **Resolve the codes you intend to filter on.** The filter values are Dow Jones taxonomy codes, not
   free text:
   - `GET /taxonomy/calendar-events/eventcodes` and `/eventcodes/{code}`
   - `GET /taxonomy/calendar-events/eventclass` and `/eventclass/eventcode/{eventcode}`
   - `GET /taxonomy/calendar-events/countries`, `/regions`, `/languages`, `/currency`,
     `/seriescodes`

2. **Search.** `GET /calendar-events/search` with bracketed filter parameters, e.g.

   ```
   GET /calendar-events/search?filter[has_confirmed_events_only]=true&filter[region]=AS&filter[event_class]=IEP_STAT
   ```

   Use `POST /calendar-events/search` when the filter set is too large for a query string.

3. **Page.** `page[limit]` and `page[offset]`, e.g.
   `?page[limit]=50&page[offset]=4302673`. Follow the `links.next` value from the response envelope
   rather than computing offsets yourself.

4. **Read one event.** `GET /calendar-events/{id}`.

5. **IPO calendar** is a parallel resource: `GET /calendar-ipo-events/search`,
   `GET /calendar-ipo-events/{ids}`, and its own taxonomy under
   `/taxonomy/calendar-ipo-events/{countries,exchanges}`.

## Rules

- Country codes are ISO-3166 alpha-2, upper case.
- Every operation declares `400`, `401`, `404`, `500` and `503`. `401` means the hour-long bearer token
  expired — re-run step 2 of `dow-jones-authenticate`.
- This API is read-only: there is nothing to undo and no idempotency key is needed.
