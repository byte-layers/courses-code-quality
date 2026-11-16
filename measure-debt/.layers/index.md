---
title: "Mesurer la dette avec Semgrep, ESLint et jscpd"
description: "Lab : mesurer trois versions du même projet (avant l'agent, code de l'agent, code corrigé) avec trois outils sans serveur : ESLint et sonarjs pour la complexité, jscpd pour la duplication, Semgrep pour des règles propres au projet. Choisir ses seuils, puis tout mesurer d'une commande."
tags:
  - lab
  - quality
difficulty: beginner
duration: 75
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Au lab [Relire et nettoyer le code d'un agent](https://byte-layers.com/fr/labs/clean-code-agent-diff), vous avez trouvé les défauts du code de l'agent en le lisant. Une relecture humaine prend du temps, et dépend de l'attention du relecteur. Dans ce lab, vous confiez une partie de ce travail à trois outils, qui tournent en quelques secondes sur votre poste :

| Outil | Ce qu'il mesure |
|---|---|
| **ESLint**, avec le plugin **sonarjs** | des erreurs fréquentes, et la **complexité cognitive** de chaque fonction |
| **jscpd** | la **duplication** : le code copié-collé |
| **Semgrep** | des **règles** écrites pour votre projet, comme « pas de jeton écrit en dur » |

Vous les passez sur trois versions du même projet : `main` (avant l'agent), `quality/agent-export` (le code de l'agent) et votre branche `clean-export`. Vous choisissez un seuil pour chaque outil, puis un petit script mesure tout d'une commande. Le lab suivant en fera une barrière dans le pipeline.

**Niveau** : débutant en qualité logicielle ; vous savez utiliser un terminal, Git et npm.

**Ce qu'il faut** :
- le lab [Relire et nettoyer le code d'un agent](https://byte-layers.com/fr/labs/clean-code-agent-diff) terminé : le projet `~/byte-layers/quality/todo-api` a une branche `clean-export` commitée ;
- Node.js 22.13 ou plus récent (ou 20.19, ou 24 et plus), la version qu'exige ESLint 10, avec npm ;
- **Docker**, pour lancer Semgrep sans l'installer ; à défaut, Semgrep installé avec `pipx install semgrep` ou `brew install semgrep` (l'étape 04 donne la commande équivalente) ;
- un accès à Internet, pour télécharger les outils.

**Un éditeur de code**, par exemple VS Code, pour modifier les fichiers de configuration : `code ~/byte-layers/quality` ouvre tout le dossier, fichiers cachés compris.

**Vos fichiers** vivent dans `~/byte-layers/quality` : la configuration des outils à la racine, et les trois versions du projet dans `~/byte-layers/quality/measure`. Les dossiers sont gardés à la fin.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
