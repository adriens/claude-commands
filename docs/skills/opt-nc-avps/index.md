---
layout: default
title: "Recherche AVPs OPT-NC"
description: "Recherche d'AVPs OPT-NC avec accompagnement complet à la préparation de candidature."
command: "/opt-nc-avps"
tags:
  - emploi
  - opt-nc
  - nouvelle-calédonie
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/opt-nc-avps/src/opt-nc-avps.md"
---
# 🚀 Recherche d'AVPs OPT-NC

!!! info "Commande Claude Code"

    **Commande** : `/opt-nc-avps`  
    **Tags** : :material-tag-outline: `emploi` :material-tag-outline: `opt-nc` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/opt-nc-avps/src/opt-nc-avps.md)

    *Accompagnement complet à la préparation de candidature pour l'OPT-NC.*

---

## 📥 Installation

L'installation se fait en deux étapes dans votre terminal Claude Code.

### 1️⃣ Serveur MCP (Prérequis)
Installez d'abord le serveur de données :
```bash
claude mcp add avps-opt-nc \
  --transport sse \
  https://opt-nc-avps.hf.space/gradio_api/mcp/sse
```

### 2️⃣ Skill (Slash Command)
Téléchargez le skill dans votre répertoire de commandes :
```bash
curl -o ~/.claude/commands/opt-nc-avps.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/opt-nc-avps/src/opt-nc-avps.md
```

---

## 📝 Utilisation

Une fois installé, lancez simplement la commande en précisant votre profil ou le type de poste :

```text
/opt-nc-avps chef de projet SI
```

### ✨ Fonctionnalités
*   🔍 **Recherche ciblée** des postes ouverts.
*   🧠 **Profilage intelligent** via questionnaire.
*   🛠️ **Plan de préparation** (atouts, points à renforcer, questions d'entretien STAR).
*   ✍️ **Aide à la rédaction** de lettre de motivation personnalisée.
