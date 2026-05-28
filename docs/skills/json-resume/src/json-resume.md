# Candidature à une offre d'emploi via JSON Resume

Charge un CV au format JSON Resume (registry.jsonresume.org, Gist, URL ou fichier local) et accompagne la candidature à une offre d'emploi : CV taillé sur mesure, lettre de motivation et document de préparation d'entretien qui démontrent que le passé répond exactement à ce que le poste demande.

TRIGGER when: user mentions JSON Resume, resume.json, registry.jsonresume.org, a CV/résumé in JSON format, fetching or reading someone's resume/CV, job application with a structured CV, cover letter from a CV.
SKIP: CV in Word/PDF without JSON Resume context, generic cover letter without a structured CV source.

## Règle de formatage AsciiDoc

> **Règle systématique** : tout fichier `.adoc` produit par cette skill est **immédiatement converti en PDF** via `asciidoctor-pdf` après sa création.
> Installation : `gem install asciidoctor-pdf` (Linux/Mac) ou `brew install asciidoctor-pdf` (Mac).
> Ne jamais proposer pandoc pour les fichiers AsciiDoc — asciidoctor-pdf est le seul outil utilisé.
>
> **Résolution du binaire** : si `asciidoctor-pdf` n'est pas dans le PATH, le localiser avant toute conversion :
> ```bash
> APDF=$(find ~ /usr/local /usr -name asciidoctor-pdf 2>/dev/null | head -1)
> $APDF {fichier}.adoc -o {fichier}.pdf
> ```

## Instructions

### Étape 1 — Chargement du CV

**Si `$ARGUMENTS` est fourni** :
- Si c'est une URL (`http://` ou `https://`) : fetcher directement via `WebFetch`
- Sinon, traiter comme un username et construire : `https://registry.jsonresume.org/$ARGUMENTS.json`

**Si `$ARGUMENTS` est vide** : demander (AskUserQuestion, header: "Source CV") :
- Username JSON Resume registry
- URL directe (raw JSON)
- Fichier local (chemin)

**Ordre de tentative pour un username** :
1. `https://registry.jsonresume.org/{username}.json` via `WebFetch`
2. Si échec : `https://gist.githubusercontent.com/{username}/resume.json/raw` via `WebFetch`
3. Si échec : `https://raw.githubusercontent.com/{username}/{username}/main/resume.json` via `WebFetch`
4. Si tout échoue : demander à l'utilisateur l'URL directe ou le fichier local

Informer l'utilisateur de la source effectivement utilisée.

**Enrichissement EAE** : après chargement du CV, demander (AskUserQuestion, header: "EAE") :
- J'ai un repo EAE sur GitHub (login/repo, ex: `adriens/eae-opt`)
- Non, continuer sans EAE

**Si EAE fourni** :
1. Lister les branches `EAE-{annee}` :
   ```bash
   gh api repos/{login}/{repo}/branches --jq '.[].name' | grep EAE
   ```
2. Prendre la branche la plus récente (ou demander laquelle si plusieurs).
3. Lire la section 06 — Autoévaluation :
   ```bash
   gh api "repos/{login}/{repo}/contents/src/06_autoevaluation.md?ref={branche}" --jq '.content' | base64 -d
   ```
4. Extraire les **réalisations concrètes** (objectifs atteints, succès, compétences acquises) et les reformuler comme `highlights` de CV (verbe d'action + résultat mesurable).
5. Lire aussi la section 03 — Fiche de poste pour enrichir le contexte :
   ```bash
   gh api "repos/{login}/{repo}/contents/src/03_fiche-de-poste.md?ref={branche}" --jq '.content' | base64 -d
   ```
6. Signaler à l'utilisateur les réalisations extraites et leur reformulation proposée.
7. Intégrer ces highlights dans le CV ciblé (étape 5A) et dans la gap analysis (étape 4) comme atouts additionnels datés de l'année en cours.

**Enrichissement portfolio** : après chargement du CV, vérifier `basics.url` :
- Si l'URL ressemble à un portfolio perso (ex: contient `github.io`, ou domaine non social — exclure : dev.to, twitter.com, linkedin.com, github.com, kaggle.com, huggingface.co, youtube.com, pypi.org) → fetcher automatiquement via `WebFetch`
- Extraire les éléments complémentaires : projets, compétences, réalisations, publications, conférences non listées dans le CV
- Signaler à l'utilisateur les éléments supplémentaires trouvés
- Intégrer ces données dans la gap analysis (étape 4) comme atouts additionnels

### Étape 2 — Offre d'emploi

Demander immédiatement (AskUserQuestion, header: "Offre") :
- URL de l'offre (le skill ira la lire)
- Texte de l'offre (coller directement)
- Numéro AVP OPT-NC (ex: `26-0689`) — fetcher via `mcp__avps-opt-nc__avps_on_card_click`

**Si URL** : fetcher via `WebFetch` et extraire :
- Titre du poste
- Missions principales
- Compétences et profil requis
- Entreprise / contexte

**Si numéro AVP OPT-NC** : appeler `mcp__avps-opt-nc__avps_on_card_click` avec `job_id` = le numéro fourni. Si erreur, fallback sur `WebFetch` de l'`url_markdown` (`https://raw.githubusercontent.com/opt-nc/avps/refs/heads/main/data/{numero}.md`).

Afficher une fiche synthèse de l'offre avant de continuer.

**Si l'offre est un AVP OPT-NC** (numéro ou URL `opt-nc.github.io/avps/`) : demander (AskUserQuestion, header: "Contexte OPT") :
- Employé actuel OPT-NC (mobilité interne)
- Fonctionnaire NC hors OPT (voie hiérarchique)
- Ancien agent OPT
- Candidat extérieur

**Si "Employé actuel OPT-NC"** : proposer de charger l'EAE :
- Utilisez-vous le template EAE de l'OPT-NC (https://github.com/opt-nc/template-eae) ?
- **Si non** : proposer de s'y mettre (guide : https://dev.to/adriens/versionner-et-builder-lebook-de-son-entretien-annuel-devaluation-sur-github-242k), puis passer à la saisie manuelle.
- **Si oui** :
  1. Demander le login GitHub
  2. Lister les branches via `gh api repos/{login}/eae-opt/branches --jq '.[].name'`
  3. Si introuvable : demander l'URL complète en fallback
  4. Récupérer `src/03_fiche-de-poste.md` via `gh api "repos/{login}/eae-opt/contents/src%2F03_fiche-de-poste.md?ref={branche}" --jq '.content' | base64 -d`
  5. Extraire : missions, activités principales/secondaires, compétences requises, **lien hiérarchique** (responsable pour la mention "sous couvert")
- **Saisie manuelle** (si pas de template EAE) : points forts du dernier EAE, compétences valorisées par la hiérarchie, nom du responsable hiérarchique direct.

> **Mention obligatoire dans toute lettre OPT-NC** : "sous couvert de [Responsable hiérarchique]" — extraire du fichier EAE ou demander explicitement.

### Étape 3 — Choix du livrable

Demander (AskUserQuestion, header: "Livrable") :
- **CV taillé sur mesure** — JSON ciblé + version AsciiDoc mise en page + PDF
- **Lettre de motivation** — lettre AsciiDoc + PDF qui démontre que les réalisations passées répondent point par point à l'offre
- **Document de préparation d'entretien** — AsciiDoc + PDF avec points forts/faibles, questions probables et réponses préparées
- **Tout** — CV + lettre + doc entretien (recommandé)

### Étape 4 — Gap analysis (automatique, avant tout livrable)

Produire un tableau de correspondance CV ↔ offre :

| Critère attendu | Ce que le CV apporte | Adéquation |
|---|---|---|
| ... | ... | 🟢 / 🟡 / 🟠 / 🔴 |

Légende :
- 🟢 Match direct et démontrable
- 🟡 Match indirect ou transposable
- 🟠 Écart gérable — à compenser dans le livrable
- 🔴 Écart significatif — à ne pas soulever ou à mentionner honnêtement

Synthèse :
- Atouts à mettre en avant (2-3, avec formulation prête à l'emploi)
- Points à compenser et comment les tourner positivement
- Verdict : 🟢 Candidature solide / 🟡 À valoriser / 🟠 Risquée

### Étape 5A — CV taillé sur mesure (si demandé)

**Principe** : ne garder que ce qui parle à ce recruteur, réordonner pour mettre en avant ce qui compte.

1. **Sélection des expériences** : garder uniquement les postes et highlights en lien avec l'offre. Pour chaque expérience conservée, reformuler les highlights pour qu'ils résonnent avec le vocabulaire de l'offre (sans inventer).

2. **Sélection des compétences** : ne lister que les skills pertinents pour le poste, triés par ordre de pertinence décroissante.

3. **Sélection des projets** : garder les projets qui illustrent les compétences clés de l'offre. Pour chaque projet conservé, mettre en avant le highlight le plus en lien avec le poste.

4. **Réalisations et awards** : ne garder que ceux qui apportent de la crédibilité pour ce poste.

5. **Résumé personnalisé** : réécrire `basics.summary` pour qu'il réponde directement au profil recherché dans l'offre (2-3 phrases, sans mensonge).

6. **Produire le CV ciblé JSON** valide (même schéma que l'original), enregistré sous `resume-{nom-du-poste-slug}.json`.

7. **Produire le CV ciblé AsciiDoc** (`resume-{nom-du-poste-slug}.adoc`) — version lisible et imprimable :
   - En-tête : nom, titre ciblé, coordonnées
   - Sections : Résumé · Expériences · Compétences · Projets · Formation · Distinctions
   - Mise en page claire, adaptée à une lecture recruteur

8. **Générer le PDF** immédiatement :
   ```bash
   asciidoctor-pdf resume-{nom-du-poste-slug}.adoc -o resume-{nom-du-poste-slug}.pdf
   ```

### Étape 5B — Lettre de motivation (si demandée)

**Principe** : chaque paragraphe répond à une exigence de l'offre avec une réalisation concrète du passé.

Demander (AskUserQuestion) :

**Signature** (header: "Signature") :
- Oui, j'ai une image (chemin à fournir)
- Non, signature textuelle

**Structure de la lettre AsciiDoc** :
- **En-tête** : coordonnées (depuis `basics`), date, destinataire si connu
- **Objet + sous couvert** (si OPT-NC interne)
- **Accroche** : lien immédiat entre le profil et la mission centrale — 1-2 phrases qui donnent envie de lire la suite
- **§1 — Ce que j'ai fait** : 2-3 réalisations concrètes issues du CV, quantifiées si possible, choisies parce qu'elles répondent directement aux missions du poste
- **§2 — Ce que ça prouve** : relier explicitement ces réalisations aux compétences et profil attendus dans l'offre — montrer que le passé garantit la capacité à réussir dans ce poste
- **§3 — Pourquoi ce poste** : motivation spécifique à l'entreprise/mission/contexte — pas une formule générique
- **Conclusion** : disponibilité, appel à l'action
- **Signature** : `image::{chemin}[Signature, 150]` ou textuelle

Créer le fichier `lettre-motivation-{numero-avp}.adoc` dans le répertoire courant.

**Générer le PDF immédiatement après création** :
```bash
asciidoctor-pdf lettre-motivation-{numero-avp}.adoc -o lettre-motivation-{numero-avp}.pdf
```

### Étape 5C — Document de préparation d'entretien (si demandé)

**Principe** : transformer la gap analysis en guide opérationnel pour l'entretien — ce qu'on dit, comment on le dit, avec quels exemples.

Créer `preparation-entretien-{nom-du-poste-slug}.adoc` dans le répertoire courant, structuré ainsi :

**Structure du document AsciiDoc** :
- **En-tête** : poste visé, date de l'entretien (si connue), candidat
- **== Points forts à valoriser** : pour chaque match 🟢🟡 de la gap analysis
  - Critère attendu par le jury
  - Réalisation concrète à citer (issue du CV)
  - Formulation recommandée (phrase prête à l'emploi, style STAR)
- **== Points faibles / écarts à préparer** : pour chaque écart 🟠🔴
  - Nature de l'écart
  - Comment le reformuler positivement ou le relativiser
  - Ce qu'on ne dit pas spontanément (mais qu'on peut concéder si on est pressé)
- **== Questions probables du jury** : 8-12 questions inférées des critères de l'offre et du contexte
  - Questions sur les compétences clés
  - Questions sur la motivation / mobilité
  - Questions sur le management ou la transversalité (si applicable)
- **== Réponses préparées** : pour les 5 questions les plus probables
  - Question
  - Réponse structurée (méthode STAR : Situation → Tâche → Action → Résultat)
  - Exemple tiré du CV
- **== Éléments de contexte à maîtriser** : chiffres clés de l'offre, vocabulaire métier du poste, points d'attention sur la direction/BU cible
- **== Check-list pré-entretien** : documents à apporter, points à réviser la veille

**Générer le PDF immédiatement après création** :
```bash
asciidoctor-pdf preparation-entretien-{nom-du-poste-slug}.adoc -o preparation-entretien-{nom-du-poste-slug}.pdf
```

## Arguments

`$ARGUMENTS` — username JSON Resume ou URL directe vers un `resume.json`

```
/json-resume adriens                 # registry.jsonresume.org/adriens
/json-resume https://…/resume.json   # URL directe
/json-resume                         # demande interactive
```
