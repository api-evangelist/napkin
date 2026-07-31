---
name: Generate a visual from text (Napkin API)
description: Turn a block of text into downloadable Napkin visuals (SVG/PNG/PPT) using the async create-poll-download flow.
api: openapi/napkin-openapi.yml
operations:
  - createVisualRequest
  - getVisualRequestStatus
  - downloadVisualFile
---

# Generate a visual from text with the Napkin API

Napkin generates visuals asynchronously. You create a request, poll its status, then download
the finished files. All calls are against `https://api.napkin.ai`.

## Authentication
Send an account API token as a bearer credential on every request:

```
Authorization: Bearer YOUR_API_TOKEN
```

Create a token at app.napkin.ai -> Account/Team Space Settings -> Developers -> "Create new API
token". (For acting on behalf of another user, use the OAuth 2.0 authorization-code flow with the
`generation` scope instead — see `authentication/napkin-authentication.yml`.)

## Steps

1. **Create the visual request** — `createVisualRequest` (`POST /v1/visual`).
   Body must include `content` (the text; max 100,000 bytes). Optional: `format` (`svg` default,
   `png`, `ppt`), `language` (auto-detected if omitted), `number_of_visuals`, `style_id`,
   `visual_query`/`visual_queries`, `orientation`, `color_mode`, `sort_strategy`. A `201` returns
   a request `id`.

2. **Poll for completion** — `getVisualRequestStatus` (`GET /v1/visual/{request-id}/status`).
   Repeat until `status` is `completed` (typically 10-30 seconds). When complete the response
   carries `generated_files[]` with each file's download metadata. Handle `failed` and check the
   `error` object / `warnings[]`.

3. **Download each file** — `downloadVisualFile` (`GET /v1/visual/{request-id}/file/{file-id}`).
   Response is raw bytes with `Content-Type` (e.g. `image/svg+xml`) and `Content-Length`.

## Rules and gotchas
- **Expiry:** status and file URLs expire **30 minutes** after generation (`410 Gone` afterward) —
  download promptly or re-create the request.
- **Ownership:** you can only read/download requests you created (`403` otherwise).
- **Rate limits:** plan-based; on `429` back off and retry (no idempotency key is available, so do
  not blindly re-POST — a retry creates a new request).
- **Auth errors:** an expired token returns `401` (not `403`).

See `conventions/napkin-conventions.yml` and `errors/napkin-problem-types.yml` for the full
cross-cutting semantics and error catalog.
