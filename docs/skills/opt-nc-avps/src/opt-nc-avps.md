# Recherche d'AVPs OPT-NC

Recherche des AVPs de l'OPT-NC adaptés au profil, avec accompagnement complet à la préparation de candidature.

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
- Employé actuel OPT (mobilité interne)
- Ancien agent OPT
- Candidat extérieur (mobilité)

> Adapter les options au domaine du poste (ex: technique réseau → certifications, terrain vs bureau).

**Si "Employé actuel OPT"** : Demander (AskUserQuestion, facultatif) :

**Option A — Via le template EAE GitHub** :
- Utilisez-vous le template EAE de l'OPT-NC (https://github.com/opt-nc/template-eae) ?
- Si non : Proposer de s'y mettre avec les ressources suivantes :
  - Template : https://github.com/opt-nc/template-eae
  - Guide complet : https://dev.to/adriens/versionner-et-builder-lebook-de-son-entretien-annuel-devaluation-sur-github-242k
  - Avantages : versioning, suivi annuel, génération PDF automatique, réutilisable pour candidatures
  - Puis passer à l'Option B (saisie manuelle) pour cette fois
- Si oui :
  1. Demander le login GitHub (ex: `username`)
  2. Tenter avec la convention : `https://github.com/{login}/eae-opt`
  3. Vérifier via `github-mcp-server-list_branches` (owner: login, repo: `eae-opt`)
  4. Si échec (repo introuvable) : demander l'URL complète de la repo en fallback
  5. Détecter les branches disponibles et proposer (ex: `2025`, `main`)
  6. Récupérer automatiquement via `github-mcp-server-get_file_contents` :
     - owner et repo (login + `eae-opt` ou extraits de l'URL)
     - path: `src/03_fiche-de-poste.md`
     - ref: `refs/heads/{branche}` (ex: `refs/heads/2025` ou `refs/heads/main`)
  7. Extraire du fichier :
     - **Identification** : Intitulé, Grade, Direction/Service
     - **Missions** (section ## Missions)
     - **Activités principales** (section ## Activités principales)
     - **Activités secondaires** (section ## Activités secondaires)
     - **Compétences requises** (section ## Compétences requises)

**Option B — Saisie manuelle** :
- Le contenu de la fiche de poste actuelle
- Les éléments clés du dernier EAE (points forts, axes d'amélioration, objectifs atteints)
- Les compétences ou réalisations valorisées par la hiérarchie

> Ces éléments permettent d'ancrer la candidature dans des faits évalués et reconnus en interne.

### Étape 4 — Plan de préparation personnalisé

#### A. Atouts à mettre en avant
- 2-3 points forts en lien avec les missions.
- Pour chaque atout : une phrase d'accroche prête à l'emploi (lettre ou entretien).
- Employé actuel OPT → valoriser la connaissance opérationnelle, réseau interne, maîtrise des processus et enjeux stratégiques.
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
- Si employé actuel OPT : référence aux réalisations de l'EAE et alignement entre fiche de poste actuelle et AVP ciblé.
- Atouts identifiés (étape 4A).
- Réponse proactive aux points faibles (étape 4B).
- Ton adapté à la fonction publique territoriale calédonienne.

## Paramètres de recherche
- Threshold par défaut : 30 — monter à 60+ si trop de bruit.
- Requête : reformuler le profil en description métier complète.

## Arguments
`$ARGUMENTS` — profil recherché (ex: "chef de projet SI MOA transformation digitale")
