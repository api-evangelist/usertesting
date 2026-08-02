---
generated: '2026-07-21'
method: generated
name: Pull all session results for a test
description: >-
  Authenticate with OAuth client credentials, page through every completed session in a
  UserTesting test, and pull the detailed v3 results (demographics + task responses) for each.
api: openapi/usertesting-results-v2-openapi.yml
operations: [SessionResultsController_getSessionSummaryResults, SessionResultsV3Controller_getSessionResults]
source: >-
  Grounded in openapi/usertesting-results-v2-openapi.yml (operationIds verified),
  docs/authorization (token flow), and conventions/usertesting-conventions.yml.
---

# Pull all session results for a test

Use this to extract a complete, analyzable dataset of sessions from one test (survey, live
conversation, interaction test, or think-out-loud test — "STUDYV2" types only).

## Auth
- `POST https://auth.usertesting.com/oauth2/aus1p3vtd8vtm4Bxv0h8/v1/token` with form fields
  `grant_type=client_credentials`, `client_id`, `client_secret`, `scope=studies:read`.
  Credentials are issued by support@usertesting.com. See `authentication/usertesting-authentication.yml`.
- Tokens expire after 3600 s — refresh before long extraction runs, and send
  `Authorization: Bearer <access_token>` on every call.

## Steps
1. **Get the test ID** — a UUID copied from the UserTesting UI (see the portal guide
   "How to Obtain a Test ID (UUID)"). There is no list-tests endpoint on this API.
2. **Page through session summaries** — `SessionResultsController_getSessionSummaryResults`
   (`GET /api/v2/sessionResults?testId=...`). Use `limit` (1-500, default 25) and `offset`
   (0-10000). Read `meta.pagination.totalCount` from the first page to compute the number of
   iterations; sessions are sorted newest to oldest.
3. **Fetch details per session** — `SessionResultsV3Controller_getSessionResults`
   (`GET /api/v3/sessionResults/{sessionId}`) for each summary; returns participant
   demographics, task-group metadata, and individual task responses.

## Rules
- Rate limit is 10 requests/minute: watch `x-ratelimit-remaining` and sleep until
  `x-ratelimit-reset` on 429. See `rate-limits/usertesting-rate-limits.yml`.
- All operations are read-only GET — retries are always safe.
- `401` = token expired/invalid (mint a new one); `404` = test or session not visible from the
  workspaces your API user belongs to. See `errors/usertesting-problem-types.yml`.
