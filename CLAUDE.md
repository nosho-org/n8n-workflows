# n8n Workflows — nosho-org

## Projet
Automatisations n8n déployées sur **https://n8n.nosho.cc**

## MCP n8n
Le serveur MCP `n8n` est configuré dans `.mcp.json` (non versionné).
Template disponible dans `.mcp.json.example`.

**URL MCP native** : `https://n8n.nosho.cc/mcp-server/http`
Transport : Streamable HTTP avec Bearer token.

### Outils MCP disponibles
- Recherche et configuration de nodes n8n
- Validation de workflows
- Création et gestion de workflows via l'API n8n

## Skills Claude Code disponibles
- `n8n-node-configuration` — Configuration de nodes, propriétés requises
- `n8n-code-javascript` — Code JS dans les nodes Code
- `n8n-code-python` — Code Python dans les nodes Code
- `n8n-workflow-patterns` — Patterns architecturaux de workflows
- `n8n-expression-syntax` — Syntaxe des expressions `{{ }}`
- `n8n-validation-expert` — Interprétation des erreurs de validation
- `n8n-mcp-tools-expert` — Guide d'utilisation des outils MCP n8n

## Conventions
- Les workflows sont exportés en JSON dans `workflows/`
- Les credentials ne sont jamais versionnés
- Nommer les workflows : `[domaine]-[action]-[déclencheur].json`
