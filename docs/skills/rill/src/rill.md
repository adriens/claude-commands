# Rill — Analyse de données locale

Skill pour travailler avec un projet [Rill](https://docs.rilldata.com) en local : démarrer le serveur, utiliser le MCP, interroger les métriques.
Réponds **dans la langue de la question posée**.

---

## 1. Démarrer Rill

Rill est installé globalement (`/usr/local/bin/rill`). Pas besoin d'utiliser `./rill` local.

```bash
# Démarrage standard (ouvre le navigateur automatiquement)
rill start

# Sans ouvrir le navigateur
rill start --no-open

# Sur un port personnalisé
rill start --port 8080
```

**Port par défaut : 9009**
Vérifier que Rill répond :
```bash
curl -s http://localhost:9009/v1/ping
```

Démarrer en arrière-plan et attendre qu'il soit prêt :
```bash
rill start --no-open &
sleep 8 && curl -s http://localhost:9009/v1/ping
```

---

## 2. Utiliser le MCP Rill

Rill expose un MCP sur `http://localhost:9009/v1/mcp`. Les outils MCP sont disponibles sous `mcp__rill-developer__*`.

### Workflow recommandé (dans l'ordre)

1. **`mcp__rill-developer__project_status`** — vérifier que tout est OK, pas d'erreurs
2. **`mcp__rill-developer__list_metrics_views`** — lister les dashboards disponibles
3. **`mcp__rill-developer__get_metrics_view`** — récupérer les dimensions et mesures d'un dashboard
4. **`mcp__rill-developer__query_metrics_view_summary`** — obtenir la plage temporelle et des valeurs d'exemple
5. **`mcp__rill-developer__query_metrics_view`** — interroger les métriques

Ne pas sauter les étapes 3 et 4 : elles donnent les noms exacts des dimensions/mesures nécessaires pour les requêtes.

### Requête de comparaison temporelle (J vs J-1)

```json
{
  "metrics_view": "mon_dashboard",
  "measures": [{"name": "ma_mesure"}],
  "time_range": {"start": "2026-06-02T00:00:00Z", "end": "2026-06-03T00:00:00Z"},
  "comparison_time_range": {"start": "2026-06-01T00:00:00Z", "end": "2026-06-02T00:00:00Z"}
}
```

### Requête groupée par jour avec filtre

```json
{
  "metrics_view": "mon_dashboard",
  "dimensions": [{"name": "timestamp", "compute": {"time_floor": {"dimension": "timestamp", "grain": "day"}}}],
  "measures": [{"name": "ma_mesure"}],
  "where": {"cond": {"op": "gte", "exprs": [{"name": "hour_of_day"}, {"val": 6}]}}
}
```

---

## 3. Rafraîchir les données

Les modèles Rill sont **matérialisés** : ils capturent un snapshot au démarrage. Si la source de données (ex: DuckDB externe) a été mise à jour, Rill ne le voit pas automatiquement.

**Forcer un rechargement** en modifiant le fichier source YAML (ex: ajouter un schedule de refresh) :

```yaml
# sources/ma_source.yaml
type: source
connector: duckdb
sql: SELECT * FROM read_duckdb('~/.config/helia/data/helia.db', table_name:='ma_table')
refresh:
  cron: "*/5 * * * *"
```

Enregistrer le fichier → le file watcher Rill déclenche une réconciliation.

Vérifier ce qu'il y a réellement dans le modèle matérialisé :
```bash
rill query --local --sql "SELECT max(timestamp), count(*) FROM ma_table" --path .
```

**Attention** : modifier un fichier source pendant que Rill est instable peut provoquer l'erreur `controller: inconsistent version`. Dans ce cas, redémarrer Rill :
```bash
pkill -f "rill start" && sleep 3 && rill start --no-open &
```

---

## 4. Points clés à retenir

- `query_metrics_view_summary` peut retourner du cache — utiliser `rill query --local --sql` pour vérifier les données réelles
- Les timestamps dans Rill sont traités comme UTC sauf si un `time_zone` est spécifié dans la requête
- Le paramètre `time_zone` dans `query_metrics_view` permet d'aligner les filtres temporels sur le fuseau local
- L'endpoint MCP est `http://localhost:9009/v1/mcp` (pas `/mcp` ni `/v1/mcp/sse`)
- `cascade_failure_rate` retourne NULL s'il n'y a aucun timeout (division par zéro protégée)

---

## 5. Requêtes SQL directes

```bash
# Requête SQL directe sur un modèle matérialisé
rill query --local --sql "SELECT * FROM mon_modele LIMIT 10" --path .

# Avec un connecteur spécifique
rill query --local --sql "SELECT ..." --connector duckdb --path .
```
