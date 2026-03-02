---
name: gated-content
description: Use when adding access control to Prismism artifacts (passwords, email capture, domain restrictions) or when programmatically unlocking gated content to retrieve file bytes.
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_).
    required: true
---

# Gated Content on Prismism

Add access control to artifacts and unlock gated content programmatically.

## Access Gates

Prismism supports two gates that can be combined:

| Gate | What It Does | Plan Required |
|------|-------------|---------------|
| **Password** | Viewers enter a password before viewing | Starter |
| **Email capture** | Viewers provide their email before viewing | Starter |
| **Domain allowlist** | Only emails from specified domains can access | Pro |

## Setting Gates at Upload

### Password protection

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@deck.pdf" \
  -F "title=Board Deck" \
  -F "password=secret123"
```

### Email gate

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@proposal.pdf" \
  -F "title=Proposal" \
  -F "requireEmail=true"
```

### Email gate with domain restriction

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@proposal.pdf" \
  -F "title=Proposal for Acme" \
  -F "requireEmail=true" \
  -F 'allowedDomains=["acme.com","partner.co"]'
```

### Combined gates (password + email)

```bash
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@sensitive.pdf" \
  -F "title=Confidential Report" \
  -F "password=secret123" \
  -F "requireEmail=true"
```

## Adding/Removing Gates After Upload

```bash
# Add password to existing artifact
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"password": "newpass"}'

# Remove password
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"password": null}'

# Enable email gate
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"requireEmail": true}'

# Disable email gate
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"requireEmail": false}'
```

## Unlocking Gated Content Programmatically

When content is gated, you need to unlock it before accessing the file bytes. This is the flow for AI agents and API callers.

### Step 1: Unlock

**Endpoint:** `POST https://prismism.dev/p/{shortId}/unlock`

```bash
# Unlock with password
curl -X POST https://prismism.dev/p/AbCdEf12/unlock \
  -H "Content-Type: application/json" \
  -d '{"gate": "password", "value": "secret123"}'

# Unlock with email
curl -X POST https://prismism.dev/p/AbCdEf12/unlock \
  -H "Content-Type: application/json" \
  -d '{"gate": "email", "value": "viewer@acme.com"}'
```

**Response:**
```json
{
  "success": true,
  "token": "eyJhbGciOiJIUzI1NiIs..."
}
```

### Step 2: Fetch content with the Bearer token

```bash
# Stream content (for rendering)
curl https://prismism.dev/p/AbCdEf12/content \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  --output file.pdf

# Download original file
curl https://prismism.dev/p/AbCdEf12/download/original \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  --output file.pdf

# Download PDF version (when available)
curl https://prismism.dev/p/AbCdEf12/download/pdf \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  --output file.pdf
```

## Important Notes

- **Password protection requires Starter plan** ($5/mo). Free plan returns 403 with `upgradeUrl`.
- **Bearer tokens are scoped** to a specific artifact and contact — they cannot be reused across artifacts.
- **For combined gates** (password + email), unlock each gate separately. Both must be unlocked before content is accessible.
- **Browser users** get an httpOnly cookie set automatically — no need to handle Bearer tokens manually.
- **No auth header needed** for the unlock endpoint itself — it's a public endpoint. The `x-api-key` is only for managing artifacts (create, update, delete), not for viewing them.

## Error Handling

| Code | Error | Meaning |
|------|-------|---------|
| 403 | `FORBIDDEN` | Wrong password, email domain not allowed, or feature requires upgrade |
| 404 | `NOT_FOUND` | Artifact doesn't exist |
| 410 | `EXPIRED` | Artifact has passed its expiration date |
