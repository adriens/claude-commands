---
layout: default
title: "Analyse CV JSON Resume"
description: "Charge un CV JSON Resume depuis le registry ou une URL, et propose gap analysis, pitch et lettre de motivation ciblée."
command: "/json-resume"
tags:
  - cv
  - emploi
  - json-resume
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/json-resume/src/json-resume.md"
---
# 📄 Analyse CV JSON Resume

!!! info "Commande Claude Code"

    **Commande** : `/json-resume`  
    **Tags** : :material-tag-outline: `cv` :material-tag-outline: `emploi` :material-tag-outline: `json-resume`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/json-resume/src/json-resume.md)

    *Analyse complète d'un CV JSON Resume : synthèse, gap analysis avec une offre, pitch et lettre de motivation.*

---

## 📥 Installation

```bash
curl -o ~/.claude/commands/json-resume.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/json-resume/src/json-resume.md
```

---

## 📝 Utilisation

```text
/json-resume adriens
```

```text
/json-resume https://raw.githubusercontent.com/username/repo/main/resume.json
```

```text
/json-resume
```

---

## ✨ Fonctionnalités

- 🔗 **Chargement automatique** depuis `registry.jsonresume.org/{username}`, une URL ou un fichier local
- 📊 **Synthèse du profil** : expériences, compétences, réalisations notables
- 🎯 **Gap analysis** CV ↔ offre d'emploi avec tableau d'adéquation coloré
- 🗣️ **Pitch** : 3 variantes (LinkedIn, entretien, email spontané)
- ✍️ **Lettre de motivation** personnalisée en Markdown ou AsciiDoc, exportable en Word

---

## 🔗 Prérequis

Un CV publié au format [JSON Resume](https://jsonresume.org/) :

- Via le registry : créer un Gist public nommé `resume.json` → disponible sur `registry.jsonresume.org/{username}`
- Ou via n'importe quelle URL publique pointant vers un fichier JSON conforme au schéma JSON Resume
