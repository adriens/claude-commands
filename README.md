# claude-commands

Collection de slash commands (skills) pour [Claude Code](https://claude.ai/code).

## Installation

```bash
# Installer un skill globalement
curl -o ~/.claude/commands/<skill>.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/<skill>.md
```

## Skills disponibles

| Skill | Commande | Description | Prérequis |
|-------|----------|-------------|-----------|
| [opt-nc-avps](opt-nc-avps.md) | `/opt-nc-avps <profil>` | Recherche d'AVPs OPT-NC avec accompagnement candidature | `claude mcp add avps-opt-nc --transport sse https://opt-nc-avps.hf.space/gradio_api/mcp/sse` |

## Prérequis MCP

Certains skills dépendent de serveurs MCP, à installer via la CLI Claude Code.

| MCP | Commande d'installation |
|-----|------------------------|
| `avps-opt-nc` | `claude mcp add avps-opt-nc --transport sse https://opt-nc-avps.hf.space/gradio_api/mcp/sse` |
