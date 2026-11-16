---
title: "03 - Corriger le code de l'agent"
description: "Extraire la mise en forme dans un module, supprimer les répétitions, le code mort et le secret écrit en dur, sans casser le comportement attendu"
---

Votre relecture liste les défauts ; il reste à les corriger. Vous travaillez sur votre propre branche, et des **tests d'acceptation** fournis disent ce que l'export doit faire : ils serviront de filet pendant que vous transformez le code.

## 03.1 - Votre branche

On part du code de l'agent, sur une nouvelle branche `clean-export` : la branche `quality/agent-export` reste intacte, pour comparer à la fin.

```exec
cd ~/byte-layers/quality/todo-api && git switch -c clean-export quality/agent-export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch -c clean-export` | crée la branche `clean-export` et se place dessus |
> | `quality/agent-export` | le point de départ de la nouvelle branche : le code de l'agent |

## 03.2 - Les tests d'acceptation

Ces tests décrivent l'export attendu, fonction par fonction. Ils visent un nouveau module, `src/export.js`, qui n'existe pas encore : la mise en forme sort du serveur HTTP, comme le demande le principe de responsabilité unique. Cette section est un **support** : le fichier est fourni complet.

```exec
cd ~/byte-layers/quality/todo-api && cat > test/export.test.js <<'FIN'
import { test } from 'node:test';
import assert from 'node:assert/strict';
import { filterByStatus, toCsv } from '../src/export.js';

const todos = [
  { id: 1, title: 'Acheter du pain', done: true },
  { id: 2, title: 'Acheter du pain', done: false },
  { id: 3, title: 'Lire "Clean Code", chapitre 2', done: false },
];

test('filterByStatus keeps every todo when no status is given, even with the same title', () => {
  assert.equal(filterByStatus(todos, null).length, 3);
});

test('filterByStatus keeps only done or only open todos', () => {
  assert.deepEqual(filterByStatus(todos, 'done').map((t) => t.id), [1]);
  assert.deepEqual(filterByStatus(todos, 'todo').map((t) => t.id), [2, 3]);
});

test('toCsv writes a header and escapes quotes and commas', () => {
  assert.equal(
    toCsv(todos),
    'id,title,done\n1,Acheter du pain,true\n2,Acheter du pain,false\n3,"Lire ""Clean Code"", chapitre 2",false\n',
  );
});
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > test/export.test.js <<'FIN'` … `FIN` | écrit les lignes entre les deux `FIN` dans le fichier `test/export.test.js` |
> | les guillemets autour de `'FIN'` | empêchent le terminal d'interpréter les `$` ou les `"` du contenu |

Ce que vérifient les trois tests :

| Test | L'exigence |
|---|---|
| le premier | sans filtre, **toutes** les tâches sont exportées, même deux tâches de même titre : c'est le bug de l'agent |
| le deuxième | `status=done` ne garde que les tâches faites, `status=todo` que les tâches restantes |
| le troisième | le CSV a une ligne d'en-tête, et un titre qui contient une virgule ou des guillemets est entouré de guillemets, ses guillemets doublés |

Pour le troisième test, la règle vient de la RFC 4180, le texte qui décrit le format CSV : un champ qui contient une virgule, un guillemet ou un retour à la ligne s'écrit entre guillemets, et chaque guillemet intérieur est doublé.

Lancez les tests :

```exec
cd ~/byte-layers/quality/todo-api && npm test
```

Le fichier `test/export.test.js` échoue avec `Cannot find module` : le module `src/export.js` n'existe pas encore. C'est votre travail.

## 03.3 - Le module d'export

Créez le fichier `src/export.js` avec ce squelette, puis complétez les trois `TODO`. Chaque fonction fait **une** chose, et aucune ne sait qu'il existe un serveur HTTP.

```exec
cd ~/byte-layers/quality/todo-api && cat > src/export.js <<'FIN'
// L'export des tâches : filtrer, puis mettre en forme. Aucun lien avec HTTP.

// Garde les tâches faites (status 'done'), restantes ('todo'), ou toutes (autre valeur).
export function filterByStatus(todos, status) {
  // TODO
}

// Met un champ au format CSV : entre guillemets s'il contient " , ou un retour à la ligne.
export function escapeCsvField(value) {
  // TODO
}

// Une ligne d'en-tête id,title,done, puis une ligne par tâche ; chaque ligne finit par \n.
export function toCsv(todos) {
  // TODO
}
FIN
```

Quelques outils de JavaScript utiles, chacun lié à un défaut de votre relecture :

| Outil | Ce qu'il remplace dans le code de l'agent |
|---|---|
| `todos.filter((todo) => todo.done)` | les boucles qui poussent les tâches une à une dans `result` |
| `/[",\n]/.test(text)` | les trois `includes` enchaînés |
| `` text.replaceAll('"', '""') `` | le `replace` avec une expression régulière |
| `todos.map(…)` puis `.join('\n')` | la chaîne `out` construite à coups de `+=` |

```hint
Pour `filterByStatus`, trois cas : `if (status === 'done') return todos.filter(…);`, la même chose pour `'todo'`, et sinon `return todos;`. Pour `toCsv`, chaque tâche donne `[todo.id, todo.title, todo.done].map(escapeCsvField).join(',')`.
```

````solution
```js
// L'export des tâches : filtrer, puis mettre en forme. Aucun lien avec HTTP.
const STATUS_FILTERS = {
  done: (todo) => todo.done,
  todo: (todo) => !todo.done,
};

// Garde les tâches faites (status 'done'), restantes ('todo'), ou toutes (autre valeur).
export function filterByStatus(todos, status) {
  const keep = STATUS_FILTERS[status];
  return keep ? todos.filter(keep) : todos;
}

// Met un champ au format CSV : entre guillemets s'il contient " , ou un retour à la ligne.
export function escapeCsvField(value) {
  const text = String(value);
  return /[",\n]/.test(text) ? `"${text.replaceAll('"', '""')}"` : text;
}

// Une ligne d'en-tête id,title,done, puis une ligne par tâche ; chaque ligne finit par \n.
export function toCsv(todos) {
  const lines = todos.map((todo) => [todo.id, todo.title, todo.done].map(escapeCsvField).join(','));
  return ['id,title,done', ...lines].join('\n') + '\n';
}
```
````

Le tableau `STATUS_FILTERS` de la solution associe chaque statut à sa règle : ajouter un statut, c'est ajouter une ligne, sans toucher à `filterByStatus`. Une suite de `if` fonctionne aussi.

Relancez `npm test` : les trois tests de `test/export.test.js` passent quand le module est complet.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
[ -f src/export.js ] || { echo "Le fichier src/export.js n'existe pas : créez-le avec la commande de la section 03.3."; exit 1; }
for f in filterByStatus escapeCsvField toCsv; do grep -q "export function $f" src/export.js || { echo "src/export.js doit exporter la fonction $f."; exit 1; }; done
node --test test/export.test.js >/tmp/bl-export.log 2>&1 || { grep -E "^not ok|expected|actual" /tmp/bl-export.log | head -12; echo "Les tests de test/export.test.js échouent (détail ci-dessus) : complétez les TODO de src/export.js."; exit 1; }
```

## 03.4 - Brancher le module dans le serveur

Le module est prêt ; le serveur doit maintenant s'en servir. Dans `src/server.js` :

1. **Supprimez** les lignes 6 à 17 : la constante `EXPORT_TOKEN`, l'ancienne version en commentaire, la fonction `formatDate` et la ligne vide qui la suit.
2. **Ajoutez** l'import du module, sous celui de `./todos.js` : `import { filterByStatus, toCsv } from './export.js';`
3. **Remplacez** toute la route de l'agent (lignes 24 à 79 du fichier d'origine), du commentaire `// Export des tâches (ajouté par l'agent)` jusqu'à l'accolade qui ferme le `if (req.url.startsWith(…))`, par deux lignes : la construction de l'adresse, `const url = new URL(req.url, 'http://localhost');`, puis l'appel à une fonction `handleExport` quand la requête est un `GET` sur le chemin exact `/todos/export`.
4. **Écrivez** la fonction `handleExport(req, res, url, store)` au-dessus de `createServer`. Elle reçoit tout ce dont elle a besoin en paramètres ; la fonction `send` n'est pas disponible ici, car elle est définie à l'intérieur de `createServer` : `handleExport` écrit donc ses réponses avec `res.writeHead` et `res.end`.

`handleExport` fait trois choses : vérifier le jeton, filtrer, mettre en forme. Le jeton change de place : il est lu dans la variable d'environnement `EXPORT_TOKEN`, et le client l'envoie dans un en-tête HTTP plutôt que dans l'adresse, car les serveurs enregistrent les adresses dans leurs journaux. L'en-tête standard pour cela s'appelle `Authorization`, et sa valeur `Bearer <jeton>` : le mot `Bearer` (« porteur ») annonce un jeton, dont la simple possession donne l'accès. Sans jeton, ou avec un mauvais jeton, la route répond `401` avec `{"error":"unauthorized"}`, comme le faisait l'agent.

```hint
La structure de `handleExport` : un `if` qui compare `req.headers.authorization` à `` `Bearer ${process.env.EXPORT_TOKEN}` `` et répond 401 sinon ; puis `const todos = filterByStatus(store.list(), url.searchParams.get('status'));` ; puis un `if` sur `url.searchParams.get('format') === 'csv'` qui répond `toCsv(todos)` avec le type `text/csv`, et sinon la réponse JSON de l'agent, `{ exportedAt, count, todos }`.
```

````solution
```js
// GET /todos/export : la route vérifie le jeton, src/export.js fait le reste.
function handleExport(req, res, url, store) {
  // Le jeton vient de l'environnement, jamais du code ; il voyage dans un en-tête, pas dans l'URL.
  const token = process.env.EXPORT_TOKEN;
  if (!token || req.headers.authorization !== `Bearer ${token}`) {
    res.writeHead(401, { 'Content-Type': 'application/json' });
    return res.end(JSON.stringify({ error: 'unauthorized' }));
  }
  const todos = filterByStatus(store.list(), url.searchParams.get('status'));
  if (url.searchParams.get('format') === 'csv') {
    res.writeHead(200, { 'Content-Type': 'text/csv' });
    return res.end(toCsv(todos));
  }
  res.writeHead(200, { 'Content-Type': 'application/json' });
  return res.end(JSON.stringify({ exportedAt: new Date().toISOString(), count: todos.length, todos }));
}
```

Et dans `createServer`, à la place de la route de l'agent, juste après la fonction `send` :

```js
    const url = new URL(req.url, 'http://localhost');
    if (req.method === 'GET' && url.pathname === '/todos/export') return handleExport(req, res, url, store);
```
````

`url.pathname === '/todos/export'` compare le chemin exact : `/todos/exportation` n'est plus pris pour l'export. Le jeton est lu au moment de la requête, pas au chargement du fichier : un test peut ainsi le définir avant d'appeler l'API.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
grep -q "exp_9f2c" src/*.js && { echo "Le jeton de l'agent est encore écrit dans src/ : supprimez la constante EXPORT_TOKEN et lisez process.env.EXPORT_TOKEN."; exit 1; }
grep -q "formatDate" src/server.js && { echo "La fonction formatDate, jamais appelée, est encore dans src/server.js."; exit 1; }
grep -qE "^// *function" src/server.js && { echo "L'ancienne version en commentaire est encore dans src/server.js."; exit 1; }
grep -q "startsWith" src/server.js && { echo "La route de l'agent, avec startsWith, est encore dans src/server.js : remplacez-la."; exit 1; }
grep -qE "=== *['\"]/todos/export['\"]" src/server.js || { echo "src/server.js doit comparer le chemin exact : url.pathname === '/todos/export'."; exit 1; }
grep -q "from './export.js'" src/server.js || { echo "src/server.js n'importe pas ./export.js."; exit 1; }
grep -q "process.env.EXPORT_TOKEN" src/server.js || { echo "src/server.js doit lire le jeton dans process.env.EXPORT_TOKEN."; exit 1; }
grep -nE "[^=!<>]==[^=]|!=[^=]" src/*.js && { echo "Les lignes ci-dessus utilisent == ou != : remplacez-les par === ou !==."; exit 1; }
node --check src/server.js || { echo "src/server.js contient une erreur de syntaxe (détail ci-dessus)."; exit 1; }
```

## 03.5 - Le test de l'agent change de contrat

Lancez les tests :

```exec
cd ~/byte-layers/quality/todo-api && npm test
```

Un test échoue : celui que l'agent avait écrit, `GET /todos/export returns the todos as JSON`. Il passe encore le jeton dans l'adresse, et ce jeton n'existe plus. C'est normal : vous avez changé le **contrat** de la route, le test doit suivre. Dans `test/server.test.js`, supprimez les lignes 22 à 31, de `test('GET /todos/export returns the todos as JSON'` jusqu'au `});` final, et mettez à leur place ce test, qui vérifie aussi le refus sans jeton et le contenu exact :

```js
test('GET /todos/export requires the token in the Authorization header', async (t) => {
  process.env.EXPORT_TOKEN = 'jeton-de-test';
  const server = createServer().listen(0);
  t.after(() => server.close());
  const base = `http://localhost:${server.address().port}`;
  await fetch(`${base}/todos`, { method: 'POST', body: JSON.stringify({ title: 'Exporter' }) });
  assert.equal((await fetch(`${base}/todos/export`)).status, 401);
  const res = await fetch(`${base}/todos/export`, { headers: { Authorization: 'Bearer jeton-de-test' } });
  assert.equal(res.status, 200);
  const body = await res.json();
  assert.deepEqual(body.todos.map((todo) => todo.title), ['Exporter']);
});
```

L'ancien test vérifiait seulement que la liste n'était pas vide : il aurait passé avec n'importe quel contenu. Le nouveau vérifie ce qui est exporté. Le stack revient sur ces tests trop complaisants dans un contenu dédié aux tests.

Relancez `npm test` : la fin de l'affichage compte 11 tests réussis et aucun échec.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
[ "$(git branch --show-current)" = "clean-export" ] || { echo "Placez-vous sur la branche clean-export : git switch clean-export"; exit 1; }
grep -q "returns the todos as JSON" test/server.test.js && { echo "Le test de l'agent est encore dans test/server.test.js : remplacez-le par celui de la section 03.5."; exit 1; }
node --test >/tmp/bl-clean-export.log 2>&1 || { grep -E "^not ok" /tmp/bl-clean-export.log | head; echo "Des tests échouent (liste ci-dessus) : lancez npm test et corrigez."; exit 1; }
grep -q "^# pass 11" /tmp/bl-clean-export.log || { grep "^# pass" /tmp/bl-clean-export.log; echo "Il doit y avoir 11 tests qui passent : vérifiez que test/export.test.js est en place."; exit 1; }
```

## 03.6 - Déclarer le réglage, puis commiter

Le jeton n'est plus dans le code : il faut maintenant dire où le mettre. Le projet déclare ses réglages dans `.env.example`, un modèle versionné ; chacun le copie en `.env`, un fichier ignoré par Git, que `npm start` lit au démarrage. Ajoutez la ligne `EXPORT_TOKEN` au modèle :

```exec
cd ~/byte-layers/quality/todo-api && printf 'EXPORT_TOKEN=change-me\n' >> .env.example && cat .env.example
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `printf 'EXPORT_TOKEN=change-me\n'` | produit la ligne, suivie d'un retour à la ligne |
> | `>> .env.example` | l'ajoute **à la fin** du fichier, sans effacer ce qu'il contient (`>` l'écraserait) |
> | `&& cat .env.example` | affiche le fichier pour vérifier |

Le fichier contient maintenant trois lignes, dont `EXPORT_TOKEN=change-me`. Pour essayer l'export pour de vrai, copiez `.env.example` en `.env`, remplacez `change-me` par un jeton de votre choix, lancez `npm start` dans un terminal, puis, dans un autre : `curl -H "Authorization: Bearer <votre-jeton>" "http://localhost:3000/todos/export?format=csv"`.

Le comportement est vérifié ; on enregistre le travail sur la branche :

```exec
cd ~/byte-layers/quality/todo-api && git add src test .env.example && git commit -m "Nettoie l'export : module src/export.js, jeton dans l'environnement"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git add src test .env.example` | prépare les modifications des dossiers `src` et `test`, nouveaux fichiers compris, et du modèle de réglages |
> | `git commit -m "…"` | enregistre ces modifications sur la branche `clean-export`, avec ce message |

> 📖 [RFC 4180, le format CSV (en anglais)](https://www.rfc-editor.org/rfc/rfc4180) · [Array.prototype.filter (MDN)](https://developer.mozilla.org/fr/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) · [L'en-tête Authorization (MDN)](https://developer.mozilla.org/fr/docs/Web/HTTP/Reference/Headers/Authorization)

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
git cat-file -e clean-export:src/export.js 2>/dev/null || { echo "Le commit de la branche clean-export ne contient pas src/export.js : lancez la commande de la section 03.6."; exit 1; }
grep -q "^EXPORT_TOKEN=" .env.example || { echo "Ajoutez la ligne EXPORT_TOKEN=change-me à .env.example (section 03.6)."; exit 1; }
[ -z "$(git status --porcelain src test .env.example)" ] || { echo "Des modifications de src ou test ne sont pas commitées : lancez la commande de la section 03.6."; exit 1; }
```
