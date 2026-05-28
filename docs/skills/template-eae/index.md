---
layout: default
title: "EAE — Entretien Annuel d'Évaluation"
description: "Prépare, complète et builde ton EAE versionné sur GitHub : onboarding guidé, remplissage section par section, export PDF/ePub/DOCX."
icon: fontawesome/solid/handshake
subtitle: "Fonction Publique de Nouvelle-Calédonie"
command: "/template-eae"
tags:
  - eae
  - rh
  - opt-nc
  - nouvelle-calédonie
  - github
source_url: "https://github.com/adriens/claude-commands/blob/main/docs/skills/template-eae/src/template-eae.md"
datasource: "https://github.com/opt-nc/template-eae"
---
# 📋 Template EAE — Entretien Annuel d'Évaluation

!!! warning "Confidentialité — point critique"

    Le repo GitHub doit **toujours** être créé en **privé**. Un repo public exposerait tes données personnelles, ta note et tes objectifs de carrière à n'importe qui sur Internet.

!!! info "Commande Claude Code"

    **Commande** : `/template-eae`  
    **Tags** : :material-tag-outline: `eae` :material-tag-outline: `rh` :material-tag-outline: `nouvelle-calédonie`  
    **Source** : [:fontawesome-brands-github: Voir le code source](https://github.com/adriens/claude-commands/blob/main/docs/skills/template-eae/src/template-eae.md)

    *Accompagnement complet pour versionner et builder son EAE sur GitHub — toutes collectivités NC.*

!!! note "Toutes les collectivités"

    Ce skill s'adresse à tous les agents permanents de la fonction publique de Nouvelle-Calédonie : **OPT-NC, Province Sud, Province Nord, Province des Îles Loyauté, Gouvernement NC, communes et établissements publics**. Le processus EAE est commun à toutes ces entités (arrêté n° 1065 du 22 août 1953).

---

## 🏛️ Contexte : l'EAE en Nouvelle-Calédonie

L'**Entretien Annuel d'Évaluation** (EAE) — aussi appelé Entretien Annuel d'Échange — est conduit chaque année pour tous les agents permanents de la fonction publique calédonienne. Il sert à :

- Évaluer la valeur professionnelle de l'agent (note sur 20)
- Fixer les objectifs pour l'année à venir
- Exprimer les souhaits d'évolution et de mobilité
- Préparer les décisions d'avancement (CAP)

### Le problème avec monportailrh.nc

La plateforme officielle souffre de trois limitations :

| Problème | Impact |
|---|---|
| Sessions qui expirent | Perte de la saisie en cours |
| Zones de texte brut | Aucun formatage possible |
| Modifications simultanées | Le dernier écrase l'autre |

### La solution : GitHub + Markdown

#### Ce que ça change concrètement

| Capacité | Ce que ça apporte |
|---|---|
| 📱 **Lecture partout** | Consultable depuis un téléphone, une tablette, une liseuse (ePub), n'importe quel navigateur |
| ✍️ **Commits signés** | Chaque modification est horodatée et attribuée à son auteur — infalsifiable |
| 🔍 **Diff entre deux années** | `git diff EAE-2024 EAE-2025 -- src/07_plan-action.md` — voir exactement ce qui a évolué dans les objectifs, les compétences ou les souhaits de carrière |
| 📊 **Analyse IA** | Les fichiers Markdown sont idéaux pour des embeddings, de la recherche sémantique ou une analyse automatisée de l'évolution de carrière sur plusieurs années |
| 🏷️ **Release officielle** | Le tag Git et la release GitHub constituent une archive horodatée et immuable, accessible à tout moment par les deux parties |
| 🤝 **Collaboration fluide** | PRs, reviews inline, issues GitHub — les mêmes outils que les équipes logicielles |

Le template [opt-nc/template-eae](https://github.com/opt-nc/template-eae) propose de versionner son EAE comme un développeur versionne son code :

- **Un fichier Markdown par section** de l'EAE
- **Une branche Git par année** (`EAE-2025`, `EAE-2026`…)
- **Pandoc** pour exporter en PDF, ePub, HTML, DOCX
- **Collaboration** avec le manager via branches et pull requests

---

## 🔀 Git flow de collaboration

```mermaid
gitGraph
   commit id: "init template"
   branch EAE-2025
   checkout EAE-2025
   commit id: "section/00 identification"
   commit id: "section/03 fiche de poste"
   branch revision/manager-objectifs
   checkout revision/manager-objectifs
   commit id: "manager: ajustement objectifs"
   checkout EAE-2025
   merge revision/manager-objectifs id: "PR validée ✅"
   commit id: "section/06 autoévaluation"
   commit id: "section/07 plan action"
   branch revision/manager-synthese
   checkout revision/manager-synthese
   commit id: "manager: synthèse + note /20"
   checkout EAE-2025
   merge revision/manager-synthese id: "PR validée ✅"
   checkout main
   merge EAE-2025 id: "EAE 2025 final" tag: "EAE-2025-final"
```

### Règles de collaboration

| Qui | Quoi | Comment |
|---|---|---|
| **Évalué** | Remplit les sections | Commits sur `EAE-{annee}` ou branches `section/xxx` |
| **Manager** | Commente, propose des ajustements | Branches `revision/xxx` + **Pull Requests** vers `EAE-{annee}` |
| **Les deux** | Discutent | **Issues GitHub** — sans modifier le document directement |
| **Les deux** | Valident une modification | Review + approbation de la PR |
| **Évalué** | Clôture l'EAE | Merge `EAE-{annee}` → `main` + **tag + release GitHub** |

### Clôture et release officielle

Une fois l'EAE validé par les deux parties, l'évalué crée une release GitHub — archive officielle horodatée et immuable :

```bash
git tag -a EAE-2025-final -m "EAE 2025 — version finale validée"
git push origin EAE-2025-final
gh release create EAE-2025-final \
  --title "EAE 2025 — Version finale" \
  --notes "EAE 2025 validé par l'évalué et le manager." \
  dist/*.pdf dist/*.epub
```

---

## 📺 Présentation vidéo

[Visionner la vidéo — "Versionner et builder l'eBook de son EAE sur Git(Hub) avec l'IA"](https://www.youtube.com/watch?v=FRVsA7NoZv8) par **DevOPS LABS**.

---

## 📥 Installation

```bash
mkdir -p ~/.claude/commands && \
curl -o ~/.claude/commands/template-eae.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/template-eae/src/template-eae.md
```

---

## 📝 Utilisation

### Onboarding (nouveau utilisateur)

```text
/template-eae
```
Choisir **"Découvrir — je ne connais pas encore cette approche"** pour le parcours guidé complet : vérification des prérequis, création du repo, remplissage des sections, build des documents.

### Analyser un EAE existant

```text
/template-eae adriens
```

### Compléter une section spécifique

```text
/template-eae adriens/eae-opt
```

---

## ✨ Fonctionnalités

- 🎓 **Onboarding "for dummies"** — parcours guidé de A à Z pour les débutants
- 🔒 **Sécurité** — alerte systématique sur la nécessité du repo privé
- 🆕 **Initialisation** — création du repo depuis le template en un seul bloc de commandes
- 📊 **Tableau de bord** — taux de complétion par section avec indicateurs 🟢🟡🔴
- ✏️ **Remplissage guidé** — questions ciblées section par section
- 🏗️ **Build** — génération PDF, ePub, HTML, DOCX via go-task + pandoc
- 🔗 **Cross-références** — vers `/opt-nc-avps` si mobilité souhaitée, vers `/json-resume` pour candidature

---

## 🔗 Ressources

### Officielles DRHFPNC
- [Guide de l'évalué](https://drhfpnc.gouv.nc/sites/default/files/atoms/files/guideevalue.pdf)
- [Guide de l'évaluateur](https://drhfpnc.gouv.nc/sites/default/files/atoms/files/guideevaluateur.pdf)
- [Formulaires officiels EAE](https://drhfpnc.gouv.nc/formulaires-agents/entretien-annuel-dechange)
- [Mon Portail RH NC](https://www.monportailrh.nc/)
- [Fiches emploi DRHFPNC](https://drhfpnc.gouv.nc/travailler-dans-la-fonction-publique-trouver-un-emploi-repertoire-des-emplois/les-fiches-emploi)

### Template GitHub
- [opt-nc/template-eae](https://github.com/opt-nc/template-eae)
- [Article dev.to](https://dev.to/adriens/versionner-et-builder-lebook-de-son-entretien-annuel-devaluation-sur-github-242k)
