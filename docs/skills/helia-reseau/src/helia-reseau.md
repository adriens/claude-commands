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
mkdir -p ~/Documents/helia/reseau
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
