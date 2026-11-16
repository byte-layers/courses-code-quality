---
title: "02 - Relire avec les principes du clean code"
description: "Nommer chaque défaut du code de l'agent avec le bon principe, et trouver le bug que ses tests ne voient pas"
---

Un relecteur qui écrit « c'est pas propre » n'aide personne. Un relecteur qui écrit « lignes 41 à 63 : la même mise en forme CSV copiée trois fois, DRY » donne à l'auteur, humain ou agent, de quoi corriger. Cette étape vous donne le vocabulaire, puis vous le mettez en pratique sur le code de l'agent.

## 02.1 - Le vocabulaire du relecteur

Ces principes viennent de livres et d'articles devenus des références, comme *Clean Code* de Robert C. Martin. Ce ne sont pas des lois, mais des noms partagés pour des défauts fréquents :

| Principe | Ce qu'il demande | Le défaut typique |
|---|---|---|
| **Nommage** | un nom dit ce que contient la variable ou ce que fait la fonction | `data`, `tmp`, `x`, `t` : il faut lire tout le code pour comprendre |
| **KISS** (*Keep It Simple, Stupid*) | la solution la plus simple qui répond au besoin | des conditions imbriquées sur cinq niveaux, du code pour un besoin que personne n'a exprimé |
| **DRY** (*Don't Repeat Yourself*) | chaque connaissance à un seul endroit | le même bloc copié-collé : une correction faite à un endroit est oubliée ailleurs |
| **SOLID**, ici le **S** (responsabilité unique) | un module, une fonction : une seule raison de changer | le routeur HTTP qui met aussi en forme du CSV |
| **Complexité algorithmique** | un coût raisonnable quand les données grandissent | deux boucles imbriquées sur la même liste : 1 000 tâches, un million de comparaisons |
| **Code mort** | pas de code qui ne sert pas | une fonction jamais appelée, une ancienne version laissée en commentaire « au cas où » |

Deux catégories complètent la liste, parce qu'une relecture ne regarde pas que le style :

| Catégorie | Ce qu'on cherche |
|---|---|
| **Sécurité** | un secret écrit dans le code, une donnée sensible qui fuit dans les journaux |
| **Bug** | un comportement faux, même si les tests passent |

Le bit [Les risques du code généré par un agent](https://byte-layers.com/fr/bits/generated-code-risks) explique pourquoi un agent produit souvent ces défauts : il écrit vite un code qui passe les tests, en ignorant ce que le projet faisait déjà.

## 02.2 - Le bug que les tests ne voient pas

Avant de relire ligne à ligne, une petite expérience. Le code de l'agent contient une boucle qui construit une liste `tmp` : que fait-elle ? Pour le savoir, ce petit script démarre le serveur, crée deux tâches **avec le même titre**, puis affiche la liste complète et l'export. On l'écrit à côté du projet, dans `~/byte-layers/quality/doublons.mjs`, pour ne pas le mélanger au code :

```exec
cd ~/byte-layers/quality && cat > doublons.mjs <<'FIN'
import { createServer } from './todo-api/src/server.js';

const server = createServer().listen(0);
const base = `http://localhost:${server.address().port}`;
for (const title of ['Acheter du pain', 'Acheter du pain']) {
  await fetch(`${base}/todos`, { method: 'POST', body: JSON.stringify({ title }) });
}
console.log('Liste :', await (await fetch(`${base}/todos`)).text());
console.log('Export :', await (await fetch(`${base}/todos/export?token=exp_9f2c4e7a1b8d6035`)).text());
server.close();
FIN
node doublons.mjs
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > doublons.mjs <<'FIN'` … `FIN` | écrit le script ; l'extension `.mjs` dit à Node que c'est un module moderne, qui accepte `import` et `await` |
> | `createServer().listen(0)` | démarre l'API sur un port libre, choisi par le système |
> | la boucle `for` | crée deux tâches identiques avec `POST /todos` |
> | les deux `console.log` | affichent la liste complète, puis l'export |
> | `server.close()` | arrête le serveur à la fin |
> | `node doublons.mjs` | exécute le script |

L'affichage ressemble à ceci (la date change) :

```text
Liste : [{"id":1,"title":"Acheter du pain","done":false},{"id":2,"title":"Acheter du pain","done":false}]
Export : {"exportedAt":"2026-11-16T12:22:52.771Z","count":1,"todos":[{"id":1,"title":"Acheter du pain","done":false}]}
```

La liste contient deux tâches, `id` 1 et 2. L'export n'en contient qu'**une** : la boucle supprime les tâches dont le titre existe déjà. Personne ne l'a demandé, et une tâche réelle disparaît de l'export. Le test de l'agent vérifie seulement que la liste exportée n'est pas vide : il ne pouvait pas le voir.

## 02.3 - À vous de relire

Ouvrez `src/server.js` et relisez tout le code ajouté par l'agent : la constante, le commentaire et la fonction en haut du fichier, puis la route `/todos/export`. Pour chaque défaut trouvé, écrivez une ligne dans le fichier `~/byte-layers/quality/review.md`, sous cette forme :

```text
- [Catégorie] ligne(s) : ce qui ne va pas, et ce qu'il faudrait faire
```

La catégorie est l'un des huit noms des tableaux ci-dessus : `Nommage`, `KISS`, `DRY`, `SOLID`, `Complexité`, `Code mort`, `Sécurité`, `Bug`. Par exemple :

```text
- [Bug] lignes 29 à 36 : la boucle retire les tâches de même titre ; l'export doit garder toutes les tâches
```

Commencez par cette commande : elle crée le fichier avec la ligne d'exemple, et ne touche à rien s'il existe déjà. Complétez-le ensuite avec votre éditeur.

```exec
cd ~/byte-layers/quality && { [ -f review.md ] || printf '%s\n' '- [Bug] lignes 29 à 36 : la boucle retire les tâches de même titre ; l'"'"'export doit garder toutes les tâches' > review.md; }
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `[ -f review.md ]` | teste si le fichier existe déjà |
> | `\|\|` | ne lance la suite que si le test a échoué, c'est-à-dire si le fichier n'existe pas |
> | `printf '%s\n' '…' > review.md` | écrit la ligne d'exemple, suivie d'un retour à la ligne, dans `review.md` |
> | `'"'"'` | écrit une apostrophe à l'intérieur d'un texte entre apostrophes |
> | `{ … ; }` | regroupe le test et l'écriture, pour qu'ils ne s'exécutent qu'une fois dans le bon dossier |

Visez au moins **six défauts**, dans au moins **quatre catégories** différentes. Prenez le temps de chercher : il y en a plus de dix.

```hint
Regardez : les noms des variables de la route ; les trois blocs qui échappent un titre pour le CSV ; les deux endroits qui filtrent par statut ; les comparaisons avec `==` et `!=` ; la fonction `formatDate` (est-elle appelée ?) ; le commentaire « gardée au cas où » ; la valeur de `EXPORT_TOKEN` ; l'endroit d'où vient le jeton ; la condition `req.url.startsWith('/todos/export')` (que se passe-t-il pour `/todos/exportation` ?).
```

````solution
```text
- [Bug] lignes 29 à 36 : la boucle retire les tâches de même titre ; l'export doit garder toutes les tâches
- [Complexité] lignes 29 à 36 : deux boucles imbriquées sur la liste, un coût quadratique ; inutile une fois le dédoublonnage supprimé
- [Nommage] route d'export : data, tmp, x, t, out, result ne disent rien ; todos, csvLines, title…
- [DRY] lignes 41 à 63 : l'échappement CSV d'un titre est copié trois fois ; une fonction escapeCsvField
- [DRY] filtres par statut : la même logique done / todo est écrite deux fois, pour le CSV et pour le JSON ; une fonction filterByStatus
- [KISS] route d'export : jusqu'à cinq niveaux d'imbrication ; filtrer d'abord, puis mettre en forme
- [SOLID] route d'export : le routeur HTTP fait aussi la mise en forme CSV et JSON ; un module src/export.js
- [Code mort] ligne 14 : formatDate n'est jamais appelée ; la supprimer
- [Code mort] lignes 9 à 12 : ancienne version en commentaire ; Git garde l'historique, la supprimer
- [Sécurité] ligne 7 : jeton écrit en dur dans le code, donc dans Git ; le lire dans une variable d'environnement
- [Sécurité] ligne 27 : jeton passé dans l'adresse, il finit dans les journaux ; un en-tête Authorization
- [Bug] ligne 25 : startsWith accepte /todos/exportation ou /todos/export/42 ; comparer le chemin exact
- [KISS] lignes 27 à 71 : == et != font des conversions de type surprenantes ; === et !==
- [Sécurité] .env.example : le projet déclare ses réglages secrets dans ce fichier, l'agent ne l'a pas suivi ; ajouter EXPORT_TOKEN à .env.example
```
````

Les numéros de ligne peuvent différer de quelques unités selon votre éditeur : ce qui compte, c'est que chaque ligne désigne un endroit précis et nomme le principe.

```verify
F=~/byte-layers/quality/review.md
[ -f "$F" ] || { echo "Le fichier ~/byte-layers/quality/review.md n'existe pas : créez-le avec la commande de la section 02.3."; exit 1; }
N=$(grep -cE '^- \[[^]]+\]' "$F")
[ "$N" -ge 6 ] || { echo "Le fichier contient $N défauts au format « - [Catégorie] … » ; il en faut au moins 6."; exit 1; }
K=$(grep -oE '^- \[(Nommage|KISS|DRY|SOLID|Complexité|Code mort|Sécurité|Bug)\]' "$F" | sort -u | wc -l | tr -d ' ')
[ "$K" -ge 4 ] || { echo "Les défauts couvrent $K catégories reconnues ; il en faut au moins 4, parmi Nommage, KISS, DRY, SOLID, Complexité, Code mort, Sécurité, Bug (écrites exactement ainsi)."; exit 1; }
```
