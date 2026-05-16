---
layout: default
title: Accueil
---

# Bibliothèque de Skills

Bienvenue dans cette collection de commandes personnalisées pour **Claude Code**.

---

{% assign skills = site.pages | filter: "layout", "default" | where_exp: "item", "item.path contains 'skills/'" | where_exp: "item", "item.path contains '.md'" | where_exp: "item", "item.path != 'skills/opt-nc-avps/README.md'" %}

<div class="skills-grid">
{% for skill in skills %}
  {% if skill.command %}
  <div class="skill-card">
    <h2><a href="{{ skill.url | relative_url }}">{{ skill.title }}</a></h2>
    <p class="skill-meta">Commande : <code>{{ skill.command }}</code></p>
    <p>{{ skill.description }}</p>
    <a href="{{ skill.url | relative_url }}" class="btn">Voir les détails &rarr;</a>
  </div>
  {% endif %}
{% endfor %}
</div>

## Comment contribuer ?
Vous pouvez ajouter vos propres skills en créant un dossier dans `skills/` avec un fichier `.md` contenant le Front Matter approprié.
