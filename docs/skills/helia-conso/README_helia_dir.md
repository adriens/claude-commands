![Logo Helia](helia.svg)

# 📡 Helia NC — Rapports automatiques

Ce répertoire contient les rapports générés automatiquement par les skills Claude Code
[`/helia-conso`](https://github.com/adriens/claude-commands/tree/main/docs/skills/helia-conso)
et [`/helia-reseau`](https://github.com/adriens/claude-commands/tree/main/docs/skills/helia-reseau).

---

## 📁 Structure

```
~/Documents/helia/
├── helia.svg                          ← Logo Helia NC
├── helia.png                          ← Logo PNG (pour les rapports PDF)
├── README.md                          ← Ce fichier
│
├── conso/                             ← Rapports de consommation personnelle
│   └── YYYY-MM-DD_rapport_conso.md   ← 1 fichier par jour (Markdown + charts Mermaid)
│
└── reseau/                            ← Rapports qualité réseau
    ├── YYYY-MM-DD_rapport_reseau.md   ← Rapport standard (Markdown + charts Mermaid)
    ├── YYYY-MM-DD_rapport_expert_reseau.qmd  ← Source Quarto
    └── YYYY-MM-DD_rapport_expert_reseau.pdf  ← Rapport expert (PDF XeLaTeX, 5+ pages)
```

---

## 📊 Rapports disponibles

### `/helia-conso report`
Rapport de **consommation personnelle** au format Markdown :
- Note d'analyse en langage naturel
- Tableau de synthèse (data, voix, SMS, rythme, projection)
- Charts Mermaid : pie data/voix, conso par jour, tendance, rythme

### `/helia-reseau report`
Rapport **qualité réseau** au format Markdown :
- Note d'analyse
- Tableau de synthèse (latence, dispo, timeouts)
- Charts Mermaid : latence horaire, dispo par jour, timeouts, pie OK/timeout

### `/helia-reseau expert`
Rapport **expert SLA** au format PDF (Quarto + R + XeLaTeX) :
- Synthèse exécutive adaptée au profil (CIO, DT, OPS, support OPT-NC)
- Tableau de bord SLA (seuils %, P50/P95/P99, MTBF)
- Charts ggplot2 professionnels (profil horaire, densité KDE, heatmap heure×jour)
- Chronologie des incidents horodatés
- Glossaire grand public
- Stack technique avec versions

---

## 🛠️ Source des données

Les rapports sont alimentés par la base DuckDB locale :
```
~/.config/helia/data/helia.db
```
Alimentée toutes les **5 minutes** par le daemon `helia-widget`.

---

## 🔗 Liens utiles

- [Skills sur GitHub](https://github.com/adriens/claude-commands)
- [helia.nc](https://helia.nc/) — Site officiel Helia NC (OPT-NC)
- [État du réseau](https://helia.nc/etat-du-reseau) — Maintenances & incidents en temps réel
