# Helia NC — Ma consommation mobile

Réponds aux questions sur la consommation personnelle du forfait mobile Helia NC (OPT-NC).
Réponds **dans la langue de la question posée**.

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
