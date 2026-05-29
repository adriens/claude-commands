# Helia NC — Qualité du réseau mobile

Réponds aux questions sur la qualité du réseau mobile Helia NC : latence API, maintenances, incidents, disponibilité.
Réponds **dans la langue de la question posée**.

---

## ROUTING — Sous-commandes disponibles

Lire `$ARGUMENTS` et router vers la section correspondante.

| Argument | Action |
|---|---|
| _(vide)_ | → **TABLEAU DE BORD** réseau complet |
| `latence` | → **FOCUS LATENCE** : latence actuelle, P95, profil horaire |
| `maintenance` | → **FOCUS MAINTENANCE** : services en cours, incidents, état du réseau |
| `dispo` | → **FOCUS DISPONIBILITÉ** : taux de dispo global et par jour |
| `heatmap` | → **FOCUS HEATMAP** : latence par heure × jour de semaine |
| `report` | → **RAPPORT MARKDOWN** : synthèse complète avec charts Mermaid, écrite dans `~/Documents/helia/reseau/` |
| `expert` | → **RAPPORT EXPERT** : SLA, percentiles P50/P90/P95/P99, MTBF, indisponibilité cumulée — pour CIO, DT, OPS, support OPT-NC |

Si l'argument ne correspond à aucune commande, afficher la liste ci-dessus et produire le tableau de bord complet.

---

## QUESTIONS FRÉQUENTES — Consommateur lambda

Répondre directement en 1-2 phrases + verdict 🟢🟡🔴.

### "Est-ce que le réseau est OK en ce moment ?"

```sql
SELECT * FROM v_api_health;
```
```bash
helia maintenance --json
```

- Aucune maintenance + latence moy ≤ 500 ms + dispo ≥ 99% → 🟢 "Oui, le réseau est opérationnel."
- Latence > 1 000 ms ou timeouts → 🟡/🔴 "Le réseau répond lentement (X ms). Vérifier si une maintenance est en cours."
- `isXxxOnMaintenance = true` → 🔴 "Une maintenance est en cours sur [service]. Consulter helia.nc/etat-du-reseau."

### "Y a-t-il une maintenance en cours ?"

```bash
helia maintenance --json
```

- Tous les champs `isXxxOnMaintenance = false` → ✅ "Aucune maintenance en cours."
- Au moins un `true` → 🔴 "Maintenance active sur [service]. Consulter helia.nc/etat-du-reseau pour la zone impactée."

### "C'est quoi la meilleure heure pour naviguer ?"

```sql
SELECT heure, latence_moy_ms FROM v_hourly_latency ORDER BY latence_moy_ms ASC LIMIT 3;
```

"Les meilleures heures sont **Xh, Xh et Xh** avec ~XXX ms de latence moyenne."

### "À quelle heure le réseau est le plus chargé ?"

```sql
SELECT heure, latence_moy_ms, nb_timeouts FROM v_hourly_latency ORDER BY latence_moy_ms DESC LIMIT 3;
```

"Le réseau est le plus lent entre **Xh et Xh** (~X XXX ms). Préfère naviguer en dehors de ces créneaux."

### "Est-ce que c'est souvent en panne ?"

```sql
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
       COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
       COUNT(*) AS nb_total
FROM api_ping;
```

- dispo ≥ 99% → 🟢 "Non, la disponibilité est excellente : XX,X% sur X mesures."
- dispo 95–99% → 🟡 "Quelques coupures : XX,X% de disponibilité (X timeouts sur X mesures)."
- dispo < 95% → 🔴 "Oui, le réseau est instable : XX,X% de disponibilité seulement."

### "Y a-t-il des timeouts en ce moment ?"

```sql
SELECT * FROM v_api_health;
```

- `nb_timeouts = 0` → ✅ "Aucun timeout sur la dernière heure."
- `nb_timeouts > 0` → 🔴 "X timeout(s) sur la dernière heure (dispo X,X%)."

### "Quel est le taux de disponibilité ?"

```sql
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct
FROM api_ping;

SELECT CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
       ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct
FROM api_ping GROUP BY 1 ORDER BY 1 DESC;
```

"Disponibilité globale : **XX,X%** — détail par jour affiché ci-dessous."

---

## FOCUS LATENCE

> Déclenché par `/helia-reseau latence`

```sql
SELECT * FROM v_api_health;
SELECT * FROM v_hourly_latency ORDER BY latence_moy_ms ASC;
SELECT timestamp + 11 * INTERVAL '1 hour' AS heure_locale, response_ms, timeout
FROM api_ping WHERE timestamp >= NOW() - INTERVAL '24 hours' ORDER BY timestamp DESC;
```

Format de réponse :

```
📡 Latence API Helia NC   🕐 Dernière mesure : HH:MM

  Moy (1h)  : XXX ms    🟢🟡🔴
  P95 (1h)  : XXX ms
  Max (1h)  : XXX ms
  Timeouts  : X / N mesures

⏰ Profil horaire (meilleures → pires)
  🟢 Xh  : XXX ms moy
  🟢 Xh  : XXX ms moy
  ...
  🔴 Xh  : X XXX ms moy  ← à éviter
```

---

## FOCUS MAINTENANCE

> Déclenché par `/helia-reseau maintenance`

```bash
helia maintenance --json
```

Interpréter chaque champ `isXxxOnMaintenance`. Si au moins un est `true` :
- Afficher le(s) service(s) concerné(s)
- Rediriger vers [helia.nc/etat-du-reseau](https://helia.nc/etat-du-reseau) pour la zone géographique

Si tout est `false` → "✅ Tous les services sont opérationnels."

---

## FOCUS DISPONIBILITÉ

> Déclenché par `/helia-reseau dispo`

```sql
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
       COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
       COUNT(*) AS nb_total
FROM api_ping;

SELECT CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
       ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
       COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
       COUNT(*) AS nb_mesures
FROM api_ping GROUP BY 1 ORDER BY 1 DESC;
```

Format de réponse :

```
📶 Disponibilité API Helia NC

  Global : XX,X%  🟢🟡🔴  (X timeouts / N mesures)

  Par jour :
  JJ/MM : XX,X%  (X timeouts)
  JJ/MM : XX,X%  (X timeouts)
  ...
```

---

## FOCUS HEATMAP

> Déclenché par `/helia-reseau heatmap`

```sql
SELECT * FROM v_latency_heatmap;
```

Produire un tableau lisible heure × jour de semaine (0=dim, 1=lun … 6=sam).
Coloriser mentalement : < 500 ms 🟢 · 500–1 000 ms 🟡 · > 1 000 ms 🔴.
Identifier et signaler le créneau optimal (heure + jour le plus rapide) et le pire créneau.

---

## FOCUS REPORT

> Déclenché par `/helia-reseau report`

### Requêtes à exécuter

```sql
-- Santé API dernière heure
SELECT * FROM v_api_health;

-- Disponibilité globale
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
       COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
       COUNT(*) AS nb_total
FROM api_ping;

-- Disponibilité par jour
SELECT CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
       ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
       COUNT(*) FILTER (WHERE timeout) AS nb_timeouts
FROM api_ping GROUP BY 1 ORDER BY 1 ASC;

-- Profil horaire complet
SELECT heure, latence_moy_ms, latence_p95_ms, nb_timeouts, nb_mesures
FROM v_hourly_latency ORDER BY heure ASC;

-- Top 5 meilleures et pires heures
SELECT heure, latence_moy_ms FROM v_hourly_latency ORDER BY latence_moy_ms ASC LIMIT 5;
SELECT heure, latence_moy_ms FROM v_hourly_latency ORDER BY latence_moy_ms DESC LIMIT 5;

-- Pings 24 dernières heures (pour sparkline)
SELECT CAST(timestamp + 11 * INTERVAL '1 hour' AS TIME) AS heure_locale,
       response_ms, timeout
FROM api_ping WHERE timestamp >= NOW() - INTERVAL '24 hours' ORDER BY timestamp ASC;

-- Historique
SELECT MIN(timestamp) + 11 * INTERVAL '1 hour' AS depuis,
       MAX(timestamp) + 11 * INTERVAL '1 hour' AS dernier,
       COUNT(*) AS nb FROM api_ping;
```

### Écriture du fichier

```bash
# Initialisation complète de l'arborescence helia
mkdir -p ~/Documents/helia/conso ~/Documents/helia/reseau
[ ! -f ~/Documents/helia/helia.svg ] && \
  curl -sL -o ~/Documents/helia/helia.svg \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/assets/logos/helia.svg
[ ! -f ~/Documents/helia/README.md ] && \
  curl -sL -o ~/Documents/helia/README.md \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/README_helia_dir.md
DATE_NC=$(date -u -d '+11 hours' '+%Y-%m-%d' 2>/dev/null || date -u -v+11H '+%Y-%m-%d')
FICHIER=~/Documents/helia/reseau/${DATE_NC}_rapport_reseau.md
```

Confirmer : `✅ Rapport écrit dans ~/Documents/helia/reseau/YYYY-MM-DD_rapport_reseau.md`

### Format du document exporté

````markdown
# 📡 Helia NC — Rapport qualité réseau
> Généré le JJ/MM/AAAA à HH:MM (heure NC) · X mesures depuis le JJ/MM/AAAA

---

## 🔍 Analyse

[2-3 phrases en langage naturel : le réseau est-il stable ? Y a-t-il des heures à éviter ?
Quelle est la tendance de disponibilité ? Ton direct, pas de jargon.]

Exemples :
- Stable → "Le réseau Helia est stable sur la période : XX,X% de disponibilité et une latence moyenne de XXX ms. La meilleure heure pour naviguer est Xh (~XXX ms)."
- Instable → "Le réseau présente des instabilités : X timeouts enregistrés, disponibilité de XX,X%. Les heures de pointe (Xh–Xh) sont à éviter avec une latence dépassant X XXX ms."

---

## 📊 Synthèse

| Indicateur | Valeur | Verdict |
|---|---|---|
| 📡 Latence moy (1h) | XXX ms | 🟢🟡🔴 |
| 📡 Latence P95 (1h) | XXX ms | 🟢🟡🔴 |
| ⛔ Timeouts (1h) | X / N mesures | ✅ ou 🔴 |
| 📶 Disponibilité globale | XX,X% | 🟢🟡🔴 |
| ⏰ Meilleure heure | Xh → XXX ms | 🟢 |
| ⚠️ Pire heure | Xh → X XXX ms | 🔴 |

---

## 📈 Latence par heure du jour

```mermaid
xychart-beta
    title "Latence moyenne par heure (ms)"
    x-axis ["0h","1h","2h","3h","4h","5h","6h","7h","8h","9h","10h","11h","12h","13h","14h","15h","16h","17h","18h","19h","20h","21h","22h","23h"]
    line [X, X, X, ...]
```

---

## 📶 Disponibilité par jour

```mermaid
xychart-beta
    title "Disponibilité par jour (%)"
    x-axis ["JJ/MM", "JJ/MM", ...]
    bar [XX.X, XX.X, ...]
```

---

## ⛔ Timeouts par jour

```mermaid
xychart-beta
    title "Timeouts par jour"
    x-axis ["JJ/MM", "JJ/MM", ...]
    bar [X, X, ...]
```

---

## 🥧 Pings OK vs Timeouts (global)

```mermaid
pie title Pings OK vs Timeouts
    "OK (N pings)" : XX.X
    "Timeout (N)" : X.X
```

---

*Données issues de `~/.config/helia/data/helia.db` · N mesures depuis le JJ/MM/AAAA · dernier ping : JJ/MM/AAAA HH:MM*
````

### Règles de construction des charts

- **Axe x latence horaire** : toutes les 24 heures, même si certaines ont 0 mesure (mettre NULL ou 0)
- **Valeurs manquantes** : si une heure n'a pas de données, ne pas l'inclure dans l'axe
- **Limiter à 10 points** pour les charts par jour si l'historique est long
- Si `v_api_health` est vide (fenêtre sans pings récents) : afficher `> Pas de mesure sur la dernière heure.`

---

## FOCUS EXPERT

> Déclenché par `/helia-reseau expert`

Destiné aux profils techniques et décisionnels : CIO, CEO, Directeur Télécom, OPS, SysAdmin, support OPT-NC.

**Format : rapport PDF compilé via Quarto + R + XeLaTeX — 4–5 pages, professionnel.**

L'R se charge de tout (connexion DuckDB, calculs, ggplot2). Claude écrit le `.qmd`, puis compile avec `quarto render`.

### Prérequis à vérifier avant de générer

```bash
quarto --version                        # ≥ 1.4
xelatex --version                       # système TeX Live
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

FontAwesome5 est fourni par TeX Live système (`/usr/share/texlive/texmf-dist/tex/latex/fontawesome5/`).
**Ne pas installer TinyTeX si TeX Live système est disponible** — conflit possible.

### Écriture et compilation

```bash
# Initialisation complète de l'arborescence helia
mkdir -p ~/Documents/helia/conso ~/Documents/helia/reseau
[ ! -f ~/Documents/helia/helia.svg ] && \
  curl -sL -o ~/Documents/helia/helia.svg \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/assets/logos/helia.svg
# Convertir SVG en PNG pour XeLaTeX (rsvg-convert ou inkscape requis)
[ ! -f ~/Documents/helia/helia.png ] && \
  rsvg-convert -f png -w 400 ~/Documents/helia/helia.svg \
    -o ~/Documents/helia/helia.png 2>/dev/null || true
[ ! -f ~/Documents/helia/README.md ] && \
  curl -sL -o ~/Documents/helia/README.md \
    https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-conso/README_helia_dir.md

# Télécharger le template QMD depuis le repo (source de vérité)
DATE_NC=$(date -u -d '+11 hours' '+%Y-%m-%d' 2>/dev/null || date -u -v+11H '+%Y-%m-%d')
QMD=~/Documents/helia/reseau/${DATE_NC}_rapport_expert_reseau.qmd
curl -sL -o "$QMD" \
  https://raw.githubusercontent.com/adriens/claude-commands/main/docs/skills/helia-reseau/src/rapport_expert_reseau_template.qmd

# Remplacer toute date résiduelle du template par la date du jour (NC)
sed -i "s/[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}_rapport_expert/${DATE_NC}_rapport_expert/g" "$QMD"

quarto render "$QMD" --to pdf
```

> **Le template QMD est maintenant versionné dans le repo.**
> Ne pas réécrire le `.qmd` manuellement — utiliser le template via `curl` ci-dessus.
> Si une modification est nécessaire (nouveau chart, nouvelle section), modifier
> `docs/skills/helia-reseau/src/rapport_expert_reseau_template.qmd` dans le repo.

**Vérifications obligatoires avant `quarto render` :**
- `date: today` dans le YAML → Quarto injecte automatiquement la date du jour ✅
- Période analysée (`periode_debut` / `periode_fin`) → calculée en live depuis DuckDB ✅
- Toutes les métriques (SLA, latence, MTBF…) → requêtes DuckDB en temps réel ✅
- Si `grep -n '[0-9]\{4\}-[0-9]\{2\}-[0-9]\{2\}' "$QMD"` retourne des dates dans des
  commentaires R ou des chaînes hardcodées, les remplacer par `$DATE_NC` avant le rendu.

### Analyse contextuelle — injection obligatoire avant le rendu

Le template contient le marqueur `HELIA_ANALYSE_PLACEHOLDER` dans la section **Analyse contextuelle**.
**Ce marqueur doit être remplacé par un texte narratif avant `quarto render`**, sinon le PDF contiendra
le marqueur brut.

#### Requêtes DuckDB pour alimenter l'analyse

```sql
-- Résumé du jour J (heure NC)
SELECT
    CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
    ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 2) AS dispo_pct,
    COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
    ROUND(AVG(response_ms) FILTER (WHERE NOT timeout), 0) AS latence_moy,
    ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY response_ms)
          FILTER (WHERE NOT timeout), 0) AS p50,
    ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY response_ms)
          FILTER (WHERE NOT timeout), 0) AS p95,
    COUNT(*) AS nb_mesures
FROM api_ping
WHERE CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) = CURRENT_DATE
GROUP BY 1;

-- Moyenne historique (hors aujourd'hui) pour comparaison
SELECT
    ROUND(AVG(dispo_pct), 2) AS dispo_moy_hist,
    ROUND(AVG(latence_moy), 0) AS latence_moy_hist
FROM (
    SELECT
        CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
        ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 2) AS dispo_pct,
        ROUND(AVG(response_ms) FILTER (WHERE NOT timeout), 0) AS latence_moy
    FROM api_ping
    WHERE CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) < CURRENT_DATE
    GROUP BY 1
) hist;

-- Heures critiques du jour (latence > 1 000 ms ou timeouts)
SELECT
    EXTRACT(HOUR FROM timestamp + 11 * INTERVAL '1 hour') AS heure,
    COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
    ROUND(AVG(response_ms) FILTER (WHERE NOT timeout), 0) AS latence_moy
FROM api_ping
WHERE CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) = CURRENT_DATE
GROUP BY 1
HAVING COUNT(*) FILTER (WHERE timeout) > 0
    OR AVG(response_ms) FILTER (WHERE NOT timeout) > 1000
ORDER BY 1;
```

#### Texte narratif à générer

À partir des résultats, rédiger **3 à 5 paragraphes en français**, ton direct, sans jargon inutile :

1. **Bilan global** — disponibilité du jour vs moyenne historique, verdict 🟢🟡🔴
2. **Comportement de la latence** — P50/P95 du jour, comparaison au profil habituel, heures de pointe
3. **Incidents** — si timeouts : quand, combien, durée estimée d'indisponibilité cumulée
4. **Tendance** — le réseau est-il stable, en amélioration, en dégradation par rapport aux jours précédents ?
5. **Recommandation** (si 🟡 ou 🔴) — action concrète (plage horaire à éviter, contacter le 1013, etc.)

Si les données du jour sont insuffisantes (< 10 mesures), le signaler explicitement et baser l'analyse
sur les dernières 24 heures disponibles.

#### Injection dans le QMD

```bash
uv run python - <<'PYEOF'
analyse = """[texte généré ci-dessus — paragraphes Markdown, sans LaTeX]"""

with open(qmd_path, "r") as f:
    content = f.read()

content = content.replace("HELIA_ANALYSE_PLACEHOLDER", analyse)

with open(qmd_path, "w") as f:
    f.write(content)
PYEOF
```

> Le texte doit être du **Markdown standard** (gras, listes, sauts de ligne) — Quarto le convertit
> en LaTeX automatiquement. Ne pas injecter de commandes LaTeX brutes.

Confirmer : `✅ PDF généré : ~/Documents/helia/reseau/YYYY-MM-DD_rapport_expert_reseau.pdf`

### Template Quarto à écrire dans le fichier .qmd

**Couleurs officielles Helia (extraites du SVG logo) :**
- Dégradé : `#FF00E3` (fuchsia) → `#FF0010` (rouge cerise)
- Milieu : `#FF0078`

**Icônes FontAwesome5 :** utiliser `\faIcon{nom-en-kebab}` pour tous les icônes sauf les rares qui ont un alias direct (`\faClipboardList`, `\faChartLine`, `\faCalendarCheck`, `\faExclamationTriangle`, `\faDatabase`, `\faClock`, `\faRobot`, `\faLock`).

**Sections requises :**
1. Page de titre avec logo PNG (`~/Documents/helia/helia.png`) + règle dégradée TikZ
2. Bloc de synthèse exécutive (`mdframed`) — utiliser chunk R `results='asis'` avec `cat()` pour injecter du LaTeX dynamique
3. Tableau de bord SLA (kableExtra, en-tête `col_mid`)
4. Analyse performances : **note explicative des percentiles avec les vrais chiffres** (P50=X ms = moitié des requêtes en dessous, P95=X ms = 19/20 en dessous, P99=X ms = 99/100 en dessous) + charts percentiles + profil horaire (gridExtra ncol=2)
5. Distribution des temps de réponse (histogramme par classes)
6. **Profil de densité KDE** : `geom_density()` + histogramme, axes linéaire et log
7. **Heatmap heure × jour de la semaine** : `geom_tile()` + `coord_equal()` (carrés style GitHub), Y = Lundi→Dimanche agrégés par DOW, couleur = latence moyenne
8. Disponibilité journalière + tableau détail
9. **Chronologie des incidents** : scatter plot de tous les pings, coloré par type (Normal/Lenteur/Timeout)
10. **Stack technique** : table des composants avec versions
11. **Glossaire** : table d'explication des termes pour non-techniciens

**Template QMD :** versionné dans le repo — récupéré via `curl` dans l'étape de compilation ci-dessus.
Ne pas réécrire le `.qmd` à la main. Pour toute modification (nouveau chart, nouvelle section),
éditer `docs/skills/helia-reseau/src/rapport_expert_reseau_template.qmd` dans le repo.

### Règles de présentation

- **Disponibilité** : 4 décimales
- **Latence** : entier ms, jamais de décimale
- **Indisponibilité** : convertir en h min (chaque ping = 5 min)
- **MTBF** : N/A si aucun timeout enregistré
- **gridExtra** requis pour la mise en page 2 colonnes — vérifier avec `install.packages("gridExtra")`

### Requêtes à exécuter

```sql
-- Percentiles de latence (hors timeouts)
SELECT
    ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY response_ms), 0) AS p50_ms,
    ROUND(PERCENTILE_CONT(0.90) WITHIN GROUP (ORDER BY response_ms), 0) AS p90_ms,
    ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY response_ms), 0) AS p95_ms,
    ROUND(PERCENTILE_CONT(0.99) WITHIN GROUP (ORDER BY response_ms), 0) AS p99_ms,
    ROUND(AVG(response_ms), 0) AS moy_ms,
    MAX(response_ms) AS max_ms
FROM api_ping WHERE NOT timeout;

-- SLA — disponibilité globale avec niveau (labels français)
SELECT
    COUNT(*) AS nb_total,
    COUNT(*) FILTER (WHERE NOT timeout) AS nb_ok,
    COUNT(*) FILTER (WHERE timeout) AS nb_timeout,
    ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 4) AS dispo_pct,
    CASE
        WHEN ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 4) >= 99.999 THEN '>= 99.999%'
        WHEN ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 4) >= 99.99  THEN '>= 99.99%'
        WHEN ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 4) >= 99.9   THEN '>= 99.9%'
        WHEN ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 4) >= 99.0   THEN '>= 99%'
        ELSE '< 99%'
    END AS sla_niveau
FROM api_ping;

-- Durée d'indisponibilité cumulée (chaque ping = 5 min)
SELECT
    COUNT(*) FILTER (WHERE timeout) * 5 AS minutes_indispo,
    ROUND(COUNT(*) FILTER (WHERE timeout) * 5 / 60.0, 2) AS heures_indispo,
    COUNT(*) * 5 AS minutes_total,
    ROUND(COUNT(*) * 5 / 60.0 / 24.0, 1) AS jours_surveilles
FROM api_ping;

-- MTBF approché : temps moyen entre deux timeouts (en minutes)
WITH timeouts AS (
    SELECT timestamp,
           LAG(timestamp) OVER (ORDER BY timestamp) AS prev_timeout
    FROM api_ping WHERE timeout
)
SELECT ROUND(AVG(EPOCH(timestamp - prev_timeout) / 60.0), 0) AS mtbf_minutes
FROM timeouts WHERE prev_timeout IS NOT NULL;

-- Distribution par plage de latence
SELECT
    CASE
        WHEN timeout THEN 'Timeout'
        WHEN response_ms <= 500   THEN '≤ 500 ms 🟢'
        WHEN response_ms <= 1000  THEN '501–1000 ms 🟡'
        WHEN response_ms <= 2000  THEN '1001–2000 ms 🟠'
        ELSE '> 2000 ms 🔴'
    END AS plage,
    COUNT(*) AS nb,
    ROUND(100.0 * COUNT(*) / (SELECT COUNT(*) FROM api_ping), 1) AS pct
FROM api_ping
GROUP BY 1 ORDER BY MIN(CASE WHEN timeout THEN 99999 ELSE response_ms END);

-- Disponibilité par jour + percentiles journaliers
SELECT
    CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
    ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 2) AS dispo_pct,
    COUNT(*) FILTER (WHERE timeout) AS nb_timeouts,
    ROUND(PERCENTILE_CONT(0.50) WITHIN GROUP (ORDER BY response_ms) FILTER (WHERE NOT timeout), 0) AS p50_ms,
    ROUND(PERCENTILE_CONT(0.95) WITHIN GROUP (ORDER BY response_ms) FILTER (WHERE NOT timeout), 0) AS p95_ms,
    COUNT(*) AS nb_mesures
FROM api_ping GROUP BY 1 ORDER BY 1 ASC;

-- Profil horaire complet
SELECT heure, latence_moy_ms, latence_p95_ms, nb_timeouts, nb_mesures
FROM v_hourly_latency ORDER BY heure ASC;

-- Top 10 pires incidents (pings les plus lents ou timeouts)
SELECT timestamp + 11 * INTERVAL '1 hour' AS heure_locale,
       response_ms, timeout, http_status
FROM api_ping
ORDER BY CASE WHEN timeout THEN 999999 ELSE response_ms END DESC
LIMIT 10;

-- Historique
SELECT MIN(timestamp) + 11 * INTERVAL '1 hour' AS depuis,
       MAX(timestamp) + 11 * INTERVAL '1 hour' AS dernier,
       COUNT(*) AS nb_pings,
       ROUND(COUNT(*) * 5 / 60.0 / 24.0, 1) AS jours_surveilles
FROM api_ping;
```

### Écriture du fichier

```bash
mkdir -p ~/Documents/helia/reseau
DATE_NC=$(date -u -d '+11 hours' '+%Y-%m-%d' 2>/dev/null || date -u -v+11H '+%Y-%m-%d')
FICHIER=~/Documents/helia/reseau/${DATE_NC}_rapport_expert_reseau.md
```

Confirmer : `✅ Rapport expert écrit dans ~/Documents/helia/reseau/YYYY-MM-DD_rapport_expert_reseau.md`

### Format du document exporté

````markdown
# 📡 Helia NC — Rapport Expert Qualité Réseau
> Généré le JJ/MM/AAAA à HH:MM (heure NC) · Période : JJ/MM/AAAA → JJ/MM/AAAA · N jours surveillés

---

## 🔍 Synthèse exécutive

[2-3 phrases niveau management : le service respecte-t-il le SLA ? Quel est l'impact opérationnel ?
Quelles actions sont recommandées ? Adapter selon le profil détecté.]

Exemples :
- Pour CIO/CEO → "Le service API Helia NC affiche une disponibilité de XX,XX% sur la période (niveau : Three nines 🟡). Le temps d'indisponibilité cumulé est de X heures X minutes. Aucune action corrective immédiate n'est requise."
- Pour DT/OPS → "XX timeouts enregistrés sur N mesures. MTBF estimé à XXX minutes. P95 à XXX ms — dans les normes pour un service mobile. Les créneaux 7h–9h présentent une dégradation systématique à surveiller."
- Pour support OPT-NC → "Rapport de disponibilité API sur N jours. X incidents de timeout détectés. Pics de latence entre Xh et Xh. Données exportables pour ticket d'incident."

---

## 📊 Tableau de bord SLA

| Métrique | Valeur | Objectif | Statut |
|---|---|---|---|
| Disponibilité globale | XX,XXXX% | ≥ 99,9% | 🟢🟡🔴 |
| Niveau SLA | Three nines / Four nines… | — | ✅🟡🔴 |
| Latence P50 | XXX ms | ≤ 500 ms | 🟢🟡🔴 |
| Latence P95 | XXX ms | ≤ 1 000 ms | 🟢🟡🔴 |
| Latence P99 | XXX ms | ≤ 2 000 ms | 🟢🟡🔴 |
| Timeouts | X / N (X,X%) | < 1% | 🟢🟡🔴 |
| Indisponibilité cumulée | X h X min | — | — |
| MTBF | XXX min | — | — |
| Période surveillée | N jours | — | — |

---

## 📈 Latence — Percentiles

```mermaid
xychart-beta
    title "Percentiles de latence (ms)"
    x-axis ["P50", "P90", "P95", "P99", "Max"]
    bar [XXX, XXX, XXX, XXX, XXXX]
```

---

## 📊 Distribution des réponses

```mermaid
pie title Distribution des temps de réponse
    "≤ 500 ms 🟢 (XX%)" : XX.X
    "501–1000 ms 🟡 (XX%)" : XX.X
    "1001–2000 ms 🟠 (XX%)" : XX.X
    "> 2000 ms 🔴 (XX%)" : XX.X
    "Timeout ⛔ (X%)" : X.X
```

---

## 📶 Disponibilité journalière

```mermaid
xychart-beta
    title "Disponibilité par jour (%)"
    x-axis ["JJ/MM", ...]
    line [XX.XX, ...]
```

---

## ⏱ Latence P95 par jour

```mermaid
xychart-beta
    title "P95 de latence par jour (ms)"
    x-axis ["JJ/MM", ...]
    bar [XXX, ...]
```

---

## ⏰ Profil horaire — Latence moyenne

```mermaid
xychart-beta
    title "Latence moyenne par heure du jour (ms)"
    x-axis ["0h","1h","2h","3h","4h","5h","6h","7h","8h","9h","10h","11h","12h","13h","14h","15h","16h","17h","18h","19h","20h","21h","22h","23h"]
    line [X, X, ...]
```

---

## 🚨 Top 10 incidents (pires mesures)

| Horodatage (NC) | Latence | Timeout | HTTP |
|---|---|---|---|
| JJ/MM/AAAA HH:MM | X XXX ms | ✅/⛔ | 200/0 |
| ... | | | |

---

## 📋 Disponibilité journalière — Détail

| Date | Dispo % | Timeouts | P50 | P95 | Mesures |
|---|---|---|---|---|---|
| JJ/MM | XX,XX% | X | XXX ms | XXX ms | N |
| ... | | | | | |

---

*Données issues de `~/.config/helia/data/helia.db` · N pings · période : JJ/MM → JJ/MM · dernier ping : JJ/MM HH:MM*
````

### Règles de présentation

- **Disponibilité** : toujours à 4 décimales (ex: 99,6552%)
- **Latence** : arrondie à l'entier (ms), jamais de décimale
- **MTBF** : si < 60 min → afficher en minutes ; si ≥ 60 min → afficher en heures et minutes
- **Indisponibilité cumulée** : convertir en h min sec
- **Top 10 incidents** : trier timeouts d'abord, puis par latence décroissante
- **Objectifs SLA** : utiliser les seuils telecom standards (P95 ≤ 1 000 ms, dispo ≥ 99,9%)
- Si aucun timeout → mentionner "aucun incident enregistré sur la période" dans la synthèse exécutive

---

## TABLEAU DE BORD — Synthèse réseau par défaut

**Si l'utilisateur ne pose pas de question précise, produire ce tableau de bord réseau.**

### Requêtes DuckDB à exécuter

```sql
-- Santé API dernière heure
SELECT * FROM v_api_health;

-- Meilleures et pires heures de la journée
SELECT * FROM v_hourly_latency ORDER BY latence_moy_ms ASC;

-- Disponibilité globale
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
       COUNT(*) FILTER (WHERE timeout) AS nb_timeouts_total,
       COUNT(*) AS nb_mesures_total
FROM api_ping;

-- Disponibilité par jour
SELECT
    CAST(timestamp + 11 * INTERVAL '1 hour' AS DATE) AS jour,
    ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
    COUNT(*) FILTER (WHERE timeout) AS nb_timeouts
FROM api_ping GROUP BY 1 ORDER BY 1 DESC;

-- Sparkline des 40 derniers pings
SELECT points FROM v_sparkline;
```

**Toujours ouvrir en lecture seule :**
```bash
duckdb -readonly ~/.config/helia/data/helia.db "SELECT ..."
```

### Format de sortie — tableau synthèse réseau

```
📡 Helia NC — Qualité réseau   🕐 Dernière mesure : HH:MM

┌──────────────┬──────────────────────────────┬────────────────────┐
│              │ Valeur                       │ Verdict            │
├──────────────┼──────────────────────────────┼────────────────────┤
│ 🟢 Latence   │ XXX ms (moy) · XXX ms (P95)  │ 🟢🟡🔴            │
├──────────────┼──────────────────────────────┼────────────────────┤
│ ⛔ Timeouts  │ X sur N mesures (XX%)        │ ✅ ou 🔴           │
├──────────────┼──────────────────────────────┼────────────────────┤
│ 📶 Dispo     │ XX.X%                        │ 🟢🟡🔴            │
├──────────────┼──────────────────────────────┼────────────────────┤
│ ⏰ Meilleure │ XXh → XXX ms moy             │ Créneau optimal    │
├──────────────┼──────────────────────────────┼────────────────────┤
│ ⚠️ Pire      │ XXh → XXX ms moy             │ À éviter           │
└──────────────┴──────────────────────────────┴────────────────────┘
```

### Règles de verdict latence

| Latence moy | Couleur |
|---|---|
| ≤ 500 ms | 🟢 Rapide |
| 500–1 000 ms | 🟡 Acceptable |
| > 1 000 ms | 🔴 Lent |
| timeout | ⛔ Indisponible |

### Règles de verdict disponibilité

| Dispo % | Verdict |
|---|---|
| ≥ 99 % | 🟢 Excellente |
| 95–99 % | 🟡 Acceptable |
| < 95 % | 🔴 Dégradée |

---

## DOMAINE 1 — Performances API & historique de latence

### Router la commande

| Question | Commande |
|---|---|
| État services / maintenance en ce moment | `helia maintenance --json` |
| Latence / timeouts dernière heure | `SELECT * FROM v_api_health` |
| Profil horaire (meilleures/pires heures) | `SELECT * FROM v_hourly_latency` |
| Heatmap heure × jour de semaine | `SELECT * FROM v_latency_heatmap` |
| Sparkline 40 derniers points | `SELECT points FROM v_sparkline` |
| Historique par jour | Requête sur `api_ping` GROUP BY jour |

Le CLI est dans le PATH ou à `/home/adriens/Github/helia/cli/helia`.

### Schéma de la table

```sql
-- api_ping (écrit toutes les 5 min)
timestamp TIMESTAMP, response_ms INTEGER, http_status INTEGER, timeout BOOLEAN
```

### Vues disponibles

```sql
-- Santé API dernière heure (moy, P95, timeouts)
SELECT * FROM v_api_health;
-- → nb_pings | nb_ok | nb_timeouts | dispo_pct | latence_moy_ms | latence_p95_ms | latence_max_ms

-- Profil de latence par heure du jour (trié du plus rapide au plus lent)
SELECT * FROM v_hourly_latency;
-- → heure | latence_moy_ms | latence_p95_ms | nb_timeouts | nb_mesures

-- Heatmap latence moyenne par heure × jour de semaine (0=dim, 1=lun, ..., 6=sam)
SELECT * FROM v_latency_heatmap;

-- Dispo globale (toutes mesures)
SELECT ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct
FROM api_ping;

-- Dispo par jour
SELECT
    CAST(timestamp + (SELECT CAST(value AS INTEGER) FROM settings WHERE key='utc_offset_hours') * INTERVAL '1 hour' AS DATE) AS jour,
    ROUND(100.0 * COUNT(*) FILTER (WHERE NOT timeout) / COUNT(*), 1) AS dispo_pct,
    COUNT(*) FILTER (WHERE timeout) AS nb_timeouts
FROM api_ping GROUP BY 1 ORDER BY 1 DESC;
```

### Requêtes ad hoc utiles

```sql
-- Pics de latence (top 10 pires mesures)
SELECT timestamp + 11 * INTERVAL '1 hour' AS heure_locale,
       response_ms, timeout
FROM api_ping
ORDER BY response_ms DESC NULLS LAST LIMIT 10;

-- Pings des dernières 24h
SELECT timestamp + 11 * INTERVAL '1 hour' AS heure_locale,
       response_ms, timeout
FROM api_ping
WHERE timestamp >= NOW() - INTERVAL '24 hours'
ORDER BY timestamp DESC;
```

---

## DOMAINE 2 — État du réseau & incidents

### Vérifier les maintenances en cours

```bash
helia maintenance --json
```

Interpréter les champs `isXxxOnMaintenance` : si `true`, le service concerné est en maintenance planifiée.

### Page officielle d'état du réseau

→ [helia.nc/etat-du-reseau](https://helia.nc/etat-du-reseau)

- **Maintenances programmées** : interventions planifiées avec dates, heures et zones géographiques
- **Incidents en cours** : perturbations en temps réel
- **Couverture** : zones de couverture mobile et fibre consultables par carte

> Si `helia maintenance --json` signale une maintenance active, rediriger vers cette page pour le détail géographique de la zone impactée.

### Infrastructure réseau (référence)

- 530+ antennes
- 8 200 km de fibre optique
- Arrêt programmé du cuivre et de la 2G (en cours)

---

## DOMAINE 3 — Assistance réseau

### Numéros utiles (problèmes réseau)

| Numéro | Service | Horaires |
|---|---|---|
| **1013** (gratuit) | Dépannage lignes & Internet | Lun–Ven 7h30–16h · Sam 7h–11h |
| **1000** (gratuit) | Assistance générale | Lun–Ven 7h30–15h45 |

### Réclamation technique persistante

→ [helia.nc/reclamations](https://helia.nc/reclamations)  
→ reclamation@helia.nc

---

## Arbre de décision

```
Question reçue
│
├─ Latence / timeouts / performances API ?
│   └─ → DOMAINE 1 : DuckDB v_api_health, v_hourly_latency, v_latency_heatmap
│
├─ Maintenance en cours / panne / incident ?
│   └─ → DOMAINE 2 : helia maintenance --json + helia.nc/etat-du-reseau
│
└─ Problème persistant / joindre le support réseau ?
    └─ → DOMAINE 3 : 1013 + reclamation@helia.nc
```

> Pour les questions sur la **consommation data/voix, les forfaits ou les recharges** → utiliser `/helia-conso`

---

## Notes techniques

- Token API : `~/.config/helia/token`
- CLI : `/home/adriens/Github/helia/cli/helia`
- DuckDB : `~/.config/helia/data/helia.db` — **toujours `read_only=True`** pour les lectures
- UTC offset NC : **+11h** stocké dans `settings` (clé `utc_offset_hours`)
- `api_ping` : écrit à chaque refresh (toutes les 5 min)
- `v_api_health` : fenêtre glissante sur la **dernière heure** de mesures
