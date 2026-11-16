---
title: "Les tests dans la qualité : couverture, mutation et agents"
description: "Ce que les tests apportent à la qualité, la pyramide des tests, ce que la couverture mesure et ne mesure pas, les tests complaisants qu'écrivent souvent les agents, les tests de mutation, et le TDD avec un agent de code."
tags:
  - bit
  - quality
difficulty: beginner
duration: 15
requires: "0.4.0"
---

« Tous les tests passent. » La phrase rassure, mais elle ne dit rien de ce que vérifient ces tests. Un agent de code produit des tests en quelques secondes, et ils passent presque toujours : c'est justement le problème. Ce bit replace les tests dans la qualité, explique ce que mesurent la couverture et les tests de mutation, puis montre comment travailler en TDD avec un agent sans le laisser écrire ses propres examens.

**Pour qui** : un développeur qui a déjà écrit ou lancé quelques tests automatisés.

> 📅 **Sources vérifiées le 16 novembre 2026.**

# Les tests dans la qualité : couverture, mutation et agents

## 1 - Ce que les tests apportent

Un test automatisé est un petit programme qui appelle votre code et vérifie le résultat. Il sert la qualité de deux façons :

| Apport | Lien avec le [modèle ISO/IEC 25010](https://byte-layers.com/fr/bits/iso-25010-quality-model) |
|---|---|
| vérifier que le code fait ce qui est attendu | l'**adéquation fonctionnelle**, en particulier la justesse |
| permettre de modifier le code sans peur | la **maintenabilité** : un code sans tests est un code que personne n'ose remanier |

On range souvent les tests en trois niveaux, en **pyramide** : beaucoup de tests rapides en bas, peu de tests lents en haut.

| Niveau | Ce qu'il teste | Vitesse | Exemple |
|---|---|---|---|
| **unitaire** | une fonction ou un module, isolé | quelques millisecondes | `toCsv` met bien une virgule entre les champs |
| **intégration** | plusieurs modules ensemble, ou avec une base, un réseau | quelques dixièmes de seconde | `POST /todos` puis `GET /todos` renvoie la tâche créée |
| **bout en bout** | l'application entière, comme un utilisateur, souvent dans un navigateur | plusieurs secondes | un utilisateur se connecte, crée une tâche, la voit dans la liste |

La forme en pyramide est un conseil d'économie : un test de bout en bout rassure beaucoup, mais il est lent, fragile et coûteux à maintenir ; on en garde quelques-uns pour les parcours essentiels.

## 2 - La couverture : ce qui est exécuté, pas ce qui est vérifié

La **couverture de code** mesure la part du code exécutée pendant les tests : 80 % des lignes, 70 % des branches des `if`. Node.js la calcule avec `node --test --experimental-test-coverage`.

Elle a une limite essentielle : une ligne **exécutée** n'est pas une ligne **vérifiée**. Ce test couvre toute la fonction d'export, et ne vérifie presque rien :

```js
test('export works', async () => {
  const body = await (await fetch(`${base}/todos/export`)).json();
  assert.ok(body.todos.length > 0); // n'importe quel contenu passe
});
```

Ce que disent les références :

- Martin Fowler : la couverture sert à **trouver le code non testé**, pas à juger la qualité des tests. Un objectif chiffré imposé pousse à écrire des tests pour le chiffre.
- Google publie des repères, 60 % « acceptable », 75 % « louable », 90 % « exemplaire », tout en refusant un chiffre idéal imposé d'en haut. Le même article note qu'au-delà d'un certain point, chaque point gagné rapporte de moins en moins, et conseille les **tests de mutation** pour juger les tests eux-mêmes.

Une couverture basse est un vrai signal : des pans entiers du code ne sont jamais exécutés. Une couverture haute, seule, ne prouve rien.

## 3 - Les tests complaisants

Un test **complaisant** passe quoi que fasse le code. Les agents de code en écrivent souvent, parce qu'on leur demande que « les tests passent » et qu'ils l'obtiennent par le chemin le plus court. Les signes :

| Signe | Exemple |
|---|---|
| une assertion vague | `assert.ok(result)`, `length > 0`, « ne lève pas d'erreur » |
| l'attendu recopié depuis le résultat | l'agent lance le code, copie la sortie dans le test : un bug devient la référence |
| tout est simulé | le test remplace la base, le réseau et le module testé par des faux : il ne teste que les faux |
| un test modifié pour passer | le test échouait, l'agent a changé l'attendu au lieu du code |
| un test désactivé | `test.skip`, ou un test commenté « temporairement » |
| seul le cas heureux | aucun titre vide, aucune valeur limite, aucune erreur |

Le dernier cas est celui de [CrowdStrike](https://byte-layers.com/fr/bits/crowdstrike-outage) : tous les tests utilisaient un joker sur le 21e champ, si bien qu'aucun ne pouvait déclencher le défaut.

## 4 - Les tests de mutation : tester les tests

L'idée est simple : si l'on introduit un petit bug dans le code et que les tests passent encore, c'est que les tests ne vérifient pas cet endroit.

Un outil de **tests de mutation** le fait automatiquement. Il crée des **mutants** : des copies du code avec une seule petite modification, par exemple `>` remplacé par `>=`, `+` par `-`, une condition remplacée par `true`. Il lance les tests sur chaque mutant :

| Résultat | Ce que ça veut dire |
|---|---|
| un test échoue : le mutant est **tué** | les tests protègent cet endroit |
| tous les tests passent : le mutant **survit** | un bug à cet endroit passerait inaperçu |

Le **score de mutation** est la part des mutants tués. Pour JavaScript, l'outil de référence est **Stryker**. Le prix : chaque mutant relance des tests (toute la suite avec un lanceur générique, seulement ceux qui exécutent la ligne mutée avec un lanceur dédié), donc l'analyse est bien plus lente que la couverture. On la lance sur les modules importants, ou de temps en temps, plutôt qu'à chaque commit sur tout le projet.

Un mutant qui survit n'est pas toujours un problème : certains ne changent rien au comportement observable (on parle de mutants équivalents). Chaque survivant se lit, et se juge.

## 5 - Le TDD avec un agent

Le **TDD** (*Test-Driven Development*), décrit par Kent Beck, inverse l'ordre habituel : on écrit d'abord un test qui échoue, puis le code le plus simple qui le fait passer, puis on remanie. Le cycle est souvent résumé en trois couleurs : rouge, vert, remanier.

Avec un agent de code, le TDD change de sens : le test devient la **spécification** que l'agent doit respecter. La règle qui le rend efficace : **l'humain est maître des tests, l'agent écrit le code.**

1. Vous écrivez le test, ou vous relisez attentivement celui que l'agent propose, avant tout code.
2. Vous vérifiez qu'il échoue, pour la bonne raison.
3. Vous demandez à l'agent de faire passer le test **sans modifier les tests**.
4. Vous relisez le diff, en commençant par les fichiers de test : s'ils ont changé, c'est suspect.

La consigne « ne modifie pas les tests » peut être ignorée. Une barrière technique est plus sûre : une permission qui refuse l'écriture dans `test/`, ou un hook qui bloque toute modification de ces fichiers, comme le montre le lab [Hooks : demander ou garantir](https://byte-layers.com/fr/labs/agent-hooks).

## Récap

- Les tests servent la justesse fonctionnelle et la **maintenabilité** ; la **pyramide** conseille beaucoup de tests unitaires et peu de tests de bout en bout.
- La **couverture** mesure le code exécuté, pas le code vérifié : elle sert à trouver le code non testé.
- Un test **complaisant** passe quoi que fasse le code : assertion vague, attendu recopié, tout simulé, cas limites oubliés.
- Les **tests de mutation** introduisent de petits bugs et regardent si les tests les détectent ; Stryker le fait pour JavaScript.
- En **TDD avec un agent**, l'humain garde la main sur les tests, et une barrière technique empêche l'agent de les modifier.

> 📖 [Martin Fowler, « Test Coverage » (en anglais)](https://martinfowler.com/bliki/TestCoverage.html) · [Ham Vocke, « The Practical Test Pyramid », sur le site de Martin Fowler (en anglais)](https://martinfowler.com/articles/practical-test-pyramid.html) · [Google Testing Blog, « Code Coverage Best Practices », 2020 (en anglais)](https://testing.googleblog.com/2020/08/code-coverage-best-practices.html) · [Stryker Mutator (en anglais)](https://stryker-mutator.io/docs/stryker-js/introduction/) · [Martin Fowler, « Test Driven Development » (en anglais)](https://martinfowler.com/bliki/TestDrivenDevelopment.html)

```quiz
title: Les tests dans la qualité
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Un module a 100 % de couverture de lignes. Que peut-on en conclure ?
    answers:
      - "Qu'il n'a pas de bug"
      - "Que ses tests sont de bonne qualité"
      - "Que chaque ligne est exécutée par au moins un test, sans savoir si le résultat est vérifié"
      - "Qu'il n'a plus besoin de tests de mutation"
    correct: 3
    explanation: |
      La couverture mesure l'exécution. Un test qui appelle le code sans rien vérifier
      augmente la couverture autant qu'un bon test.
  - prompt: Un test vérifie seulement `assert.ok(body.todos.length > 0)`. Quel défaut a-t-il ?
    answers:
      - "Il est trop lent"
      - "Il teste trop de choses"
      - "Il n'en a aucun"
      - "C'est un test complaisant : une assertion vague, qui passe avec presque n'importe quel contenu"
    correct: 4
    explanation: |
      Une liste qui contient une mauvaise tâche, ou une tâche en moins, passe aussi. Le test
      doit vérifier le contenu attendu.
  - prompt: Un outil de tests de mutation remplace `>` par `>=` dans votre code, et tous les tests passent. Que s'est-il passé ?
    answers:
      - "Le mutant a été tué, tout va bien"
      - "Le mutant a survécu : aucun test ne vérifie la valeur limite à cet endroit"
      - "L'outil a un bug"
      - "Le code est faux"
    correct: 2
    explanation: |
      Un mutant qui survit montre un endroit où un bug passerait inaperçu. Ici, il manque un
      test sur la valeur exacte de la limite.
  - prompt: Dans quel niveau de la pyramide des tests faut-il le plus de tests ?
    answers:
      - "Les tests de bout en bout"
      - "Les tests manuels"
      - "Les tests unitaires, rapides et peu coûteux"
      - "Autant à chaque niveau"
    correct: 3
    explanation: |
      Les tests unitaires s'exécutent en millisecondes et cassent rarement sans raison. Les
      tests de bout en bout, lents et fragiles, se réservent aux parcours essentiels.
  - prompt: En TDD avec un agent de code, quelle règle rend la démarche efficace ?
    answers:
      - "Laisser l'agent écrire les tests et le code, pour aller plus vite"
      - "Écrire le code d'abord, puis demander à l'agent les tests"
      - "Désactiver les tests qui échouent"
      - "L'humain garde la main sur les tests ; l'agent fait passer les tests sans les modifier, et une barrière technique l'en empêche"
    correct: 4
    explanation: |
      Le test est la spécification. Si l'agent peut la modifier, il peut faire passer n'importe
      quel code.
```
