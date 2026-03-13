# Prismism MCP — Client Configuration Guide

Copy-paste configs for every major MCP client. All use the same hosted endpoint.

---

## Claude Code (CLI)

**One-liner:**
```bash
claude mcp add --transport http prismism https://prismism.dev/mcp \
  --header "x-api-key: YOUR_API_KEY"
```

**Project config** (`.mcp.json` in project root):
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

**Global config** (`~/.claude.json`):
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

---

## Claude Desktop

**`claude_desktop_config.json`:**
```json
{
  "mcpServers": {
    "prismism": {
      "url": "https://prismism.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

---

## Cursor

**`.cursor/mcp.json`** (project) or **`~/.cursor/mcp.json`** (global):
```json
{
  "mcpServers": {
    "prismism": {
      "url": "https://prismism.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

No `type` field needed — Cursor auto-detects from URL.

---

## Windsurf

**`~/.windsurf/mcp.json`:**
```json
{
  "mcpServers": {
    "prismism": {
      "url": "https://prismism.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

---

## VS Code (GitHub Copilot)

**`.vscode/mcp.json`** or VS Code user settings:
```json
{
  "mcpServers": {
    "prismism": {
      "type": "http",
      "url": "https://prismism.dev/mcp",
      "headers": {
        "x-api-key": "YOUR_API_KEY"
      }
    }
  }
}
```

---

## npm/stdio (alternative — local install)

For clients that don't support remote HTTP transport:

```json
{
  "mcpServers": {
    "prismism": {
      "command": "npx",
      "args": ["@prismism/mcp-server"],
      "env": {
        "PRISMISM_API_KEY": "YOUR_API_KEY"
      }
    }
  }
}
```

---

## Environment Variables

All configs that use `${PRISMISM_API_KEY}` require:
```bash
export PRISMISM_API_KEY="pal_your_key_here"
```

Add to your shell profile (`~/.zshrc`, `~/.bashrc`) for persistence.
