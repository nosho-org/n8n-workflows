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
├── workflows/
│   ├── chatbot-slack-openrouter.json          # ChatBot — Slack + AI Agent + OpenRouter
│   ├── whatsbot-waha-openrouter.json          # WhatsBot — WhatsApp (WAHA) + AI Agent + OpenRouter
│   ├── qonto-invoice-supabase-mollie.json     # Qonto Invoice — Supabase + Mollie → Qonto
│   └── pappers-siret-supabase-webhook.json    # Pappers → Supabase — Recherche SIRET
├── .mcp.json                                  # Claude Code MCP config (local only)
├── .mcp.json.example                          # MCP config template
└── CLAUDE.md                                  # Claude Code project instructions
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

### WhatsBot (`whatsbot-waha-openrouter.json`)

WhatsApp chatbot powered by an n8n AI Agent with OpenRouter as LLM, using [WAHA](https://waha.nosho.cc) as WhatsApp gateway.

**Flow :**
```
WAHA Webhook → Filtre messages entrants → Extraire message → AI Agent → Envoyer réponse WhatsApp
                                                                   ↑
                                                       OpenRouter Chat Model (glm-4.7-flash)
                                                       Mémoire conversation (10 msgs)
```

**Nodes :**
| Node | Type | Role |
|---|---|---|
| WAHA Webhook | `n8n-nodes-base.webhook` | Reçoit les events WAHA (`message`) |
| Filtre messages entrants | `n8n-nodes-base.filter` | Bloque `fromMe` et non-messages |
| Extraire message | `n8n-nodes-base.code` | Normalise `waFrom`, `waText`, `sessionId` |
| AI Agent | `@n8n/n8n-nodes-langchain.agent` | Agent conversationnel |
| OpenRouter Chat Model | `@n8n/n8n-nodes-langchain.lmChatOpenRouter` | LLM via OpenRouter (`z-ai/glm-4.7-flash`) |
| Mémoire conversation | `@n8n/n8n-nodes-langchain.memoryBufferWindow` | Mémoire glissante (10 échanges) |
| Envoyer réponse WhatsApp | `n8n-nodes-base.httpRequest` | POST `/api/sendText` sur WAHA |

**Webhook URL n8n :**
```
https://n8n.nosho.cc/webhook/c7a2e3f1-4d8b-4e9a-b012-5c6d7e8f9a1b
```

**Variables d'environnement WAHA (Coolify) :**
| Variable | Valeur |
|---|---|
| `WHATSAPP_API_KEY` | *(voir `.mcp.json` local)* |
| `WHATSAPP_HOOK_URL` | `https://n8n.nosho.cc/webhook/c7a2e3f1-4d8b-4e9a-b012-5c6d7e8f9a1b` |
| `WHATSAPP_HOOK_EVENTS` | `message` |

**Credentials requis :**
- `OpenRouter API` — Clé API [openrouter.ai](https://openrouter.ai)

### Qonto Invoice (`qonto-invoice-supabase-mollie.json`)

Création automatique de factures dans Qonto à partir des paiements Mollie et des données Supabase. Gère le backlog de factures passées et les nouvelles commandes.

**Flow :**
```
Supabase DB Webhook ──┐
Chat Trigger ─────────┼──> Normaliser Input
HTTP Webhook Manuel ──┘          │
                          Has Order Item ID?
                         ┌──YES──┴──NO──┐
                         ▼              ▼
               Supabase Lookup    Mollie ID Disponible?
                    │            ┌──YES──┴──NO──┐
               En Supabase?      ▼              ▼
              ┌─YES──┴─NO─┐  Mollie API    Erreur
              ▼            ▼     │
         Données HT    Mollie ID?│
              │            │     │
              └──────┬─────┘     │
                     ▼           │
              Get Organisation ◄─┘
                     │
                Org Valide?
               ┌─YES──┴─NO─┐
               ▼            ▼
        Construire       Erreur
         Facture
            │
            ▼
    Qonto POST Invoice (draft)
            │
            ▼
       Répondre Webhook
```

**Nodes (19) :**
| Node | Type | Role |
|---|---|---|
| Supabase DB Webhook | `webhook` | Trigger sur INSERT `order_items` |
| Chat Trigger | `chatTrigger` | Rattrapage backlog via n8n UI |
| HTTP Webhook Manuel | `webhook` | `POST /webhook/create-invoice` |
| Normaliser Input | `code` | Unifie les 3 formats d'entrée |
| Has Order Item ID ? | `if` | Route selon présence d'un UUID |
| Supabase: Get Order Item | `supabase` | Lookup `order_items` par ID |
| Trouve en Supabase ? | `if` | Vérifie si l'item existe |
| Preparer Donnees Supabase | `code` | `unit_price` cents → EUR HT |
| Mollie ID Disponible ? | `if` | Vérifie présence d'un `tr_xxx` |
| Mollie: Get Payment | `httpRequest` | `GET /v2/payments/{id}` |
| Preparer Donnees Mollie | `code` | TTC → HT (`amount / 1.20`) |
| Erreur: Pas de Mollie ID | `code` | Erreur si aucune source |
| Supabase: Get Organisation | `supabase` | Lookup `organizations` par ID |
| Org Valide ? | `if` | Vérifie `qonto_client_id` + `billing_name` |
| Construire Facture Qonto | `code` | Build payload invoice |
| Qonto: POST Invoice Draft | `httpRequest` | `POST /v2/client_invoices` |
| Resultat Final | `code` | Format réponse |
| Repondre Webhook | `respondToWebhook` | Réponse HTTP |
| Erreur: Org Incomplete | `code` | Erreur si org incomplète |

**Calcul TVA :**
- Les montants Mollie sont **TTC** (taxe incluse)
- HT = TTC / 1.20, TVA = 20% (`vat_rate: "0.2000"`)
- Les montants Supabase (`unit_price`) sont déjà en **centimes HT**

**Dates :**
- `issue_date` = date de paiement Mollie (`paidAt`) ou `created_at` Supabase
- `due_date` = `issue_date` (facture déjà réglée)

**Utilisation :**
```bash
# Créer une facture depuis un paiement Mollie
curl -X POST https://n8n.nosho.cc/webhook/create-invoice \
  -H "Content-Type: application/json" \
  -d '{"mollie_payment_id": "tr_WoLiBK5qxDZ7u9UHrobMJ"}'

# Créer une facture depuis un order_item Supabase
curl -X POST https://n8n.nosho.cc/webhook/create-invoice \
  -H "Content-Type: application/json" \
  -d '{"order_item_id": "uuid-de-lorder-item"}'
```

**Credentials requis :**
- `Qonto API` — Header auth `Authorization: {login}:{secret_key}`
- `Mollie API Live` — Header auth `Authorization: Bearer {api_key}`
- `Supabase account` — API Supabase

### Pappers → Supabase (`pappers-siret-supabase-webhook.json`)

Recherche d'entreprises via l'API Pappers et enrichissement des données de facturation dans Supabase. Met également à jour le client Qonto si existant.

**Flow :**
```
POST /webhook/update-billing-data
         │
    SIRET fourni ?
   ┌─YES──┴──NO─┐
   ▼             ▼
Pappers      Pappers
Détails      Recherche → Format Résultats → Répondre (liste)
   │
   ▼
Extraire Données Org
   │
   ├──> Supabase Upsert (filtre ilike sur name)
   ├──> Répondre Confirmation
   └──> Get Organisation → A un Qonto ID ? → Update Qonto Client
```

**Utilisation :**
```bash
# Recherche par nom
curl -X POST https://n8n.nosho.cc/webhook/update-billing-data \
  -H "Content-Type: application/json" \
  -d '{"query": "Nom de la société"}'

# Confirmation par SIRET
curl -X POST https://n8n.nosho.cc/webhook/update-billing-data \
  -H "Content-Type: application/json" \
  -d '{"siret": "88782780600027"}'
```

**Credentials requis :**
- `Supabase account` — API Supabase
- `Qonto API` — Header auth (pour mise à jour client Qonto)

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
