---
title: "Relire et nettoyer le code d'un agent"
description: "Lab : un agent a ajouté un export JSON et CSV à une petite API. Les tests passent. Vous relisez son code avec les principes du clean code (nommage, KISS, DRY, SOLID, complexité), vous listez les défauts, puis vous les corrigez sans casser le comportement attendu."
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

Un agent de code a ajouté une fonctionnalité d'export à `todo-api`, une petite API de tâches en Node.js. Ses tests passent, la fonctionnalité répond. Faut-il fusionner ? Dans ce lab, vous faites ce que ferait un relecteur : vous lisez le code, vous nommez chaque défaut avec le vocabulaire du **clean code**, puis vous le corrigez sur votre propre branche. Des tests d'acceptation fournis vérifient que votre version fait ce qu'on attend, y compris un cas que l'agent a cassé sans que ses tests le voient.

**Niveau** : débutant en qualité logicielle ; vous savez lire du JavaScript et utiliser Git (cloner, changer de branche, commiter).

**Ce qu'il faut** :
- Git ;
- Node.js 22.9 ou plus récent (`node --version` pour vérifier) ;
- un éditeur de code.

Aucun agent d'IA n'est nécessaire : le code de l'agent est fourni, sur une branche du dépôt.

**Vos fichiers** vivent dans `~/byte-layers/quality/todo-api`, sur une nouvelle branche `clean-export`, et votre relecture dans `~/byte-layers/quality/review.md`. Les dossiers sont gardés à la fin : le lab suivant mesure la dette de ces mêmes branches avec des outils.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
