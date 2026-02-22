# n8n Workflows — nosho-org

Collection of automation workflows deployed on [n8n.nosho.cc](https://n8n.nosho.cc).

Built and maintained with **Claude Code** using the n8n MCP integration.

---

## Structure

```
.
├── workflows/          # Exported n8n workflow JSON files
├── .mcp.json           # Claude Code MCP config (local only, not committed)
├── .mcp.json.example   # MCP config template
└── CLAUDE.md           # Claude Code project instructions
```

## Setup

### Prerequisites

- [Node.js](https://nodejs.org) ≥ 18
- [Claude Code](https://claude.ai/claude-code)
- Access to an n8n instance

### MCP Configuration

Copy the example config and fill in your credentials:

```bash
cp .mcp.json.example .mcp.json
```

Edit `.mcp.json` with your n8n instance URL and API key:

```json
{
  "mcpServers": {
    "n8n": {
      "command": "npx",
      "args": ["-y", "n8n-mcp"],
      "env": {
        "N8N_HOST": "https://your-n8n-instance.example.com",
        "N8N_API_KEY": "your-api-key",
        "N8N_VERSION": "1"
      }
    }
  }
}
```

Then reload Claude Code in this project — the `n8n` MCP server will be available.

### Generate an n8n API Key

1. Open your n8n instance → **Settings** → **API**
2. Click **Create API key**
3. Copy the key into `.mcp.json`

## Workflows

Workflows are stored as JSON exports in `workflows/`. To import one:

1. Open n8n → **Workflows** → **Import from File**
2. Select the desired JSON file

## Available Claude Code Skills

| Skill | Usage |
|---|---|
| `n8n-node-configuration` | Node setup, required fields, property dependencies |
| `n8n-code-javascript` | JS code in Code nodes (`$input`, `$json`, `$node`) |
| `n8n-code-python` | Python code in Code nodes |
| `n8n-workflow-patterns` | Workflow architecture patterns |
| `n8n-expression-syntax` | `{{ }}` expression syntax and debugging |
| `n8n-validation-expert` | Validation error interpretation |
| `n8n-mcp-tools-expert` | Guide for using n8n MCP tools |

## License

MIT
