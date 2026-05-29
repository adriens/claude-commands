# Helia NC — Ma consommation mobile

Réponds aux questions sur la consommation personnelle du forfait mobile Helia NC (OPT-NC).
Réponds **dans la langue de la question posée**.

---

## ROUTING — Sous-commandes disponibles

Lire `$ARGUMENTS` et router vers la section correspondante.

| Argument | Action |
|---|---|
| _(vide)_ | → **TABLEAU DE BORD** complet |
| `data` | → **FOCUS DATA** : Go restants, rythme, projection data |
| `voix` | → **FOCUS VOIX** : minutes restantes, rythme, tient ? |
| `projection` | → **FOCUS PROJECTION** : data et voix tiennent-elles jusqu'au renouvellement ? |
| `recharge` | → **FOCUS RECHARGE** : dois-je recharger ? quelle recharge ? |
| `rythme` | → **FOCUS RYTHME** : je consomme normalement pour ce stade du mois ? |
| `hf` | → **FOCUS HORS-FORFAIT** : frais hors-forfait via CLI |
| `report` | → **RAPPORT MARKDOWN** : synthèse complète avec charts Mermaid, écrite dans `~/Documents/helia/conso/` |
| `expert` | → **RAPPORT EXPERT PDF** : Quarto + R + XeLaTeX — consommation, projection, heatmap, conseil forfait |

Si l'argument ne correspond à aucune commande, afficher la liste ci-dessus et produire le tableau de bord complet.

---

## FOCUS DATA

> Déclenché par `/helia-conso data`

```sql
SELECT data_left_go, data_initial_go,
       ROUND((data_initial_go - data_left_go) / data_initial_go * 100, 1) AS pct_conso,
       days_renewal
FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;

SELECT go_par_jour, data_tient_jours, jours_renouvellement, statut FROM v_projection;

SELECT jour, data_conso_mo, data_restant_go FROM v_daily_conso ORDER BY jour DESC LIMIT 7;
```

Format de réponse :

```
📶 Data — Forfait M X Go

  Restant    : X,XXX Go (XX% consommé)
  Rythme     : X,XX Go/jour
  Projection : tient ~X,X jours → OK ✅ / ALERTE 🔴
  Renouvellement dans N jours

📅 Conso par jour (7 derniers jours)
  JJ/MM : X Mo
  ...
```

Verdict : appliquer les seuils du TABLEAU DE BORD. Si `statut = 'ALERTE'`, suggérer une recharge.

---

## FOCUS VOIX

> Déclenché par `/helia-conso voix`

```sql
SELECT voice_initial_sec, voice_left_sec,
       ROUND(voice_left_sec * 100.0 / voice_initial_sec, 1) AS pct_conso,
       ROUND((voice_initial_sec - voice_left_sec) / 60.0, 1) AS restant_min,
       days_renewal
FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;

SELECT jour, voice_conso_min FROM v_voice_daily ORDER BY jour DESC LIMIT 7;

SELECT ROUND(AVG(voice_conso_sec) / 60.0, 1) AS moy_min_par_jour FROM v_voice_daily WHERE voice_conso_sec > 0;
```

Format de réponse :

```
📞 Voix — Forfait X h

  Restant    : X min X sec (XX% consommé)  🟢🟡🔴
  Rythme     : ~X,X min/jour
  Projection : tient ~X jours → OK ✅ / risque 🔴
  Renouvellement dans N jours

📅 Conso voix par jour
  JJ/MM : X,X min
  ...
```

Verdict : `voix_restante_min / moy_min_par_jour >= days_renewal` → ✅ sinon 🔴.

---

## FOCUS PROJECTION

> Déclenché par `/helia-conso projection`

```sql
SELECT data_left_go, days_renewal FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
SELECT go_par_jour, data_tient_jours, jours_renouvellement, statut FROM v_projection;
SELECT voice_initial_sec, voice_left_sec, days_renewal FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
SELECT ROUND(AVG(voice_conso_sec) / 60.0, 1) AS moy_min_par_jour FROM v_voice_daily WHERE voice_conso_sec > 0;
```

Format de réponse :

```
🏁 Projection jusqu'au renouvellement (N jours)

  📶 Data  : X,XX Go restants · X,XX Go/j · tient ~X,X jours → OK ✅ / ALERTE 🔴
  📞 Voix  : X min restantes · X,X min/j  · tient ~X jours   → OK ✅ / risque 🔴

  ⚠️ Si < 3 jours de snapshots : projection estimée, fiable dans X jours.
```

Si les deux sont OK → "Rien à faire, attends le renouvellement."
Si alerte → passer directement à la recommandation de recharge.

---

## FOCUS RECHARGE

> Déclenché par `/helia-conso recharge`

Évaluer si une recharge est nécessaire :

```sql
SELECT data_tient_jours, jours_renouvellement, statut FROM v_projection;
SELECT voice_initial_sec, voice_left_sec, days_renewal FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
SELECT ROUND(AVG(voice_conso_sec) / 60.0, 1) AS moy_min_par_jour FROM v_voice_daily WHERE voice_conso_sec > 0;
```

Logique de décision :

| Situation | Recommandation |
|---|---|
| Tout OK | 🟢 Pas besoin de recharger, renouvellement dans N jours |
| Data ALERTE uniquement | 🔴 Recharge Internet Mobile 1 Go / 24h — **400 F** |
| Voix épuisée avant renouvellement | 🔴 Recharge packagée 1h + 1 Go + SMS illim. — **1 000 F** |
| Data + Voix en alerte | 🔴 Recharge packagée 1h + 1 Go + SMS illim. — **1 000 F** |
| Besoin confort | Recharge packagée 2h + 5 Go + SMS illim. — **3 000 F** |

Canaux disponibles :
- 📱 App Helia (App Store / Google Play) — le plus rapide
- 🌐 helia.nc → "Mes démarches en ligne"
- 📞 **1013** (gratuit) — Lun–Ven 7h30–16h / Sam 7h–11h
- 🏪 46 agences / 21 revendeurs en NC

---

## FOCUS RYTHME

> Déclenché par `/helia-conso rythme`

```sql
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1) AS pct_temps_ecoule,
    MIN(days_renewal) AS jours_restants
FROM conso_snapshot;

SELECT jour, data_conso_mo FROM v_daily_conso ORDER BY jour DESC LIMIT 7;
SELECT go_par_jour FROM v_projection;
```

Format de réponse :

```
⏱ Rythme de consommation

  Data consommée : XX%
  Temps écoulé   : XX%
  Écart          : XX points → 🟢 en avance / 🟡 dans les clous / 🔴 dépasse

  Rythme moyen : X,XX Go/jour
  Jour le plus gourmand : JJ/MM — X Mo
```

Règles de verdict :
- `pct_data_conso < pct_temps_ecoule - 10` → 🟢 En avance
- `ABS(pct_data_conso - pct_temps_ecoule) <= 10` → 🟡 Dans les clous
- `pct_data_conso > pct_temps_ecoule + 10` → 🔴 Consomme trop vite

---

## FOCUS HORS-FORFAIT

> Déclenché par `/helia-conso hf`

DuckDB ne stocke pas `amountHF`. Requêter le CLI :

```bash
helia status --json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('amountHF', 0))"
```

- `amountHF == 0` → ✅ "Aucun frais hors-forfait."
- `amountHF > 0` → 🔴 "Tu as **X F** de hors-forfait. Ces frais seront régularisés au renouvellement."

---

## FOCUS REPORT

> Déclenché par `/helia-conso report`

Exécuter toutes les requêtes ci-dessous, puis produire **un seul bloc Markdown complet** à copier-coller.

### Requêtes à exécuter

```sql
-- État instantané
SELECT data_left_go, data_initial_go,
       ROUND((data_initial_go - data_left_go) / data_initial_go * 100, 1) AS pct_data_conso,
       ROUND((data_initial_go - data_left_go), 3) AS data_conso_go,
       voice_initial_sec, voice_left_sec,
       ROUND(voice_left_sec * 100.0 / voice_initial_sec, 1) AS pct_voix_conso,
       ROUND((voice_initial_sec - voice_left_sec) / 60.0, 1) AS voix_restante_min,
       ROUND(voice_left_sec / 60.0, 1) AS voix_conso_min,
       days_renewal
FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;

-- Projection
SELECT go_par_jour, data_tient_jours, jours_renouvellement, statut FROM v_projection;

-- Rythme
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1) AS pct_temps_ecoule
FROM conso_snapshot;

-- Conso data par jour
SELECT jour, ROUND(data_conso_mo, 1) AS mo, data_restant_go FROM v_daily_conso ORDER BY jour ASC;

-- Conso voix par jour
SELECT jour, ROUND(voice_conso_min, 1) AS min FROM v_voice_daily ORDER BY jour ASC;

-- Tendance data restante
SELECT CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
       ROUND(MIN(data_left_go), 3) AS data_restant_go
FROM conso_snapshot
GROUP BY 1 ORDER BY 1 ASC;

-- Historique + horodatage de génération
SELECT MIN(timestamp) + 11 * INTERVAL '1 hour' AS depuis,
       COUNT(*) AS nb_snapshots,
       MAX(timestamp) + 11 * INTERVAL '1 hour' AS dernier_snapshot_local
FROM conso_snapshot;
```

### Format du document exporté

Produire **exactement** ce document, en substituant toutes les valeurs réelles :

````markdown
# 📱 Helia NC — Rapport de consommation
> Généré le JJ/MM/AAAA à HH:MM (heure NC) · Forfait M X Go · Renouvellement dans N jours

---

## 🔍 Analyse

> Rédiger 2-3 phrases en langage naturel qui résument la situation globale : ce qui va bien, ce qui mérite attention, et ce qu'il faut faire (ou ne pas faire). Ton direct, pas de jargon.
>
> Exemples de formulations selon la situation :
> - Tout OK → "Ton forfait est bien géré ce mois-ci : tu n'as consommé que X% de ta data pour X% du temps écoulé. Rien à faire, attends le renouvellement dans N jours."
> - Alerte data → "Ta data s'épuise plus vite que prévu : X% consommé pour X% du temps écoulé. À ce rythme tu seras à sec dans X jours, avant le renouvellement. Une recharge s'impose."
> - Alerte voix → "Ta data est tranquille mais tu as presque épuisé tes minutes (X% consommé). Il te reste X min pour N jours — évite les longs appels d'ici le renouvellement."
> - Double alerte → "Situation tendue : data et voix sont toutes les deux dans le rouge. Envisage une recharge packagée (1h + 1 Go — 1 000 F) pour finir le mois sereinement."

---

## 📊 Synthèse

| Indicateur | Valeur | Verdict |
|---|---|---|
| 📶 Data restante | X,XXX Go / X Go (XX%) | 🟢🟡🔴 |
| 📞 Voix restante | X min / Xh (XX% consommé) | 🟢🟡🔴 |
| 💬 SMS | Illimité | ✅ |
| ⏱ Rythme data | XX% consommé · XX% temps écoulé | 🟢🟡🔴 |
| 🏁 Projection data | tient ~X,X jours / N restants | ✅ OK / 🔴 ALERTE |
| 🔄 Renouvellement | Dans N jours | — |

---

## 📶 Data

```mermaid
pie title 📶 Data — Forfait M X Go
    "Consommé (X,XXX Go)" : XX.X
    "Restant (X,XXX Go)" : XX.X
```

```mermaid
xychart-beta
    title "Conso data par jour (Mo)"
    x-axis ["JJ/MM", "JJ/MM", ...]
    bar [X, X, ...]
```

```mermaid
xychart-beta
    title "Data restante au fil des jours (Go)"
    x-axis ["JJ/MM", "JJ/MM", ...]
    line [X.XXX, X.XXX, ...]
```

---

## 📞 Voix

```mermaid
pie title 📞 Voix — Forfait X h
    "Consommé (X min)" : XX.X
    "Restant (X min)" : XX.X
```

```mermaid
xychart-beta
    title "Conso voix par jour (min)"
    x-axis ["JJ/MM", "JJ/MM", ...]
    bar [X.X, X.X, ...]
```

---

## 🏁 Projection

| | Data | Voix |
|---|---|---|
| Restant | X,XXX Go | X min |
| Rythme moyen | X,XX Go/jour | X,X min/jour |
| Tient jusqu'au renouvellement | ✅ Oui (X,X j) / 🔴 Non | ✅ Oui / 🔴 Non |

---

## ⏱ Rythme

```mermaid
xychart-beta
    title "Rythme : % data consommée vs % temps écoulé"
    x-axis ["% data conso", "% temps écoulé"]
    bar [XX.X, XX.X]
```

---

*Données issues de `~/.config/helia/data/helia.db` · X snapshots depuis le JJ/MM/AAAA · dernier snapshot : JJ/MM/AAAA HH:MM*
````

### Écriture du fichier

```bash
# Initialisation complète de l'arborescence helia (première fois)
mkdir -p ~/Documents/helia/conso ~/Documents/helia/reseau
[ ! -f ~/Documents/helia/helia.svg ] && \
  curl -sL -o ~/Documents/helia/helia.svg \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/assets/logos/helia.svg
[ ! -f ~/Documents/helia/README.md ] && \
  curl -sL -o ~/Documents/helia/README.md \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/README_helia_dir.md

# Télécharger le template depuis le repo (source de vérité)
DATE_NC=$(date -u -d '+11 hours' '+%Y-%m-%d' 2>/dev/null || date -u -v+11H '+%Y-%m-%d')
FICHIER=~/Documents/helia/conso/${DATE_NC}_rapport_conso.md
curl -sL -o "$FICHIER" \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/src/rapport_conso_template.md
```

Le template contient les placeholders suivants — **tous doivent être remplacés** avant de confirmer à l'utilisateur :

| Placeholder | Contenu attendu |
|---|---|
| `HELIA_CONSO_HEADER` | `> Généré le JJ/MM/AAAA à HH:MM (heure NC) · Forfait M X Go · Renouvellement dans N jours` |
| `HELIA_CONSO_ANALYSE` | Texte narratif contextuel (voir ci-dessous) |
| `HELIA_CONSO_TABLE` | Tableau Markdown synthèse complet (6 lignes) |
| `HELIA_CONSO_DATA` | Section data : pie + bar conso/jour + line tendance |
| `HELIA_CONSO_VOIX` | Section voix : pie + bar conso/jour |
| `HELIA_CONSO_PROJECTION` | Tableau projection data + voix |
| `HELIA_CONSO_RYTHME` | Section rythme : xychart-beta |
| `HELIA_CONSO_FOOTER` | `*Données issues de ... · X snapshots depuis le JJ/MM/AAAA · dernier : ...*` |

#### Injection via uv

```bash
uv run python - <<'PYEOF'
import re

fichier = f"{FICHIER}"  # chemin défini ci-dessus

replacements = {
    "HELIA_CONSO_HEADER":     "...",  # généré depuis les données
    "HELIA_CONSO_ANALYSE":    "...",  # texte narratif ci-dessous
    "HELIA_CONSO_TABLE":      "...",
    "HELIA_CONSO_DATA":       "...",
    "HELIA_CONSO_VOIX":       "...",
    "HELIA_CONSO_PROJECTION": "...",
    "HELIA_CONSO_RYTHME":     "...",
    "HELIA_CONSO_FOOTER":     "...",
}

with open(fichier, "r") as f:
    content = f.read()

for placeholder, value in replacements.items():
    content = content.replace(placeholder, value)

with open(fichier, "w") as f:
    f.write(content)
PYEOF
```

> **Ne jamais écrire le rapport directement avec le Write tool.** Toujours passer par le template + injection.

Confirmer : `✅ Rapport écrit dans ~/Documents/helia/conso/YYYY-MM-DD_rapport_conso.md`

### Analyse contextuelle — texte narratif à générer

Rédiger **2 à 4 paragraphes en français**, ton direct, pour `HELIA_CONSO_ANALYSE` :

1. **Bilan global** — data et voix : quel % consommé, verdict 🟢🟡🔴 vs stade du mois
2. **Rythme** — en avance, dans les clous, ou en dépassement ? Comparaison % data / % temps écoulé
3. **Projection** — la data va-t-elle tenir jusqu'au renouvellement ? La voix ?
4. **Recommandation** (si 🟡 ou 🔴) — action concrète : réduire, recharger, attendre le renouvellement

Si l'historique est < 3 jours, le signaler et baser la projection sur le débit manuel
(`(data_initial - data_left) / jours_ecoules * jours_restants`).

### Règles de construction des charts

- **Axes x** : utiliser le format `JJ/MM` pour les dates (convertir avec UTC+11)
- **Valeurs à 0** : les inclure pour préserver l'échelle temporelle
- **Pie** : arrondir à 1 décimale, les deux segments doivent totaliser 100
- **xychart-beta** : limiter à 10 points max — si plus, regrouper par semaine
- Si une vue est vide (pas encore de données), remplacer le chart par `> Pas encore assez de données.`

---

## FOCUS EXPERT

> Déclenché par `/helia-conso expert`

Rapport **consommation PDF** destiné au client final : forfait, projection, heatmap d'usage, conseil d'offre.
Même pipeline que `/helia-reseau expert` — template Quarto versionné dans le repo.

### Prérequis à vérifier avant de générer

```bash
quarto --version
xelatex --version
Rscript -e "packageVersion('duckdb')"
Rscript -e "packageVersion('ggplot2')"
Rscript -e "packageVersion('kableExtra')"
Rscript -e "packageVersion('scales')"
Rscript -e "packageVersion('dplyr')"
Rscript -e "packageVersion('gridExtra')"
```

Si un package manque :
```r
install.packages(c("duckdb", "ggplot2", "kableExtra", "scales", "dplyr", "gridExtra"))
```

### Écriture et compilation

```bash
# Initialisation complète de l'arborescence helia
mkdir -p ~/Documents/helia/conso ~/Documents/helia/reseau
[ ! -f ~/Documents/helia/helia.svg ] && \
  curl -sL -o ~/Documents/helia/helia.svg \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/assets/logos/helia.svg
[ ! -f ~/Documents/helia/helia.png ] && \
  rsvg-convert -f png -w 400 ~/Documents/helia/helia.svg \
    -o ~/Documents/helia/helia.png 2>/dev/null || true
[ ! -f ~/Documents/helia/README.md ] && \
  curl -sL -o ~/Documents/helia/README.md \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/README_helia_dir.md

# Télécharger le template QMD depuis le repo (source de vérité)
DATE_NC=$(date -u -d '+11 hours' '+%Y-%m-%d' 2>/dev/null || date -u -v+11H '+%Y-%m-%d')
QMD=~/Documents/helia/conso/${DATE_NC}_rapport_conso_expert.qmd
curl -sL -o "$QMD" \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/src/rapport_conso_expert_template.qmd

# Remplacer toute date résiduelle du template par la date du jour
sed -i "s/[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}_rapport_conso_expert/${DATE_NC}_rapport_conso_expert/g" "$QMD"

quarto render "$QMD" --to pdf
```

> Le template QMD est versionné dans le repo.
> Pour toute modification, éditer `docs/skills/helia-conso/src/rapport_conso_expert_template.qmd`.

### Étape 0 — Demander l'alias de la ligne avant de générer

**Avant toute génération**, demander à l'utilisateur :

> "Je vais générer le rapport pour **Tonton Marcel** — je laisse comme ça ou tu veux
> mettre un autre alias pour le propriétaire de la ligne ?
> (ex : `Maman`, `Bureau`, `Jean DUPONT — 98 XX XX XX`)"

- Si l'utilisateur répond : utiliser sa réponse telle quelle.
- Si l'utilisateur ne précise pas ou dit "par défaut" : utiliser `Tonton Marcel`.
- Ne jamais mettre le nom de l'auteur du rapport (Adrien SALES) comme valeur par défaut —
  l'auteur figure déjà dans le pied de page du rapport.

> **Distinction importante**
> - **Auteur du rapport** = Adrien SALES — toujours présent dans le footer (ne pas modifier)
> - **Propriétaire de la ligne** = alias libre, affiché sur la page de couverture — peut être
>   un prénom, un alias métier, un numéro de ligne, utile en contexte flotte d'entreprise

---

### Analyse contextuelle — injection obligatoire avant le rendu

Le template contient deux marqueurs à remplacer :
- `HELIA_CONSO_ANALYSE_PLACEHOLDER` — analyse contextuelle (texte Markdown narratif)
- `HELIA_OWNER_PLACEHOLDER` — alias du propriétaire de la ligne (saisi à l'étape 0)

```bash
uv run python - <<'PYEOF'
analyse = """[texte narratif généré ci-dessous]"""
proprietaire = "[alias saisi par l'utilisateur à l'étape 0]"

with open(qmd_path, "r", encoding="utf-8") as f:
    content = f.read()
content = content.replace("HELIA_CONSO_ANALYSE_PLACEHOLDER", analyse)
content = content.replace("HELIA_OWNER_PLACEHOLDER", proprietaire)
with open(qmd_path, "w", encoding="utf-8") as f:
    f.write(content)
PYEOF
```

#### Requêtes DuckDB pour l'analyse contextuelle

```sql
-- Snapshot instantané
SELECT data_left_go, data_initial_go, voice_initial_sec, voice_left_sec, days_renewal
FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;

-- Rythme
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1) AS pct_temps_ecoule
FROM conso_snapshot;

-- Conso journalière (7 derniers jours)
SELECT jour, data_conso_mo FROM v_daily_conso ORDER BY jour DESC LIMIT 7;
```

#### Texte narratif à générer

3 à 4 paragraphes en français, ton direct :
1. **Bilan** — data et voix restantes, verdict 🟢🟡🔴
2. **Rythme** — en avance / dans les clous / dépassement
3. **Projection** — la data tient-elle jusqu'au renouvellement ?
4. **Recommandation** — recharger / changer de forfait / rien à faire

> Texte Markdown standard — Quarto le convertit en LaTeX automatiquement.

**Vérifications avant rendu :**
- `date: today` → Quarto injecte la date automatiquement ✅
- Toutes les métriques (data, voix, projection) → R + DuckDB en temps réel ✅
- `grep '[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}' "$QMD"` → aucune date hardcodée ✅

Confirmer : `✅ PDF généré : ~/Documents/helia/conso/YYYY-MM-DD_rapport_conso_expert.pdf`

---

## TABLEAU DE BORD — Synthèse par défaut

**Si l'utilisateur ne pose pas de question précise, ou demande un état général, produire systématiquement ce tableau de bord complet.**

### Règle fondamentale : DuckDB en priorité sur le CLI

**Pour toutes les questions de conso (data, voix, SMS, rythme, projection) : requêter DuckDB d'abord.**
Le CLI (`helia status --json`) n'est un recours que si la DB est vide ou pour confirmer une donnée temps réel.

### Requêtes DuckDB à exécuter

```sql
-- État instantané
SELECT data_left_go, data_initial_go, voice_initial_sec, voice_left_sec,
       days_renewal FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;

-- Rythme data vs temps écoulé
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1)                AS pct_temps_ecoule
FROM conso_snapshot;

-- Projection (fiable seulement si ≥ 3 jours de snapshots)
SELECT go_par_jour, data_tient_jours, jours_renouvellement, statut FROM v_projection;

-- Disponibilité de l'historique
SELECT MIN(timestamp) + 11 * INTERVAL '1 hour' AS premier_snapshot_local,
       COUNT(*) AS nb_snapshots,
       COUNT(DISTINCT CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE)) AS nb_jours
FROM conso_snapshot;
```

> ⚠️ **v_projection non fiable si < 3 jours de données** : le débit calculé sera aberrant (ex: 17 Go/j).
> Dans ce cas, calculer manuellement : `(data_initial - data_left) / jours_ecoules * jours_restants`.
> Toujours signaler explicitement si la projection est non fiable.

### Format de sortie — tableau synthèse (emojis, screenshot-ready)

Produire **exactement** ce format, en substituant les valeurs réelles :

```
📱 Helia NC — Forfait M X Go   🔄 Renouvellement dans N jours

┌──────────┬──────────────────────────────┬───────────────────────┐
│          │ Situation                    │ Verdict               │
├──────────┼──────────────────────────────┼───────────────────────┤
│ 📶 Data  │ X,XX Go restants / X Go      │ 🟢🟡🔴 + message     │
│          │ XX% consommé                 │                       │
├──────────┼──────────────────────────────┼───────────────────────┤
│ 📞 Voix  │ Xh XX restantes / Xh totales │ 🟢🟡🔴 + message     │
│          │ XX% consommé                 │                       │
├──────────┼──────────────────────────────┼───────────────────────┤
│ 💬 SMS   │ Illimité / X restants        │ ✅ ou ⚠️              │
├──────────┼──────────────────────────────┼───────────────────────┤
│ 💸 HF    │ X F ou Aucun                 │ ✅ ou 🔴              │
├──────────┼──────────────────────────────┼───────────────────────┤
│ ⏱ Rythme │ XX% data / XX% temps écoulé  │ 🟢 en avance          │
│          │                              │ 🟡 dans les clous     │
│          │                              │ 🔴 dépasse            │
└──────────┴──────────────────────────────┴───────────────────────┘

🏁 Projection jusqu'au renouvellement (N jours)
  📶 Data  : ~XXX Mo prévus sur les N jours → OK ✅ / ALERTE 🔴
  📞 Voix  : tient ~X jours → OK ✅ / épuisée dès le prochain appel 🔴
  ⚠️ Si historique < 3 jours : "(projection estimée — fiable dans X jours)"
```

### Règles de verdict

| Indicateur | 🟢 OK | 🟡 Attention | 🔴 Alerte |
|---|---|---|---|
| Data % consommé | < 60 % | 60–80 % | > 80 % |
| Voix % consommé | < 60 % | 60–80 % | > 80 % |
| Projection data | tient > jours restants | tient jours restants ±1 | épuisée avant renouvellement |
| Projection voix | tient > jours restants | tient jours restants ±1 | épuisée avant renouvellement |
| Hors-forfait | 0 F | — | > 0 F |

### Focus automatique sur ce qui est problématique

Après le tableau, **mettre en avant uniquement les indicateurs en 🟡 ou 🔴** avec :
- Le chiffre exact et ce que ça signifie concrètement
- Une action recommandée si 🔴 (recharge, économie, attente renouvellement)
- Rester silencieux sur les indicateurs 🟢 (ne pas les répéter)

### Données historisées — toujours mentionner et exploiter

**À la fin de chaque réponse**, signaler la disponibilité de l'historique DuckDB :

```
📈 Historique disponible depuis le JJ/MM/AAAA
   Tu peux me demander : tendance de conso, jour le plus gourmand,
   rythme semaine vs semaine, évolution de ta voix...
```

**Si l'historique est suffisant (≥ 3 jours de snapshots)**, enrichir le tableau avec :

```sql
-- Conso moyenne journalière réelle (remplace la projection approximative)
SELECT * FROM v_daily_conso ORDER BY jour DESC LIMIT 7;

-- Rythme : % data consommée vs % temps écoulé
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1)                AS pct_temps_ecoule
FROM conso_snapshot;
```

Et ajouter une ligne **⏱ Rythme** dans le tableau :

| ⏱ Rythme | X% data consommée pour X% du temps écoulé | en avance 🟢 / dans les clous 🟡 / dépasse 🔴 |

Si l'historique est trop récent (< 3 jours), le signaler clairement et indiquer quand les projections seront fiables.

### Options de recharge (à afficher si 🔴 sur voix ou data)

| Canal | Détail |
|---|---|
| 📱 App Helia | Recharge packagée **1h + 1 Go + SMS illim. — 1 000 F** |
| 📱 App Helia | Recharge Internet Mobile **1 Go / 24h — 400 F** |
| 🌐 helia.nc | "Mes démarches en ligne" |
| 📞 1013 | Gratuit — Lun–Ven 7h30–16h / Sam 7h–11h |

---

## QUESTIONS FRÉQUENTES — Consommateur lambda

Répondre directement et simplement à ces questions du quotidien. Pas de jargon, réponse en 1-2 phrases + verdict 🟢🟡🔴.

### "Est-ce que ma data va tenir jusqu'au renouvellement ?"

```sql
SELECT data_tient_jours, jours_renouvellement, statut FROM v_projection;
```

- `statut = 'OK'` → ✅ "Oui, ta data tient encore ~X jours pour N jours restants."
- `statut = 'ALERTE'` → 🔴 "Non, elle s'épuise dans ~X jours, renouvellement dans N jours. Envisage une recharge."
- Si < 3 jours de snapshots → calculer manuellement et signaler l'incertitude.

### "Est-ce que mes minutes vont tenir ?"

```sql
SELECT voice_initial_sec, voice_left_sec, days_renewal FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
SELECT SUM(voice_conso_sec) / COUNT(DISTINCT jour) AS moy_sec_par_jour FROM v_voice_daily;
```

Calculer : `voix_restante_sec = voice_initial_sec - voice_left_sec`, puis `voix_tient_jours = voix_restante_sec / moy_sec_par_jour`.
- `voix_tient_jours >= days_renewal` → ✅ "Oui, il te reste ~X min pour N jours."
- `voix_tient_jours < days_renewal` → 🔴 "Non, tu risques de manquer de minutes. Il reste X min pour N jours."

### "Dans combien de jours je renouvelle ?"

```sql
SELECT days_renewal FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
```

Répondre simplement : "Ton forfait se renouvelle dans **N jours**."

### "Combien de Go il me reste ?"

```sql
SELECT data_left_go, data_initial_go FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
```

"Il te reste **X,XX Go** sur X Go (XX% consommé)."

### "Combien de minutes il me reste ?"

```sql
SELECT voice_initial_sec, voice_left_sec FROM conso_snapshot ORDER BY timestamp DESC LIMIT 1;
```

Calculer `voix_restante_sec = voice_initial_sec - voice_left_sec`, convertir en min/sec.
"Il te reste **X min X sec** de voix (XX% consommé)."

### "J'ai consommé combien depuis le début du mois ?"

```sql
SELECT ROUND(MAX(data_initial_go) - MIN(data_left_go), 3) AS data_conso_go,
       ROUND((MAX(voice_left_sec)) / 60.0, 1) AS voix_conso_min
FROM conso_snapshot;
```

"Tu as consommé **X,XXX Go** de data et **X min** de voix ce mois-ci."

### "Je consomme trop vite ou j'ai de la marge ?"

```sql
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1) AS pct_temps_ecoule
FROM conso_snapshot;
```

- `pct_data_conso < pct_temps_ecoule - 10` → 🟢 "Tu es très en avance : X% de data consommé pour X% du temps écoulé."
- `ABS(pct_data_conso - pct_temps_ecoule) <= 10` → 🟡 "Tu es dans les clous : rythme conforme au temps écoulé."
- `pct_data_conso > pct_temps_ecoule + 10` → 🔴 "Tu consommes trop vite : X% de data pour seulement X% du temps écoulé."

### "Quel jour j'ai le plus consommé ?"

```sql
SELECT jour, data_conso_mo FROM v_daily_conso ORDER BY data_conso_mo DESC LIMIT 1;
```

"Le jour le plus gourmand était le **JJ/MM** avec **X Mo** consommés."
Si toutes les valeurs sont à 0 → "Pas encore assez de données pour identifier un pic."

### "Quel est mon rythme moyen ?"

```sql
SELECT go_par_jour FROM v_projection;
SELECT SUM(voice_conso_min) / COUNT(DISTINCT jour) AS moy_voix_min_par_jour FROM v_voice_daily;
```

"Tu consommes en moyenne **X,XX Go/jour** de data et **X,X min/jour** de voix."

### "Est-ce que j'ai des frais hors-forfait ?"

Le champ `amountHF` n'est pas stocké dans DuckDB. Lancer le CLI :
```bash
helia status --json | python3 -c "import sys,json; d=json.load(sys.stdin); print(d.get('amountHF', 0))"
```
- `amountHF == 0` → ✅ "Aucun frais hors-forfait."
- `amountHF > 0` → 🔴 "Tu as **X F** de hors-forfait."

### "Est-ce que je dois recharger ?"

Logique de décision :

| Situation | Recommandation |
|---|---|
| `v_projection.statut = 'OK'` ET voix tient | 🟢 Non, attends le renouvellement |
| `v_projection.statut = 'ALERTE'` | 🔴 Oui → recharge Internet Mobile 1 Go / 24h (400 F) ou packagée 1h+1Go (1 000 F) |
| Voix épuisée avant renouvellement | 🔴 Oui → recharge packagée 1h+1Go+SMS illim. (1 000 F) |
| `amountHF > 0` | ⚠️ Signaler — pas de recharge recommandée, attendre le renouvellement |

### "Quelle recharge me conseilles-tu ?"

| Besoin | Recharge recommandée | Prix |
|---|---|---|
| Data seulement (urgence 24h) | Internet Mobile 1 Go / 24h | 400 F |
| Data + voix + SMS | Packagée 1h + 1 Go + SMS illim. / 30j | 1 000 F |
| Data + voix + SMS (plus confortable) | Packagée 2h + 5 Go + SMS illim. / 30j | 3 000 F |

Disponible via : 📱 App Helia · 🌐 helia.nc · 📞 1013 · 🏪 agence

---

## DOMAINE 1 — Ma consommation (data & voix)

### Router la commande

| Question | Commande |
|---|---|
| Conso actuelle (data, voix, SMS, jours) | `helia status --json` |
| Profil / forfait souscrit | `helia profile --json` |
| Historique / tendance / évolution | Requête DuckDB `v_daily_conso` |
| Projection / est-ce que ça va tenir ? | `helia status --json` + `SELECT * FROM v_projection` |

Le CLI est dans le PATH ou à `/home/adriens/Github/helia/cli/helia`.

### Calculer les valeurs (`helia status --json`)

**ATTENTION** : `voiceLeft` = voix **consommée** (pas restante).

```
voix_restante_sec  = voiceInitial - voiceLeft
voix_pct_consommé  = voiceLeft / voiceInitial * 100

jours_écoulés      = 30 - daysRenewal
data_consommée     = dataInitial - dataLeft
débit_data_jour    = data_consommée / jours_écoulés        (Go/jour)
data_tient_jours   = dataLeft / débit_data_jour            (jours)
```

### Historique DuckDB — vues disponibles

**Toujours ouvrir en lecture seule** pour éviter les conflits avec les widgets :
```bash
duckdb -readonly ~/.config/helia/data/helia.db "SELECT ..."
```
```python
con = duckdb.connect('/home/adriens/.config/helia/data/helia.db', read_only=True)
```

**Schéma de la table principale :**
```sql
-- conso_snapshot (écrit toutes les 5 min si valeurs changées)
timestamp TIMESTAMP, data_left_go DOUBLE, data_initial_go DOUBLE,
voice_left_sec INTEGER,   -- secondes CONSOMMÉES (pas restantes !)
voice_initial_sec INTEGER, days_renewal INTEGER
```

**Vues prêtes à l'emploi :**

```sql
-- Conso par jour (LAG sur la veille, en Go et Mo)
SELECT * FROM v_daily_conso;
SELECT * FROM v_daily_conso WHERE jour >= CURRENT_DATE - 7;

-- Macro paramétrée (équivalent ci-dessus)
SELECT * FROM get_conso_since(7);

-- Conso voix par jour
SELECT * FROM v_voice_daily ORDER BY jour DESC;

-- Projection : débit 7j + jours avant épuisement vs renouvellement
SELECT * FROM v_projection;
-- → go_par_jour | data_tient_jours | jours_renouvellement | statut (OK/ALERTE)
```

**Requêtes ad hoc utiles :**

```sql
-- Rythme : % data consommée vs % temps écoulé
SELECT
    ROUND((1 - MIN(data_left_go)/MAX(data_initial_go))*100, 1) AS pct_data_conso,
    ROUND((30 - MIN(days_renewal))/30.0*100, 1) AS pct_temps_ecoule
FROM conso_snapshot;

-- 10 derniers snapshots
SELECT timestamp,
       data_left_go,
       ROUND((voice_initial_sec - voice_left_sec)/3600.0, 2) AS voix_restante_h,
       days_renewal
FROM conso_snapshot ORDER BY timestamp DESC LIMIT 10;
```

### Alertes proactives

| Seuil | Action |
|---|---|
| Data `>= 80%` consommée | 🟡 Attention |
| `v_projection.statut = 'ALERTE'` | 🔴 Data épuisée avant renouvellement — suggérer recharge |
| Voix `>= 80%` consommée | 🔴 Alerte |
| `amountHF > 0` | Signaler le montant |

**Comment recharger :**
- 📱 App Helia (App Store / Google Play)
- 🌐 helia.nc → "Mes démarches en ligne"
- 🏪 46 agences / 21 revendeurs en NC
- 📞 **1013** (gratuit) Dépannage — Lun–Ven 7h30–16h / Sam 7h–11h
- 📞 **1052** (gratuit) Perte/vol — Tous les jours 6h–21h

---

## DOMAINE 2 — Offres & Forfaits

### Forfaits M (abonnement mensuel)

| Forfait | Data | Tarif TTC |
|---|---|---|
| M 2 Go | 2 Go | 1 000 F |
| M 10 Go | 10 Go | 3 000 F |
| M 30 Go | 30 Go | 6 000 F |
| M 100 Go | 100 Go | 10 000 F |

- Après épuisement : Internet illimité à débit réduit (pas de coupure)
- Recharge data possible par SMS surtaxé
- **Changement de forfait** : gratuit, sans frais après le 1er mois → [formulaire en ligne](https://helia.nc/formulaire-changer-de-forfait)
  - Informations requises : nom, prénom, date/lieu de naissance, numéro concerné, date souhaitée, adresse de facturation
  - Remboursement au prorata de l'ancien forfait · réinitialisation mensuelle du nouveau
  - Impossible avant le mois suivant la souscription · perte du report de communication et du TOP UP en cours

### Kits prépayés Liberté (sans abonnement)
- 15 Go et 30 Go disponibles
- Crédit et data rechargeables via app, agence ou revendeur

### Autres offres
- Forfaits objets connectés (IoT)
- eSIM disponible
- Pass Internet Voyage (depuis l'app)
- Internet fixe / Fibre 1 Gbit/s / Téléphonie fixe

---

## DOMAINE 3 — Application Helia

Téléchargement gratuit — [App Store](https://apps.apple.com) / [Google Play](https://play.google.com)
- Android 5.1+ · iOS 13+

### Ce qu'on peut faire depuis l'app

| Forfait M | Kit Liberté | Objets connectés |
|---|---|---|
| Suivi data/voix/SMS temps réel | Suivi data/voix/SMS | Suivi data/SMS |
| Consulter hors-forfait | Recharger crédit Liberté | Consulter hors-forfait |
| Recharger data mobile | Acheter recharges internet | Recharger data |
| Pass Internet Voyage | Pass Internet Voyage | Pass Internet Voyage |
| Date de renouvellement | Expiration numéro & crédit | Date de renouvellement |

---

## DOMAINE 4 — Assistance & Contact

### Numéros utiles

| Numéro | Service | Horaires |
|---|---|---|
| **1000** (gratuit) | Assistance générale | Lun–Ven 7h30–15h45 |
| **1013** (gratuit) | Dépannage lignes & Internet | Lun–Ven 7h30–16h · Sam 7h–11h |
| **1052** (gratuit) | Perte / vol de téléphone | Tous les jours 6h–21h |

### Agences sans rendez-vous (Lun & Mar, 7h45–10h30 et 12h30–15h)
- Quartier Latin (siège)
- Dumbéa Panda
- Pont des Français
- Ducos

**46 agences + 21 revendeurs** en Nouvelle-Calédonie → [Trouver un point de vente](https://helia.nc/trouver-un-point-de-vente)

### FAQ & aide en ligne
Plus de 40 articles couvrant : réseau, SMS, changement de forfait, paiement de facture, compatibilité appareil, roaming/itinérance.
→ [helia.nc/assistance](https://helia.nc/assistance)

### Démarches en ligne
Paiement facture · déménagement · changement de forfait · résiliation · changement de titulaire
→ [helia.nc](https://helia.nc/) > "Mes démarches en ligne"

---

## DOMAINE 5 — Réclamations

**3 canaux disponibles :**

1. **Formulaire en ligne** (recommandé) → [helia.nc/reclamations](https://helia.nc/reclamations)
2. **Email** : reclamation@helia.nc
3. **Courrier** : Helia – Service Réclamations, 2 rue Paul Montchovet, 98800 Nouméa

**Types de réclamations acceptées :**
- Insatisfaction services mobiles, fixes ou Internet
- Erreur de facturation / anomalie administrative
- Problèmes techniques persistants (mobile, fibre, ADSL, fixe)

**Traitement :**
- Accusé de réception automatique avec numéro de dossier
- Réponse par email (possibilité de suivre en répondant directement)
- Données conservées 3 ans

---

## Arbre de décision

```
Question reçue
│
├─ Données temps réel (data, voix, SMS, jours) ?
│   └─ → DOMAINE 1 : helia status --json + calculs DuckDB
│
├─ Quel forfait choisir / combien ça coûte ?
│   └─ → DOMAINE 2 : tableau des forfaits M + lien formulaire changement
│
├─ Comment recharger / que peut faire l'app ?
│   └─ → DOMAINE 3 : fonctionnalités app + lien téléchargement
│
├─ Problème technique / joindre le support ?
│   └─ → DOMAINE 4 : numéros (1000/1013/1052) + horaires + agences
│
└─ Erreur de facture / insatisfaction / litige ?
    └─ → DOMAINE 5 : reclamation@helia.nc + formulaire + courrier
```

> Pour les questions sur la **qualité du réseau, les maintenances ou la latence API** → utiliser `/helia-reseau`

---

## Notes techniques

- Token API : `~/.config/helia/token`
- CLI : `/home/adriens/Github/helia/cli/helia`
- DuckDB : `~/.config/helia/data/helia.db` — **toujours `read_only=True`** pour les lectures
- Python venv : `/home/adriens/Github/helia/.venv/bin/python`
- UTC offset NC : **+11h** stocké dans `settings` (clé `utc_offset_hours`)
- `voiceLeft` = voix **consommée** (API mal nommée) — restante = `voiceInitial - voiceLeft`
- `conso_snapshot` : écrit si valeurs changées (toutes les 5 min)
- Widgets refresh : toutes les **5 min** (helia-widget et helia-status)
