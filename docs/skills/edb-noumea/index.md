---
layout: default
title: "Eaux de Baignade Nouméa"
description: "Consulte la qualité sanitaire des eaux de baignade sur les plages de Nouméa (E. coli, Entérocoques)."
icon: fontawesome/solid/water
subtitle: "Qualité des plages de Nouméa"
command: "/edb-noumea"
tags:
  - environnement
  - baignade
  - nouvelle-calédonie
  - nouméa
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/edb-noumea/src/edb-noumea.md"
datasource: "https://github.com/adriens/edb-noumea-data"
---
# 🏖️ Eaux de Baignade Nouméa

![Logo Ville de Nouméa](../../assets/logos/edb-noumea.svg)

!!! info "Commande Claude Code"

    **Commande** : `/edb-noumea`  
    **Tags** : :material-tag-outline: `environnement` :material-tag-outline: `baignade` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/edb-noumea/src/edb-noumea.md)

    *Consulte la qualité sanitaire des eaux de baignade sur les plages de Nouméa.*

---

## 🌊 Qu'est-ce que l'EDB ?

Le programme de surveillance des **Eaux De Baignade (EDB)** est un suivi microbiologique régulier des plages de **Nouméa, Nouvelle-Calédonie**. Des prélèvements sont effectués pour mesurer la concentration en bactéries indicatrices de contamination fécale :

- **E. coli** : indicateur de contamination d'origine humaine ou animale
- **Entérocoques intestinaux** : indicateur complémentaire de la qualité sanitaire

La qualité est classée en quatre niveaux :

| Statut | Signification |
|--------|--------------|
| 🟢 **Bonne** | Baignade recommandée |
| 🟡 **Acceptable** | Baignade possible avec prudence |
| 🔴 **Mauvaise** | Baignade déconseillée |
| ⚫ **Insuffisante** | Données insuffisantes |

---

## 📥 Installation

L'installation se fait en deux étapes dans votre terminal Claude Code.

### 1️⃣ Serveur MCP (Prérequis)
Installez d'abord le serveur de données :
```bash
claude mcp add --transport sse edb-noumea \
  https://rastadidi-edb-noumea.hf.space/gradio_api/mcp/sse
```

### 2️⃣ Skill (Slash Command)
Téléchargez le skill dans votre répertoire de commandes :
```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/edb-noumea.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/edb-noumea/src/edb-noumea.md
```

---

## 📝 Utilisation

```text
/edb-noumea
```

Ou directement avec le nom d'une plage :

```text
/edb-noumea Anse Vata
```

```text
/edb-noumea Citrons
```

---

## ✨ Fonctionnalités

- 🗺️ **Vue d'ensemble** : statut sanitaire de toutes les plages de Nouméa
- 🔬 **Détail par plage** : valeurs E. coli et Entérocoques avec historique
- 🏊 **Avis de baignade** : recommandation claire selon les seuils réglementaires
- 📊 **Historique** : évolution de la qualité dans le temps
