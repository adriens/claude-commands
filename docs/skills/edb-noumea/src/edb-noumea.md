# EDB Nouméa — Qualité des eaux de baignade

Consulte la qualité sanitaire des eaux de baignade sur les plages de Nouméa (E. coli, Entérocoques) et donne un avis de sécurité pour la baignade.

## Instructions

### Étape 0 — Intention de l'utilisateur

**Si `$ARGUMENTS` est fourni** : l'interpréter comme le nom d'une plage (passer à l'étape 2).

**Si vide** : demander (AskUserQuestion) :

**Je veux…** (header: "Action") :
- Voir le statut de toutes les plages
- Consulter une plage en particulier

---

### Étape 1 — Vue d'ensemble de toutes les plages

1. Appeler `mcp__edb-noumea__on_load`.
2. Extraire le résumé (markdown) et le tableau des plages.
3. Présenter un tableau markdown avec pour chaque plage :
   - Nom de la plage
   - Statut sanitaire avec indicateur coloré : 🟢 Bonne · 🟡 Acceptable · 🔴 Mauvaise · ⚫ Insuffisante
   - Date du dernier prélèvement
4. Ajouter un résumé global : combien de plages praticables vs. déconseillées.
5. Proposer de détailler une plage spécifique (passer à l'étape 2).

---

### Étape 2 — Détail d'une plage

1. Appeler `mcp__edb-noumea__handle_dropdown` avec le `site_name` de la plage choisie.
2. Extraire et présenter :
   - **Statut actuel** : qualité de l'eau (avec indicateur 🟢/🟡/🔴)
   - **Valeurs mesurées** : taux d'E. coli et d'Entérocoques (µg/100mL)
   - **Historique récent** : tableau des derniers prélèvements (date, valeurs, statut)
3. **Avis de baignade** :
   - 🏊 **Baignade recommandée** si qualité Bonne ou Acceptable
   - ⚠️ **Baignade déconseillée** si qualité Mauvaise
   - 🚫 **Baignade interdite** si qualité Insuffisante ou données absentes
4. Expliquer brièvement ce que mesurent E. coli et Entérocoques (indicateurs de contamination fécale).

---

## Contexte

Les **Eaux De Baignade (EDB)** sont surveillées régulièrement par des prélèvements microbiologiques sur les plages de Nouméa. Les indicateurs clés sont :
- **E. coli** : bactérie indicatrice de contamination fécale d'origine humaine ou animale
- **Entérocoques intestinaux** : indicateur complémentaire de contamination fécale

Les seuils de qualité suivent la réglementation française :
- 🟢 **Bonne** : E. coli < 250 UFC/100mL, Entérocoques < 100 UFC/100mL
- 🟡 **Acceptable** : valeurs modérées
- 🔴 **Mauvaise** : dépassement significatif des seuils
- ⚫ **Insuffisante** : données insuffisantes ou valeurs très élevées

## Arguments
`$ARGUMENTS` — nom de la plage (ex: `"Anse Vata"`, `"Citrons"`, `"Lemon"`
