---
title: "02 - Un mutant à la main"
description: "Introduire volontairement un petit bug, et constater que les tests ne le voient pas"
---

Un test utile échoue quand le code devient faux. Pour le vérifier, le plus simple est de rendre le code faux exprès, et de regarder les tests. Ce code modifié s'appelle un **mutant**.

## 02.1 - Changer un seul caractère

Dans `src/todos.js`, la ligne 11 refuse les titres trop longs :

```js
    if (clean.length > MAX_TITLE_LENGTH) throw new Error(`title is longer than ${MAX_TITLE_LENGTH} characters`);
```

`MAX_TITLE_LENGTH` vaut 200 : un titre de 200 caractères est accepté, un titre de 201 est refusé. Remplaçons `>` par `>=` : désormais, un titre de 200 caractères pile serait refusé. C'est un bug. La commande `sed` fait le remplacement dans le fichier, puis on lance les tests :

```exec
cd ~/byte-layers/quality/todo-api && sed -i.bak 's/clean.length > MAX_TITLE_LENGTH/clean.length >= MAX_TITLE_LENGTH/' src/todos.js && grep -n "MAX_TITLE_LENGTH)" src/todos.js && npm test
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `sed -i.bak 's/avant/après/' src/todos.js` | remplace le texte `avant` par `après` dans le fichier ; `-i.bak` modifie le fichier sur place et garde une copie `src/todos.js.bak` (cette forme marche sous Linux, macOS et Git Bash) |
> | `grep -n "MAX_TITLE_LENGTH)" src/todos.js` | affiche la ligne modifiée, avec son numéro, pour vérifier le remplacement |
> | `npm test` | lance les tests du projet |

La ligne affichée contient bien `>=`, et les tests passent tous : 7 réussis, aucun échec. Le bug est invisible pour eux. Le test `add refuses a title that is too long` essaie 201 caractères, refusés dans les deux versions ; aucun test n'essaie 200 caractères, la **valeur limite**.

## 02.2 - Remettre le code en état

On annule le mutant : `git restore` remet le fichier tel qu'il est dans le dernier commit, et on supprime la copie de `sed` :

```exec
cd ~/byte-layers/quality/todo-api && git restore src/todos.js && rm -f src/todos.js.bak && git status --short
```

`git status --short` n'affiche plus rien : le projet est revenu à l'état du commit.

Faire cela à la main pour chaque opérateur de chaque ligne serait interminable. C'est le travail d'un outil de tests de mutation.

> 📖 [git restore (en anglais)](https://git-scm.com/docs/git-restore)

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
grep -q "clean.length >= MAX_TITLE_LENGTH" src/todos.js && { echo "Le mutant est encore dans src/todos.js : lancez la commande de la section 02.2."; exit 1; }
[ -f src/todos.js.bak ] && { echo "La copie src/todos.js.bak existe encore : lancez la commande de la section 02.2."; exit 1; }
git diff --quiet src/todos.js || { echo "src/todos.js est modifié : lancez git restore src/todos.js"; exit 1; }
```
