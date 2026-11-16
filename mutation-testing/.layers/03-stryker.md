---
title: "03 - Laisser Stryker créer les mutants"
description: "Configurer Stryker, lancer l'analyse, et lire les mutants qui survivent"
---

Stryker fait automatiquement ce que vous venez de faire à la main : il crée des dizaines de mutants, lance les tests sur chacun, et compte ceux qui survivent. Cette étape est un **support** : la configuration et la commande sont fournies, votre travail est de lire le résultat.

## 03.1 - La configuration

Stryker lit ses réglages dans `stryker.config.json`, à la racine du projet :

```exec
cd ~/byte-layers/quality/todo-api && cat > stryker.config.json <<'FIN'
{
  "testRunner": "command",
  "commandRunner": { "command": "node --test" },
  "mutate": ["src/todos.js"],
  "reporters": ["clear-text", "html"]
}
FIN
grep -qx 'reports/' .gitignore || printf 'reports/\n.stryker-tmp/\n' >> .gitignore
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > stryker.config.json <<'FIN'` … `FIN` | écrit la configuration dans le fichier |
> | `grep -qx 'reports/' .gitignore \|\|` | vérifie si `.gitignore` contient déjà la ligne `reports/` ; la suite ne s'exécute que si elle manque, pour ne pas l'ajouter deux fois |
> | `printf 'reports/\n.stryker-tmp/\n' >> .gitignore` | ajoute deux lignes à la fin de `.gitignore` : le rapport et le dossier de travail de Stryker ne seront pas commités |

| Réglage | Rôle |
|---|---|
| `"testRunner": "command"` | Stryker lance les tests avec une simple commande, quel que soit l'outil de test |
| `"commandRunner": { "command": "node --test" }` | la commande : le lanceur de tests de Node ; si elle échoue, le mutant est tué |
| `"mutate": ["src/todos.js"]` | les fichiers à muter : ici le seul module de stockage |
| `"reporters": ["clear-text", "html"]` | un résumé dans le terminal, et un rapport HTML dans `reports/mutation/mutation.html` |

## 03.2 - L'analyse

`npx` télécharge Stryker la première fois, sans l'ajouter aux dépendances du projet, puis le lance :

```exec
cd ~/byte-layers/quality/todo-api && npx -y -p @stryker-mutator/core@10.1.1 stryker run
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `npx` | lance un outil distribué sur npm, en le téléchargeant si besoin |
> | `-y` | accepte le téléchargement sans poser de question |
> | `-p @stryker-mutator/core@10.1.1` | le paquet à télécharger, dans une version fixée pour que le lab donne toujours le même résultat |
> | `stryker run` | la commande fournie par ce paquet : lance l'analyse avec `stryker.config.json` |

L'analyse dure quelques secondes. Elle se termine par un tableau :

```text
----------|------------------|----------|-----------|------------|----------|----------|
          | % Mutation score |          |           |            |          |          |
File      |  total | covered | # killed | # timeout | # survived | # no cov | # errors |
----------|--------|---------|----------|-----------|------------|----------|----------|
All files |  90.32 |   90.32 |       28 |         0 |          3 |        0 |        0 |
 todos.js |  90.32 |   90.32 |       28 |         0 |          3 |        0 |        0 |
```

Les chiffres exacts de `killed` et `timeout` peuvent varier légèrement d'une machine à l'autre ; un mutant en `timeout` a fait boucler les tests, Stryker le compte comme tué. Ce qui compte : **3 mutants survivent**, sur un module couvert à 100 %.

## 03.3 - Lire les survivants

Au-dessus du tableau, Stryker affiche chaque mutant qui a survécu, avec la ligne d'origine (`-`) et la ligne mutée (`+`). Vous devez retrouver ces trois-là :

| Mutant | Le changement | Le bug qu'il représente |
|---|---|---|
| `ConditionalExpression`, ligne 9 | `typeof title === 'string' ? title.trim() : ''` devient `true ? title.trim() : ''` | un titre qui n'est pas du texte, comme le nombre `42`, ferait planter `trim()` au lieu de donner l'erreur `title is required` |
| `EqualityOperator`, ligne 11 | `>` devient `>=` | votre mutant de l'étape 02 : un titre de 200 caractères pile serait refusé |
| `UpdateOperator`, ligne 12 | `nextId++` devient `nextId--` | les identifiants décroîtraient : 1, 0, -1… |

Le premier mutant mérite une remarque : la ligne 9 est bien exécutée par le test du serveur qui envoie un corps vide. Mais ce test vérifie seulement le code `400`, et le serveur répond `400` pour **toute** erreur, y compris le plantage de `trim()`. Un test qui ne regarde que le code HTTP laisse passer ce bug.

Chaque survivant dit la même chose : **aucun test ne vérifie ce comportement**. Le rapport HTML, `reports/mutation/mutation.html`, montre les mêmes informations sur le code coloré ; ouvrez-le dans un navigateur si vous le souhaitez.

> 📖 [Stryker : la configuration (en anglais)](https://stryker-mutator.io/docs/stryker-js/configuration/) · [Stryker : les mutations possibles (en anglais)](https://stryker-mutator.io/docs/mutation-testing-elements/supported-mutators/)

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
[ -f stryker.config.json ] || { echo "Le fichier stryker.config.json n'existe pas : lancez la commande de la section 03.1."; exit 1; }
grep -q '"src/todos.js"' stryker.config.json || { echo "stryker.config.json doit muter src/todos.js."; exit 1; }
grep -qx "reports/" .gitignore || { echo "Ajoutez reports/ à .gitignore (section 03.1)."; exit 1; }
[ -f reports/mutation/mutation.html ] || { echo "Le rapport reports/mutation/mutation.html n'existe pas : lancez Stryker (section 03.2)."; exit 1; }
```
