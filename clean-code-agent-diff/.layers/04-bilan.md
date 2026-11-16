---
title: "04 - Comparer et prendre du recul"
description: "Mettre côte à côte le code de l'agent et le vôtre, et voir ce que la relecture n'a pas réglé"
---

Deux versions de la même fonctionnalité existent maintenant : celle de l'agent, sur `quality/agent-export`, et la vôtre, sur `clean-export`. Elles passent toutes deux leurs tests. Qu'est-ce qui a vraiment changé ?

## 04.1 - Les deux versions côte à côte

`git diff --stat` résume les changements entre deux branches. On compare chacune à `main`, le projet avant l'export :

```exec
cd ~/byte-layers/quality/todo-api && git diff --stat main quality/agent-export && git diff --stat main clean-export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git diff --stat main quality/agent-export` | résume ce que l'agent a ajouté par rapport à `main` |
> | `&&` | n'enchaîne la seconde commande que si la première a réussi |
> | `git diff --stat main clean-export` | résume ce que votre branche ajoute par rapport à `main` |

Ce que vous devez constater, avec la solution du lab :

| | Code de l'agent | Votre version |
|---|---|---|
| lignes ajoutées dans `src/server.js` | 69 | une vingtaine (21 avec la solution) |
| fichiers | tout dans le serveur | la mise en forme dans `src/export.js` |
| tâches de même titre | une seule exportée | toutes exportées |
| coût du filtrage | deux boucles imbriquées : 1 000 tâches, jusqu'à un million de comparaisons | un seul passage sur la liste |
| jeton | écrit dans le code, passé dans l'adresse | lu dans l'environnement, déclaré dans `.env.example`, passé dans un en-tête |
| tests | 8, dont un qui vérifie seulement qu'une liste n'est pas vide | 11, qui vérifient le contenu |

Le code de l'agent n'était pas « faux » au sens où il plantait : il répondait. Ses défauts touchent ce que le [modèle ISO/IEC 25010](https://byte-layers.com/fr/bits/iso-25010-quality-model) appelle la **maintenabilité** (lisibilité, facilité de modification), la **sécurité** et, pour le dédoublonnage, la **justesse fonctionnelle**. Les tests de l'agent ne mesuraient rien de tout cela.

## 04.2 - Ce que la relecture n'a pas réglé

Votre version n'est pas parfaite, et c'est voulu : un lab corrige ce qu'il cible.

- **La fonction `createServer` reste longue.** Ses `if` successifs, un par route, existaient avant l'agent : c'est une dette d'origine du projet. Le lab suivant la mesure avec un outil, et montre comment éviter qu'elle grossisse.
- **Le jeton est comparé avec `!==`.** Pour un vrai secret, on préfère une comparaison à durée constante, `crypto.timingSafeEqual` de Node, qui ne laisse pas deviner le jeton caractère par caractère en mesurant le temps de réponse.
- **Le CSV n'est pas protégé contre l'injection de formules.** Un titre qui commence par `=` peut être interprété comme une formule par un tableur. Pour un export ouvert dans Excel, on préfixe ces titres d'une apostrophe.

Une relecture humaine trouve ce que vous avez trouvé, à condition d'avoir le vocabulaire et le temps. Ce temps manque souvent : c'est pourquoi on confie une partie de cette relecture à des outils qui tournent à chaque merge request.

```acknowledge
J'ai comparé les deux versions et relevé ce que ma correction laisse encore à faire.
```

## Récap

- Des tests verts disent que le code fait ce que les tests demandent, pas qu'il est lisible, sûr ou juste dans tous les cas.
- Une relecture utile **nomme** chaque défaut : nommage, KISS, DRY, responsabilité unique, complexité, code mort, sécurité, bug.
- Le code de l'agent dupliquait la mise en forme CSV, mélangeait HTTP et mise en forme, laissait du code mort, écrivait un secret en dur, et supprimait silencieusement des tâches.
- On corrige sur une branche, avec des **tests d'acceptation** comme filet ; un changement de contrat entraîne un changement de test.
- Certaines dettes précèdent l'agent : il faut les mesurer pour les suivre.

> 📖 [Martin Fowler, « Refactoring » (en anglais)](https://refactoring.com/) · [OWASP, CSV Injection (en anglais)](https://owasp.org/www-community/attacks/CSV_Injection) · [crypto.timingSafeEqual (en anglais)](https://nodejs.org/api/crypto.html#cryptotimingsafeequala-b)
