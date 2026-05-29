---
layout: default
title: "Helia / Mobilis — Mon forfait mobile"
description: "Suivi de consommation personnelle (data & voix) et qualité du réseau mobile OPT-NC / Helia."
icon: fontawesome/solid/mobile-screen-button
subtitle: "Forfait mobile OPT-NC"
command: "/helia"
tags:
  - helia
  - mobile
  - opt-nc
  - nouvelle-calédonie
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/helia/src/helia.md"
---
# 📱 Helia / Mobilis — Mon forfait mobile

!!! info "Commande Claude Code"

    **Commande** : `/helia`  
    **Tags** : :material-tag-outline: `helia` :material-tag-outline: `mobile` :material-tag-outline: `opt-nc` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/helia/src/helia.md)

    *Suivi de consommation data & voix, performances réseau et gestion de ton forfait mobile Helia NC.*

---

## 📡 Qu'est-ce que Helia ?

**[Helia](https://helia.nc/)** est la marque mobile de l'**OPT-NC** (Office des Postes et Télécommunications de Nouvelle-Calédonie), anciennement connue sous le nom **Mobilis**. Elle propose des forfaits M (abonnement mensuel) et des kits prépayés Liberté, sur un réseau couvrant l'ensemble de la Nouvelle-Calédonie (530+ antennes, 8 200 km de fibre).

Ce skill s'appuie sur :

- Le **CLI `helia`** — outil local qui interroge l'API OPT-NC en temps réel
- Une base **DuckDB** alimentée toutes les 5 minutes — pour l'historique, les tendances et les projections
- Les **vues SQL** précalculées — `v_projection`, `v_daily_conso`, `v_api_health`, `v_hourly_latency`

---

## 📥 Installation

### Skill (Slash Command)
```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/helia.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia/src/helia.md
```

> **Prérequis** : le CLI `helia` doit être installé et configuré (`~/.config/helia/token`) ainsi que la base DuckDB `~/.config/helia/data/helia.db`.

---

## 📝 Utilisation

```text
/helia
```

Sans argument, le skill affiche le **tableau de bord complet** : data, voix, SMS, hors-forfait, rythme et projection.

```text
/helia data
/helia voix
/helia réseau
/helia latence
```

---

## ✨ Fonctionnalités

### Domaine 1 — Consommation
- 📶 **Data** : Go restants, % consommé, rythme jour par jour
- 📞 **Voix** : heures restantes, % consommé
- 💬 **SMS** : illimité ou quota
- 💸 **Hors-forfait** : montant en F CFP
- ⏱ **Rythme** : % data consommée vs % temps écoulé (en avance / dans les clous / dépasse)
- 🏁 **Projection** : data et voix tiennent-elles jusqu'au renouvellement ?

### Domaine 2 — Qualité réseau & API
- 🟢/🟡/🔴 Latence API (≤500 ms / 500–1000 ms / >1000 ms)
- ⛔ Timeouts et taux de disponibilité
- 📊 Profil horaire (meilleures / pires heures de la journée)
- 🗓 Heatmap heure × jour de semaine

### Domaine 3 — Offres & forfaits
- Tableau des forfaits M (2 / 10 / 30 / 100 Go)
- Kits prépayés Liberté
- Changement de forfait sans frais après le 1er mois

### Domaines 4–7
- **App Helia** — fonctionnalités par type de forfait
- **Assistance** — numéros 1000 / 1013 / 1052, agences sans RDV
- **État du réseau** — maintenances et incidents temps réel
- **Réclamations** — formulaire, email, courrier

---

## 🔀 Flow de travail

```mermaid
flowchart TD
    classDef input fill:#2980b9,stroke:#1a5276,color:#fff,rx:8
    classDef process fill:#e67e22,stroke:#a04000,color:#fff
    classDef result fill:#27ae60,stroke:#1a7a42,color:#fff
    classDef alert fill:#c0392b,stroke:#7b241c,color:#fff
    classDef decision fill:#7f8c8d,stroke:#2c3e50,color:#fff,shape:diamond

    A(["📱 /helia"]):::input --> B{"❓ Question précise ?"}:::decision
    B -->|"Non / tableau de bord"| C["🗄️ Requête DuckDB\nconso_snapshot + v_projection"]:::process
    B -->|"Réseau / latence"| D["🗄️ DuckDB\nv_api_health · v_hourly_latency"]:::process
    B -->|"Temps réel"| E["⚡ helia status --json"]:::process
    B -->|"Forfait / offre"| F["📋 Tableau forfaits M\n+ lien changement"]:::result
    B -->|"Incident / panne"| G["🔴 helia maintenance --json\n+ helia.nc/etat-du-reseau"]:::alert
    C --> H["📊 Tableau de bord\n🟢🟡🔴 Data · Voix · Rythme · Projection"]:::result
    D --> I["📡 Rapport réseau\nLatence · Dispo · Heatmap"]:::result
    E --> H
    H --> J{"🔴 Alerte ?"}:::decision
    J -->|"Data / voix épuisée"| K["💳 Options recharge\nApp · 1013 · Agence"]:::alert
    J -->|"Tout OK"| L["📈 Historique dispo\ndepuis JJ/MM/AAAA"]:::result
```
