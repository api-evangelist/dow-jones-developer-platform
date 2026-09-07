---
name: dow-jones-top-stories-sync
description: Keep a local cache of entitled Dow Jones Top Stories collections in sync without breaching the published 15-minute polling ceiling.
api: Dow Jones Newswires Top Stories API
operations:
  - GetCollectionIdList
  - GetContentCollectionByRef
  - GetContentByRef
generated: '2026-09-07'
method: generated
source: openapi/dow-jones-developer-platform-newswires-top-stories-api-openapi.yml, https://developer.dowjones.com/documents/site-docs-newswires_apis-certification_process
---

# Sync Top Stories collections

Base URL `https://api.dowjones.com/`. Media type `application/vnd.dowjones.dna+json`.
Authenticate first — see `dow-jones-authenticate`.

This is the one Newswires flow Dow Jones explicitly permits an application to run on a **schedule**
rather than on human action.

## Steps

1. **List entitled collections.** `GET /content-collections` (`GetCollectionIdList`) returns the
   collection IDs the account is entitled to, each with an `updated_datetime_utc`.

2. **Diff.** Compare each `updated_datetime_utc` against what you last stored. Fetch only the
   collections that moved.

3. **Fetch a changed collection.** `GET /content-collections/{ref}` (`GetContentCollectionByRef`) with
   the collection DRN, e.g.
   `drn:consumer.contentcollection.originid.FP_US_LEAD_1`.

4. **Update the cache** with the headlines and article IDs from the response.

5. **Fetch full text lazily.** `GET /content/{ref}` (`GetContentByRef`) the first time a user opens an
   article, then cache it.

## Rules

- **`GET /content-collections` no more than once every 15 minutes.**
- **An individual collection may not be refreshed more than once every 15 minutes.**
- Store articles only for the archive duration your contract states, then purge.
- `404` on a content ref may mean the article does not exist *or* that the account is not entitled to
  it — treat both the same and do not retry-loop.
- Errors on this API use `JsonApiError`, whose `status` is a **string enum of .NET status names**
  (`badRequest`, `unauthorized`, `notFound`) rather than an integer. Do not parse it as a number.
