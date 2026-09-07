---
name: dow-jones-newswires-realtime-search
description: Search Dow Jones Newswires real-time content by DJN taxonomy code and retrieve the linked articles.
api: Dow Jones Newswires Real-Time API
operations:
  - POST /content/realtime/search
  - GetRealtimeLinkedContentByDrn
generated: '2026-09-07'
method: generated
source: openapi/dow-jones-developer-platform-newswires-real-time-api-openapi.yml, openapi/dow-jones-developer-platform-newswires-content-api-openapi.yml
---

# Search Newswires real-time content

Base URL `https://api.dowjones.com/`. Authenticate first — see `dow-jones-authenticate`.

## Steps

1. **Search.** `POST /content/realtime/search` with
   `Content-Type: application/vnd.dowjones.dna.content.v_1.0+json`.

   Body shape (from the contract):

   ```json
   {"data": {"id": "Search", "type": "content", "attributes": {
      "query": {"search_string": [{"mode": "Unified", "value": "djn=p/pmdm"}],
                "date": {"days_range": "LastDay"}},
      "formatting": {"is_return_rich_article_id": true},
      "navigation": {"is_return_headline_coding": true, "is_return_djn_headline_coding": true},
      "page_offset": 0, "page_limit": 10}}}
   ```

   `data.id` is the fixed value `Search` and `data.type` is the fixed value `content`.

2. **Build the query string.** The `search_string.value` is a DJN taxonomy expression, not free text.
   Operators are `and`, `or`, `not`, `<`, `>`, `=`; fields include `djn=` (taxonomy code), `hd=`
   (headline), `la=` (language) and `pdt:` (published date-time, `YYYYMMDDHHMMSSmmm`). Group with
   parentheses. Example:
   `djn=p/pmdm and (pdt:>20220307080000000 and pdt:<20220307163059000)`.

   Company relevance uses `djn:sig:<ticker|isin>` for significant coverage and
   `djn:djnabout:<ticker|isin>` for about-coverage.

3. **Page.** Use `page_offset` / `page_limit` in the body. The whole POST — authentication included —
   is capped at **10,000 characters**.

4. **Retrieve a linked article.** `GET /content/{drn}` (`GetRealtimeLinkedContentByDrn`) on the
   Newswires Content API, using a DRN from the search response.

## Rules

- **Rate limit:** the Real-Time APIs may be called at most **once every 15 seconds**. Dow Jones directs
  higher-volume consumers to a feed product. No `Retry-After` or `X-RateLimit-*` header is returned, so
  schedule against the published frequency rather than reacting to a signal.
- `days_range` accepts `LastDay`, `LastWeek`, `LastMonth`, `Last3Months`, `Last6Months`, `LastYear`,
  `Last2Years`, `Last5Years`, `AllDates`. Dow Jones recommends `pdt:` timestamps over the `date.custom`
  object.
- Errors come back as a JSON:API-style `{"errors":[{"code","title","status","detail","meta"}]}` envelope —
  see `errors/dow-jones-developer-platform-problem-types.yml`.
- Content use is contractual: an application serving users must pass Dow Jones technical **and**
  business certification first.
