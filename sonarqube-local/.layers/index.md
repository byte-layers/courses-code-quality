---
title: "SonarQube sur son poste : profil et quality gate"
description: "Lab optionnel : lancer SonarQube Community Build dans Docker, analyser les trois versions du projet, constater que les réglages par défaut ne voient rien dans le code de l'agent, puis écrire un profil et une quality gate qui l'arrêtent."
tags:
  - lab
  - quality
difficulty: beginner
duration: 60
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Le bit [Les outils d'analyse de code et leurs alternatives](https://byte-layers.com/fr/bits/code-analysis-tools) présente SonarQube, la plateforme de qualité la plus répandue en entreprise. Dans ce lab **optionnel**, vous la lancez sur votre poste, dans un conteneur Docker, et vous analysez les trois versions du projet mesurées au lab [Mesurer la dette avec Semgrep, ESLint et jscpd](https://byte-layers.com/fr/labs/measure-debt).

Vous verrez qu'un SonarQube tout juste installé laisse passer le code de l'agent : ses réglages par défaut sont volontairement prudents. Vous écrirez alors deux réglages, un **profil** (les règles appliquées) et une **quality gate** (les conditions du verdict), pour qu'il arrête ce code comme les outils du lab précédent.

**Niveau** : débutant en qualité logicielle ; vous savez utiliser un terminal et lancer un conteneur Docker.

**Ce qu'il faut** :
- le lab [Mesurer la dette avec Semgrep, ESLint et jscpd](https://byte-layers.com/fr/labs/measure-debt) terminé : les trois versions sont dans `~/byte-layers/quality/measure/main`, `agent` et `clean` ;
- **Docker**, avec au moins 4 Go de mémoire disponibles pour les conteneurs, et environ 2 Go de téléchargement ;
- Node.js 22.13 ou plus récent, pour mesurer la couverture des tests ;
- un navigateur, pour l'interface de SonarQube.

**Vos fichiers** restent dans `~/byte-layers/quality`. SonarQube tourne dans un conteneur nommé `sonarqube`, qui écoute sur le port 9000 de votre poste : si ce port est déjà pris, arrêtez d'abord le programme qui l'occupe.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
