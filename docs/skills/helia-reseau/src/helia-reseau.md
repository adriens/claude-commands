# Helia NC — Qualité du réseau mobile

Réponds aux questions sur la qualité du réseau mobile Helia NC : latence API, maintenances, incidents, disponibilité.
Réponds **dans la langue de la question posée**.

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
