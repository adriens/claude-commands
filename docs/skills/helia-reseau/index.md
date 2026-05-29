---
layout: default
title: "Helia NC — Qualité du réseau mobile"
description: "Suivi de la qualité du réseau mobile Helia NC : latence API, maintenances, incidents et disponibilité."
icon: fontawesome/solid/tower-cell
subtitle: "Qualité réseau mobile OPT-NC"
command: "/helia-reseau"
tags:
  - helia
  - réseau
  - latence
  - maintenance
  - opt-nc
  - nouvelle-calédonie
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/helia-reseau/src/helia-reseau.md"
---
# 📡 Helia NC — Qualité du réseau mobile

!!! info "Commande Claude Code"

    **Commande** : `/helia-reseau`  
    **Tags** : :material-tag-outline: `helia` :material-tag-outline: `réseau` :material-tag-outline: `latence` :material-tag-outline: `maintenance` :material-tag-outline: `opt-nc`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/helia-reseau/src/helia-reseau.md)

    *Latence API, disponibilité, profil horaire, maintenances programmées et incidents réseau Helia NC.*

---

## 📡 Contexte

Ce skill se concentre sur la **qualité du réseau mobile Helia NC** : est-ce que l'API répond vite ? Y a-t-il une maintenance en cours ? Quelles sont les meilleures heures pour une connexion rapide ?

Il s'appuie sur :
- La table **`api_ping`** — mesure de latence toutes les 5 min, stockée dans DuckDB
- Le **CLI `helia maintenance`** — état des services en temps réel
- Les vues SQL `v_api_health`, `v_hourly_latency`, `v_latency_heatmap`

> Pour la consommation data/voix, les forfaits et les recharges → voir [`/helia-conso`](../helia-conso/index.md)

---

## 📥 Installation

### Prérequis

```bash
# DuckDB CLI (pour les requêtes historiques)
brew install duckdb
```

> Le CLI `helia` et la base `~/.config/helia/data/helia.db` doivent être configurés au préalable.

### Skill (Slash Command)

```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/helia-reseau.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-reseau/src/helia-reseau.md
```

---

## 📝 Utilisation

```text
/helia-reseau
```

Sans argument → tableau de bord réseau complet (latence, dispo, meilleures/pires heures).

```text
/helia-reseau latence
/helia-reseau maintenance
/helia-reseau incident
/helia-reseau heatmap
```

---

## ✨ Fonctionnalités

- 🟢/🟡/🔴 **Latence API** : moyenne, P95, max (dernière heure)
- ⛔ **Timeouts** : nombre et taux sur la période
- 📶 **Disponibilité** : % global et par jour
- ⏰ **Profil horaire** : meilleures et pires heures de la journée
- 🗓 **Heatmap** : latence par heure × jour de semaine
- 🔧 **Maintenances** : services en cours de maintenance via `helia maintenance --json`
- 🗺 **Incidents** : redirection vers [helia.nc/etat-du-reseau](https://helia.nc/etat-du-reseau)

---

## 🔀 Flow de travail

```mermaid
flowchart TD
    classDef input fill:#2980b9,stroke:#1a5276,color:#fff,rx:8
    classDef process fill:#e67e22,stroke:#a04000,color:#fff
    classDef result fill:#27ae60,stroke:#1a7a42,color:#fff
    classDef alert fill:#c0392b,stroke:#7b241c,color:#fff
    classDef decision fill:#7f8c8d,stroke:#2c3e50,color:#fff,shape:diamond

    A(["📡 /helia-reseau"]):::input --> B{"❓ Question ?"}:::decision
    B -->|"Tableau de bord"| C["🗄️ DuckDB\nv_api_health + v_hourly_latency"]:::process
    B -->|"Maintenance / incident"| D["⚡ helia maintenance --json"]:::process
    B -->|"Heatmap / profil"| E["🗄️ DuckDB\nv_latency_heatmap"]:::process
    B -->|"Panne / zone"| F["🌐 helia.nc/etat-du-reseau"]:::alert
    C --> G["📊 Tableau réseau\n🟢🟡🔴 Latence · Dispo · Timeouts"]:::result
    D --> H{"🔴 Maintenance ?"}:::decision
    H -->|"Oui"| F
    H -->|"Non"| G
    E --> I["🗓 Heatmap\nheure × jour de semaine"]:::result
    G --> J{"🔴 Dégradé ?"}:::decision
    J -->|"Oui"| K["📞 1013 · reclamation@helia.nc"]:::alert
    J -->|"Non"| L["✅ Réseau nominal"]:::result
```
