# Recherche d'AVPs OPT-NC

Recherche des AVPs de l'OPT-NC adaptés au profil, avec accompagnement complet à la préparation de candidature.

## Instructions

### Étape 0 — Chargement du profil candidat

**Si `$ARGUMENTS` est fourni** : l'utiliser directement comme requête (passer à l'étape 1).

**Si `$ARGUMENTS` est vide** : demander (AskUserQuestion) :

**Source du profil** (header: "Profil") :
- J'ai un CV sur registry.jsonresume.org
- Décrire mon profil manuellement

**Si JSON Resume** :
1. Demander le username (ex: `adriens`)
2. Fetcher `https://registry.jsonresume.org/{username}.json` via `WebFetch`
3. Si échec : `https://gist.githubusercontent.com/{username}/resume.json/raw`
4. Extraire pour construire la requête :
   - `basics.summary` — résumé du profil
   - `work[].position` et `work[].highlights` — postes et réalisations
   - `skills[].name` — compétences clés
   - `education[].studyType` + `education[].area` — niveau et domaine
5. **Enrichissement portfolio** : si `basics.url` est présent et ne ressemble pas à un réseau social connu (dev.to, twitter, linkedin, github.com, kaggle, youtube, huggingface) → fetcher automatiquement via `WebFetch` et extraire les projets, compétences et réalisations supplémentaires pour enrichir la requête. Signaler à l'utilisateur les éléments trouvés en plus du CV.
6. Construire une requête enrichie (description métier complète, 50-100 mots)
7. **Mémoriser le CV chargé** pour les étapes ultérieures (CV optimisé étape 7, lettre)

**Si profil manuel** : demander le profil (métier, niveau, compétences clés), puis passer à l'étape 1.

---

### Étape 1 — Recherche des postes
1. Lance `mcp__avps-opt-nc__avps_search_avps` avec une requête enrichie.
2. **Alerte urgence** : calculer `date_cloture - date_du_jour` pour chaque résultat.
   - Si au moins un poste clôture dans ≤ 7 jours : afficher ⚠️ **URGENT — X poste(s) clôturent dans N jours** avant le tableau.
   - Si clôture ≤ 3 jours : ⛔ **TRÈS URGENT — clôture imminente**
3. Tableau markdown : titre, numéro, score, dispo immédiate, date clôture, lien.
4. Mettre en avant : postes disponibles immédiatement et score > 0.6.
5. Si 3+ AVPs avec score > 0.5 : proposer d'abord la comparaison (étape 1.5) avant de détailler.
6. Sinon : proposer d'ouvrir le détail via `mcp__avps-opt-nc__avps_on_card_click`.

### Étape 1.5 — Comparaison multi-AVPs (optionnelle)

**Quand** : Si la recherche retourne 3+ AVPs avec score > 0.5

**Proposer** (AskUserQuestion) : "Souhaitez-vous comparer plusieurs postes avant de choisir ?"

**Si oui** :
1. Demander de sélectionner 2-3 AVPs à comparer (par numéros)
2. Récupérer les détails de chaque AVP via `mcp__avps-opt-nc__avps_on_card_click`
3. Générer un **tableau comparatif markdown** avec :
   - Titre du poste
   - Missions principales (résumé 2-3 points clés)
   - Score de matching
   - Disponibilité immédiate (✓/✗)
   - Date de clôture
   - Localisation / Direction
   - Niveau requis vs profil candidat (adéquation)
4. **Analyse comparative** :
   - 🟢 Match parfait (score > 0.7 + dispo immédiate + profil aligné)
   - 🟡 Bon match (score > 0.6 + 1-2 écarts gérables)
   - 🟠 Match partiel (nécessite préparation significative)
5. **Recommandation stratégique** :
   - Lequel prioriser et pourquoi (3-4 arguments)
   - Si pertinent : suggérer candidatures multiples avec ordre de préférence
   - Timeline recommandée (quel poste traiter en premier)
6. Laisser le candidat choisir l'AVP sur lequel continuer (étape 2)

**Si non** : Passer directement à l'étape 2 avec l'AVP choisi.

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
     - **Lien hiérarchique** : Responsable hiérarchique et fonction (section # Lien hiérarchique)

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

**Format de rédaction** : Proposer (AskUserQuestion) :
1. **AsciiDoc (.adoc)** — format recommandé, structuré, exportable
2. **Markdown (.md)** — format simple et universel

**Signature** : Demander (AskUserQuestion) si le candidat dispose d'une image de signature :
- Si oui : demander le chemin du fichier image (ex: `~/Documents/signature.png`)
- Intégrer l'image dans le document (syntaxe AsciiDoc : `image::chemin/signature.png[width=200]` ou Markdown : `![Signature](chemin/signature.png)`)
- Positionner la signature en fin de lettre, après la formule de politesse

Proposer de rédiger un brouillon personnalisé intégrant :
- **Mention obligatoire** : "sous couvert de [Responsable hiérarchique]" (extrait de l'EAE si disponible, sinon à demander).
- Accroche sur le lien OPT-NC / NC si pertinent.
- Si employé actuel OPT : référence aux réalisations de l'EAE et alignement entre fiche de poste actuelle et AVP ciblé.
- Atouts identifiés (étape 4A).
- Réponse proactive aux points faibles (étape 4B).
- Ton adapté à la fonction publique territoriale calédonienne.

**Génération Word** : Une fois le fichier créé, proposer de générer un document Word via pandoc :
```bash
# Pour AsciiDoc
pandoc lettre-motivation.adoc -o lettre-motivation.docx

# Pour Markdown
pandoc lettre-motivation.md -o lettre-motivation.docx
```

> **Note importante** : Les candidatures AVP à l'OPT-NC doivent être faites **sous couvert du responsable hiérarchique**. La lettre doit mentionner explicitement cette information (extraite automatiquement de l'EAE si disponible).

> Note technique : pandoc doit être installé via brew (`brew install pandoc`) si non disponible.

### Étape 6 — Document de préparation à l'entretien (optionnel)

Proposer (AskUserQuestion, 2 questions simultanées) :

**Q1 — Générer la doc entretien ?** (header: "Entretien") :
- Oui, générer le document
- Non, ce n'est pas nécessaire

**Q2 — Format** (header: "Format entretien") :
- AsciiDoc (.adoc) — recommandé, exportable PDF
- Markdown (.md) — universel

**Si oui, contenu du document** :
- **Fiche poste synthétique** : missions clés, compétences attendues, enjeux
- **Mon profil vs le poste** : tableau atouts / écarts (basé sur l'analyse étape 4)
- **5 questions probables du jury** avec réponses structurées STAR (Situation, Tâche, Action, Résultat) — personnalisées avec les éléments du profil candidat
- **2 questions à poser au jury** (posture proactive, montrer la maturité)
- **Éléments différenciants** à mentionner impérativement
- **Points de vigilance** (écarts à ne pas sous-estimer, comment les aborder)

Créer le fichier `preparation-entretien-{slug-poste}.adoc` ou `.md` dans le répertoire courant.

**Proposer la conversion Word** après création :
```bash
pandoc preparation-entretien-{slug-poste}.adoc -o preparation-entretien-{slug-poste}.docx
# ou
pandoc preparation-entretien-{slug-poste}.md -o preparation-entretien-{slug-poste}.docx
```

---

### Étape 7 — CV optimisé pour le poste (conditionnel)

**Conditionnel** : uniquement si un JSON Resume a été chargé à l'étape 0.

Proposer (AskUserQuestion) :
- Oui, générer un CV ciblé pour ce poste
- Non

**Principe** : sélectionner et reformuler — jamais inventer.

**Ce qui change** :
- Sélection des expériences, compétences et projets pertinents pour ce poste
- Reformulation des highlights pour résonner avec le vocabulaire de l'AVP (sans inventer de faits)
- Réordonnancement des sections pour mettre en avant ce que ce recruteur cherche
- Réécriture de `basics.summary` pour répondre directement au profil attendu

**Ce qui ne change pas** : faits, dates, employeurs, diplômes, chiffres — aucune fabrication.

Produire le CV ciblé en JSON Resume valide, enregistré sous `resume-{slug-poste}.json`.

> Objectif : maximiser le score de matching sémantique avec l'AVP, en valorisant honnêtement ce que le candidat a vraiment fait.

---

## Paramètres de recherche
- Threshold :
  - **30** = large (nombreux résultats, plus de bruit) — utile pour explorer un profil atypique
  - **50** = équilibré (défaut recommandé)
  - **70+** = strict (peu de résultats, très pertinents) — utile si trop de bruit à 50
- Requête : reformuler le profil en description métier complète (50-100 mots, pas de mots-clés isolés).

## Arguments
`$ARGUMENTS` — profil recherché (ex: "chef de projet SI MOA transformation digitale")
