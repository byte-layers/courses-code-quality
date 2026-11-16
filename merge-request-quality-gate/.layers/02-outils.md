---
title: "02 - Mettre les outils dans le projet"
description: "Déclarer les outils dans package.json et copier leur configuration dans le dépôt, pour que le pipeline les retrouve"
---

Au lab précédent, les outils et leur configuration vivaient dans `~/byte-layers/quality`, **à côté** du projet. Un pipeline, lui, ne voit que ce qui est **dans le dépôt** : il clone le projet sur une machine neuve, et rien d'autre. Il faut donc y mettre trois choses :

| Ce qu'il faut | Où dans le dépôt |
|---|---|
| la liste des outils et leurs versions | `package.json` et `package-lock.json` |
| la configuration d'ESLint | `eslint.config.js` |
| vos règles Semgrep | `.semgrep/rules.yml` |

## 02.1 - Installer les outils dans le projet

On travaille sur `main` : c'est la base que les merge requests viseront. Cette commande installe les outils comme **dépendances de développement** du projet :

```exec
cd ~/byte-layers/quality/todo-api && git switch main && npm install --save-dev eslint@10.15.0 @eslint/js@10.0.1 eslint-plugin-sonarjs@4.2.2 globals@17.14.0 eslint-formatter-gitlab@7.2.0 jscpd@5.4.3
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch main` | se place sur la branche `main` |
> | `npm install --save-dev …` | installe les outils dans `node_modules`, les note dans `package.json`, et fige leurs versions exactes, et celles de leurs propres dépendances, dans `package-lock.json` |
> | `eslint-formatter-gitlab@7.2.0` | le seul paquet absent du lab précédent : il écrit les résultats d'ESLint dans le format des rapports de qualité de GitLab |

Le dossier `node_modules` n'est pas versionné : le fichier `.gitignore` du projet l'exclut déjà. Le pipeline le recréera à partir de `package-lock.json`.

## 02.2 - Copier la configuration

Les deux fichiers de configuration du lab précédent sont copiés tels quels dans le projet, avec vos réglages : le seuil de complexité que vous avez choisi, et vos deux règles Semgrep.

```exec
cd ~/byte-layers/quality/todo-api && cp ../eslint.config.js . && mkdir -p .semgrep && cp ../.semgrep/rules.yml .semgrep/
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cp ../eslint.config.js .` | copie la configuration d'ESLint du dossier parent dans le projet (`.` désigne le dossier courant) |
> | `mkdir -p .semgrep` | crée le dossier `.semgrep` dans le projet |
> | `cp ../.semgrep/rules.yml .semgrep/` | y copie vos règles |

## 02.3 - Vérifier sur votre poste

Avant d'envoyer quoi que ce soit, vérifiez que les outils tournent depuis le projet, et que `main` passe :

```exec
cd ~/byte-layers/quality/todo-api && npx eslint src && npx jscpd --min-tokens 30 --threshold 5 --reporters silent src && npm test
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `npx eslint src` | analyse `src` avec la configuration du projet ; ESLint prend le fichier `eslint.config.js` le plus proche des fichiers analysés |
> | `npx jscpd … src` | mesure la duplication, avec les réglages du lab précédent |
> | `npm test` | lance les tests du projet |
> | `&&` | n'enchaîne la commande suivante que si la précédente a réussi |

ESLint et jscpd ne signalent aucun problème, puis les tests finissent sur `pass 7` et `fail 0`. Semgrep sera vérifié par le pipeline.

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
[ "$(git branch --show-current)" = main ] || { echo "Le projet n'est pas sur la branche main : lancez git switch main dans ~/byte-layers/quality/todo-api."; exit 1; }
for p in eslint eslint-plugin-sonarjs eslint-formatter-gitlab jscpd; do grep -q "\"$p\"" package.json || { echo "$p n'est pas déclaré dans package.json : lancez la commande de la section 02.1."; exit 1; }; done
[ -f package-lock.json ] || { echo "package-lock.json manque : lancez la commande de la section 02.1."; exit 1; }
[ -f eslint.config.js ] && [ -f .semgrep/rules.yml ] || { echo "eslint.config.js ou .semgrep/rules.yml manque dans le projet : lancez la commande de la section 02.2."; exit 1; }
npx eslint src >/dev/null 2>&1 || { echo "ESLint échoue sur main depuis le projet : vérifiez que vous êtes sur main (git switch main) et que eslint.config.js est celui du lab précédent."; exit 1; }
exit 0
```
