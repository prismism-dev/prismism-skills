# Prismism for Claude Code

Give Claude Code the ability to publish any file as a tracked, shareable link. DocSend for AI agents.

## Quick Setup

### Option 1: One-liner (recommended)

```bash
claude mcp add --transport http prismism https://prismism.dev/mcp \
  --header "x-api-key: YOUR_API_KEY"
```

### Option 2: Project config (shareable with team)

Add `.mcp.json` to your project root:

```json
{
  "mcpServers": {
    "prismism": {
      "type": "http",
      "url": "https://prismism.dev/mcp",
      "headers": {
        "x-api-key": "${PRISMISM_API_KEY}"
      }
    }
  }
}
```

Then set your API key:
```bash
export PRISMISM_API_KEY="your_key_here"
```

### Option 3: Global config

Add to `~/.claude.json` under `mcpServers` to make Prismism available in all projects.

## Add the Skill (optional but powerful)

Copy `CLAUDE.md` to your project root (or merge with existing):

```bash
cp CLAUDE.md /path/to/your/project/CLAUDE.md
```

This teaches Claude *when* to use Prismism — not just *how*.

## Get an API Key

Don't have one? Claude can create an account for you:

> "Register me for Prismism with name 'Your Name' and email 'you@example.com'"

The API key is returned once — save it immediately.

## What You Can Do

| Command | What it does |
|---------|-------------|
| "Publish this report to Prismism" | Uploads file → returns tracked link |
| "Share my spec as a Prismism link" | Same — any file format works |
| "List my Prismism artifacts" | Shows all your published files |
| "How many views did my report get?" | Fetches analytics for an artifact |
| "Delete the old version" | Removes an artifact |
| "Password-protect this document" | Updates artifact settings |

## Supported Formats

PDF, HTML, Markdown, Images (PNG/JPG/GIF/SVG/WebP), Video (MP4), JSON, CSV, plain text.

## Links

- **API Docs**: [prismism.dev/docs](https://prismism.dev/docs)
- **Dashboard**: [prismism.dev](https://prismism.dev)
- **npm package** (stdio alternative): `npx @prismism/mcp-server`
