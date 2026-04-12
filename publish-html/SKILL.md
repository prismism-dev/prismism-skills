---
name: publish-html
description: Use when publishing raw HTML or Markdown content (reports, dashboards, one-pagers) to Prismism without a file upload. Sends content as JSON, returns a shareable tracked link. Content can be updated in place — the URL stays the same.
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_).
    required: true
---

# Publish HTML or Markdown to Prismism

Publish content directly without a file upload. Ideal for agent-generated reports, dashboards, and one-pagers.

**Endpoint:** `POST https://prismism.dev/v1/artifacts`
**Content-Type:** `application/json`
**Auth:** `x-api-key` header

## Quick Start — HTML

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Sales Dashboard",
    "html": "<h1>Q4 Sales</h1><p>Revenue: $1.2M</p>"
  }'
```

## Quick Start — Markdown

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Meeting Notes",
    "markdown": "# Team Standup\n\n- Sprint goal on track\n- Demo scheduled Friday"
  }'
```

**Response:**
```json
{
  "id": "d4f7a1b2-3c9e-4e8f-b5d6-1a2b3c4d5e6f",
  "shortId": "AbCdEf12",
  "url": "https://prismism.dev/p/AbCdEf12",
  "title": "Sales Dashboard",
  "format": "html",
  "access": "public",
  "status": "ready"
}
```

The `url` is immediately shareable.

## Parameters

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `html` | **One of** | string | Raw HTML content. Can be a full document or fragment. Mutually exclusive with `markdown`. |
| `markdown` | **One of** | string | Raw Markdown content. Stored and rendered as Markdown. Mutually exclusive with `html`. |
| `title` | No | string | Display name shown in the wrapper and link previews. |
| `access` | No | string | `public` (default), `private` (owner only), or `allowlist` (verified emails, Plus+). |
| `password` | No | string | Password-protect the artifact. All plans. |
| `requireEmail` | No | boolean | Require email before viewing (Pro+). |
| `allowedDomains` | No | string[] | Allowed email domains (only with requireEmail, Pro+). |
| `allowedEmails` | No | string[] | Allowed emails for allowlist access. Required when access is `allowlist`. |
| `allowNetwork` | No | boolean | Allow external network requests — fetch, images, scripts (default: false). |
| `expiresAt` | No | string | Auto-expire date (RFC 3339). |

**Important:** Use `html` or `markdown` — not both. Using the wrong field name (`content`, `text`, `body`, `data`) returns a helpful error with the correct field names.

## Network Access

By default, HTML content is **sandboxed** — no external fetch, images, or scripts load. This is a security boundary.

Set `"allowNetwork": true` to allow external resources:

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Interactive Dashboard",
    "html": "...",
    "allowNetwork": true
  }'
```

Only enable `allowNetwork` when the HTML genuinely needs external resources (CDN libraries, external images, API calls).

## Updating Content

HTML and Markdown artifacts can be **updated in place**. The shareable URL stays the same — viewers always see the latest version.

```bash
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"html": "<h1>Updated Dashboard</h1><p>Revenue: $1.5M</p>"}'
```

```bash
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"markdown": "# Updated Notes\n\nNew content here."}'
```

This is powerful for living documents — update the content, same URL, no broken links.

## Best Practices

1. **Include `<html>` and `<body>` tags** — while fragments work, full documents render more predictably.
2. **Inline CSS when possible** — external stylesheets require `allowNetwork: true`.
3. **Set a meaningful `title`** — it appears in the wrapper header and Open Graph link previews.
4. **Use `allowNetwork` sparingly** — sandboxed content is safer and loads faster.

## Example: Agent-Generated Report

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Weekly Analytics Report - Week 12",
    "html": "<h1>Weekly Analytics</h1><table><tr><th>Metric</th><th>This Week</th><th>Change</th></tr><tr><td>Page Views</td><td>12,450</td><td>+18%</td></tr><tr><td>Unique Visitors</td><td>3,200</td><td>+12%</td></tr></table>",
    "expiresAt": "2026-03-15T00:00:00Z"
  }'
```
