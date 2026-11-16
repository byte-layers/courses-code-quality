---
title: "Trivy, mars 2026 : quand l'outil de sécurité vole les secrets du pipeline"
description: "Des identifiants mal révoqués, 76 étiquettes de l'action GitHub trivy-action déplacées vers un code malveillant, et des pipelines qui livrent leurs secrets en croyant scanner leurs vulnérabilités. À quelle phase est né le problème ?"
tags:
  - bit
  - quality
difficulty: beginner
duration: 10
requires: "0.4.0"
---

Trivy, d'Aqua Security, est l'un des scanners de vulnérabilités open source les plus utilisés : on l'ajoute à un pipeline pour vérifier qu'une image ou un projet ne contient pas de failles connues. Le 19 mars 2026, des attaquants publient de fausses versions de Trivy et de ses actions GitHub. Les pipelines qui les utilisent continuent de scanner normalement, mais envoient d'abord leurs secrets aux attaquants. Ce bit suit l'avis de sécurité du projet et pose la question : **à quelle phase est né le problème ?**

**Pour qui** : un développeur qui a déjà vu un pipeline de CI. Les phases utilisées sont présentées dans [Ariane 5 vol 501](https://byte-layers.com/fr/bits/ariane-5-failure).

> 📅 **Sources vérifiées le 16 novembre 2026.**

# Trivy, mars 2026 : quand l'outil de sécurité vole les secrets du pipeline

## 1 - Une étiquette n'est pas une version figée

Dans GitHub Actions, un pipeline appelle une action par son nom, suivi d'une **référence** :

```yaml
# une étiquette : le propriétaire du dépôt peut la déplacer vers un autre commit
- uses: aquasecurity/trivy-action@0.28.0

# l'empreinte complète d'un commit : elle désigne un contenu, et un seul
- uses: aquasecurity/trivy-action@<empreinte-sha-complete-de-40-caracteres>
```

Une étiquette Git, comme `0.28.0`, est un simple pointeur vers un commit. Quelqu'un qui a les droits d'écriture sur le dépôt peut la **déplacer** vers un autre commit, avec un `git push --force`. Le pipeline, lui, ne voit rien : il demande `0.28.0` et reçoit le nouveau contenu. Seule l'empreinte SHA complète d'un commit garantit le code exécuté.

## 2 - Ce qui s'est passé

| Date | Événement |
|---|---|
| quelques semaines avant | les attaquants exploitent une erreur de configuration des GitHub Actions du projet Trivy et récupèrent un jeton d'accès privilégié. Le projet signale l'incident et renouvelle ses identifiants, mais **pas tous** : certains restent probablement valides |
| 19 mars 2026 | avec ces identifiants, les attaquants publient une version malveillante de Trivy, la 0.69.4, et **déplacent 76 des 77 étiquettes** de `aquasecurity/trivy-action`, ainsi que les 7 étiquettes de `aquasecurity/setup-trivy`, vers des commits malveillants |
| 19 et 20 mars | les composants malveillants restent en ligne de 3 à 12 heures selon le cas, avant d'être retirés |
| 22 mars | avec des identifiants Docker Hub volés séparément, les attaquants publient des images Docker malveillantes de Trivy, 0.69.5 et 0.69.6 |

Le code malveillant s'exécutait **avant** le vrai scan : le pipeline se terminait normalement, avec un rapport de vulnérabilités, pendant qu'il avait déjà collecté et envoyé les jetons, les clés de cloud, les clés SSH et les autres secrets présents sur la machine de CI. Le logiciel malveillant se présente lui-même comme l'œuvre du groupe TeamPCP. La faille est référencée CVE-2026-33634.

**Qui a échappé à l'attaque** : les pipelines qui appelaient l'action par l'empreinte complète d'un commit postérieur au 9 avril 2025 (les plus anciens appelaient eux-mêmes une référence mobile), ou l'image par son empreinte (son *digest*), et ceux qui utilisaient une version déjà publiée et protégée par les *immutable releases* de GitHub, comme Trivy 0.69.3 ou l'étiquette `0.35.0` de `trivy-action`.

## 3 - La chaîne des causes

1. **Une configuration de CI trop permissive** chez le projet : elle a laissé fuiter un jeton privilégié.
2. **Une révocation incomplète** après le premier incident : elle a probablement laissé aux attaquants l'accès utilisé le 19 mars. Le 22, ils publient sur Docker Hub avec d'autres identifiants volés.
3. **Des références mobiles chez les utilisateurs** : appeler une action par une étiquette, ou une image par `latest`, c'est accepter d'exécuter ce que le propriétaire, ou celui qui a volé ses accès, publiera demain.
4. **Des secrets à portée de main** : sur la machine de CI, le code d'une action peut lire tous les secrets que le pipeline lui expose.

## 4 - À quelle phase est né le problème ?

| Phase | Chez qui | Ce qui s'y est passé |
|---|---|---|
| Exploitation | le projet Trivy | une configuration de CI qui laisse fuiter un jeton ; une révocation incomplète après le premier incident ; des identifiants Docker Hub volés |
| **Dépendances** | les utilisateurs | des pipelines qui exécutent un code tiers désigné par une référence que n'importe quel détenteur des accès peut changer |

Pour l'équipe qui utilise Trivy, le problème est né dans ses **dépendances** : un code qu'elle n'a pas écrit, choisi pour sa sécurité, et exécuté avec ses secrets. Elle ne pouvait pas empêcher la compromission du projet ; elle pouvait choisir de n'exécuter que des versions figées.

## 5 - Ce que ça change pour vous

- **Figer les références.** Une action par l'empreinte SHA complète d'un commit, une image Docker par son *digest*. Un outil comme Dependabot ou Renovate propose ensuite les mises à jour, que l'équipe relit.
- **Ne pas se fier aux badges.** L'avis d'Aqua note que l'interface de GitHub affichait un badge « Immutable » à côté d'étiquettes empoisonnées : seule l'empreinte complète protège vraiment.
- **Donner peu de secrets, et pour peu de temps.** Un job de scan n'a pas besoin des clés de déploiement ; un jeton à courte durée de vie vaut moins cher quand il est volé.
- **Révoquer tout, après un incident.** Une révocation partielle a probablement laissé la porte ouverte.

## Récap

- Le 19 mars 2026, Trivy 0.69.4, 76 des 77 étiquettes de `trivy-action` et les 7 de `setup-trivy` pointent vers du code qui vole les secrets du pipeline.
- La cause côté projet : un jeton fuité par une configuration de CI, puis une révocation incomplète.
- Le code malveillant s'exécutait avant le vrai scan : les pipelines semblaient normaux.
- Pour les utilisateurs, le problème est né dans les **dépendances** ; les pipelines qui appelaient l'action par l'empreinte complète d'un commit récent ont échappé à l'attaque.

> 📖 [Avis de sécurité GHSA-69fq-xp46-6x23, CVE-2026-33634 (en anglais)](https://github.com/advisories/GHSA-69fq-xp46-6x23) · [Aqua Security, « Trivy supply chain attack » (en anglais)](https://www.aquasec.com/blog/trivy-supply-chain-attack/) · [GitHub, utiliser les actions tierces en sécurité (en anglais)](https://docs.github.com/en/actions/reference/security/secure-use#using-third-party-actions)

```quiz
title: Trivy, mars 2026
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Pourquoi un pipeline qui appelait `aquasecurity/trivy-action@0.28.0` a-t-il exécuté le code malveillant ?
    answers:
      - "Il avait installé une mauvaise version par erreur"
      - "L'étiquette 0.28.0 avait été déplacée vers un commit malveillant ; le pipeline a reçu ce nouveau contenu sans rien voir"
      - "GitHub avait remplacé l'action par une autre"
      - "Le pipeline avait désactivé les contrôles de sécurité"
    correct: 2
    explanation: |
      Une étiquette Git est un pointeur. Avec les droits d'écriture, on la déplace vers un autre
      commit ; ceux qui l'appellent reçoivent le nouveau code.
  - prompt: Pourquoi les équipes n'ont-elles pas remarqué le vol ?
    answers:
      - "Le pipeline échouait, mais personne ne le lisait"
      - "Le code malveillant s'exécutait avant le vrai scan, et le pipeline se terminait normalement"
      - "Le vol avait lieu la nuit"
      - "Trivy était désactivé"
    correct: 2
    explanation: |
      Le rapport de vulnérabilités était bien produit. Rien, dans le résultat du job,
      n'indiquait l'envoi des secrets.
  - prompt: Côté projet Trivy, quelle erreur a probablement permis l'attaque du 19 mars ?
    answers:
      - "Une révocation incomplète après le premier incident : des identifiants restaient valides"
      - "Un mot de passe affiché dans la documentation"
      - "Une version de Go trop ancienne"
      - "L'absence de tests unitaires"
    correct: 1
    explanation: |
      Après le premier incident, le renouvellement des identifiants n'a pas été complet. Les
      attaquants ont gardé un accès.
  - prompt: Pour une équipe qui utilise Trivy dans son pipeline, à quelle phase est né le problème ?
    answers:
      - "Exigences"
      - "Codage de son application"
      - "Dépendances : un code tiers, désigné par une référence mobile, exécuté avec ses secrets"
      - "Test de son application"
    correct: 3
    explanation: |
      L'équipe n'a écrit aucune ligne fautive. Le risque venait du code tiers qu'elle exécutait,
      et de la façon dont elle le désignait.
  - prompt: Quelle pratique protège le mieux d'une étiquette déplacée ?
    answers:
      - "Appeler l'action par l'empreinte SHA complète d'un commit, et relire ses mises à jour"
      - "Utiliser l'étiquette `latest`"
      - "Faire confiance au badge « Immutable »"
      - "Lancer le scan deux fois"
    correct: 1
    explanation: |
      L'empreinte complète désigne un contenu unique. L'avis d'Aqua rappelle que des étiquettes
      empoisonnées portaient le badge « Immutable ».
```
