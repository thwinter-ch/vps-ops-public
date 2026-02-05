# MCP Inventory

*Model Context Protocol servers we use for AI assistant integrations.*

---

## Active MCPs

| MCP | Purpose | Source |
|-----|---------|--------|
| **hostinger-mcp** | Manage VPS, domains, DNS via Hostinger API | [hostinger-api-mcp](https://www.npmjs.com/package/hostinger-api-mcp) |
| **typefully** | Schedule and manage social posts (Twitter/LinkedIn) | [typefully-mcp-server](https://github.com/pascalporedda/typefully-mcp-server) |
| **notion** | Read/write Notion databases and pages | Built-in |
| **perplexity** | Web search and research | Built-in |
| **playwright** | Browser automation and testing | Built-in |

---

## Configuration

MCPs are configured in `.mcp.json` at the project root (not committed to git).

**Template:**
```json
{
  "mcpServers": {
    "typefully": {
      "type": "stdio",
      "command": "cmd",
      "args": ["/c", "npx", "-y", "typefully-mcp-server"],
      "env": {
        "TYPEFULLY_API_KEY": "your-key-here"
      }
    }
  }
}
```

---

## Getting API Keys

| MCP | Where to get key |
|-----|------------------|
| Hostinger | Hostinger panel → API section |
| Typefully | Settings → Integrations → API |

Store keys in 1Password. Reference with `op read` where possible.

---

*Last updated: February 5, 2026*
