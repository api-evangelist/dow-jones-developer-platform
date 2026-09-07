---
name: dow-jones-authenticate
description: Obtain a Dow Jones bearer token via the documented two-step OAuth exchange before calling any api.dowjones.com endpoint.
api: Dow Jones Developer Platform
operations: []
generated: '2026-09-07'
method: generated
source: https://developer.dowjones.com/documents/site-docs-newswires_apis-oauth-migration-guide
---

# Authenticate against the Dow Jones Developer Platform

Every Dow Jones API call carries `Authorization: Bearer <token>`. Getting that token takes **two**
POSTs, not one. A token that comes back from step 1 will not authenticate an API call — this is the
single most common failure on this platform.

## Preconditions

- A `client_id` and a `refresh_token` issued by Dow Jones. There is no self-serve signup; credentials
  come from a Dow Jones representative (https://developer.dowjones.com/request-trial/).
- EU-resident accounts use `https://eu.accounts.dowjones.com/oauth2/v1/token` instead.

## Steps

1. **Exchange the refresh token for an intermediate token.**

   `POST https://accounts.dowjones.com/oauth2/v1/token`
   with `Content-Type: application/json` and body
   `{"client_id": "...", "grant_type": "refresh_token", "refresh_token": "..."}`.

   The response `access_token` is an **intermediate** token, valid 432000 seconds (~5 days). It cannot
   call any Dow Jones API. Cache it — it is what makes the exchange cheap.

2. **Exchange the intermediate token for the bearer token.**

   `POST https://accounts.dowjones.com/oauth2/v1/token`
   with body
   `{"assertion": "<INTERMEDIATE_TOKEN>", "client_id": "...", "grant_type": "urn:ietf:params:oauth:grant-type:jwt-bearer", "scope": "openid pib"}`.

   The `access_token` returned here is the bearer token. It expires in **3600 seconds**.

3. **Call the API.** Send `Authorization: Bearer <FINAL_ACCESS_TOKEN>` on every request to
   `https://api.dowjones.com/...`.

## Rules

- On `401`, re-run step 2 from the cached intermediate token. Only re-run step 1 when the intermediate
  token has also expired.
- On `406`, the Accept-header version you asked for is no longer supported — see
  `conventions/dow-jones-developer-platform-conventions.yml`.
- The legacy `user-key` header is still accepted on Factiva Analytics endpoints during the OAuth
  migration; both methods work in parallel and no cutover date has been announced.
- The identity service is discoverable at
  `https://accounts.dowjones.com/.well-known/openid-configuration` (issuer `https://sso.accounts.dowjones.com/`).
