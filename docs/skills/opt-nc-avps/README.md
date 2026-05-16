# Skill : opt-nc-avps

Recherche des AVPs (Avis de Vacance de Poste) de l'OPT-NC adaptés à votre profil, avec un accompagnement complet à la préparation de votre candidature.

## Fonctionnalités

Ce skill vous accompagne tout au long du processus de candidature :

*   🔍 **Recherche ciblée** : Identifie les postes ouverts à l'OPT-NC via un serveur MCP dédié.
*   📄 **Analyse de fiche** : Décortique les missions et le profil requis en un clic.
*   🧠 **Profilage intelligent** : Évalue votre adéquation via un questionnaire interactif.
*   🛠️ **Plan de préparation** : Génère vos atouts, points de vigilance et questions d'entretien (méthode STAR).
*   ✍️ **Aide à la rédaction** : Rédige un brouillon de lettre de motivation personnalisé et adapté au contexte local.

## Installation

```bash
# 1. Installer le serveur MCP requis
claude mcp add avps-opt-nc --transport sse https://opt-nc-avps.hf.space/gradio_api/mcp/sse

# 2. Installer le skill
curl -o ~/.claude/commands/opt-nc-avps.md \
  https://raw.githubusercontent.com/adriens/claude-commands/main/skills/opt-nc-avps/opt-nc-avps.md
```

## Exemple d'utilisation

Lancez la commande dans Claude Code en précisant votre profil ou le type de poste recherché :

```text
/opt-nc-avps chef de projet SI transformation digitale
```

### Déroulement type :
1. **Claude** affiche une liste de postes (ex: "Chef de projet MOA", "Responsable d'applications").
2. **Vous** choisissez un poste en cliquant sur le lien ou en donnant le numéro.
3. **Claude** analyse la fiche et vous pose 3 questions sur votre expérience, votre maîtrise de la conduite du changement et votre connaissance du contexte local (Nouvelle-Calédonie).
4. **Claude** génère votre stratégie de candidature et un projet de lettre.
