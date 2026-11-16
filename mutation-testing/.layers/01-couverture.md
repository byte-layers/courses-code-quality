---
title: "01 - Mesurer la couverture"
description: "Préparer une branche de travail, puis voir que les tests exécutent toutes les lignes du module"
---

On commence par la mesure la plus répandue : la couverture. Cette étape est un **support** : toutes les commandes sont fournies.

## 01.1 - Une branche de travail

Le projet est peut-être déjà sur votre poste, si vous avez fait le lab précédent. Dans ce cas, vérifiez d'abord qu'il n'a pas de modification en cours : `cd ~/byte-layers/quality/todo-api && git status --short` ne doit rien afficher. S'il affiche des fichiers, commitez-les sur leur branche, ou mettez-les de côté avec `git stash`.

Cette commande clone le projet seulement s'il manque, puis crée la branche `stronger-tests` à partir de `main`, la version d'origine du projet :

```exec
cd ~ && mkdir -p ~/byte-layers/quality && { [ -d ~/byte-layers/quality/todo-api ] || git clone https://github.com/byte-layers/todo-api.git ~/byte-layers/quality/todo-api; } && cd ~/byte-layers/quality/todo-api && git switch -c stronger-tests --no-track origin/main
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `mkdir -p ~/byte-layers/quality` | crée le dossier de travail du stack, s'il n'existe pas |
> | `[ -d ~/byte-layers/quality/todo-api ]` | teste si le projet est déjà là |
> | `\|\| git clone …` | ne clone que si le test a échoué, c'est-à-dire si le dossier manque |
> | `{ … ; }` | regroupe le test et le clonage, pour que la suite s'enchaîne dans les deux cas |
> | `git switch -c stronger-tests --no-track origin/main` | crée la branche `stronger-tests` à partir de la branche `main` du dépôt, et se place dessus |
> | `--no-track` | ne relie pas la nouvelle branche à `main` sur le dépôt distant : un futur `git push` ne visera pas `main` par erreur |

Si Git refuse avec `a branch named 'stronger-tests' already exists`, la branche existe d'un essai précédent : `cd ~/byte-layers/quality/todo-api && git switch stronger-tests` suffit.

## 01.2 - La couverture

Node.js mesure la couverture sans rien installer, avec l'option `--experimental-test-coverage` :

```exec
cd ~/byte-layers/quality/todo-api && node --test --experimental-test-coverage
```

Après les résultats des tests, un tableau donne, pour chaque fichier, la part des lignes, des branches et des fonctions exécutées :

```text
# file            | line % | branch % | funcs % | uncovered lines
# src             |        |          |         |
#  server.js      |  77.50 |    52.94 |   75.00 | 27-29 31-33 38-40
#  todos.js       | 100.00 |   100.00 |  100.00 |
# test            |        |          |         |
#  server.test.js | 100.00 |   100.00 |  100.00 |
#  todos.test.js  | 100.00 |   100.00 |  100.00 |
# all files       |  92.80 |    81.82 |   95.00 |
```

La ligne de `todos.js` est à 100 % partout : chaque ligne, chaque branche de chaque `if` et chaque fonction sont exécutées au moins une fois. Ouvrez `src/todos.js` et `test/todos.test.js` : cinq tests, pour quatre fonctions, `add`, `list`, `complete` et `remove`. Une partie de `todos.js` est aussi exécutée par les tests du serveur, dans `test/server.test.js` : par exemple, `POST /todos` avec un corps vide appelle `add` sans titre. La question de ce lab : tous ces tests vérifient-ils vraiment ce qu'ils exécutent ?

> 📖 [La couverture de code dans Node.js (en anglais)](https://nodejs.org/api/test.html#collecting-code-coverage)

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas : lancez la commande de la section 01.1."; exit 1; }
[ "$(git branch --show-current)" = "stronger-tests" ] || { echo "Placez-vous sur la branche stronger-tests : git switch stronger-tests"; exit 1; }
git diff --quiet origin/main -- src test || { echo "src ou test diffèrent de main : la branche stronger-tests doit partir de main, sans modification. Mettez vos changements de côté (git stash) et recréez la branche."; exit 1; }
```
