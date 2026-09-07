---
name: dow-jones-riskcenter-third-party-onboarding
description: Onboard a third party into Dow Jones RiskCenter and put it under ongoing monitoring, with the replay and reversal hazards called out.
api: RiskCenter Third Party Platform API 0.2
operations:
  - GetBusinessUnits
  - GetGroups
  - GetFields
  - PostThirdParty
  - PutBatchThirdPartyProperties
  - PostMonitoredEntities
  - GetThirdParties
  - GetThirdParty
  - PatchThirdParty
  - DeleteThirdParty
  - DeleteMonitoredEntities
generated: '2026-09-07'
method: generated
source: openapi/dow-jones-developer-platform-riskcenter-third-party-api-0-2-openapi.yml
---

# Onboard a third party into RiskCenter

Base URL `https://api-thirdparty.riskcenter.dowjones.com/`. This host is **not** `api.dowjones.com`:
the RiskCenter Third Party Platform runs on its own domain with its own token endpoint,
`https://auth-thirdparty.riskcenter.dowjones.com/connect/token`. Media type
`application/vnd.dowjones.dna.risk-third-parties.v_0.2-beta+json`.

> This contract is **beta** (`info.version: v0.2-beta`). Version 0.1 is still published alongside it.

## Steps

1. **Read the tenant's shape first.**
   - `GetBusinessUnits` — `GET /business-units`
   - `GetGroups` — `GET /groups`
   - `GetFields` — `GET /third-parties/properties/fields` (and `GetFieldList` —
     `GET /third-parties/properties/fields/{id}/list` — for a field's allowed values)
   - `GetProcesses` — `GET /third-parties/processes`

2. **Check for an existing record before creating one.** `GetThirdParties` — `GET /third-parties` with
   the filter parameters. A duplicate create returns `409 Conflict`.

3. **Create.** `PostThirdParty` — `POST /third-parties`.

4. **Set properties.** `PutBatchThirdPartyProperties` —
   `PUT /third-parties/{thirdpartyid}/properties/batch-update` for many at once, or
   `PutThirdPartyProperties` — `PUT /third-parties/{thirdpartyid}/properties/{id}` for one.

5. **Attach monitored entities.** `PostMonitoredEntities` —
   `POST /third-parties/{thirdpartyid}/monitored-entities`. Resolve the allowed values first with
   `GetMonitoredEntitiesEntityTypes` (`GET /third-parties/monitored-entities/types`) and
   `GetMonitoredEntitiesIdTypes` (`GET /third-parties/monitored-entities/id-types`).

6. **Verify.** `GetThirdParty` — `GET /third-parties/{id}` and
   `GetMonitoredEntities` — `GET /third-parties/{thirdpartyid}/monitored-entities`.

## Rules — read these before writing anything

- **There is no idempotency key.** Dow Jones documents no `Idempotency-Key` header and none appears in
  this contract. A retried `POST /third-parties` after a timeout will create a **second** third party.
  Always re-read with `GetThirdParties` before retrying a create, and treat `409` as "it already
  landed", not as a failure to retry.
- **There is no documented undo.** `DeleteThirdParty` and `DeleteMonitoredEntities` are described only
  as "Deletes a third party" and "Removes a monitored entity". No restore, undelete, archive or
  retention window is published. Do not tell a user a deletion can be reversed.
- `POST /files` uploads and `GET /files/{fileId}` downloads; the upload has no documented deletion.
- Errors arrive as `ErrorResponseModelV2` (`errors[]` of `{title,status,code,detail}`). Version 0.1 of
  this API additionally declares an RFC 7807 `ProblemDetails` schema.
- Every operation declares `401`. Tokens are hour-long; refresh, do not retry blindly.
