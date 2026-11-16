---
title: "01 - Récupérer le code de l'agent"
description: "Cloner le projet, se placer sur la branche de l'agent, et constater que ses tests passent"
---

Avant de juger un code, il faut l'avoir sous les yeux, et savoir ce qui a changé. Cette étape est un **support** : toutes les commandes sont fournies.

## 01.1 - Cloner le projet

Le dépôt `todo-api` contient deux branches utiles ici : `main`, le projet avant l'agent, et `quality/agent-export`, le même projet après le passage de l'agent. On le clone dans le dossier de travail du stack, puis on se place sur la branche de l'agent :

```exec
cd ~ && mkdir -p ~/byte-layers/quality && git clone https://github.com/byte-layers/todo-api.git ~/byte-layers/quality/todo-api && cd ~/byte-layers/quality/todo-api && git switch quality/agent-export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cd ~ &&` | part de votre dossier personnel |
> | `mkdir -p ~/byte-layers/quality` | crée le dossier de travail du stack, s'il n'existe pas |
> | `git clone <adresse> ~/byte-layers/quality/todo-api` | télécharge le dépôt dans ce dossier |
> | `cd ~/byte-layers/quality/todo-api` | entre dans le projet |
> | `git switch quality/agent-export` | se place sur la branche de l'agent ; Git la crée en local à partir de celle du dépôt distant |

Si le dossier existe déjà, `git clone` refuse : entrez dans le dossier et lancez seulement `git switch quality/agent-export`.

## 01.2 - Ce que l'agent a changé

Une relecture commence par la liste des fichiers modifiés. `git diff main --stat` compare la branche courante à `main` et résume les changements :

```exec
cd ~/byte-layers/quality/todo-api && git diff main --stat
```

Deux fichiers ont changé : `src/server.js`, qui reçoit la nouvelle route, et `test/server.test.js`, qui reçoit un test. Ouvrez `src/server.js` dans votre éditeur. La demande faite à l'agent était courte : « ajoute un export des tâches en JSON et en CSV, filtrable par statut, protégé par un jeton ». L'agent a ajouté :

| Ajout | Où |
|---|---|
| une constante `EXPORT_TOKEN` | en haut du fichier |
| une route `/todos/export` | au début du gestionnaire de requêtes, avant les autres routes |
| les paramètres `format` (`csv` ou JSON par défaut), `status` (`done`, `todo` ou rien) et `token` | lus dans l'adresse, par exemple `/todos/export?format=csv&status=done&token=…` |

## 01.3 - Les tests passent

Lancez les tests du projet :

```exec
cd ~/byte-layers/quality/todo-api && npm test
```

La fin de l'affichage compte 8 tests réussis et aucun échec : `ℹ pass 8` et `ℹ fail 0` dans un terminal (ou `# pass 8` selon l'affichage). Le code fonctionne, du moins pour ce que les tests vérifient. C'est souvent là qu'une revue s'arrête, et c'est exactement le piège : des tests verts disent que le code fait ce que les tests demandent, pas qu'il est lisible, maintenable ou juste dans tous les cas.

> 📖 [git diff (en anglais)](https://git-scm.com/docs/git-diff) · [Le lanceur de tests de Node.js (en anglais)](https://nodejs.org/api/test.html)

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas : lancez la commande de la section 01.1."; exit 1; }
git rev-parse --verify -q quality/agent-export >/dev/null || { echo "La branche quality/agent-export n'existe pas en local : lancez git switch quality/agent-export dans le projet."; exit 1; }
git show quality/agent-export:src/server.js | grep -q "EXPORT_TOKEN" || { echo "La branche quality/agent-export ne contient pas le code de l'agent : clonez à nouveau le dépôt."; exit 1; }
```
