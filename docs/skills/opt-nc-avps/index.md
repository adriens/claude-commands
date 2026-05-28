---
layout: default
title: "Recherche AVPs OPT-NC"
description: "Recherche d'AVPs OPT-NC avec accompagnement complet à la préparation de candidature."
icon: fontawesome/solid/magnifying-glass
subtitle: "Offres d'emploi OPT-NC"
command: "/opt-nc-avps"
tags:
  - emploi
  - opt-nc
  - nouvelle-calédonie
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/opt-nc-avps/src/opt-nc-avps.md"
datasource: "https://data.gouv.nc/explore/dataset/avis-de-vacances-de-poste-avp-drhfpnc/"
---
# 🔍 Recherche d'AVPs OPT-NC

![Logo OPT-NC](../../assets/logos/opt-nc.png)

!!! info "Commande Claude Code"

    **Commande** : `/opt-nc-avps`  
    **Tags** : :material-tag-outline: `emploi` :material-tag-outline: `opt-nc` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/opt-nc-avps/src/opt-nc-avps.md)

    *Accompagnement complet à la préparation de candidature pour l'OPT-NC.*

---

## 🏛️ Contexte : OPT-NC et AVPs

### L'OPT-NC

L'[Office des Postes et Télécommunications de Nouvelle-Calédonie (OPT-NC)](https://opt.nc/) est un établissement public calédonien qui gère les services postaux, les télécommunications et les services financiers (CCP) sur l'ensemble du territoire. C'est l'un des principaux employeurs publics de Nouvelle-Calédonie.

### Qu'est-ce qu'un AVP ?

Un **Avis de Vacance de Poste (AVP)** est l'équivalent calédonien d'une offre d'emploi dans la fonction publique. Les AVPs sont publiés par la **Direction des Ressources Humaines de la Fonction Publique de Nouvelle-Calédonie (DRHFPNC)** et recensent les postes ouverts au sein des administrations et établissements publics du territoire.

Contrairement aux offres d'emploi privées, les AVPs obéissent à des règles spécifiques :

- **Lettre sous couvert hiérarchique** : la candidature doit transiter par la hiérarchie de l'agent
- **EAE (Entretien Annuel d'Évaluation)** : souvent demandé en pièce jointe
- **Délai de clôture strict** : les candidatures hors délai ne sont pas acceptées

Les AVPs de l'OPT-NC sont publiés en open data sur [data.gouv.nc](https://data.gouv.nc/explore/dataset/avis-de-vacances-de-poste-avp-drhfpnc/) et mis à jour régulièrement.

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
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/opt-nc-avps.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/opt-nc-avps/src/opt-nc-avps.md```
---

## 📝 Utilisation

Une fois installé, dans Claude, lancez simplement la commande en précisant votre profil ou le type de poste :

```text
/opt-nc-avps chef de projet SI
```

### ✨ Fonctionnalités
*   🔍 **Recherche ciblée** des postes ouverts.
*   🧠 **Profilage intelligent** via questionnaire.
*   🛠️ **Plan de préparation** (atouts, points à renforcer, questions d'entretien STAR).
*   ✍️ **Aide à la rédaction** de lettre de motivation personnalisée.

---

## 🔀 Flow de travail

```mermaid
flowchart TD
    classDef input fill:#2980b9,stroke:#1a5276,color:#fff,rx:8
    classDef process fill:#e67e22,stroke:#a04000,color:#fff
    classDef result fill:#27ae60,stroke:#1a7a42,color:#fff
    classDef crossref fill:#8e44ad,stroke:#5b2c6f,color:#fff
    classDef decision fill:#c0392b,stroke:#7b241c,color:#fff,shape:diamond

    A(["🔍 /opt-nc-avps"]):::input --> B{"👤 Profil candidat ?"}:::decision
    B -->|"📄 JSON Resume"| C["📥 Fetch CV + portfolio"]:::input
    B -->|"✍️ Manuel"| D["📝 Saisie profil"]:::input
    C --> E["🤖 Recherche AVPs
par similarité sémantique"]:::process
    D --> E
    E --> F{"📊 3+ bons
résultats ?"}:::decision
    F -->|"Oui"| G["⚖️ Tableau comparatif
multi-AVPs"]:::process
    F -->|"Non"| H["📋 Détail AVP
missions + profil requis"]:::result
    G --> H
    H --> I["🚀 Handoff"]:::process
    I --> J["📄 /json-resume
CV ciblé + lettre + entretien"]:::crossref
    I --> K["🤝 /template-eae
vérifier s.03 compétences"]:::crossref
```
