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
| [opt-nc-avps](opt-nc-avps.md) | `/opt-nc-avps <profil>` | Recherche d'AVPs OPT-NC avec accompagnement candidature | MCP `avps-opt-nc` |

## Prérequis MCP

Certains skills dépendent de serveurs MCP. Consultez la fiche de chaque skill pour les détails.

- **avps-opt-nc** : serveur MCP OPT-NC pour l'accès aux Avis de Vacances de Poste
