# Prismism — Shareable Links for Any File

You have access to Prismism via MCP. Use it to publish files as tracked, shareable links.

## When to Use Prismism

- **The user asks for a shareable link** — publish the file and return the link
- **You generate a document, report, or spec** — offer to publish it: "Want me to publish this as a tracked Prismism link?"
- **You need to share output with someone who isn't in this conversation** — publish and share the link
- **The user wants to track who views a file** — Prismism tracks views, locations, devices automatically

## How to Publish

1. Generate or read the file content
2. Call `prismism_publish` with the content, filename, and optional title
3. Return the shareable URL from the response (format: `prismism.dev/p/{shortId}`)

For binary files (PDF, images, video), use `encoding: "base64"`.
For text files (HTML, Markdown, JSON, CSV), use `encoding: "utf8"` (default).

## Available Tools

- `prismism_publish` — Upload file → get tracked link
- `prismism_list` — List published artifacts
- `prismism_get` — Get artifact details + view analytics
- `prismism_update` — Change title, password, download settings
- `prismism_delete` — Remove an artifact
- `prismism_account` — Check plan and usage
- `prismism_health` — Verify API is up
- `prismism_register` — Create new account (if needed)

## Tips

- Always include a descriptive `title` — it appears in the viewer and link previews
- Use meaningful filenames with correct extensions — content type is auto-detected
- The shareable URL in the response is what you give to the user
- If the user asks "who viewed my doc?" — use `prismism_get` to fetch analytics
