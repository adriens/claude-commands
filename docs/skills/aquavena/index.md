---
layout: default
title: "Menus Aquavena"
description: "Explore les menus et tarifs Aquavena, le service de gamelle healthy livré à domicile à Nouméa."
icon: fontawesome/solid/utensils
command: "/aquavena"
tags:
  - alimentation
  - healthy
  - nouvelle-calédonie
  - nouméa
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/aquavena/src/aquavena.md"
datasource: "https://www.aquavena.nc/"
---
# 🥗 Menus Aquavena

!!! info "Commande Claude Code"

    **Commande** : `/aquavena`  
    **Tags** : :material-tag-outline: `alimentation` :material-tag-outline: `healthy` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/aquavena/src/aquavena.md)

    *Explore les menus et tarifs Aquavena, le service de gamelle healthy livré à domicile à Nouméa.*

---

## 📥 Installation

L'installation se fait en deux étapes dans votre terminal Claude Code.

### 1️⃣ Serveur MCP (Prérequis)
Installez d'abord le serveur de données :
```bash
claude mcp add --transport sse aquavena \
  https://rastadidi-aquavena.hf.space/gradio_api/mcp/sse
```

### 2️⃣ Skill (Slash Command)
Téléchargez le skill dans votre répertoire de commandes :
```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/aquavena.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/aquavena/src/aquavena.md
```

---

## 📝 Utilisation

Une fois installé, dans Claude Code :

```text
/aquavena
```

Ou directement avec un objectif :

```text
/aquavena végétarien
```

```text
/aquavena menu low-carb de la semaine
```

---

## ✨ Fonctionnalités

- 📋 **Liste des régimes** : tous les programmes alimentaires disponibles
- 🍽️ **Menu de la semaine** : consultation par régime, présenté jour par jour
- 💰 **Grille tarifaire** : prix HT et TTC en XPF pour toutes les formules
- 🎯 **Conseil personnalisé** : recommandation de régime selon votre objectif (sport, minceur, végé, famille…)

---

## 🥗 Qu'est-ce qu'Aquavena ?

[Aquavena](https://www.aquavena.nc/) est un service de gamelle healthy basé à **Nouméa, Nouvelle-Calédonie**. Ils proposent des repas équilibrés, savoureux et livrés à domicile, déclinés en plusieurs formules adaptées à différents objectifs alimentaires : sportifs, végétariens, low-carb, méditerranéen, et plus encore.

Aquavena s'adresse à celles et ceux qui veulent bien manger sans sacrifier le temps ni le plaisir.
