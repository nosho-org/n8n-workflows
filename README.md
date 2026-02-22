# n8n Workflows — nosho-org

Collection of automation workflows deployed on [n8n.nosho.cc](https://n8n.nosho.cc).

Built and maintained with **Claude Code** using the n8n MCP integration.

---

## MCP Server

n8n exposes a native **Streamable HTTP MCP server** that allows Claude Code to interact directly with the n8n instance.

**URL** : `https://n8n.nosho.cc/mcp-server/http`

### Configuration (`.mcp.json`)

```json
{
  "mcpServers": {
    "n8n": {
      "url": "https://n8n.nosho.cc/mcp-server/http",
      "headers": {
        "Authorization": "Bearer <your-api-key>"
      }
    }
  }
}
```

Copy `.mcp.json.example` → `.mcp.json` and fill in your API key, then reload Claude Code.

> `.mcp.json` is gitignored — API keys are never committed.

---

## Structure

```
.
├── workflows/                            # Exported n8n workflow JSON files
│   └── chatbot-slack-openrouter.json     # ChatBot — Slack + AI Agent + OpenRouter
├── .mcp.json                             # Claude Code MCP config (local only)
├── .mcp.json.example                     # MCP config template
└── CLAUDE.md                             # Claude Code project instructions
```

---

## Workflows

### ChatBot (`chatbot-slack-openrouter.json`)

Slack chatbot powered by an n8n AI Agent with OpenRouter as LLM.

**Flow :**
```
Slack Trigger → Filtre messages bot → AI Agent → Réponse Slack
                                           ↑
                                    OpenRouter Chat Model
                                    Mémoire conversation (10 msgs)
```

**Nodes :**
| Node | Type | Role |
|---|---|---|
| Slack Trigger | `n8n-nodes-base.slackTrigger` | Écoute les messages Slack |
| Filtre messages bot | `n8n-nodes-base.filter` | Bloque les messages de bots (anti-boucle) |
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | Agent conversationnel |
| OpenRouter Chat Model | `@n8n/n8n-nodes-langchain.lmChatOpenRouter` | LLM via OpenRouter (`openai/gpt-4o-mini`) |
| Mémoire conversation | `@n8n/n8n-nodes-langchain.memoryBufferWindow` | Mémoire glissante (10 échanges) |
| Réponse Slack | `n8n-nodes-base.slack` | Répond en thread dans Slack |

**Credentials requis :**
- `Slack Bot Token` — OAuth bot token avec scopes `channels:history`, `chat:write`, `reactions:write`
- `OpenRouter API` — Clé API [openrouter.ai](https://openrouter.ai)

---

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

---

## License

MIT
