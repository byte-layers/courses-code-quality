---
title: "Tester les tests avec la mutation"
description: "Lab : un module couvert à 100 % par ses tests, et pourtant trois bugs passeraient inaperçus. Vous les trouvez à la main, puis avec Stryker, et vous écrivez les tests qui les attrapent."
tags:
  - lab
  - quality
difficulty: beginner
duration: 45
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Le module `src/todos.js` du projet `todo-api` gère le stockage des tâches. Ses tests passent, et ils exécutent **toutes** ses lignes : 100 % de couverture. Sont-ils bons pour autant ? Dans ce lab, vous introduisez vous-même un petit bug pour voir si les tests le remarquent, puis vous laissez un outil, **Stryker**, en introduire des dizaines. Ceux qui passent inaperçus vous disent quels tests manquent ; vous les écrivez.

Le bit [Les tests dans la qualité](https://byte-layers.com/fr/bits/tests-and-quality) explique la couverture et les tests de mutation ; ce lab les met en pratique.

**Niveau** : débutant en qualité logicielle ; vous savez lire du JavaScript et utiliser Git.

**Ce qu'il faut** :
- Git ;
- Node.js 22.9 ou plus récent (`node --version` pour vérifier), avec `npx`, installé avec Node ;
- un accès à Internet, pour que `npx` télécharge Stryker au premier lancement.

**Vos fichiers** vivent dans `~/byte-layers/quality/todo-api`, le projet du lab [Relire et nettoyer le code d'un agent](https://byte-layers.com/fr/labs/clean-code-agent-diff), sur une nouvelle branche `stronger-tests`. Si vous n'avez pas fait ce lab, la première étape clone le projet. Les dossiers sont gardés à la fin.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
