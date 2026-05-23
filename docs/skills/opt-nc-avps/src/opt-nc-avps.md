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
7. **Mémoriser le CV chargé** pour le handoff vers `/json-resume` (gap analysis, CV ciblé, lettre, doc entretien)

**Si profil manuel** : demander le profil (métier, niveau, compétences clés), puis passer à l'étape 1.

---

### Étape 1 — Recherche des postes
1. Lance `mcp__avps-opt-nc__avps_search_avps` avec une requête enrichie.
   - **Si aucun résultat** : baisser le threshold à 30 et relancer. Si toujours vide, proposer de reformuler la requête en termes plus génériques (ex: domaine métier seul, sans les technologies).
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

### Étape 2 — Détail d'un poste et handoff
1. Appelle `mcp__avps-opt-nc__avps_on_card_click`. Si erreur, `WebFetch` sur l'`url_markdown`.
2. Présente la fiche complète : missions, activités, profil requis, modalités.
3. **Handoff vers `/json-resume`** :
   - Si un JSON Resume a été chargé à l'étape 0 :
     > 👉 Lance `/json-resume {username}` — quand la skill demande l'offre, réponds avec le numéro AVP `{numero}` ; quand elle demande les livrables, choisis **Tout** pour obtenir CV + lettre + document de préparation d'entretien
   - Si aucun CV chargé :
     > 👉 Lance `/json-resume` — la skill guidera le chargement de ton CV, puis donne le numéro AVP `{numero}` comme offre

> `/json-resume` gère : gap analysis CV↔offre · CV ciblé JSON+AsciiDoc+PDF · lettre AsciiDoc+PDF · **document de préparation d'entretien AsciiDoc+PDF** (points forts/faibles, 12 questions probables, 5 réponses STAR, check-list) · spécificités OPT-NC (EAE, lettre sous couvert hiérarchique).

---

## Paramètres de recherche
- Threshold :
  - **30** = large (nombreux résultats, plus de bruit) — utile pour explorer un profil atypique
  - **50** = équilibré (défaut recommandé)
  - **70+** = strict (peu de résultats, très pertinents) — utile si trop de bruit à 50
- Requête : reformuler le profil en description métier complète (50-100 mots, pas de mots-clés isolés).

## Arguments
`$ARGUMENTS` — profil recherché (ex: "chef de projet SI MOA transformation digitale")
