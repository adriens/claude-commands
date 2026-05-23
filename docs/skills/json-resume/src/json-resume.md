# Candidature à une offre d'emploi via JSON Resume

Charge un CV au format JSON Resume (registry.jsonresume.org, Gist, URL ou fichier local) et accompagne la candidature à une offre d'emploi : CV taillé sur mesure et/ou lettre de motivation qui démontre que le passé répond exactement à ce que le poste demande.

TRIGGER when: user mentions JSON Resume, resume.json, registry.jsonresume.org, a CV/résumé in JSON format, fetching or reading someone's resume/CV, job application with a structured CV, cover letter from a CV.
SKIP: CV in Word/PDF without JSON Resume context, generic cover letter without a structured CV source.

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

### Étape 2 — Offre d'emploi

Demander immédiatement (AskUserQuestion, header: "Offre") :
- URL de l'offre (le skill ira la lire)
- Texte de l'offre (coller directement)

**Si URL** : fetcher via `WebFetch` et extraire :
- Titre du poste
- Missions principales
- Compétences et profil requis
- Entreprise / contexte

Afficher une fiche synthèse de l'offre avant de continuer.

### Étape 3 — Choix du livrable

Demander (AskUserQuestion, header: "Livrable") :
- **CV taillé sur mesure** — extraire et réorganiser les éléments du JSON Resume pour coller au mieux à l'offre
- **Lettre de motivation** — rédiger une lettre qui démontre que les réalisations passées répondent point par point à l'offre
- **Les deux** — CV ciblé puis lettre cohérente avec le CV produit

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

6. **Produire le CV ciblé** en JSON Resume valide (même schéma que l'original), enregistré sous `resume-{nom-du-poste-slug}.json`.

### Étape 5B — Lettre de motivation (si demandée)

**Principe** : chaque paragraphe répond à une exigence de l'offre avec une réalisation concrète du passé.

Demander (AskUserQuestion, 2 questions simultanées) :

**Q1 — Format** (header: "Format") :
- Markdown (.md)
- AsciiDoc (.adoc)

**Q2 — Signature** (header: "Signature") :
- Oui, j'ai une image (chemin à fournir)
- Non, signature textuelle

**Structure de la lettre** :
- **En-tête** : coordonnées (depuis `basics`), date, destinataire si connu
- **Accroche** : lien immédiat entre le profil et la mission centrale — 1-2 phrases qui donnent envie de lire la suite
- **§1 — Ce que j'ai fait** : 2-3 réalisations concrètes issues du CV, quantifiées si possible, choisies parce qu'elles répondent directement aux missions du poste
- **§2 — Ce que ça prouve** : relier explicitement ces réalisations aux compétences et profil attendus dans l'offre — montrer que le passé garantit la capacité à réussir dans ce poste
- **§3 — Pourquoi ce poste** : motivation spécifique à l'entreprise/mission/contexte — pas une formule générique
- **Conclusion** : disponibilité, appel à l'action
- **Signature** : image intégrée ou textuelle

Créer le fichier (`lettre-motivation.md` ou `.adoc`) dans le répertoire courant.

**Proposer la conversion Word** après création :
```bash
pandoc lettre-motivation.md -o lettre-motivation.docx
# ou
pandoc lettre-motivation.adoc -o lettre-motivation.docx
```
> pandoc requis : `brew install pandoc` ou `sudo apt install pandoc`

## Arguments

`$ARGUMENTS` — username JSON Resume ou URL directe vers un `resume.json`

```
/json-resume adriens                 # registry.jsonresume.org/adriens
/json-resume https://…/resume.json   # URL directe
/json-resume                         # demande interactive
```
