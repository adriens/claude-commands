---
layout: default
title: "Analyse CV JSON Resume"
description: "Charge un CV JSON Resume depuis le registry ou une URL, et propose gap analysis, pitch et lettre de motivation ciblée."
icon: fontawesome/solid/file-lines
subtitle: "CV, lettre & préparation entretien"
command: "/json-resume"
tags:
  - cv
  - emploi
  - json-resume
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/json-resume/src/json-resume.md"
datasource: "https://registry.jsonresume.org/"
---
# 📄 Analyse CV JSON Resume

![Logo JSON Resume](../../assets/logos/json-resume.png)

!!! info "Commande Claude Code"

    **Commande** : `/json-resume`  
    **Tags** : :material-tag-outline: `cv` :material-tag-outline: `emploi` :material-tag-outline: `json-resume`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/json-resume/src/json-resume.md)

    *Analyse complète d'un CV JSON Resume : synthèse, gap analysis avec une offre, pitch et lettre de motivation.*

---

## 📖 Qu'est-ce que JSON Resume ?

[JSON Resume](https://jsonresume.org/) est un standard open source communautaire qui définit un schéma JSON universel pour les CV. L'idée : stocker son CV dans un fichier `resume.json` structuré et interopérable, indépendant de tout outil de mise en forme.

Avantages :
- **Versionnable** : stocké dans un Gist ou un repo Git, l'historique est conservé
- **Interopérable** : un seul fichier source, des dizaines de thèmes de rendu disponibles
- **Exploitable par l'IA** : structure normalisée → facile à analyser, comparer, adapter

Le registry officiel (`registry.jsonresume.org/{username}`) publie automatiquement tout Gist GitHub public nommé `resume.json`.

---

## 🔗 Prérequis

Un CV publié au format [JSON Resume](https://jsonresume.org/) :

- Via le registry : créer un Gist public nommé `resume.json` → disponible sur `registry.jsonresume.org/{username}`
- Ou via n'importe quelle URL publique pointant vers un fichier JSON conforme au schéma JSON Resume

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

## 🔀 Flow de travail

```mermaid
flowchart TD
    classDef input fill:#2980b9,stroke:#1a5276,color:#fff
    classDef enrich fill:#16a085,stroke:#0e6655,color:#fff
    classDef process fill:#e67e22,stroke:#a04000,color:#fff
    classDef output fill:#27ae60,stroke:#1a7a42,color:#fff
    classDef crossref fill:#8e44ad,stroke:#5b2c6f,color:#fff
    classDef decision fill:#c0392b,stroke:#7b241c,color:#fff

    A(["📄 /json-resume"]):::input --> B["📥 Chargement CV\nregistry / URL / fichier"]:::input
    B --> C{"🤝 Repo EAE\ndisponible ?"}:::decision
    C -->|"Oui"| D["📖 Lecture s.06 réalisations\n+ s.03 fiche de poste"]:::enrich
    C -->|"Non"| E["🌐 Enrichissement\nportfolio"]:::enrich
    D --> E
    E --> F["📋 Chargement offre\nURL / texte / numéro AVP"]:::input
    F --> G{"🏛️ AVP OPT-NC\n+ agent interne ?"}:::decision
    G -->|"Oui"| H["🔑 Lecture EAE GitHub\nresponsable hiérarchique"]:::enrich
    G -->|"Non"| I["🎯 Gap analysis\nCV ↔ offre"]:::process
    H --> I
    I --> J{"📦 Livrable"}:::decision
    J --> K["📄 CV ciblé\nJSON + AsciiDoc + PDF"]:::output
    J --> L["✉️ Lettre de motivation\nAsciiDoc + PDF"]:::output
    J --> M["🎤 Doc préparation entretien\n12 questions + STAR + check-list"]:::output
    I -->|"Écarts 🟠🔴"| N["🤝 /template-eae\nplan d'action s.07"]:::crossref
```
