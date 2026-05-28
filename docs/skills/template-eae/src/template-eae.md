# Template EAE — Entretien Annuel d'Évaluation (Fonction Publique NC)

Prépare, complète et analyse un EAE (Entretien Annuel d'Échange) versionné sur GitHub via le template [opt-nc/template-eae](https://github.com/opt-nc/template-eae).

> ⚠️ **Confidentialité — point critique** : le repo GitHub doit TOUJOURS être créé en **privé** (`--private`). Un repo public exposerait toutes tes données personnelles, ton évaluation, ta note et tes souhaits de carrière à n'importe qui sur Internet. Ne jamais créer ou passer un repo EAE en public.

> ℹ️ **Toutes les collectivités** : ce template est utilisable par tout agent permanent de la fonction publique de Nouvelle-Calédonie, quelle que soit sa collectivité — OPT-NC, Province Sud, Province Nord, Province des Îles, Gouvernement de la Nouvelle-Calédonie, communes, établissements publics. Le processus EAE est commun à toutes ces entités (arrêté n° 1065 du 22 août 1953).

## Instructions

### Étape 0 — Routing

**Si `$ARGUMENTS` est fourni** : l'interpréter comme un login GitHub (ex: `adriens`) ou `login/repo` → aller directement à l'**Étape 2** (Analyser un EAE existant).

**Si vide** : demander (AskUserQuestion) :

**Je veux…** (header: "EAE") :
- 🎓 Découvrir — je ne connais pas encore cette approche (onboarding)
- 🆕 Initialiser un nouvel EAE depuis le template GitHub
- 📖 Lire et analyser mon EAE existant
- ✏️ Compléter une section de mon EAE
- 🏗️ Builder les documents (PDF, ePub, HTML, DOCX)
- 🎯 Mode coach — j'utilise déjà le template, aide-moi à optimiser mon EAE

---

### Étape Onboarding — Scénario "for dummies" : de zéro à l'EAE buildé

> Parcours guidé complet pour un agent qui démarre de zéro. Chaque étape est validée avant de passer à la suivante.

#### Phase 0 — Comprendre en 2 minutes

Présenter ce pitch synthétique à l'utilisateur :

---
**L'EAE (Entretien Annuel d'Évaluation)** — c'est l'entretien obligatoire chaque année avec ton responsable. Tu y es noté, tu y fixes tes objectifs, tu exprimes tes souhaits de mobilité.

**Le problème :** sur monportailrh.nc, les sessions expirent, tu perds ta saisie, et si ton manager et toi modifiez en même temps, le dernier écrase l'autre.

**La solution :** versionner son EAE sur GitHub comme un développeur versionne son code. Un fichier Markdown par section, une branche par année, et on génère un beau PDF/ePub/Word d'un seul coup.

Et ce qui est vraiment puissant :
- 📱 **Consultable partout** — depuis un téléphone, une tablette, une liseuse (format ePub), n'importe quel navigateur
- ✍️ **Commits signés** — chaque modification est horodatée et attribuée à son auteur, infalsifiable
- 🔍 **Diff entre deux années** — `git diff EAE-2024 EAE-2025 -- src/07_plan-action.md` pour voir exactement ce qui a changé dans les objectifs d'une année à l'autre
- 📊 **Analyse IA** — les fichiers Markdown sont parfaits pour faire des embeddings, de la recherche sémantique ou une analyse automatisée de l'évolution de carrière sur plusieurs années
- 🏷️ **Release officielle** — le tag Git et la release GitHub servent d'archive horodatée et immuable, consultable à tout moment

⚠️ **Important** : ton repo doit être **privé**. Un repo public rendrait tes données personnelles, ta note et tes objectifs accessibles à tout le monde sur Internet.

📺 Pour voir ça en action : https://www.youtube.com/watch?v=FRVsA7NoZv8
---

Demander (AskUserQuestion, header: "Départ") :
- On y va — guide-moi pas à pas
- Je veux d'abord regarder la vidéo (je reviendrai avec `/template-eae`)

#### Phase 1 — Vérifier les prérequis

Demander d'abord le système d'exploitation (AskUserQuestion, header: "OS") :
- Linux (Debian/Ubuntu/WSL)
- macOS

Faire chaque vérification séquentiellement et s'arrêter à la première qui échoue :

**1a. GitHub CLI (`gh`)**
```bash
gh auth status
```
- Si OK → continuer.
- Si `gh` non installé :
  ```bash
  # Linux
  sudo apt install gh
  # Mac
  brew install gh
  ```
  Puis : `gh auth login` (suivre le flow interactif)
  > **Arrêter ici et relancer `/template-eae` une fois authentifié.**
- Si non authentifié → guider : `gh auth login`

**1b. Pandoc (conversion des documents)**
```bash
which pandoc || echo "absent"
```
Si absent :
```bash
# Linux
sudo apt-get install -y pandoc
# Mac
brew install pandoc
```

**1c. Moteur LaTeX (pour le PDF)**
```bash
which xelatex || which lualatex || echo "absent"
```
Si absent :
```bash
# Linux — installation complète (longue mais nécessaire)
sudo apt-get install -y texlive-full
# Mac
brew install --cask mactex
# Alternative Mac légère (plus rapide)
brew install --cask basictex && sudo tlmgr install collection-fontsrecommended
```

**1d. go-task (automatisation du build)**
```bash
which task || echo "absent"
```
Si absent :
```bash
# Linux
sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin
# Vérifier que ~/.local/bin est dans le PATH
echo $PATH | grep -q ".local/bin" || echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc && source ~/.bashrc
# Mac
brew install go-task
```

Afficher un récapitulatif des prérequis :
> ✅ GitHub CLI authentifié · ✅ pandoc · ✅ LaTeX (xelatex/lualatex) · ✅ task — tout est prêt !

#### Phase 2 — Créer le repo EAE

Expliquer le git flow recommandé avant de créer :

> **Git flow EAE recommandé :**
>
> ```mermaid
> gitGraph
>    commit id: "init template"
>    branch EAE-2025
>    checkout EAE-2025
>    commit id: "section/00 identification"
>    commit id: "section/03 fiche de poste"
>    branch revision/manager-objectifs
>    checkout revision/manager-objectifs
>    commit id: "manager: ajustement objectifs"
>    checkout EAE-2025
>    merge revision/manager-objectifs id: "PR validée"
>    commit id: "section/06 autoévaluation"
>    commit id: "section/07 plan action"
>    branch revision/manager-synthese
>    checkout revision/manager-synthese
>    commit id: "manager: synthèse + note"
>    checkout EAE-2025
>    merge revision/manager-synthese id: "PR validée #2"
>    checkout main
>    merge EAE-2025 id: "EAE 2025 final" tag: "EAE-2025-final"
> ```
>
> **Règles de collaboration :**
> - L'**évalué** crée et pousse ses commits sur des branches `section/xxx` ou directement sur `EAE-{annee}`
> - Le **manager** travaille sur des branches `revision/xxx` et ouvre des **Pull Requests** vers `EAE-{annee}`
> - Les **Issues GitHub** servent à commenter, questionner ou signaler un point à retravailler — sans modifier le document directement
> - Les deux parties **discutent et valident via les PRs** (review, commentaires inline, approbation)
> - Une fois l'EAE validé par les deux parties : l'**évalué** merge `EAE-{annee}` dans `main`
> - L'**évalué** crée ensuite un **tag** et une **release GitHub** pour officialiser la version finale :
>   ```bash
>   git tag -a EAE-{annee}-final -m "EAE {annee} — version finale validée"
>   git push origin EAE-{annee}-final
>   gh release create EAE-{annee}-final \
>     --title "EAE {annee} — Version finale" \
>     --notes "EAE {annee} validé par l'évalué et le manager. Documents joints : PDF, ePub." \
>     dist/*.pdf dist/*.epub
>   ```
> - La release GitHub sert d'**archive officielle** horodatée et immuable de l'EAE signé

Demander le login GitHub de l'utilisateur, puis :

```bash
gh repo create {login}/eae-opt \
  --description "Mon EAE — Entretien Annuel d'Évaluation" \
  --private \
  --template opt-nc/template-eae
```

Créer la branche pour l'année en cours :
```bash
SHA=$(gh api repos/{login}/eae-opt/git/refs/heads/main --jq '.object.sha')
ANNEE=$(date +%Y)
gh api repos/{login}/eae-opt/git/refs \
  -f ref="refs/heads/EAE-$ANNEE" \
  -f sha="$SHA"
```

Afficher :
> ✅ Repo créé : https://github.com/{login}/eae-opt
> ✅ Branche EAE-{annee} prête

#### Phase 3 — Remplir l'identification (section 00)

Expliquer : "On commence par te présenter — ces données seront dans l'en-tête de tous tes documents."

Poser les questions une par une :
1. Nom de famille ?
2. Prénom ?
3. Matricule agent (ex: 123456) ?
4. Nom de ton responsable hiérarchique direct ?
5. Ton référent DRH ?
6. Nom de ton administration (ex: OPT-NC) ?
7. Ton affectation (direction/service) ?

Générer le fichier `src/00_identification_agent.md` complété, puis l'écrire sur GitHub :
```bash
SHA=$(gh api "repos/{login}/eae-opt/contents/src/00_identification_agent.md?ref=EAE-{annee}" --jq '.sha')
CONTENT=$(printf '%s' '{contenu}' | base64 -w0)
gh api "repos/{login}/eae-opt/contents/src/00_identification_agent.md" \
  -X PUT \
  -f message="feat(eae): identification agent — section 00" \
  -f content="$CONTENT" \
  -f sha="$SHA" \
  -f branch="EAE-{annee}"
```

#### Phase 4 — Remplir la fiche de poste (section 03)

Expliquer : "La fiche de poste décrit ton rôle. C'est la section la plus importante — elle sert aussi si tu candidates à un autre poste."

Poser les questions :
1. Intitulé exact de ton poste ?
2. Ton grade ?
3. Emploi selon le référentiel DRHFPNC (cf https://drhfpnc.gouv.nc) ?
4. Collectivité (ex: OPT-NC) ?
5. Direction et service ?
6. Localisation (ville/site) ?
7. Tes missions principales en 2-3 phrases ?
8. Nom et fonction de ton responsable hiérarchique direct ?
9. Tes 3 à 5 activités principales ?
10. Tes 2 à 3 activités secondaires ?
11. Les 3 à 5 compétences requises pour ce poste ?

Générer et écrire `src/03_fiche-de-poste.md` sur la branche EAE-{annee}.

#### Phase 5 — Cloner et builder les documents

Cloner le repo localement :
```bash
gh repo clone {login}/eae-opt
cd eae-opt
git checkout EAE-{annee}
```

Demander (AskUserQuestion, header: "Format de sortie") :
- PDF + ePub (recommandé)
- PDF uniquement
- Tous les formats (PDF, ePub, HTML, DOCX)

Lancer le build :
```bash
task pdf epub   # ou task / task pdf selon le choix
```

Afficher les fichiers générés dans `dist/` et leur chemin absolu.

#### Phase 6 — Récapitulatif et suite

Afficher un résumé de ce qui a été fait :
```
✅ Repo créé    : https://github.com/{login}/eae-opt
✅ Branche      : EAE-{annee}
✅ Section 00   : Identification agent
✅ Section 03   : Fiche de poste
✅ Documents    : {liste des fichiers dans dist/}
```

Proposer les prochaines étapes :
> **Sections restantes à compléter** (lance `/template-eae {login}` pour reprendre) :
> - 06 — Autoévaluation (tes réalisations de l'année)
> - 07 — Plan d'action (tes objectifs pour l'année suivante)
> - 08 — Évolution professionnelle (tes souhaits de mobilité)
>
> **Si tu cherches un autre poste** : lance `/opt-nc-avps` pour explorer les AVPs, puis `/json-resume` pour préparer ta candidature — ta fiche de poste sera réutilisée automatiquement.

---

### Étape 1 — Initialiser un nouvel EAE

1. Demander le login GitHub cible.
2. Proposer un nom de repo (défaut : `eae-opt`).
3. Créer le repo depuis le template :
   ```bash
   gh repo create {login}/{repo} \
     --description "Mon EAE {annee}" \
     --private \
     --template opt-nc/template-eae
   ```
4. Créer une branche pour l'année courante :
   ```bash
   SHA=$(gh api repos/{login}/{repo}/git/refs/heads/main --jq '.object.sha')
   gh api repos/{login}/{repo}/git/refs \
     -f ref="refs/heads/EAE-{annee}" \
     -f sha="$SHA"
   ```
5. Guider pour remplir `src/00_identification_agent.md` : demander nom, prénom, matricule, nom du responsable, référent DRH, administration, affectation.
6. Écrire le fichier via gh api (voir Étape 3 pour la procédure d'écriture).
7. Afficher le lien vers le repo créé.
8. Proposer de passer à l'Étape 3 (Compléter une section) ou directement builder (Étape 4).

---

### Étape 2 — Lire et analyser un EAE existant

1. Si login non fourni, demander le login GitHub et le nom du repo (défaut : `eae-opt`).
2. Lister les branches disponibles :
   ```bash
   gh api repos/{login}/{repo}/branches --jq '.[].name'
   ```
3. Si plusieurs branches `EAE-{annee}` → demander (AskUserQuestion) laquelle analyser.
4. Lire les fichiers clés :
   ```bash
   gh api "repos/{login}/{repo}/contents/src/{fichier}.md?ref={branche}" --jq '.content' | base64 -d
   ```
   Fichiers à lire : `00_identification_agent`, `01_entete`, `03_fiche-de-poste`, `06_autoevaluation`, `07_plan-action`, `08_evolution-professionnelle`.
5. Calculer le taux de complétion par fichier : compter les placeholders `VOTRE_*` et les `...` restants.
6. Présenter un tableau de bord :

| Section | Titre | État | Points manquants |
|---|---|---|---|
| 00 | Identification agent | 🟢/🟡/🔴 | liste |
| 01 | Entête | 🟢/🟡/🔴 | liste |
| 03 | Fiche de poste | 🟢/🟡/🔴 | liste |
| 06 | Autoévaluation | 🟢/🟡/🔴 | liste |
| 07 | Plan d'action | 🟢/🟡/🔴 | liste |
| 08 | Évolution professionnelle | 🟢/🟡/🔴 | liste |

Légende : 🟢 Complet · 🟡 Partiel · 🔴 Non rempli

7. **Si section 08 mentionne mobilité souhaitée** → proposer :
   > 👉 Tu souhaites de la mobilité ? Lance `/opt-nc-avps` pour explorer les AVPs adaptés à ton profil.

8. **Si section 03 complète** → proposer :
   > 👉 Ta fiche de poste peut alimenter une candidature. Lance `/json-resume` — la skill chargera automatiquement cette fiche de poste.

9. Proposer de compléter une section (Étape 3) ou de builder (Étape 4).

---

### Étape 3 — Compléter une section

1. Si repo/branche non encore identifiés → demander login, repo (défaut : `eae-opt`), branche.
2. Demander (AskUserQuestion, header: "Section") quelle section compléter :
   - 00 — Identification (nom, matricule, responsable)
   - 01 — Entête (date de l'entretien, collectivité, direction)
   - 03 — Fiche de poste (missions, activités, compétences, lien hiérarchique)
   - 04 — Tenue du poste (écarts fiche de poste, résultats des objectifs N-1)
   - 05 — Appréciation des compétences (grilles technique, savoir-être, managérial)
   - 06 — Autoévaluation (réalisations, succès, difficultés de l'année)
   - 07 — Plan d'action (objectifs N+1, indicateurs, délais, moyens)
   - 08 — Évolution professionnelle (mobilité, perspectives, concours, VAE)

3. Lire le fichier actuel de la section choisie.
4. Identifier les champs non remplis (`VOTRE_*`, `...`, cases `[ ]`).
5. Poser des questions ciblées par section :

   **Section 00** : nom de famille, prénom, matricule agent, nom du responsable direct, référent DRH, nom de l'administration (ex: OPT-NC), affectation (direction/service).

   **Section 03** : intitulé exact du poste, grade, emploi (référentiel DRHFPNC), collectivité, direction/service, localisation, missions globales (2-3 bullet points), nom + fonction du responsable hiérarchique direct, activités principales (3-5), activités secondaires (2-3), compétences requises (3-5).

   **Section 06** : particularités ou changements depuis l'entretien précédent, objectifs atteints (lesquels, comment), compétences nouvellement acquises, succès marquants de l'année (avec chiffres si possible), difficultés rencontrées, solutions mises en œuvre.

   **Section 07** : jusqu'à 5 objectifs professionnels pour l'année à venir. Pour chacun : libellé de l'objectif, indicateur de mesure, délai. Objectifs de progrès individuels. Moyens demandés : matériels, financiers, autres.

   **Section 08** : souhait de mobilité (oui/non), horizon temporel (< 1 an, 1-2 ans, 2-4 ans), type de mobilité (géographique, fonctionnelle, changement de métier), périmètre souhaité (service, direction, collectivité, autre collectivité), projet de concours ou VAE, perspective de retraite.

6. Générer le contenu markdown complété avec les réponses fournies.
7. Demander (AskUserQuestion, header: "Enregistrement") :
   - Écrire directement dans le repo GitHub via `gh api`
   - Afficher le contenu pour copie manuelle

   **Si écriture directe** :
   ```bash
   # Récupérer le SHA du fichier actuel (nécessaire pour le PUT)
   SHA=$(gh api "repos/{login}/{repo}/contents/src/{fichier}.md?ref={branche}" --jq '.sha')
   # Encoder le nouveau contenu en base64
   CONTENT=$(printf '%s' '{contenu_echappe}' | base64 -w0)
   # Mettre à jour le fichier sur la branche
   gh api "repos/{login}/{repo}/contents/src/{fichier}.md" \
     -X PUT \
     -f message="feat(eae): compléter section {numero} — {titre}" \
     -f content="$CONTENT" \
     -f sha="$SHA" \
     -f branch="{branche}"
   ```

8. **Si section 08 complétée avec mobilité souhaitée** :
   > 👉 Lance `/opt-nc-avps` pour trouver des AVPs correspondant à ton profil de mobilité.

9. **Si section 03 complétée** :
   > 👉 Ces données sont directement exploitables par `/json-resume` pour cibler une candidature — ta fiche de poste sera chargée automatiquement et le responsable hiérarchique sera intégré dans la lettre de motivation (mention "sous couvert de…").

---

### Étape 4 — Builder les documents

1. Si repo non encore identifié → demander login/repo.
2. Vérifier que le repo est cloné localement :
   ```bash
   ls {repo}/Taskfile.yml 2>/dev/null && echo "ok" || echo "absent"
   ```
   Si absent → cloner : `gh repo clone {login}/{repo}`.
3. Vérifier les prérequis :
   ```bash
   which task && which pandoc
   ```
   Si `task` absent :
   ```bash
   # Linux
   sh -c "$(curl --location https://taskfile.dev/install.sh)" -- -d -b ~/.local/bin
   # Mac
   brew install go-task
   ```
   Si `pandoc` absent :
   ```bash
   sudo apt-get install pandoc   # Linux
   brew install pandoc            # Mac
   ```
4. Demander (AskUserQuestion, header: "Format") les formats à générer :
   - Tous les formats (`task`)
   - PDF uniquement (`task pdf`)
   - ePub uniquement (`task epub`)
   - HTML uniquement (`task html`)
   - DOCX uniquement (`task docx`)
5. Lancer le build depuis le répertoire cloné :
   ```bash
   cd {repo} && task {cible}
   ```
6. Lister les fichiers générés dans `dist/`.
7. Si erreur liée à lualatex/xelatex → proposer : `sudo apt-get install texlive-full` (Linux) ou `brew install --cask mactex` (Mac).

---

### Mode Coach — Optimiser son EAE pour un résultat optimal

> Pour les agents qui utilisent déjà le template et veulent aller plus loin : qualité des formulations, cohérence entre sections, préparation de l'entretien oral.

#### Phase 0 — Charger le contexte

1. Demander le login GitHub et le repo (défaut : `eae-opt`).
2. Demander la branche (année de l'EAE à travailler).
3. Lire toutes les sections remplies :
   ```bash
   for f in 00 01 03 04 06 07 08; do
     gh api "repos/{login}/{repo}/contents/src/${f}_*.md?ref={branche}" --jq '.content' | base64 -d
   done
   ```

#### Phase 1 — Audit de qualité

Analyser chaque section complétée selon ces critères :

**Section 03 (fiche de poste)** :
- Les missions sont-elles rédigées avec des verbes d'action (piloter, coordonner, assurer, gérer…) ?
- Les compétences requises correspondent-elles aux activités listées ?
- Le lien hiérarchique est-il clairement identifié (important pour la lettre de candidature) ?

**Section 06 (autoévaluation)** :
- Les réalisations sont-elles quantifiées (chiffres, %, délais, volumes) ?
- Les difficultés sont-elles suivies de solutions concrètes (pas juste une liste de problèmes) ?
- Le bilan est-il équilibré (succès + difficultés) ?

**Section 07 (plan d'action)** :
- Chaque objectif a-t-il un indicateur de mesure et un délai précis ?
- Les objectifs sont-ils SMART (Spécifique, Mesurable, Atteignable, Réaliste, Temporel) ?
- Les moyens demandés sont-ils justifiés par les objectifs ?

**Section 08 (évolution professionnelle)** :
- Les souhaits de mobilité sont-ils cohérents avec le profil et les compétences ?
- Les perspectives (concours, VAE) sont-elles réalistes et documentées ?

Produire un tableau d'audit :

| Section | Point fort | Point à améliorer | Score |
|---|---|---|---|
| 03 Fiche de poste | ... | ... | 🟢/🟡/🔴 |
| 06 Autoévaluation | ... | ... | 🟢/🟡/🔴 |
| 07 Plan d'action | ... | ... | 🟢/🟡/🔴 |
| 08 Évolution | ... | ... | 🟢/🟡/🔴 |

#### Phase 2 — Reformulations ciblées

Pour chaque point à améliorer identifié :
1. Afficher la formulation actuelle
2. Proposer une reformulation améliorée (plus précise, plus percutante, mieux structurée)
3. Demander (AskUserQuestion) si l'agent valide la reformulation proposée
4. Si oui → écrire la modification dans le repo via `gh api` (même procédure qu'Étape 3)

#### Phase 3 — Cohérence inter-sections

Vérifier la cohérence entre les sections :
- Les compétences de la section 03 sont-elles illustrées par des réalisations en section 06 ?
- Les objectifs N-1 de la section 04 sont-ils alignés avec les résultats en section 06 ?
- Les objectifs N+1 (section 07) sont-ils en lien avec les souhaits d'évolution (section 08) ?

Signaler toute incohérence et proposer des ajustements.

#### Phase 4 — Préparation de l'entretien oral

Générer un guide de préparation à l'entretien basé sur le contenu de l'EAE :

**Questions probables du manager** (inférées des sections 04, 06, 07) :
- Sur les objectifs non atteints : "Qu'est-ce qui a bloqué ?"
- Sur les points forts : "Pouvez-vous me donner un exemple concret ?"
- Sur les objectifs N+1 : "Comment comptez-vous mesurer cet objectif ?"
- Sur la mobilité (si section 08 remplie) : "Qu'est-ce qui vous attire dans ce changement ?"

**Formulations prêtes à l'emploi** (méthode STAR) pour les 3 réalisations les plus marquantes :
- **S**ituation : contexte
- **T**âche : ce qui était attendu
- **A**ction : ce que l'agent a fait concrètement
- **R**ésultat : résultat mesurable obtenu

**Points à ne pas oublier** :
- Demander la note avant la fin de l'entretien
- Faire signer le document (circuit de transmission : agent → supérieur → DRH → DRHFPNC pour la CAP)
- Demander une copie signée pour ses archives

#### Phase 5 — Cross-références finales

Si l'EAE révèle des souhaits de mobilité :
> 👉 Lance `/opt-nc-avps` pour explorer les AVPs correspondant à ton profil — les données de ta fiche de poste (section 03) seront utilisées pour affiner la recherche.

Si l'agent veut candidater à un poste :
> 👉 Lance `/json-resume` — la skill lira automatiquement ta fiche de poste (section 03) et ton autoévaluation pour construire le CV ciblé, la lettre "sous couvert" et le document de préparation d'entretien.

---

## Contexte

L'**EAE (Entretien Annuel d'Évaluation)** est conduit chaque année pour tous les agents permanents (titulaires et non titulaires) de la fonction publique de Nouvelle-Calédonie, conformément à l'article 41 de l'arrêté n° 1065 du 22 août 1953.

Le template [opt-nc/template-eae](https://github.com/opt-nc/template-eae) structure l'EAE en fichiers Markdown versionnés sur Git, avec une branche par campagne annuelle. Le build via `task` (go-task + pandoc) produit PDF, ePub, HTML et DOCX depuis les sources.

### Structure des sections

| Fichier | Section | Contenu |
|---|---|---|
| `00_identification_agent.md` | Identification | Nom, matricule, responsable, DRH |
| `01_entete.md` | Entête | Date, collectivité, circuit transmission |
| `02_resume.md` | Résumé dossier | Parcours, formation, situation administrative |
| `03_fiche-de-poste.md` | Fiche de poste | Missions, activités, compétences, lien hiérarchique |
| `04_tenue-maitrise-du-poste.md` | Tenue du poste | Écarts, résultats objectifs N-1 |
| `05_appreciation_competences.md` | Appréciation | Grilles technique / savoir-être / managérial |
| `06_autoevaluation.md` | Autoévaluation | Réalisations, succès, difficultés |
| `07_plan-action.md` | Plan d'action | Objectifs N+1, indicateurs, délais |
| `08_evolution-professionnelle.md` | Évolution | Mobilité, perspectives, concours |
| `09_synthese-evaluation.md` | Synthèse | Appréciation générale, note /20 |
| `10_avancement.md` | Avancement | Avis changement de classe/grade |

### Ressources officielles

- [Guide de l'évalué — DRHFPNC](https://drhfpnc.gouv.nc/sites/default/files/atoms/files/guideevalue.pdf)
- [Guide de l'évaluateur — DRHFPNC](https://drhfpnc.gouv.nc/sites/default/files/atoms/files/guideevaluateur.pdf)
- [Formulaires officiels EAE](https://drhfpnc.gouv.nc/formulaires-agents/entretien-annuel-dechange)
- [Mon Portail RH NC](https://www.monportailrh.nc/)
- [Fiches emploi DRHFPNC](https://drhfpnc.gouv.nc/travailler-dans-la-fonction-publique-trouver-un-emploi-repertoire-des-emplois/les-fiches-emploi)

### Ressources template

- [📺 Vidéo de présentation](https://www.youtube.com/watch?v=FRVsA7NoZv8) — DevOPS LABS
- [📝 Article dev.to](https://dev.to/adriens/versionner-et-builder-lebook-de-son-entretien-annuel-devaluation-sur-github-242k)
- [📘 Template GitHub](https://github.com/opt-nc/template-eae)

### Cross-références

- `/opt-nc-avps` — si l'EAE révèle un souhait de mobilité, rechercher les AVPs adaptés
- `/json-resume` — pour transformer la fiche de poste et l'autoévaluation en candidature ciblée (lettre "sous couvert", doc préparation entretien)

## Arguments

`$ARGUMENTS` — login GitHub (ex: `adriens`) ou `login/repo` (ex: `adriens/eae-opt`)
