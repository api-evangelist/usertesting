---
generated: '2026-07-21'
method: generated
name: Track QXscore and export highlight reels
description: >-
  Pull the QXscore experience-quality metric for an interaction test and download its
  highlight reels for stakeholder sharing.
api: openapi/usertesting-results-v2-openapi.yml
operations: [TestResultsController_getQxScoreResponse, TestResultsController_getHighlightReels, TestResultsController_getReelDetails]
source: >-
  Grounded in openapi/usertesting-results-v2-openapi.yml (operationIds verified) and the
  "Build a UX Metrics Dashboard" tutorial on developer.usertesting.com.
---

# Track QXscore and export highlight reels

Use this to feed a UX-metrics dashboard (QXscore over time) and to pull shareable highlight
reels out of a test.

## Auth
- OAuth client-credentials bearer token, 3600 s lifetime. See
  `authentication/usertesting-authentication.yml`.

## Steps
1. **QXscore** — `TestResultsController_getQxScoreResponse`
   (`GET /api/v2/testResults/{testId}/qxScores`). Interaction tests only ("STUDYV2" +
   "NON_THINK_OUT_LOUD"); computed across all completed sessions, with component and
   subcomponent values (behavioral + attitudinal) you can chart individually.
2. **List highlight reels** — `TestResultsController_getHighlightReels`
   (`GET /api/v2/testResults/{testId}/highlightreels`) returns each reel's ID, name,
   duration, clip count, and last-updated timestamp.
3. **Download a reel** — `TestResultsController_getReelDetails`
   (`GET /api/v2/testResults/{testId}/highlightreels/{reelId}`). Poll every few seconds while
   `status: GENERATING`; when `status: GENERATED`, fetch `reelUrl` within **15 minutes**.

## Rules
- QXscore requests against non-interaction tests return `404` — check the test's product type
  first via session summaries. See `errors/usertesting-problem-types.yml`.
- 10 requests/minute shared across all Results API calls — poll reels gently and honor
  `x-ratelimit-reset` on 429. See `rate-limits/usertesting-rate-limits.yml`.
