---
title: "01 - Trois versions côte à côte"
description: "Sortir les trois branches du projet dans trois dossiers, avec git worktree"
---

Pour comparer trois versions avec les mêmes outils, le plus simple est de les avoir sous les yeux en même temps, chacune dans son dossier. Cette étape est un **support** : toutes les commandes sont fournies.

## 01.1 - Un dossier par version

Changer de branche à chaque mesure serait fastidieux, et risqué si des fichiers ne sont pas commités. Git sait sortir une autre version du même dépôt dans un autre dossier : c'est un **worktree**, un « arbre de travail » supplémentaire, qui partage l'historique du dépôt sans le copier.

```exec
cd ~/byte-layers/quality/todo-api && git fetch -q origin && git worktree add --detach ../measure/main origin/main && git worktree add --detach ../measure/agent origin/quality/agent-export && git worktree add --detach ../measure/clean clean-export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git fetch -q origin` | met à jour, sans bavarder, la copie locale des branches du dépôt distant |
> | `git worktree add <dossier> <version>` | sort la version demandée dans un nouveau dossier, relié au même dépôt |
> | `--detach` | ne rattache pas le dossier à une branche : on veut lire ces versions, pas y travailler |
> | `../measure/main origin/main` | le projet avant l'agent |
> | `../measure/agent origin/quality/agent-export` | le code de l'agent |
> | `../measure/clean clean-export` | votre version corrigée, la branche locale du lab précédent |

Si Git répond `invalid reference: clean-export`, la branche du lab précédent manque : terminez d'abord le lab [Relire et nettoyer le code d'un agent](https://byte-layers.com/fr/labs/clean-code-agent-diff). Si un dossier existe déjà d'un essai précédent, Git refuse de l'écraser, et la suite de la commande ne s'exécute pas. Retirez alors les trois dossiers, puis relancez la commande ci-dessus :

```exec
cd ~/byte-layers/quality/todo-api && for d in main agent clean; do git worktree remove --force ../measure/$d 2>/dev/null; done; git worktree prune
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `for d in main agent clean; do … done` | répète la suite pour chaque dossier |
> | `git worktree remove --force ../measure/$d` | retire le dossier et le détache du dépôt, même s'il contient des fichiers modifiés |
> | `2>/dev/null` | cache l'erreur si le dossier n'existe pas |
> | `git worktree prune` | oublie les worktrees dont le dossier a disparu |

## 01.2 - Vérifier

Listez les worktrees du dépôt :

```exec
cd ~/byte-layers/quality/todo-api && git worktree list
```

Quatre lignes s'affichent : le dossier `todo-api` lui-même, puis `measure/main`, `measure/agent` et `measure/clean`, chacun avec l'identifiant court de son commit et la mention `(detached HEAD)`. Les outils des étapes suivantes s'installent dans `~/byte-layers/quality`, le dossier parent : ils verront les trois versions.

> 📖 [git worktree (en anglais)](https://git-scm.com/docs/git-worktree)

```verify
Q=~/byte-layers/quality/measure
for d in main agent clean; do [ -f "$Q/$d/src/server.js" ] || { echo "Le dossier $Q/$d n'existe pas ou n'a pas de src/server.js : lancez la commande de la section 01.1."; exit 1; }; done
grep -q "EXPORT_TOKEN = '" "$Q/agent/src/server.js" || { echo "measure/agent ne contient pas le code de l'agent."; exit 1; }
grep -q "EXPORT_TOKEN" "$Q/main/src/server.js" && { echo "measure/main contient déjà l'export : il doit être sur origin/main."; exit 1; }
grep -rq "EXPORT_TOKEN = '" "$Q/clean/src" && { echo "measure/clean contient encore le jeton écrit en dur : la correction du lab précédent n'est pas commitée sur clean-export."; exit 1; }
grep -rq "todos/export" "$Q/clean/src" || { echo "measure/clean n'a pas de route d'export : la branche clean-export du lab précédent n'est pas commitée."; exit 1; }
```
