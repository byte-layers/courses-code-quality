---
title: "Un pipeline de sécurité : dépendances, secrets, images épinglées"
description: "Lab GitLab CI : ajouter au pipeline un audit des dépendances et une détection de secrets, épingler les images par empreinte, puis planter une dépendance vulnérable et un secret, les voir arrêtés, et corriger, historique de Git compris."
tags:
  - lab
  - quality
  - security
  - gitlab
  - ci
difficulty: beginner
duration: 60
introTitle: "Le lab"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Le bit [SAST, SCA, secrets, épinglage](https://byte-layers.com/fr/bits/sast-sca-secrets) décrit quatre portes par lesquelles une faille entre dans un logiciel. Le pipeline du lab [Une barrière qualité sur les merge requests](https://byte-layers.com/fr/labs/merge-request-quality-gate) en garde déjà une : Semgrep et vos règles surveillent votre code. Dans ce lab, vous gardez les trois autres :

| Porte | Ce que vous ajoutez |
|---|---|
| les dépendances | un job `audit`, avec `npm audit` |
| les secrets | un job `secrets`, avec **gitleaks** |
| les outils de build | les images Docker du pipeline, **épinglées** par leur empreinte |

Puis vous jouez l'attaque, ou plutôt l'erreur : une merge request ajoute une dépendance vulnérable et un secret écrit en dur. Le pipeline l'arrête, et vous corrigez, jusqu'à l'historique de Git.

**Niveau** : débutant en sécurité ; vous avez fait le lab de la barrière qualité, et vous savez ouvrir une merge request.

**Ce qu'il faut** :
- le lab [Une barrière qualité sur les merge requests](https://byte-layers.com/fr/labs/merge-request-quality-gate) terminé : le projet public `todo-api-quality` sur gitlab.com a son pipeline, la merge request de `clean-export` est fusionnée, et `~/byte-layers/quality/gitlab-url` contient son adresse ;
- **Docker**, pour lire l'empreinte des images ;
- Node.js 22.13 ou plus récent, avec npm, et un éditeur de code.

Le pipeline consomme environ trois minutes de calcul de gitlab.com par exécution, et ce lab en lance quatre ou cinq.

**Vos fichiers** restent dans `~/byte-layers/quality/todo-api`. Le secret planté dans ce lab est **faux** : il ne donne accès à rien. Ne mettez jamais un vrai secret dans un dépôt, même pour un essai.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
