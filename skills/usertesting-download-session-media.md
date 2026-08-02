---
generated: '2026-07-21'
method: generated
name: Download session videos, transcripts, and clips
description: >-
  Retrieve the WebVTT transcript, the full session video (1-hour pre-signed URL), and every
  clip (poll GENERATING -> GENERATED) for a UserTesting session.
api: openapi/usertesting-results-v2-openapi.yml
operations: [SessionResultsController_getTranscriptVtt, SessionResultsController_getVideoDownloadUrl, SessionResultsController_getClips, SessionResultsController_getClipDetails]
source: >-
  Grounded in openapi/usertesting-results-v2-openapi.yml (operationIds verified) and the
  "Process Session Videos at Scale" tutorial on developer.usertesting.com.
---

# Download session videos, transcripts, and clips

Use this to archive or post-process the media behind a session (think-out-loud, live
conversation, or interaction test sessions).

## Auth
- OAuth client-credentials bearer token, 3600 s lifetime. See
  `authentication/usertesting-authentication.yml`.

## Steps
1. **Transcript** — `SessionResultsController_getTranscriptVtt`
   (`GET /api/v2/sessionResults/{sessionId}/transcript`) returns WebVTT text; save it with a
   `.vtt` extension. Only THINK_OUT_LOUD and LIVE_CONVERSATION sessions have transcripts.
2. **Full video** — `SessionResultsController_getVideoDownloadUrl`
   (`GET /api/v2/sessionResults/{sessionId}/videoDownloadUrl`) returns a pre-signed URL valid
   for **1 hour**; download immediately and request a fresh URL after expiry.
3. **List clips** — `SessionResultsController_getClips`
   (`GET /api/v2/sessionResults/{sessionId}/clips`) returns clip timing, media flags, layout,
   and video-region metadata.
4. **Download each clip** — `SessionResultsController_getClipDetails`
   (`GET /api/v2/sessionResults/{sessionId}/clips/{clipUuid}`). While the clip is being
   prepared the response has `status: GENERATING` and no link — poll every few seconds until
   `status: GENERATED`, then fetch `clipUrl` within **15 minutes**.

## Rules
- 10 requests/minute — budget the polling loop against `x-ratelimit-remaining`, and back off
  on 429 until `x-ratelimit-reset`. See `rate-limits/usertesting-rate-limits.yml`.
- Pre-signed URLs are temporary by design; never persist them, persist the downloaded media.
- `404` on any media call usually means the session's product type does not support that
  asset. See `errors/usertesting-problem-types.yml`.
