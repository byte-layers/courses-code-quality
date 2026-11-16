---
title: "05 - Ce que la barrière garantit, et ce qu'elle ne garantit pas"
description: "Les limites d'une barrière qualité, la revue par une IA, et le pipeline sans les minutes de gitlab.com"
---

Le code de l'agent est arrêté, le vôtre est fusionné. Reste à savoir ce que vaut vraiment cette barrière.

## 05.1 - Les limites

| Limite | Ce qu'on en fait |
|---|---|
| La barrière ne vaut que par ses règles : elle n'a pas vu le bug des tâches de même titre | les tests et la relecture humaine restent nécessaires ; une règle Semgrep ou un test s'ajoute à chaque défaut trouvé à la main |
| Un propriétaire peut encore envoyer directement sur `main`, comme à la section 03.2 | protéger `main` : **Settings**, **Repository**, **Protected branches**, *Allowed to push and merge* sur *No one*, et *Allowed to merge* sur *Maintainers* ; tout changement passe alors par une merge request |
| Le seuil de complexité vaut 20, une tolérance au-dessus de la dette d'origine | le baisser par une merge request, une fois `createServer` découpée : le pipeline vérifie que le code suit |
| Une règle trop stricte ou fausse bloque tout le monde | on corrige la règle par une merge request, plutôt que de désactiver la barrière |

## 05.2 - Et la revue par une IA ?

GitLab propose une revue des merge requests par son assistant, GitLab Duo, dans ses offres payantes. Avec l'agent installé dans le stack [Coder avec un agent d'IA](https://byte-layers.com/fr/stacks/ai-coding-agents), vous pouvez faire la même chose sur votre poste : lui donner le diff de la merge request, avec la grille de relecture du lab [Relire et nettoyer le code d'un agent](https://byte-layers.com/fr/labs/clean-code-agent-diff), puis coller sa réponse en commentaire dans la merge request.

La place de cette revue est à côté de la barrière, pas dedans. Une IA ne répond pas deux fois la même chose, et peut se tromper avec assurance : elle **conseille** un relecteur humain. La barrière, elle, doit être **déterministe** : même code, même verdict.

## 05.3 - Sans les minutes de gitlab.com

Si vos minutes de calcul gratuites sont épuisées, ou si votre code ne doit pas quitter votre réseau, un **runner** installé sur votre poste peut exécuter le même `.gitlab-ci.yml`, sans changer le fichier. Deux réglages sont nécessaires : le runner utilise l'exécuteur **Docker**, puisque les jobs demandent une image (`image:`), et les runners partagés sont désactivés pour le projet (**Settings**, **CI/CD**, **Runners**), pour que GitLab envoie les jobs au vôtre. Le bit [GitLab Runner](https://byte-layers.com/fr/bits/gitlab-runner) explique ce qu'est un runner et comment GitLab choisit celui qui exécute un job.

La merge request du code de l'agent reste ouverte : fermez-la avec **Close merge request**, elle a rempli son rôle.

```acknowledge
J'ai compris ce que la barrière garantit, ce qu'elle laisse aux tests et à la relecture, et où placer une revue par une IA.
```

## Récap

- Une **barrière qualité** fait tourner les outils sur chaque merge request, et GitLab refuse la fusion tant qu'ils échouent (**Pipelines must succeed**).
- Le pipeline ne voit que le dépôt : outils dans `package.json` et `package-lock.json`, configuration versionnée, `npm ci` pour installer exactement les mêmes versions.
- Un job par outil donne un verdict lisible ; `artifacts: reports: codequality` affiche les défauts d'ESLint dans l'onglet **Reports** de la merge request, comparés à ceux de `main`.
- Une merge request lance le pipeline de **sa** branche : une branche plus ancienne que le pipeline doit d'abord récupérer `main`.
- La barrière arrête les défauts de forme ; les bugs de sens restent l'affaire des tests et de la relecture. Une revue par une IA conseille, elle ne décide pas.

> 📖 [GitLab : les branches protégées (en anglais)](https://docs.gitlab.com/user/project/repository/branches/protected/) · [GitLab : Code Quality (en anglais)](https://docs.gitlab.com/ci/testing/code_quality/) · [GitLab Duo (en anglais)](https://docs.gitlab.com/user/gitlab_duo/)
