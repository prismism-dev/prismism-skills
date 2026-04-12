---
name: gated-content
description: Use when adding access control to Prismism artifacts (access levels, passwords, email capture, domain restrictions) or when programmatically unlocking gated content to retrieve file bytes.
inputs:
  - name: PRISMISM_API_KEY
    description: Prismism API key (prefixed with pal_).
    required: true
---

# Gated Content on Prismism

Control who can view your artifacts and unlock gated content programmatically.

## Access Levels

Every artifact has an `access` level that controls who can view it:

| Level | Who Can View | Plan Required |
|-------|-------------|---------------|
| **`public`** | Anyone with the link (default) | Free |
| **`private`** | Only the owner (via API key or dashboard) | Free |
| **`allowlist`** | Only verified emails via magic link | Plus+ |

### Setting access at upload

```bash
# Private artifact (owner only)
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@internal.pdf" \
  -F "title=Internal Notes" \
  -F "access=private"

# Allowlist artifact (verified emails only)
curl -X POST https://prismism.dev/v1/artifacts \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -F "file=@proposal.pdf" \
  -F "title=Proposal" \
  -F "access=allowlist" \
  -F 'allowedEmails=["alice@acme.com","bob@acme.com"]'
```

### Changing access after upload

```bash
# Make private
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"access": "private"}'

# Set allowlist with specific emails
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"access": "allowlist", "allowedEmails": ["alice@acme.com"]}'

# Make public again
curl -X PATCH https://prismism.dev/v1/artifacts/{id} \
  -H "x-api-key: $PRISMISM_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"access": "public"}'
```

### Magic links for allowlist access

When an artifact has `access: "allowlist"`, visitors see a branded gate page. They enter their email, and if it's on the allowlist, they receive a magic link via email. Clicking the link sets a viewer session cookie (30-day expiry, scoped to that specific artifact).

Magic link tokens expire after 15 minutes. Rate limited to 3 magic links per email per artifact per hour.

## Content Gates

On top of access levels, public artifacts can have additional gates:

| Gate | What It Does | Plan Required |
|------|-------------|---------------|
| **Password** | Viewers enter a password before viewing | Free (all plans) |
| **Email capture** | Viewers provide their email before viewing | Pro+ |
| **Domain allowlist** | Only emails from specified domains can access | Pro+ |

**Important:** Password and email gates only apply to `public` artifacts. Setting access to `private` or `allowlist` silently clears any existing password and email gates.

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

There are two ways for agents to access gated content:

### Method 1: Owner API key bypass (recommended for artifact owners)

If you are the artifact owner, simply use your API key with the raw content endpoint. Owner API key access **bypasses all gates unconditionally** — no password or email needed.

```bash
curl https://prismism.dev/v1/artifacts/{id}/content \
  -H "x-api-key: $PRISMISM_API_KEY" \
  --output file.pdf
```

This is the simplest approach when the agent has the owner's API key.

### Method 2: Gate negotiation via content endpoint (for non-owners)

When accessing gated content without owner credentials, the content endpoint returns 402 with `_hints` explaining what's needed:

**Password-protected content:**
```bash
# First attempt — returns 402 with hints
curl https://prismism.dev/v1/artifacts/{id}/content

# Response: {"error": "PASSWORD_REQUIRED", "_hints": {"unlock": "...retry with X-Prismism-Password header..."}}

# Retry with password
curl https://prismism.dev/v1/artifacts/{id}/content \
  -H "X-Prismism-Password: secret123" \
  --output file.pdf
```

**Email-gated content:**
```bash
# First attempt — returns 402 with hints
curl https://prismism.dev/v1/artifacts/{id}/content

# Response: {"error": "EMAIL_REQUIRED", "_hints": {"unlock": "...retry with X-Prismism-Email header..."}}

# Retry with email
curl https://prismism.dev/v1/artifacts/{id}/content \
  -H "X-Prismism-Email: viewer@acme.com" \
  --output file.pdf
```

**Alternative: query parameters**
```bash
curl "https://prismism.dev/v1/artifacts/{id}/content?password=secret123"
curl "https://prismism.dev/v1/artifacts/{id}/content?email=viewer@acme.com"
```

### Method 3: Unlock endpoint (for browser-based flows)

The unlock endpoint is used by browser viewers and sets a session cookie:

```bash
# Unlock with password
curl -X POST https://prismism.dev/p/{shortId}/unlock \
  -H "Content-Type: application/json" \
  -d '{"gate": "password", "value": "secret123"}'

# Unlock with email
curl -X POST https://prismism.dev/p/{shortId}/unlock \
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

The token can be used as a Bearer token for subsequent content/download requests:

```bash
curl https://prismism.dev/p/{shortId}/content \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIs..." \
  --output file.pdf
```

**For combined gates** (password + email), unlock each gate separately. Both must be unlocked before content is accessible.

## Important Notes

- **Owner API key always bypasses all gates** — this is the simplest path for agents acting on behalf of the artifact owner.
- **Private artifacts** return a branded "This link is private" page to non-owners. Only the owner (via API key or dashboard session) can view.
- **Allowlist artifacts** show a gate where visitors enter their email to receive a magic link. Only emails on the allowlist receive the link.
- **Bearer tokens are scoped** to a specific artifact and contact — they cannot be reused across artifacts.
- **For combined gates** (password + email), unlock each gate separately. Both must be unlocked before content is accessible.

## Error Handling

| Code | Error | Meaning |
|------|-------|---------|
| 402 | `PASSWORD_REQUIRED` | Artifact is password protected. Provide password via `X-Prismism-Password` header or `?password=` param. |
| 402 | `EMAIL_REQUIRED` | Artifact requires email. Provide via `X-Prismism-Email` header or `?email=` param. |
| 402 | `INVALID_PASSWORD` | Wrong password. Retry with correct password. |
| 403 | `PRIVATE_ARTIFACT` | Artifact is private — owner only. Use owner API key to bypass. |
| 403 | `ALLOWLIST_ONLY` | Artifact is restricted to approved emails. Visit the viewer page to request access via magic link. |
| 403 | `FORBIDDEN` | Email domain not allowed, or feature requires upgrade |
| 404 | `NOT_FOUND` | Artifact doesn't exist |
| 410 | `EXPIRED` | Artifact has passed its expiration date |
