# Bibliothèque de Skills

Bienvenue dans cette collection de commandes personnalisées pour **Claude Code**.

## Skills disponibles

### 🚀 Recherche AVPs OPT-NC
Accompagnement complet pour postuler à l'OPT-NC : recherche ciblée, profilage intelligent et aide à la rédaction de candidature.
*   **Commande** : `/opt-nc-avps`
*   [En savoir plus](skills/opt-nc-avps/index.md)

---

## 📦 Installation rapide

L'installation se fait en deux étapes :

### 1. Serveur MCP
```bash
claude mcp add avps-opt-nc --transport sse https://opt-nc-avps.hf.space/gradio_api/mcp/sse
```

### 2. Skill (Slash Command)
```bash
curl -o ~/.claude/commands/opt-nc-avps.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/opt-nc-avps/index.md
```

## 🤝 Comment contribuer ?

Les contributions sont les bienvenues ! Si vous avez créé un skill utile, n'hésitez pas à **ouvrir une Pull Request** pour l'ajouter à la collection.

1. Forkez le projet.
2. Créez votre dossier dans `docs/skills/`.
3. Proposez votre modification via une PR.
