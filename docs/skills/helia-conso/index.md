---
layout: default
title: "Helia NC — Ma consommation mobile"
description: "Suivi personnel de la consommation data, voix et SMS sur le forfait mobile Helia NC (OPT-NC)."
icon: fontawesome/solid/mobile-screen-button
subtitle: "Suivi conso forfait mobile"
command: "/helia-conso"
tags:
  - helia
  - mobile
  - consommation
  - opt-nc
  - nouvelle-calédonie
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/helia-conso/src/helia-conso.md"
---
# 📱 Helia NC — Ma consommation mobile

![Logo Helia](../../assets/logos/helia.svg)

!!! info "Commande Claude Code"

    **Commande** : `/helia-conso`  
    **Tags** : :material-tag-outline: `helia` :material-tag-outline: `mobile` :material-tag-outline: `consommation` :material-tag-outline: `opt-nc`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/helia-conso/src/helia-conso.md)

    *Tableau de bord data/voix/SMS, projection de fin de forfait, recharges et gestion du forfait Helia NC.*

---

## 📡 Contexte

**[Helia](https://helia.nc/)** est la marque mobile de l'**OPT-NC**, anciennement Mobilis. Ce skill se concentre sur le **suivi de ta consommation personnelle** : combien il reste, si ça va tenir jusqu'au renouvellement, et comment recharger si besoin.

Il s'appuie sur :
- Le **CLI `helia`** — interroge l'API OPT-NC en temps réel
- Une base **DuckDB** alimentée toutes les 5 min — historique, tendances, projections

> Pour la qualité du réseau, les maintenances et la latence API → voir [`/helia-reseau`](../helia-reseau/index.md)

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
curl -o ~/.claude/commands/helia-conso.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/src/helia-conso.md
```

---

## 📝 Utilisation

```text
/helia-conso
```

Sans argument → tableau de bord complet (data, voix, SMS, hors-forfait, rythme, projection).

```text
/helia-conso data
/helia-conso voix
/helia-conso projection
/helia-conso recharge
```

---

## ✨ Fonctionnalités

- 📶 **Data** : Go restants, % consommé, rythme jour par jour
- 📞 **Voix** : heures restantes, % consommé, conso par jour
- 💬 **SMS** : illimité ou quota restant
- 💸 **Hors-forfait** : montant en F CFP
- ⏱ **Rythme** : % data consommée vs % temps écoulé (en avance / dans les clous / dépasse)
- 🏁 **Projection** : data et voix tiennent-elles jusqu'au renouvellement ?
- 💳 **Recharges** : options packagées, internet mobile, agences, app
- 📋 **Forfaits** : tableau comparatif M 2/10/30/100 Go + changement de forfait
- 🗂 **Réclamations** : formulaire, email, courrier

---

## 🔀 Flow de travail

```mermaid
flowchart TD
    classDef input fill:#2980b9,stroke:#1a5276,color:#fff,rx:8
    classDef process fill:#e67e22,stroke:#a04000,color:#fff
    classDef result fill:#27ae60,stroke:#1a7a42,color:#fff
    classDef alert fill:#c0392b,stroke:#7b241c,color:#fff
    classDef decision fill:#7f8c8d,stroke:#2c3e50,color:#fff,shape:diamond

    A(["📱 /helia-conso"]):::input --> B{"❓ Question ?"}:::decision
    B -->|"Tableau de bord"| C["🗄️ DuckDB\nconso_snapshot + v_projection"]:::process
    B -->|"Temps réel"| D["⚡ helia status --json"]:::process
    B -->|"Forfait / offre"| E["📋 Forfaits M\n+ lien changement"]:::result
    B -->|"Réclamation"| F["📧 reclamation@helia.nc\n+ formulaire"]:::result
    C --> G["📊 Tableau de bord\n🟢🟡🔴 Data · Voix · Rythme · Projection"]:::result
    D --> G
    G --> H{"🔴 Alerte ?"}:::decision
    H -->|"Data / voix épuisée"| I["💳 Options recharge\nApp · 1013 · Agence"]:::alert
    H -->|"Tout OK"| J["📈 Historique dispo\ndepuis JJ/MM/AAAA"]:::result
```
