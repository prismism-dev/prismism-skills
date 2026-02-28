---
name: prismism
description: Use when publishing files or HTML as shareable tracked links via Prismism. Supports PDF, HTML, Markdown, images, and text. Use for sharing reports, docs, dashboards, or any artifact that needs a durable URL with optional analytics, password protection, and email gating.
license: MIT
metadata:
  author: prismism
  version: "1.0.0"
  homepage: https://prismism.dev
  source: https://github.com/prismism-dev/prismism-skills
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_). Get one by registering at https://prismism.dev or via POST /v1/auth/register.
    required: true
---

# Prismism

Upload any file → get a shareable link with built-in analytics and access control. API-first, built for AI agents.

**Base URL:** `https://prismism.dev`
**Auth:** `x-api-key` header with your API key
**OpenAPI spec:** `https://prismism.dev/openapi.yaml`

## Sub-Skills

| Feature | Skill | Use When |
|---------|-------|----------|
| **Upload files** | `publish-file` | Publishing PDFs, images, markdown, or text files as shareable links |
| **Publish HTML** | `publish-html` | Publishing raw HTML content without a file (reports, dashboards) |
| **Manage artifacts** | `manage-artifacts` | Listing, updating, or deleting previously published artifacts |
| **Gated content** | `gated-content` | Adding password protection or email gates, unlocking gated artifacts |

## Quick Routing

**Need to share a file?** → `publish-file`
Upload any supported format and get a URL like `https://prismism.dev/p/AbCdEf12`

**Need to publish generated HTML?** → `publish-html`
Send raw HTML string directly — no file needed. Great for reports, dashboards, one-pagers.

**Need to manage existing artifacts?** → `manage-artifacts`
List all artifacts, update titles or expiration, delete artifacts.

**Need access control?** → `gated-content`
Password-protect, require email, restrict by domain, unlock programmatically.

## Setup

### Get an API Key

Register an account to get your API key:

```bash
curl -X POST https://prismism.dev/v1/auth/register \
  -H "Content-Type: application/json" \
  -d '{"name": "Your Name", "email": "you@example.com", "password": "securepass123"}'
```

The API key is returned **once** in the registration response. Store it securely.

Every new account includes a **30-day free Pro trial** — 5 GB storage, full analytics, password protection, email capture. No credit card required.

### Store the Key

```bash
export PRISMISM_API_KEY=pal_your_key_here
```

### Verify It Works

```bash
curl https://prismism.dev/v1/account \
  -H "x-api-key: $PRISMISM_API_KEY"
```

## Common Mistakes

| Mistake | Fix |
|---------|-----|
| Using `Authorization: Bearer` for API auth | Use `x-api-key` header. Bearer tokens are only for unlocking gated content. |
| Expecting API at `/api/` | Correct base path is `/v1/` |
| Not storing the API key at registration | Key is returned once and cannot be retrieved again |
| Sending `allowedDomains` as repeated form fields | Must be a JSON-encoded string: `'["acme.com"]'` |

## Rate Limits

60 requests per minute per API key. Response headers:
- `x-ratelimit-limit` — max requests in window
- `x-ratelimit-remaining` — requests left
- `x-ratelimit-reset` — UTC epoch when window resets
