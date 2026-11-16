---
title: "02 - ESLint, sonarjs et la complexité cognitive"
description: "Installer les outils, mesurer la complexité de chaque fonction, et choisir un seuil qui tient compte de la dette existante"
---

ESLint est l'analyseur de code le plus répandu en JavaScript : il lit le code sans l'exécuter et signale ce qui enfreint ses **règles**. Le plugin **sonarjs**, publié par SonarSource, l'éditeur de SonarQube, lui ajoute les règles de SonarQube, dont la complexité cognitive.

## 02.1 - Installer les outils

Les outils s'installent une fois, dans `~/byte-layers/quality`, le dossier parent des trois versions. Cette section est un **support** :

```exec
cd ~/byte-layers/quality && npm init -y >/dev/null && npm pkg set type=module && npm install --save-dev eslint@10.15.0 @eslint/js@10.0.1 eslint-plugin-sonarjs@4.2.2 globals@17.14.0 jscpd@5.4.3
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `npm init -y >/dev/null` | crée un `package.json` avec les valeurs par défaut, sans afficher son contenu |
> | `npm pkg set type=module` | déclare que les fichiers `.js` du dossier sont des modules modernes, avec `import` : la configuration d'ESLint en a besoin |
> | `npm install --save-dev …` | installe les outils dans `node_modules` et les note dans `package.json` comme outils de développement |
> | `@x.y.z` après chaque nom | fixe la version, pour que le lab donne toujours les mêmes résultats |

| Paquet | Rôle |
|---|---|
| `eslint` | l'analyseur |
| `@eslint/js` | les règles recommandées par ESLint pour JavaScript |
| `eslint-plugin-sonarjs` | les règles de SonarQube, dont `cognitive-complexity` |
| `globals` | la liste des variables globales de Node.js (`process`, `console`…), pour qu'ESLint ne les prenne pas pour des oublis |
| `jscpd` | le détecteur de duplication, utilisé à l'étape suivante |

## 02.2 - La complexité cognitive

La **complexité cognitive** estime l'effort pour **comprendre** une fonction. Elle a été définie par SonarSource en 2017. Chaque rupture dans la lecture linéaire coûte un point (`if`, `else`, boucle, `catch`, enchaînement de `&&` et de `||`), et chaque niveau d'imbrication ajoute un point de plus :

```js
function exemple(items, status) {
  for (const item of items) {        // +1 (boucle)
    if (status === 'done') {         // +2 (if, imbriqué une fois)
      if (item.done) keep(item);     // +3 (if, imbriqué deux fois)
    }
  }
}                                    // complexité cognitive : 6
```

Une fonction plate, avec des sorties rapides (`if (…) return …;`), reste basse ; une fonction aux conditions imbriquées monte vite. SonarQube propose par défaut un maximum de **15** par fonction.

## 02.3 - Le fichier de configuration

ESLint lit sa configuration dans `eslint.config.js`. Cette commande écrit ce fichier : il active les règles recommandées d'ESLint et de sonarjs, et durcit deux règles.

```exec
cd ~/byte-layers/quality && cat > eslint.config.js <<'FIN'
import js from '@eslint/js';
import sonarjs from 'eslint-plugin-sonarjs';
import globals from 'globals';

export default [
  js.configs.recommended,
  sonarjs.configs.recommended,
  {
    languageOptions: { globals: globals.node },
    rules: {
      eqeqeq: 'error',
      'sonarjs/cognitive-complexity': ['error', 15],
    },
  },
];
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > eslint.config.js` | écrit dans le fichier `eslint.config.js` ce que `cat` reçoit, en remplaçant son contenu s'il existe |
> | `<<'FIN'` … `FIN` | fournit à `cat` toutes les lignes jusqu'à `FIN` ; les apostrophes autour de `FIN` empêchent le terminal d'interpréter les `$` ou les `` ` `` du texte |

Le contenu du fichier, ligne par ligne :

| Ligne | Rôle |
|---|---|
| `js.configs.recommended` | les règles de base : variable jamais utilisée, code inatteignable… |
| `sonarjs.configs.recommended` | les règles recommandées de sonarjs |
| `languageOptions: { globals: globals.node }` | le code tourne sous Node.js |
| `eqeqeq: 'error'` | interdit `==` et `!=`, qui convertissent les types en silence |
| `'sonarjs/cognitive-complexity': ['error', 15]` | une erreur pour toute fonction dont la complexité cognitive dépasse 15 |

## 02.4 - Mesurer les trois versions

Lancez ESLint sur le code de chaque version :

```exec
cd ~/byte-layers/quality && for v in main agent clean; do echo "== $v"; npx eslint measure/$v/src; done
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `for v in main agent clean; do … done` | répète la suite pour chaque version, avec `$v` qui vaut `main`, puis `agent`, puis `clean` |
> | `echo "== $v"` | affiche le nom de la version, pour s'y retrouver |
> | `npx eslint measure/$v/src` | analyse le dossier `src` de cette version avec `eslint.config.js` |

Pour `main`, ESLint affiche une seule erreur :

```text
  7:45  error  Refactor this function to reduce its Cognitive Complexity from 18 to the 15 allowed  sonarjs/cognitive-complexity
```

Le message se lit : « remaniez cette fonction pour faire passer sa complexité cognitive de 18 aux 15 autorisés ». Les trois constats :

| Version | Ce qu'ESLint signale |
|---|---|
| `main` | une complexité de **18** pour la fonction de `createServer` qui traite les requêtes |
| `agent` | 9 erreurs : `formatDate` jamais utilisée (`no-unused-vars`), sept `==` ou `!=` (`eqeqeq`), et une complexité cognitive de **91** |
| `clean` | une complexité de **20** avec la solution du lab précédent (un peu plus ou un peu moins avec la vôtre), pour la même fonction |

La surprise est `main` : le projet **avant** l'agent dépasse déjà le seuil. C'est une **dette d'origine** : un `if` par route, dans une seule fonction. Si l'on garde 15, l'outil échoue sur tout le projet, y compris sur du code que personne n'a touché, et l'équipe apprend vite à ignorer ses alertes.

## 02.5 - À vous de choisir le seuil

La pratique courante face à une dette existante est le **cliquet** : on fixe le seuil au niveau actuel du projet, pour que la dette ne **grossisse** plus, puis on l'abaisse à mesure qu'on rembourse. Ici, le niveau actuel est 18, mais votre version ajoute une route, donc un `if` de plus dans la même fonction : environ 2 points. Une équipe a deux choix honnêtes : découper `createServer` d'abord, puis fixer 18 ; ou accepter une petite **tolérance** au-dessus de 18, en notant la dette à rembourser. Ce lab prend le second, plus rapide.

Ouvrez `~/byte-layers/quality/eslint.config.js` dans votre éditeur, et modifiez le seuil de `cognitive-complexity` (le nombre `15`) pour que :

- `main` passe : la dette d'origine est acceptée, pour l'instant ;
- `clean` passe, avec le seuil le plus bas possible : votre version ne doit aggraver la dette que de l'ajout de sa route ;
- `agent` échoue toujours.

Relancez la commande de la section 02.4 après chaque essai. Les erreurs `eqeqeq` et `no-unused-vars` de `agent` ne dépendent pas du seuil.

```hint
Lisez le nombre qu'ESLint affiche pour `clean` : `Refactor this function to reduce its Cognitive Complexity from N to the 15 allowed`. Le seuil doit être au moins égal à ce nombre, et bien en dessous de 91. Si votre `clean` dépasse 25, sa route d'export est sans doute restée dans `createServer` : sortez-la dans une fonction `handleExport`, comme dans la solution du lab précédent.
```

````solution
```js
      'sonarjs/cognitive-complexity': ['error', 20],
```

Avec la solution du lab précédent, `clean` vaut 20 : un seuil de 20 accepte `main` (18) et `clean`, et refuse `agent` (91). Ce n'est pas un cliquet strict, c'est une tolérance de 2 points : chaque nouvelle route en ajouterait encore. Notez dans un ticket la dette restante : découper `createServer`, une fonction par route, pour revenir sous 15, puis baisser le seuil.
````

> 📖 [SonarSource, « Cognitive Complexity » (PDF, en anglais)](https://www.sonarsource.com/docs/CognitiveComplexity.pdf) · [ESLint : les fichiers de configuration (en anglais)](https://eslint.org/docs/latest/use/configure/configuration-files) · [ESLint : la règle eqeqeq (en anglais)](https://eslint.org/docs/latest/rules/eqeqeq)

```verify
cd ~/byte-layers/quality 2>/dev/null || { echo "Le dossier ~/byte-layers/quality n'existe pas."; exit 1; }
[ -f node_modules/.bin/eslint ] || { echo "ESLint n'est pas installé dans ~/byte-layers/quality : lancez la commande de la section 02.1."; exit 1; }
[ -f eslint.config.js ] || { echo "Le fichier eslint.config.js n'existe pas : lancez la commande de la section 02.3."; exit 1; }
grep -q "sonarjs/cognitive-complexity" eslint.config.js || { echo "eslint.config.js doit contenir la règle sonarjs/cognitive-complexity."; exit 1; }
T=$(grep -oE "cognitive-complexity['\"]?: *\[ *['\"]error['\"], *[0-9]+" eslint.config.js | grep -oE "[0-9]+$")
[ -n "$T" ] || { echo "Le seuil de sonarjs/cognitive-complexity est introuvable : gardez la forme ['error', N]."; exit 1; }
[ "$T" -lt 30 ] || { echo "Le seuil vaut $T : c'est beaucoup trop haut, il laisserait passer des fonctions très complexes. Visez le nombre affiché pour clean."; exit 1; }
npx eslint measure/main/src >/dev/null 2>&1 || { echo "ESLint échoue sur measure/main : le seuil est trop bas pour la dette d'origine (18)."; exit 1; }
npx eslint measure/clean/src >/tmp/bl-eslint-clean.log 2>&1 || { grep -E "error" /tmp/bl-eslint-clean.log | head -5; echo "ESLint échoue sur measure/clean (détail ci-dessus) : relevez le seuil, ou simplifiez votre version."; exit 1; }
npx eslint measure/agent/src 2>&1 | grep -q "Cognitive Complexity" || { echo "La complexité du code de l'agent passe le seuil : il est trop haut."; exit 1; }
exit 0
```
