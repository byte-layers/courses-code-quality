---
title: "04 - Écrire les tests qui tuent les mutants"
description: "Ajouter trois tests, un par survivant, jusqu'à un score de mutation de 100 %"
---

Chaque mutant survivant désigne un comportement que personne ne vérifie. Votre travail : un test par survivant, dans `test/todos.test.js`. Un bon test échoue sur le mutant et passe sur le vrai code.

## 04.1 - À vous d'écrire

Ajoutez trois tests à la fin de `test/todos.test.js`, en suivant le modèle des tests existants (`test('…', () => { … });`, `createStore()`, `assert`) :

| Test à écrire | Ce qu'il vérifie | Le mutant qu'il tue |
|---|---|---|
| un titre qui n'est pas du texte | `createStore().add(42)` lève l'erreur `title is required` | ligne 9 |
| la valeur limite | un titre de `MAX_TITLE_LENGTH` caractères exactement est **accepté** | ligne 11 |
| des identifiants croissants | deux tâches ajoutées reçoivent les identifiants 1 puis 2 | ligne 12 |

Lancez `npm test` après chaque test ajouté : il doit passer sur le vrai code.

```hint
Le premier se calque sur `add refuses an empty title`, avec `42` à la place de `'   '`. Pour le deuxième, `'x'.repeat(MAX_TITLE_LENGTH)` fabrique un titre de 200 caractères ; vérifiez la longueur du titre renvoyé par `add`. Pour le troisième, appelez deux fois `store.add(…)` sur le même `store` et comparez les `id`.
```

````solution
```js
test('add refuses a title that is not a string', () => {
  assert.throws(() => createStore().add(42), /title is required/);
});

test('add accepts a title of exactly MAX_TITLE_LENGTH characters', () => {
  const todo = createStore().add('x'.repeat(MAX_TITLE_LENGTH));
  assert.equal(todo.title.length, MAX_TITLE_LENGTH);
});

test('add gives increasing ids', () => {
  const store = createStore();
  assert.equal(store.add('Premier').id, 1);
  assert.equal(store.add('Second').id, 2);
});
```
````

Ce contrôle rejoue vos tests sur les trois mutants, dans une copie temporaire du projet : chacun doit faire échouer au moins un test.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
node --test >/dev/null 2>&1 || { echo "Des tests échouent sur le vrai code : lancez npm test ; un bon test passe sur le code d'origine."; exit 1; }
check() {
  T=$(mktemp -d); cp -r src test package.json "$T"/
  sed -i.bak "$1" "$T/src/todos.js"
  if (cd "$T" && node --test >/dev/null 2>&1); then rm -rf "$T"; echo "Aucun test n'échoue quand $2 : le test « $3 » manque ou ne vérifie pas assez."; exit 1; fi
  rm -rf "$T"
}
check "s/typeof title === 'string' ? title.trim()/true ? title.trim()/" "un titre qui n'est pas du texte n'est plus refusé proprement" "titre qui n'est pas du texte"
check "s/clean.length > MAX_TITLE_LENGTH/clean.length >= MAX_TITLE_LENGTH/" "un titre de MAX_TITLE_LENGTH caractères est refusé" "valeur limite"
check "s/nextId++/nextId--/" "les identifiants décroissent" "identifiants croissants"
```

## 04.2 - Relancer Stryker

Relancez l'analyse :

```exec
cd ~/byte-layers/quality/todo-api && npx -y -p @stryker-mutator/core@10.1.1 stryker run
```

Le score de `todos.js` passe à `100.00`, avec `0` dans la colonne `# survived`. Si un mutant survit encore, relisez la ligne `+` qu'affiche Stryker : elle dit quel comportement votre test ne vérifie pas.

La couverture, elle, n'a pas bougé : elle était déjà à 100 %. Vos trois tests n'exécutent aucune ligne nouvelle ; ils **vérifient** des comportements que les anciens tests exécutaient sans les contrôler.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
git diff --quiet origin/main -- src/todos.js || { echo "src/todos.js a été modifié : le but est d'ajouter des tests, pas de changer le code. Lancez git restore --source=origin/main src/todos.js"; exit 1; }
node --test >/tmp/bl-mutation-tests.log 2>&1 || { grep "^not ok" /tmp/bl-mutation-tests.log; echo "Des tests échouent sur le vrai code (liste ci-dessus) : un bon test doit passer sur le code d'origine."; exit 1; }
npx -y -p @stryker-mutator/core@10.1.1 stryker run >/tmp/bl-stryker.log 2>&1 || { tail -15 /tmp/bl-stryker.log; echo "Stryker a échoué (détail ci-dessus)."; exit 1; }
S=$(grep -E "^ *todos\.js *\|" /tmp/bl-stryker.log | tail -1 | awk -F'|' '{gsub(/ /,"",$2); print $2}')
[ "$S" = "100.00" ] || { grep -A2 "\[Survived\]" /tmp/bl-stryker.log | head -12; echo "Le score de mutation de todos.js est $S au lieu de 100.00 : les mutants ci-dessus survivent encore."; exit 1; }
```

## 04.3 - Commiter

Les nouveaux tests et la configuration de Stryker rejoignent la branche :

```exec
cd ~/byte-layers/quality/todo-api && git add test/todos.test.js stryker.config.json .gitignore && git commit -m "Renforce les tests de todos.js : score de mutation 100 %"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git add test/todos.test.js stryker.config.json .gitignore` | prépare les tests, la configuration et le `.gitignore` modifié |
> | `git commit -m "…"` | les enregistre sur la branche `stronger-tests` |

Si Git répond `Please tell me who you are`, il ne connaît pas encore votre nom : `git config --global user.name "Votre Nom"` et `git config --global user.email "vous@exemple.fr"`, puis relancez la commande.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
git cat-file -e stronger-tests:stryker.config.json 2>/dev/null || { echo "Le commit de stronger-tests ne contient pas stryker.config.json : lancez la commande de la section 04.3."; exit 1; }
[ -z "$(git status --porcelain test stryker.config.json .gitignore)" ] || { echo "Des modifications ne sont pas commitées : lancez la commande de la section 04.3."; exit 1; }
```

## 04.4 - Et avec un agent ?

Ce lab donne une méthode pour juger les tests qu'écrit un agent de code : passez-les à Stryker. Un agent à qui l'on demande « écris des tests pour `todos.js` » produit souvent des tests qui couvrent tout et vérifient peu ; le score de mutation le montre en quelques secondes, là où la couverture affiche fièrement 100 %.

Deux limites à garder en tête :

- **Le coût.** Avec le lanceur `command` utilisé ici, chaque mutant relance toute la suite de tests ; les lanceurs dédiés de Stryker (Jest, Vitest, Mocha…) ne relancent que les tests qui exécutent la ligne mutée, mais l'analyse reste bien plus longue qu'une simple exécution des tests. Sur un gros projet, on mute les modules importants, ou seulement les fichiers modifiés par une merge request, pas tout le code à chaque commit.
- **Les mutants équivalents.** Certains mutants ne changent rien au comportement observable ; aucun test ne peut les tuer. Un score de 100 % n'est pas toujours atteignable, ni utile : chaque survivant se lit et se juge.

```acknowledge
J'ai compris ce que le score de mutation ajoute à la couverture, et ses limites.
```

## Récap

- `src/todos.js` était couvert à 100 %, et trois bugs y seraient pourtant passés inaperçus.
- Un **mutant** est une copie du code avec un petit bug ; un test utile le **tue** en échouant.
- **Stryker** crée les mutants, lance les tests sur chacun, et donne le **score de mutation**.
- Chaque survivant indique un comportement non vérifié : une valeur limite, un type inattendu, un ordre.
- Écrire les tests qui tuent les survivants ne change pas la couverture, mais rend les tests vraiment utiles.
