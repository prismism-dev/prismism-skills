---
name: manage-artifacts
description: Use when listing, viewing details, updating metadata, changing access levels, or deleting existing Prismism artifacts. Covers pagination, filtering, access control, and metadata updates.
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_).
    required: true
---

# Manage Prismism Artifacts

List, view, update, and delete artifacts you've published.

**Base URL:** `https://prismism.dev`
**Auth:** `x-api-key` header

## List Artifacts

```bash
curl https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY"
```

### Pagination Parameters

| Parameter | Default | Max | Description |
|-----------|---------|-----|-------------|
| `page` | 1 | — | Page number (1-indexed) |
| `limit` | 20 | 100 | Items per page |
| `status` | — | — | Filter: `processing`, `converting`, `ready`, `failed` |

### Response

```json
{
  "data": [
    {
      "id": "d4f7a1b2-...",
      "shortId": "AbCdEf12",
      "url": "https://prismism.dev/p/AbCdEf12",
      "title": "Q4 Report",
      "format": "pdf",
      "status": "ready",
      "access": "public",
      "fileSize": 54321,
      "viewCount": 42,
      "hasPassword": false,
      "requireEmail": false,
      "isGated": false,
      "createdAt": "2026-01-15T10:30:00Z"
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 20,
    "total": 42,
    "hasMore": true
  }
}
```

### Pagination Example

```bash
# Get page 2 with 50 items per page
curl "https://prismism.dev/v1/artifacts?page=2&limit=50" \
  -H "x-api-key: $PRISMISM_API_KEY"

# Filter to only ready artifacts
curl "https://prismism.dev/v1/artifacts?status=ready" \
  -H "x-api-key: $PRISMISM_API_KEY"
```

## Get Single Artifact

```bash
curl https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY"
```

Returns the full artifact object including all metadata.

## Update Artifact

```bash
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "New Title"}'
```

### Updatable Fields

| Field | Type | Notes |
|-------|------|-------|
| `title` | string | Display name |
| `html` | string | Replace content (**HTML format only**) |
| `markdown` | string | Replace content (**Markdown format only**) |
| `access` | string | `public`, `private`, or `allowlist` (Plus+) |
| `allowedEmails` | string[] | Emails for allowlist access. Used when `access` is `allowlist`. |
| `password` | string \| null | Set password, or `null` to remove it |
| `requireEmail` | boolean | Toggle email gate (Pro+) |
| `allowedDomains` | string[] | Set allowed email domains (Pro+) |
| `expiresAt` | string \| null | Set expiration (RFC 3339), or `null` to remove |
| `allowNetwork` | boolean | Toggle network access for HTML/Markdown |

**Note:** File content (non-HTML/Markdown) cannot be updated after upload. Create a new artifact instead.

### Access Level Transitions

When changing access levels, be aware of side effects:

- **Setting access to `private` or `allowlist`**: Password and email gates are silently cleared. Response includes `_warnings` explaining what was cleared.
- **Changing away from `allowlist`**: Allowed emails are cleared and all viewer sessions are revoked. Response includes `_warnings`.
- **Setting `allowlist` with empty emails**: Succeeds with a `_warnings` note that nobody will be able to request access.

### Update Examples

```bash
# Change title and add expiration
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"title": "Updated Report", "expiresAt": "2026-06-01T00:00:00Z"}'

# Make private (owner only)
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"access": "private"}'

# Set allowlist access with specific emails
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"access": "allowlist", "allowedEmails": ["alice@acme.com", "bob@partner.co"]}'

# Add password protection
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"password": "newpassword"}'

# Remove password
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"password": null}'

# Make public again
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"access": "public"}'

# Remove expiration (make permanent)
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"expiresAt": null}'
```

## Delete Artifact

```bash
curl -X DELETE https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY"
```

**Response:**
```json
{
  "deleted": true,
  "id": "d4f7a1b2-3c9e-4e8f-b5d6-1a2b3c4d5e6f"
}
```

Permanently deletes the artifact, its files, and frees storage quota immediately. The shareable URL will return 404.

## Check Account Usage

```bash
curl https://prismism.dev/v1/account \
  -H "x-api-key: $PRISMISM_API_KEY"
```

Returns current usage (artifact count, storage bytes) and plan limits. Useful for checking storage before large uploads.
