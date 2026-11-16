---
title: "Une barrière qualité sur les merge requests"
description: "Lab GitLab CI : faire tourner ESLint, jscpd et Semgrep sur chaque merge request, afficher les défauts dans l'onglet des rapports, et refuser la fusion du code de l'agent tant qu'il dépasse les seuils."
tags:
  - lab
  - quality
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

Au lab [Mesurer la dette avec Semgrep, ESLint et jscpd](https://byte-layers.com/fr/labs/measure-debt), vous avez mesuré trois versions du projet sur votre poste. Une mesure qu'il faut penser à lancer finit par être oubliée. Dans ce lab, les mêmes outils tournent tout seuls sur chaque **merge request** (MR), dans un pipeline GitLab CI, et GitLab refuse la fusion tant qu'ils échouent : c'est une **barrière qualité** (*quality gate*).

Vous publiez le projet sur gitlab.com, vous y ajoutez les outils et le pipeline, puis vous proposez deux merge requests : le code de l'agent, que la barrière doit arrêter, et votre version corrigée, qu'elle doit laisser passer.

**Niveau** : débutant en qualité logicielle ; vous savez pousser une branche et ouvrir une merge request. Les bases d'un pipeline sont rappelées au fil du lab ; le stack [Automatiser avec GitLab CI](https://byte-layers.com/fr/stacks/gitlab-ci) les détaille.

**Ce qu'il faut** :
- le lab [Mesurer la dette avec Semgrep, ESLint et jscpd](https://byte-layers.com/fr/labs/measure-debt) terminé : `~/byte-layers/quality` contient `eslint.config.js` et `.semgrep/rules.yml`, et le projet `~/byte-layers/quality/todo-api` a sa branche `clean-export` ;
- un compte gratuit sur [gitlab.com](https://gitlab.com/), avec une clé SSH enregistrée ;
- Node.js 22.13 ou plus récent, avec npm, et un éditeur de code.

Sur gitlab.com, les jobs sont exécutés par des **runners partagés** : vous n'avez rien à installer. Si un job échoue en vous demandant de valider votre compte, suivez le lien affiché : GitLab demande par exemple un numéro de téléphone ou une carte bancaire, qui ne sera pas débitée. Les minutes de calcul gratuites sont limitées : ce pipeline en consomme environ deux par exécution.

**Vos fichiers** restent dans `~/byte-layers/quality`. Le projet GitLab doit être **public** : les vérifications lisent son état sans mot de passe.

**Sous Windows**, tapez les commandes dans le terminal **Git Bash**.
