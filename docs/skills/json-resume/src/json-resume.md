# Candidature à une offre d'emploi via JSON Resume

Charge un CV au format JSON Resume et accompagne la rédaction d'une lettre de motivation ciblée sur une offre d'emploi.

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
2. Si échec (4xx/5xx) : `https://gist.githubusercontent.com/{username}/resume.json/raw` via `WebFetch`
3. Si échec : `https://raw.githubusercontent.com/{username}/{username}/main/resume.json` via `WebFetch`
4. Si tout échoue : demander à l'utilisateur de fournir l'URL directe ou le fichier local

**Informer l'utilisateur** de la source effectivement utilisée (registry ou fallback).

### Étape 2 — Synthèse rapide du profil

Afficher en markdown (compact, pas de grands tableaux) :

- **Nom & titre** (depuis `basics`)
- **Résumé** : 2 phrases extraites ou reformulées depuis `basics.summary`
- **Dernière expérience** : poste + entreprise + durée
- **Top 5 compétences** (par niveau décroissant depuis `skills`)

### Étape 3 — Offre d'emploi

Demander immédiatement (AskUserQuestion, header: "Offre") :
- URL de l'offre (le skill ira la lire)
- Texte de l'offre (coller directement)

**Si URL** : fetcher via `WebFetch` et extraire :
- Titre du poste
- Missions principales
- Compétences et profil requis
- Entreprise / contexte

**Afficher une fiche synthèse de l'offre** avant de continuer.

### Étape 4 — Gap analysis automatique

Produire un tableau de correspondance CV ↔ offre :

| Critère attendu | Ce que le CV apporte | Adéquation |
|---|---|---|
| ... | ... | 🟢 / 🟡 / 🟠 / 🔴 |

Légende :
- 🟢 Match direct et démontrable
- 🟡 Match indirect ou transposable
- 🟠 Écart gérable — à compenser dans la lettre
- 🔴 Écart significatif — à mentionner honnêtement ou à ne pas soulever

**Synthèse en 3 points** :
- Atouts principaux à mettre en avant (2-3 max, avec formulation prête à l'emploi)
- Points à compenser et comment les tourner positivement
- Verdict : 🟢 Candidature solide / 🟡 À valoriser / 🟠 Risquée

### Étape 5 — Lettre de motivation

Demander (AskUserQuestion, 2 questions simultanées) :

**Q1 — Format** (header: "Format") :
- Markdown (.md)
- AsciiDoc (.adoc)

**Q2 — Signature** (header: "Signature") :
- Oui, j'ai une image (chemin à fournir)
- Non, signature textuelle

**Rédiger la lettre** en intégrant :
- **En-tête** : coordonnées du candidat (depuis `basics`), date, destinataire si connu
- **Accroche** : lien direct entre le profil et la mission centrale du poste (1-2 phrases percutantes)
- **§1 — Qui je suis** : parcours synthétique ancré dans ce que le poste demande
- **§2 — Ce que j'apporte** : 2-3 réalisations concrètes issues du CV, quantifiées si possible, en lien avec l'offre
- **§3 — Pourquoi ce poste** : motivation spécifique (entreprise, mission, contexte) — pas générique
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
/json-resume adriens              # registry.jsonresume.org/adriens
/json-resume https://…/resume.json  # URL directe
/json-resume                      # demande interactive
```
