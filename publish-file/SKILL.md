---
name: publish-file
description: Use when uploading files (PDF, HTML, Markdown, images, video, text) to Prismism to create shareable tracked links. Handles multipart upload, format detection, and returns the public URL.
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_).
    required: true
---

# Publish File to Prismism

Upload any supported file to get a shareable, tracked link.

**Endpoint:** `POST https://prismism.dev/v1/artifacts`
**Content-Type:** `multipart/form-data`
**Auth:** `x-api-key` header

## Supported Formats

| MIME Type | Extensions | Max Size |
|-----------|-----------|----------|
| `application/pdf` | .pdf | 50 MB |
| `text/html` | .html, .htm | 10 MB |
| `text/markdown` | .md | 5 MB |
| `image/png` | .png | 20 MB |
| `image/jpeg` | .jpg, .jpeg | 20 MB |
| `image/gif` | .gif | 20 MB |
| `image/svg+xml` | .svg | 5 MB |
| `image/webp` | .webp | 20 MB |
| `video/mp4` | .mp4 | 100 MB (Free/Plus), 500 MB (Pro), 1 GB (Business) |
| `text/plain` | .txt, .csv | 5 MB |
| `application/json` | .json | 5 MB |

## Quick Start

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@report.pdf" \
  -F "title=Q4 Report"
```

**Response:**
```json
{
  "id": "d4f7a1b2-3c9e-4e8f-b5d6-1a2b3c4d5e6f",
  "shortId": "AbCdEf12",
  "url": "https://prismism.dev/p/AbCdEf12",
  "title": "Q4 Report",
  "format": "pdf",
  "status": "ready",
  "access": "public",
  "viewCount": 0,
  "hasPassword": false,
  "requireEmail": false,
  "isGated": false
}
```

The `url` field is the shareable link. Share it anywhere — Slack, email, docs, tweets.

## Optional Parameters

All passed as form fields alongside `file`:

| Parameter | Type | Description |
|-----------|------|-------------|
| `title` | string | Display name. Defaults to original filename. |
| `access` | string | Access level: `public` (default), `private` (owner only), `allowlist` (verified emails, Plus+). |
| `password` | string | Password-protect the artifact. All plans. |
| `requireEmail` | boolean | Require email before viewing (Pro+). |
| `allowedDomains` | string | JSON array of allowed email domains, e.g. `'["acme.com"]'`. Only applies when requireEmail is true (Pro+). |
| `allowedEmails` | string | JSON array of allowed emails for allowlist access, e.g. `'["alice@acme.com"]'`. Required when access is `allowlist`. |
| `allowNetwork` | boolean | Allow HTML/Markdown to make external network requests (default: false). |
| `expiresAt` | string | Auto-expire date (RFC 3339), e.g. `2026-04-01T00:00:00Z`. |

**IMPORTANT:** `allowedDomains` and `allowedEmails` must be JSON-encoded strings in multipart form data, not repeated fields.

## Examples

### Upload a video

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@demo.mp4" \
  -F "title=Product Demo"
```

### Upload an image

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@screenshot.png" \
  -F "title=App Screenshot"
```

### Upload with password protection

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@confidential.pdf" \
  -F "title=Board Deck" \
  -F "password=secret123"
```

### Upload as private (owner only)

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@internal.pdf" \
  -F "title=Internal Notes" \
  -F "access=private"
```

### Upload with allowlist access

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@proposal.pdf" \
  -F "title=Proposal for Acme" \
  -F "access=allowlist" \
  -F 'allowedEmails=["alice@acme.com","bob@acme.com"]'
```

### Upload with email gate and domain restriction

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@proposal.pdf" \
  -F "title=Proposal for Acme" \
  -F "requireEmail=true" \
  -F 'allowedDomains=["acme.com","partner.co"]'
```

### Upload with expiration

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@temp-report.pdf" \
  -F "title=Weekly Report" \
  -F "expiresAt=2026-03-15T00:00:00Z"
```

## Error Handling

| Code | Error | Action |
|------|-------|--------|
| 400 | `INVALID_REQUEST` | Fix request parameters, don't retry |
| 401 | `UNAUTHORIZED` | Check API key is correct and in `x-api-key` header |
| 403 | `FORBIDDEN` | Feature requires a higher plan |
| 413 | `FILE_TOO_LARGE` | Reduce file size or compress |
| 415 | `UNSUPPORTED_FORMAT` | Check supported formats table above |
| 429 | `RATE_LIMIT_EXCEEDED` | Wait for `x-ratelimit-reset`, then retry |

## After Upload

- The returned `url` is immediately shareable
- `status` will be `ready` for directly-supported formats
- Share the URL anywhere — it renders in a branded wrapper with analytics
- View analytics in the Prismism dashboard at https://prismism.dev/dashboard
