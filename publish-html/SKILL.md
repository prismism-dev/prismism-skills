---
name: publish-html
description: Use when publishing raw HTML content (reports, dashboards, one-pagers) to Prismism without a file upload. Sends HTML as JSON, returns a shareable tracked link. HTML can be updated in place — the URL stays the same.
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_).
    required: true
---

# Publish HTML to Prismism

Publish HTML content directly without a file upload. Ideal for agent-generated reports, dashboards, and one-pagers.

**Endpoint:** `POST https://prismism.dev/v1/artifacts`
**Content-Type:** `application/json`
**Auth:** `x-api-key` header

## Quick Start

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Sales Dashboard",
    "html": "<html><body><h1>Q4 Sales</h1><p>Revenue: $1.2M</p></body></html>"
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
  "status": "ready"
}
```

The `url` is immediately shareable.

## Parameters

| Parameter | Required | Type | Description |
|-----------|----------|------|-------------|
| `html` | **Yes** | string | Raw HTML content. Can be a full document or fragment. |
| `title` | No | string | Display name shown in the wrapper and link previews. |
| `password` | No | string | Password-protect the artifact (Starter plan required). |
| `requireEmail` | No | boolean | Require email before viewing. |
| `allowedDomains` | No | string[] | Allowed email domains (only with requireEmail). |
| `allowNetwork` | No | boolean | Allow external network requests — fetch, images, scripts (default: false). |
| `expiresAt` | No | string | Auto-expire date (RFC 3339). |

## Network Access

By default, HTML content is **sandboxed** — no external fetch, images, or scripts load. This is a security boundary.

Set `"allowNetwork": true` to allow external resources:

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "title": "Interactive Dashboard",
    "html": "<html><head><script src=\"https://cdn.plot.ly/plotly-2.35.2.min.js\"></script></head><body>...</body></html>",
    "allowNetwork": true
  }'
```

Only enable `allowNetwork` when the HTML genuinely needs external resources (CDN libraries, external images, API calls).

## Updating HTML Content

Unlike file uploads, HTML artifacts can be **updated in place**. The shareable URL stays the same — viewers always see the latest version.

```bash
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"html": "<html><body><h1>Updated Dashboard</h1><p>Revenue: $1.5M</p></body></html>"}'
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
    "html": "<!DOCTYPE html><html><head><style>body{font-family:system-ui;max-width:800px;margin:0 auto;padding:2rem}h1{color:#1e293b}table{width:100%;border-collapse:collapse}td,th{padding:8px;border:1px solid #e2e8f0;text-align:left}</style></head><body><h1>Weekly Analytics</h1><table><tr><th>Metric</th><th>This Week</th><th>Change</th></tr><tr><td>Page Views</td><td>12,450</td><td>+18%</td></tr><tr><td>Unique Visitors</td><td>3,200</td><td>+12%</td></tr></table></body></html>",
    "expiresAt": "2026-03-15T00:00:00Z"
  }'
```
