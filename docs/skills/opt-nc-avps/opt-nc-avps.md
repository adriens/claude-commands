---
layout: default
title: "Recherche AVPs OPT-NC"
description: "Recherche d'AVPs OPT-NC avec accompagnement complet à la préparation de candidature."
command: "/opt-nc-avps"
---
# Recherche d'AVPs OPT-NC

!!! info "Commande Claude Code"

    **Commande** : `{{ page.command | default: "/opt-nc-avps" }}`  
    *Accompagnement complet à la préparation de candidature pour l'OPT-NC.*

---

## Instructions

### Étape 1 — Recherche des postes
1. Si l'utilisateur ne fournit pas de profil dans `$ARGUMENTS`, demande son profil (métier, niveau, compétences clés).
2. Lance `mcp__avps-opt-nc__avps_search_avps` avec une requête enrichie.
3. Tableau markdown : titre, numéro, score, dispo immédiate, date clôture, lien.
4. Mettre en avant : postes disponibles immédiatement et score > 0.6.
5. Proposer d'ouvrir le détail via `mcp__avps-opt-nc__avps_on_card_click`.

### Étape 2 — Détail d'un poste
1. Appelle `mcp__avps-opt-nc__avps_on_card_click`. Si erreur, `WebFetch` sur l'`url_markdown`.
2. Présente la fiche complète : missions, activités, profil requis, modalités.
3. Enchaîne immédiatement avec l'étape 3.

### Étape 3 — Profilage candidat (AskUserQuestion, 3 questions simultanées)

**Q1 — Expérience** (header: "Expérience") :
- Chef de projet confirmé (5 ans+, bout en bout)
- Chef de projet junior (2-5 ans)
- Profil MOA/AMOA (besoins, specs, recette)
- En reconversion (compétences transverses)

**Q2 — Conduite du changement** (header: "Conduite du chgt") :
- Oui, formalisée (méthodologie, plan de com)
- Oui, opérationnelle (animation terrain)
- Partielle (ateliers ponctuels)
- Non

**Q3 — Lien NC / OPT-NC** (header: "Contexte local") :
- Résident en NC
- Fonctionnaire NC (voie hiérarchique)
- Candidat extérieur (mobilité)
- Ancien agent OPT

> Adapter les options au domaine du poste (ex: technique réseau → certifications, terrain vs bureau).

### Étape 4 — Plan de préparation personnalisé

#### A. Atouts à mettre en avant
- 2-3 points forts en lien avec les missions.
- Pour chaque atout : une phrase d'accroche prête à l'emploi (lettre ou entretien).
- Ancien agent OPT → valoriser la connaissance terrain, processus internes, enjeux institutionnels.

#### B. Points à renforcer
- Écarts entre profil et exigences du poste.
- Pour chaque écart : stratégie concrète (formation courte, expérience indirecte, angle de com).
- Ne jamais laisser un écart sans solution.

#### C. Préparation à l'entretien
- 5 questions types du jury, basées sur les missions du poste.
- Pour chaque question : structure STAR (Situation, Tâche, Action, Résultat) avec éléments à personnaliser.
- 2 questions à poser au jury (montrer engagement et maturité).

### Étape 5 — Lettre de candidature
Proposer de rédiger un brouillon personnalisé intégrant :
- Accroche sur le lien OPT-NC / NC si pertinent.
- Atouts identifiés (étape 4A).
- Réponse proactive aux points faibles (étape 4B).
- Ton adapté à la fonction publique territoriale calédonienne.

## Paramètres de recherche
- Threshold par défaut : 30 — monter à 60+ si trop de bruit.
- Requête : reformuler le profil en description métier complète.

## Arguments
`$ARGUMENTS` — profil recherché (ex: "chef de projet SI MOA transformation digitale")
