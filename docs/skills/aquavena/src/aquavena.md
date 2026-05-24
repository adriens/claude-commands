# Aquavena — Repas healthy Nouvelle-Calédonie

Explore les menus et tarifs Aquavena, le service de gamelle healthy livré à domicile à Nouméa, et aide à choisir la formule adaptée.

## Instructions

### Étape 0 — Intention de l'utilisateur

**Si `$ARGUMENTS` est fourni** : l'interpréter comme un objectif ou un régime recherché (passer à l'étape 1 option D).

**Si vide** : demander (AskUserQuestion) ce que l'utilisateur souhaite :

**Je veux…** (header: "Action") :
- Découvrir les régimes disponibles
- Consulter le menu de la semaine
- Connaître les tarifs
- Choisir le régime adapté à mon objectif

---

### Étape 1 — Selon l'intention

#### Option A — Lister les régimes
1. Appeler `mcp__aquavena__list_regimes`.
2. Présenter un tableau markdown : nom du régime, slug.
3. Proposer de consulter le menu d'un régime ou les tarifs.

#### Option B — Menu de la semaine
1. Si le régime n'est pas encore connu : appeler `mcp__aquavena__list_regimes` et demander le choix (AskUserQuestion).
2. Appeler `mcp__aquavena__get_menus` avec le `regime_slug` choisi.
3. Présenter le menu de façon lisible (tableau par jour : repas, plat, description si disponible).
4. Proposer d'afficher les tarifs ou de consulter un autre régime.

#### Option C — Tarifs
1. Appeler `mcp__aquavena__get_tarifs`.
2. Présenter la grille tarifaire en tableau markdown (colonnes : formule, prix HT, prix TTC en XPF).
3. Mettre en évidence les formules les plus populaires si identifiables.
4. Rappeler le lien pour commander.

#### Option D — Conseil régime personnalisé
1. Demander l'objectif (AskUserQuestion) :
   - Perte de poids / minceur
   - Performance sportive
   - Végétarien / végétalien
   - Alimentation en famille
   - Bien-être général / équilibre
2. Appeler `mcp__aquavena__list_regimes` pour avoir la liste complète.
3. Recommander 1-2 régimes adaptés avec une justification courte (2-3 points).
4. Proposer d'afficher le menu et les tarifs de la formule recommandée.

---

### Étape 2 — Appel à l'action

Toujours terminer par :

> 🛒 **Commander** : [aquavena.nc](https://www.aquavena.nc/) — Livraison à domicile sur Nouméa, Nouvelle-Calédonie.

---

## Arguments
`$ARGUMENTS` — objectif ou régime recherché (ex: `"végétarien"`, `"sportif"`, `"menu low-carb de la semaine"`)
