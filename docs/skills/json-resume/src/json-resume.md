# Analyse de CV JSON Resume

Charge un CV au format JSON Resume depuis le registry ou une URL, et propose un accompagnement complet : analyse, gap analysis avec une offre d'emploi, génération de lettre de motivation ciblée.

## Instructions

### Étape 1 — Chargement du CV

1. Si `$ARGUMENTS` contient un nom d'utilisateur ou une URL, l'utiliser directement.
2. Sinon, demander (AskUserQuestion) :

**Q — Source du CV** (header: "Source CV") :
- Registry JSON Resume (username)
- URL directe (raw JSON)
- Fichier local (chemin)

**Selon le choix** :
- **Registry** : construire l'URL `https://registry.jsonresume.org/{username}.json` et fetcher via `WebFetch`
- **URL directe** : fetcher directement via `WebFetch`
- **Fichier local** : lire le fichier via `Read`

3. Si le fetch échoue sur le registry, tenter `https://raw.githubusercontent.com/{username}/{username}/main/resume.json` comme fallback.

### Étape 2 — Synthèse du profil

Présenter un résumé structuré du CV en markdown :

- **Identité** : nom, titre, localisation, email, profils (GitHub, LinkedIn, etc.)
- **Résumé** : reformulation concise du `summary` (2-3 phrases)
- **Compétences clés** : top 10 skills triés par niveau si disponible
- **Expérience** : tableau avec poste, entreprise, durée, faits saillants
- **Formation** : diplômes et certifications
- **Réalisations notables** : publications, projets open source, conférences, articles

Puis proposer (AskUserQuestion) :

**Q — Que faire avec ce CV ?** (header: "Action") :
- Analyser face à une offre d'emploi (gap analysis)
- Générer un pitch / résumé exécutif
- Rédiger une lettre de motivation ciblée
- Les trois (dans cet ordre)

### Étape 3 — Gap analysis CV ↔ Offre (si demandé)

1. Demander l'offre d'emploi (AskUserQuestion) :
   - URL de l'offre
   - Texte collé directement

2. Si URL : fetcher via `WebFetch` et extraire : titre, missions, compétences requises, profil recherché, avantages.

3. Produire un tableau de gap analysis :

| Critère | Requis par l'offre | Présent dans le CV | Niveau d'adéquation |
|---|---|---|---|
| ... | ... | ... | 🟢 / 🟡 / 🟠 / 🔴 |

Légende :
- 🟢 Match parfait
- 🟡 Match partiel (expérience indirecte ou transposable)
- 🟠 Écart gérable (à travailler / à mettre en valeur différemment)
- 🔴 Écart significatif (formation ou expérience manquante)

4. **Synthèse stratégique** :
   - Points forts à mettre en avant (3-5 arguments)
   - Points à compenser (avec stratégie de communication pour chacun)
   - Verdict global : 🟢 Candidature solide / 🟡 Candidature à valoriser / 🟠 Candidature risquée

### Étape 4 — Pitch / Résumé exécutif (si demandé)

Générer 3 variantes de pitch selon le contexte :

1. **Pitch LinkedIn** (300 caractères max) : accroche percutante pour "About"
2. **Pitch entretien** (2 minutes, ~250 mots) : structure Passé → Présent → Futur
3. **Pitch email de candidature spontanée** (5-6 lignes) : contexte, valeur ajoutée, appel à l'action

Adapter le ton au domaine détecté dans le CV (tech, management, data, etc.).

### Étape 5 — Lettre de motivation (si demandée)

**Prérequis** : avoir une offre d'emploi (étape 3) ou en demander une.

**Format** (AskUserQuestion, header: "Format lettre") :
- Markdown (.md)
- AsciiDoc (.adoc)

**Signature** (AskUserQuestion, header: "Signature") :
- Oui, j'ai une image de signature (demander le chemin)
- Non, signature textuelle

**Contenu de la lettre** :
- En-tête : coordonnées du candidat (extraites du CV), destinataire si connu
- Accroche : lien direct entre le profil et la mission centrale du poste
- Paragraphe 1 — Qui je suis : synthèse du parcours en lien avec le poste
- Paragraphe 2 — Ce que j'apporte : 2-3 réalisations concrètes issues du CV, quantifiées si possible
- Paragraphe 3 — Pourquoi ce poste : motivation spécifique à l'entreprise/mission
- Conclusion : disponibilité, appel à l'action
- Signature (image ou textuelle)

**Génération Word** : proposer après création du fichier :
```bash
# Markdown
pandoc lettre-motivation.md -o lettre-motivation.docx

# AsciiDoc
pandoc lettre-motivation.adoc -o lettre-motivation.docx
```

> Note : pandoc doit être installé (`brew install pandoc` ou `sudo apt install pandoc`).

## Arguments

`$ARGUMENTS` — username JSON Resume registry ou URL directe vers un fichier `resume.json`

Exemples :
- `/json-resume adriens` — charge depuis `registry.jsonresume.org/adriens`
- `/json-resume https://example.com/resume.json` — charge depuis une URL
- `/json-resume` — demande interactivement la source
