---
layout: default
title: "Kalolo — Expressions caldoches"
description: "Active le mode caldoche dans Claude Code : parsème tes échanges d'expressions typiques de Nouvelle-Calédonie."
icon: fontawesome/solid/hat-cowboy
subtitle: "Le parler du Caillou dans Claude Code"
command: "/kalolo"
tags:
  - culture
  - langue
  - nouvelle-calédonie
  - caldoche
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/kalolo/src/kalolo.md"
datasource: "https://github.com/adriens/kalolo-api"
---
# 🌴 Kalolo — Expressions caldoches

!!! info "Commande Claude Code"

    **Commande** : `/kalolo`  
    **Tags** : :material-tag-outline: `culture` :material-tag-outline: `langue` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/kalolo/src/kalolo.md)

    *Active le mode caldoche : Claude parsème ses réponses d'expressions typiques du Caillou.*

---

## 🗣️ Qu'est-ce que le caldoche ?

Le **caldoche** est le parler créole francophone de **Nouvelle-Calédonie**, né du métissage entre le français populaire, les langues kanak et les expressions forgées par les générations vivant sur le **Caillou** (surnom affectueux de la Nouvelle-Calédonie).

Ce lexique vivant reflète la culture locale : la convivialité mélanésienne, les fêtes coutumières, la vie entre mer et montagne, et l'humour particulier des gens du Caillou.

Ce skill est basé sur le projet [Kalolo-API](https://github.com/adriens/kalolo-api) — une API open source répertoriant plus de **250 expressions caldoches** classées par registre émotionnel.

---

## 📥 Installation

L'installation est simple : pas de serveur MCP requis, les expressions sont embarquées directement dans le skill.

```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/kalolo.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/kalolo/src/kalolo.md
```

---

## 📝 Utilisation

```text
/kalolo
```

Ou directement avec une catégorie :

```text
/kalolo joie
```

```text
/kalolo surprise
```

```text
/kalolo fin choc
```

---

## ✨ Fonctionnalités

- 🦜 **Mode caldoche** : Claude intègre naturellement des expressions NC dans ses réponses pour toute la conversation
- 📚 **Dictionnaire par catégorie** : approbation, désapprobation, surprise, colère, joie, tristesse, insultes…
- 🎲 **Expression aléatoire** : tirage au sort avec explication culturelle et contextuelle
- 🔄 **Traduction en caldoche** : reformule n'importe quelle phrase avec des expressions locales
- 🌺 **Contexte culturel** : explications sur le lexique fondamental (fin, aïta, AWA, kaï-kaï, fiu…)

---

## 📖 Quelques expressions emblématiques

| Expression | Signification |
|-----------|--------------|
| `kalolo !` | Super ! Génial ! Excellent ! |
| `fin choc !` | Vraiment top, impressionnant |
| `AWA !` | Surprise / Non ! (du kanak) |
| `aïta` | Non, rien à faire (du kanak) |
| `jamais fini cassé !` | C'est nul, raté, lamentable |
| `ça bombarde !` | Ça déchire, c'est au top |
| `fiu` | Fatigué, blasé |
| `c'est l'heure du kaï-kaï` | C'est l'heure de manger |
| `ayaoué` | Expression de surprise / lassitude |
| `le Caillou` | La Nouvelle-Calédonie |

---

## 🔗 Sources

- [Kalolo-API sur GitHub](https://github.com/adriens/kalolo-api) — base de données open source des expressions
- [Article LinkedIn : EaaS — Expressions as a Service](https://www.linkedin.com/pulse/eaas-expressions-service-nos-ont-une-api-sont-fin-barr%C3%A9s-adrien-sales)
- [Kalolo in the shell](https://www.linkedin.com/posts/adrien-sales_shell-unix-linux-activity-6697227023942868992-Z26_) — intégration terminal
