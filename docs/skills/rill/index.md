---
layout: default
title: "Rill — Analyse de données locale"
description: "Démarrer Rill, utiliser son MCP et interroger les métriques en local avec DuckDB."
icon: fontawesome/solid/chart-bar
subtitle: "BI locale avec Rill"
command: "/rill"
tags:
  - rill
  - data
  - mcp
  - duckdb
  - analytics
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/rill/src/rill.md"
---
# 📊 Rill — Analyse de données locale

!!! info "Commande Claude Code"

    **Commande** : `/rill`  
    **Tags** : :material-tag-outline: `rill` :material-tag-outline: `data` :material-tag-outline: `mcp` :material-tag-outline: `duckdb`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/rill/src/rill.md)

    *Skill pour travailler avec un projet [Rill](https://docs.rilldata.com) en local : démarrer le serveur, utiliser le MCP, interroger les métriques et rafraîchir les données.*

---

## 📥 Installation

```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/rill.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/rill/src/rill.md
```

---

## 📝 Utilisation

```text
/rill
```

---

## ✨ Ce que couvre ce skill

- **Démarrer Rill** : `rill start`, options `--no-open`, `--port` — installation globale `/usr/local/bin/rill`
- **MCP Rill** : workflow complet `project_status` → `list_metrics_views` → `get_metrics_view` → `query_metrics_view_summary` → `query_metrics_view`
- **Requêtes métriques** : comparaison temporelle J vs J-1, filtre par dimension, groupement par jour
- **Rafraîchir les données** : forcer la réconciliation quand la source DuckDB externe est mise à jour
- **SQL direct** : requêtes via `rill query --local`

---

## ⚙️ Prérequis

- Rill installé globalement : `rill version`
- Un projet Rill dans le répertoire courant (fichier `rill.yaml` présent)
- Rill démarré : `rill start --no-open`

---

## 🔗 Sources

- [Rill documentation](https://docs.rilldata.com)
- [Rill GitHub](https://github.com/rilldata/rill)
