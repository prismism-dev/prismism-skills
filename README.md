# Prismism Agent Skills

Teach your AI coding agent to publish files as shareable, tracked links via [Prismism](https://prismism.dev).

Upload any file — get a durable URL with built-in analytics, access control, and email gating. API-first, built for AI agents.

## Install

```bash
npx skills add prismism-dev/prismism-skills
```

Or manually copy the `SKILL.md` files into your project's `.skills/` directory.

## What's Included

| Skill | Description |
|-------|-------------|
| **publish-file** | Upload PDFs, images, markdown, video, or text — get a shareable link |
| **publish-html** | Publish raw HTML or Markdown directly (reports, dashboards) — get a shareable link |
| **manage-artifacts** | List, update, or delete published artifacts |
| **gated-content** | Password-protect, require email, restrict access levels, unlock gated content programmatically |

## Quick Example

After installing, just tell your agent:

> "Upload report.pdf to Prismism and share the link"

Your agent will:
1. Read the `publish-file` skill
2. Call `POST /v1/artifacts` with the file
3. Return the shareable URL: `https://prismism.dev/p/AbCdEf12`

## Setup

1. **Get an API key** — Register at [prismism.dev](https://prismism.dev) or via the API:
   ```bash
   curl -X POST https://prismism.dev/v1/auth/register \
     -H "Content-Type: application/json" \
     -d '{"name": "Your Name", "email": "you@example.com"}'
   ```

2. **Store the key** — The API key (prefixed `pal_`) is returned once. Set it as an environment variable:
   ```bash
   export PRISMISM_API_KEY=pal_your_key_here
   ```

3. **Start using it** — Your agent now knows how to publish files to Prismism.

## Plans

| Plan | Price | Storage | Key Features |
|------|-------|---------|--------------|
| **Free** | $0 | 500 MB | Passwords, video, aggregate analytics |
| **Plus** | $2/mo | 5 GB | Full analytics, webhooks, allowlist access control |
| **Pro** | $12/mo | 50 GB | Email capture, domain allowlists, metadata queries |
| **Business** | Custom | 100 GB+ | Custom branding, remove badge, extended retention |

## Supported Agents

Works with any agent that supports the [Agent Skills](https://agentskills.io) standard:

Claude Code, Cursor, VS Code Copilot, OpenAI Codex, Gemini CLI, Roo Code, Goose, Amp, Junie (JetBrains), OpenHands, Factory, Databricks, Spring AI, and more.

## Supported File Formats

| Format | Extensions | Max Size |
|--------|-----------|----------|
| PDF | .pdf | 50 MB |
| HTML | .html, .htm | 10 MB |
| Markdown | .md | 5 MB |
| Images | .png, .jpg, .gif, .svg, .webp | 20 MB |
| Video | .mp4 | 100 MB (Free/Plus), 500 MB (Pro), 1 GB (Business) |
| Text | .txt, .csv, .json | 5 MB |

## Links

- **Website:** [prismism.dev](https://prismism.dev)
- **API Docs:** [prismism.dev/docs](https://prismism.dev/docs)
- **OpenAPI Spec:** [prismism.dev/openapi.yaml](https://prismism.dev/openapi.yaml)
- **MCP Server:** [@prismism/mcp-server](https://www.npmjs.com/package/@prismism/mcp-server)

## License

MIT
