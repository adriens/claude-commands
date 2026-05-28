# CLAUDE.md — Conventions du projet claude-commands

## Description

Collection de skills (slash commands) pour Claude Code, dédiées à la Nouvelle-Calédonie et à la fonction publique NC.

## Conventions de commits

Ce projet utilise les **Conventional Commits** (https://www.conventionalcommits.org/).

### Format

```
<type>(<scope>): <description courte>
```

### Types autorisés

| Type | Usage |
|---|---|
| `feat` | Ajout d'un nouveau skill **ou amélioration** du contenu d'un skill existant |
| `fix` | Correction d'une erreur (commande incorrecte, lien cassé, instruction fausse) |
| `docs` | Modification de la documentation du site uniquement (index.md, README) — pas le skill lui-même |
| `refactor` | Réécriture d'un skill sans changer son comportement |
| `chore` | Mise à jour de config, dépendances, fichiers de build |

> **Règle** : toute modification du fichier source d'un skill (`src/*.md` ou `~/.claude/commands/*.md`) est un `feat`, même si c'est un enrichissement mineur. Le `fix` est réservé aux vraies erreurs (mauvaise commande, faute, lien mort).

### Scopes

Le scope correspond au nom du skill concerné :

```
feat(skill/template-eae): ...
feat(skill/kalolo): ...
fix(skill/json-resume): ...
docs(skill/edb-noumea): ...
```

Pour les changements transversaux : `feat(skills): ...`

### Exemples

```
feat(skill/template-eae): ajout skill EAE avec onboarding for dummies
fix(skill/opt-nc-avps): corriger le threshold de recherche par défaut
docs(skill/kalolo): ajouter exemples d'utilisation dans index.md
refactor(skill/json-resume): restructurer les étapes de gap analysis
chore: mise à jour de la structure des docs
```

## Conventions de tags

Les tags suivent le **semantic versioning** (https://semver.org/) : `vMAJEUR.MINEUR.PATCH`

| Incrément | Quand |
|---|---|
| **MAJEUR** (`v2.0.0`) | Refonte complète d'un skill, breaking change |
| **MINEUR** (`v1.1.0`) | Ajout d'un nouveau skill ou fonctionnalité significative |
| **PATCH** (`v1.0.1`) | Correction de bug, amélioration mineure |

### Créer un tag

```bash
git tag -a v1.1.0 -m "feat: ajout skill template-eae"
git push origin v1.1.0
```

### Règle

Créer un tag à chaque fois qu'un nouveau skill est publié ou qu'une amélioration significative est apportée à un skill existant, afin que les utilisateurs puissent épingler une version stable.

## Structure des skills

```
docs/skills/{nom-skill}/
├── index.md           # Page de documentation (site web)
└── src/
    └── {nom-skill}.md # Source du skill (= contenu de ~/.claude/commands/{nom-skill}.md)
```

Le fichier source `src/{nom-skill}.md` est installé par l'utilisateur via :
```bash
curl -o ~/.claude/commands/{nom-skill}.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/{nom-skill}/src/{nom-skill}.md
```

## Ajouter un nouveau skill

1. Créer `docs/skills/{nom}/src/{nom}.md` — le skill lui-même
2. Créer `docs/skills/{nom}/index.md` — la page de documentation
3. Committer : `feat(skill/{nom}): ajout skill {description}`
4. Tagger si c'est un nouveau skill : `git tag -a vX.Y.0 -m "feat: ajout skill {nom}"`
5. Pousser : `git push && git push origin vX.Y.0`
