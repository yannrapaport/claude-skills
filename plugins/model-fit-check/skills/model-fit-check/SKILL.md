---
name: model-fit-check
description: Vérifie que le modèle Claude actif correspond à la complexité de la tâche, et alerte si mismatch. À DÉCLENCHER au démarrage d'une tâche quand le tier risque de ne pas matcher le modèle actif — soit tâche lourde (architecture, design, plan, tradeoff, stratégie, refactor structurant, debug complexe) sous Haiku/Sonnet, soit tâche mécanique (renommage, extraction, listing, reformatage, boilerplate, bulk haut-volume) sous Opus. Aussi sur demande : "quel modèle pour ça", "suis-je dans le bon modèle", "/model-fit-check".
---

# Model fit check

Compare le **modèle actuellement actif** au tier que la tâche demande, et alerte s'il y a un décalage — surdimensionné (gaspillage) ou sous-dimensionné (risque qualité).

## La grille (tier → modèle actuel)

| Tier | Modèle actuel | Pour |
|------|---------------|------|
| **Léger** | Haiku 4.5 | Bulk mécanique, no judgment : renommage, extraction, listing, reformatage, boilerplate, haut volume, pattern-matching |
| **Standard** | Sonnet 5 | Recherche cadrée, exploration code, synthèse in-scope, coding standard multi-étapes. **C'est le défaut.** |
| **Lourd** | Opus 4.8 | Planning, tradeoffs réels, architecture, décisions structurantes, debug complexe, nuance/jugement |

*Fable 5 = pointe extrême (raisonnement le plus dur) — à ne suggérer que si Opus est visiblement en peine.*

## Le check (3 gestes)

1. **Ton modèle actif** — tu le connais, c'est toi (Opus / Sonnet / Haiku).
2. **Classe la tâche** par signaux de surface, pas par auto-évaluation de tes capacités :
   - mots « architect / design / plan / stratégie / tradeoff / refactor entier / comprendre pourquoi » → **Lourd**
   - mots « renommer / extraire / lister / reformater / bulk / répétitif / 40 fichiers » → **Léger**
   - tout le reste, ou ambigu → **Standard**
3. **Compare.** Si tier ≠ modèle actif → alerte. Sinon → tais-toi (ou une ligne max).

## Sortie en cas de mismatch (recette exacte)

Trois lignes, rien de plus :

```
⚠️ Modèle : tu es en <actif> — cette tâche vise le tier <cible> (<modèle cible>).
Pourquoi : <une ligne — le signal qui classe la tâche>.
Action : `/model` pour basculer, ou délègue à un subagent <cible>. Sinon dis-moi de continuer.
```

Puis **attends** avant d'attaquer la tâche (sur-dimensionné : tu peux continuer, juste signaler ; sous-dimensionné : laisse l'utilisateur trancher).

## Quand NE PAS alerter (éviter le bruit)

- Modèle actif = tier cible → rien, ou une demi-ligne.
- Tâche **Standard** ou ambiguë → Sonnet est le défaut, ne jamais alerter dans le doute.
- Question conversationnelle, lookup trivial, suite d'un échange en cours → pas de check.
- Ne jamais répéter l'alerte deux fois pour la même tâche.

## Limites (à assumer, pas à cacher)

- **Pas d'auto-switch** : Claude Code ne change pas de modèle seul en cours de session. La sortie est une **alerte + action manuelle** (`/model` ou subagent), jamais une bascule.
- **Angle mort d'auto-évaluation** : un modèle faible peut sous-estimer une tâche. C'est pourquoi le check se base sur des **signaux de surface** (mots-clés, forme de la tâche) que même Haiku pattern-match, pas sur « est-ce que je me sens capable ».
- Ne classe **Lourd** que sur signal réel de complexité. Une tâche longue n'est pas une tâche lourde.

## Adapter la grille

Les IDs de modèles (Haiku 4.5, Sonnet 5, Opus 4.8, Fable 5) et le mapping tier→modèle sont volontairement explicites : mets-les à jour quand la gamme évolue. La logique — classer par signaux de surface, comparer au modèle actif, alerter en 3 lignes — reste stable.
