---
title: "Le modèle de qualité ISO/IEC 25010"
description: "Les neuf caractéristiques de la qualité d'un produit logiciel selon ISO/IEC 25010:2023 et leurs sous-caractéristiques, ce qui a changé depuis 2011, la place de l'accessibilité, et la différence avec ISO/IEC 27001."
tags:
  - bit
  - quality
difficulty: beginner
duration: 15
requires: "0.4.0"
---

« Ce logiciel est de mauvaise qualité » ne dit rien d'utile : trop lent ? plein de bugs ? impossible à modifier ? inutilisable au clavier ? Pour parler de qualité précisément, il faut un vocabulaire commun. La norme internationale **ISO/IEC 25010** en fournit un : elle découpe la qualité d'un produit en neuf caractéristiques, chacune divisée en sous-caractéristiques. Ce bit les présente, avec un exemple de défaut pour chacune.

**Pour qui** : un développeur qui veut nommer ce qui ne va pas dans un logiciel, pour le mesurer, le prioriser, ou l'écrire dans une exigence. Aucun prérequis.

> 📅 **Sources vérifiées le 16 novembre 2026**, sur l'extrait officiel de la norme ISO/IEC 25010:2023 (deuxième édition).

# Le modèle de qualité ISO/IEC 25010

## 1 - Une norme pour nommer la qualité

**Ce que c'est.** ISO/IEC 25010 fait partie de la famille **SQuaRE** (*Systems and software Quality Requirements and Evaluation*), un ensemble de normes sur les exigences et l'évaluation de la qualité des logiciels. Elle définit un **modèle de qualité du produit** : une liste de caractéristiques qui couvre ce qu'on peut attendre d'un produit logiciel ou numérique.

À quoi elle sert, concrètement :

| Usage | Exemple |
|---|---|
| écrire des exigences complètes | passer chaque caractéristique en revue pour vérifier qu'aucune attente n'est oubliée : performance, sécurité, accessibilité |
| classer un incident ou une dette | « ce bug touche la fiabilité, cette dette la maintenabilité » |
| choisir quoi mesurer | un outil d'analyse de code mesure surtout la maintenabilité, une partie de la fiabilité et de la sécurité, rien sur la capacité d'interaction |
| se comprendre entre métiers | développeurs, testeurs, acheteurs et auditeurs emploient les mêmes mots |

ISO 25010 n'est pas une norme d'exigences, comme ISO 27001 : c'est un **cadre de référence**, que chaque projet adapte en choisissant les caractéristiques qui comptent pour lui et la façon de les mesurer. Il existe des certifications de produit qui s'appuient sur elle, comme celle d'AENOR en Espagne, mais elles restent rares et ne couvrent que quelques caractéristiques.

La version en vigueur est la **deuxième édition, de 2023**. Elle remplace celle de 2011, qui reste très citée : vous croiserez souvent ses termes.

## 2 - Les neuf caractéristiques

Les noms officiels sont en anglais ; la traduction est celle de ce bit. Chaque ligne donne les sous-caractéristiques, et un exemple de défaut qui touche la caractéristique.

| Caractéristique | Sous-caractéristiques | Un défaut typique |
|---|---|---|
| **Adéquation fonctionnelle** (*functional suitability*) | complétude, exactitude, pertinence des fonctions | l'export oublie les tâches terminées ; un calcul de TVA arrondit mal |
| **Efficacité des performances** (*performance efficiency*) | comportement temporel, utilisation des ressources, capacité | la page met huit secondes à s'afficher ; le service sature à cent utilisateurs |
| **Compatibilité** (*compatibility*) | coexistence, interopérabilité | l'application bloque le port d'un autre service ; l'export est illisible par l'outil du client |
| **Capacité d'interaction** (*interaction capability*) | reconnaissance de l'adéquation, facilité d'apprentissage, facilité d'utilisation, protection contre les erreurs, engagement de l'utilisateur, inclusivité, assistance à l'utilisateur, auto-description | un bouton « Supprimer » sans confirmation ; un formulaire inutilisable au clavier |
| **Fiabilité** (*reliability*) | absence de défauts, disponibilité, tolérance aux pannes, récupérabilité | le service plante une fois par jour ; une coupure de la base de données fait tout tomber |
| **Sécurité** (*security*) | confidentialité, intégrité, non-répudiation, imputabilité, authenticité, résistance | un utilisateur lit les données d'un autre ; aucun journal ne dit qui a supprimé un compte |
| **Maintenabilité** (*maintainability*) | modularité, réutilisabilité, analysabilité, modifiabilité, testabilité | une fonction de 400 lignes que personne n'ose toucher ; du code copié-collé en cinq endroits |
| **Flexibilité** (*flexibility*) | adaptabilité, scalabilité, installabilité, remplaçabilité | l'application ne tourne que sur un système précis ; impossible d'ajouter des serveurs quand la charge monte |
| **Innocuité** (*safety*) | contrainte opérationnelle, identification des risques, sécurité en cas de défaillance, alerte de danger, intégration sûre | le logiciel d'une pompe à perfusion ne se met pas en position sûre quand un capteur lâche |

## 3 - Deux sécurités : *security* et *safety*

En français, un seul mot, « sécurité », recouvre deux caractéristiques différentes de la norme. Les confondre fait passer à côté d'exigences.

| | *Security* : sécurité de l'information | *Safety* : innocuité, sûreté de fonctionnement |
|---|---|---|
| Ce qu'on protège | les données et le système, contre un attaquant | les personnes, les biens et l'environnement, contre les effets du système lui-même |
| La question | « quelqu'un peut-il voler, modifier ou bloquer ? » | « ce système peut-il blesser, détruire ou mettre en danger ? » |
| Exemples de domaines | toute application en ligne | santé, transport, industrie, objets connectés qui agissent sur le monde physique |

La caractéristique *safety* est **nouvelle dans l'édition 2023**. Elle reconnaît que de plus en plus de logiciels pilotent des objets physiques.

## 4 - Ce qui a changé depuis 2011

Si vous avez appris le modèle de 2011, ou si un document le cite, voici la correspondance :

| Édition 2011 | Édition 2023 |
|---|---|
| 8 caractéristiques | 9 : ajout de **safety** |
| *Usability* (utilisabilité) | **Interaction capability** (capacité d'interaction) |
| *Portability* (portabilité) | **Flexibility** (flexibilité), avec une nouvelle sous-caractéristique, la **scalabilité** |
| *Accessibility*, une sous-caractéristique de l'utilisabilité | partagée en **inclusivité** et **assistance à l'utilisateur** |
| *User interface aesthetics* | **user engagement** (engagement de l'utilisateur) |
| *Maturity*, sous la fiabilité | **faultlessness** (absence de défauts) |
| — | **resistance** (résistance) ajoutée sous la sécurité ; **self-descriptiveness** (auto-description) sous la capacité d'interaction |
| un seul document, avec le modèle de **qualité en usage** | la qualité en usage, celle que vit l'utilisateur dans son contexte, part dans une norme à part, **ISO/IEC 25019** |

## 5 - L'accessibilité dans le modèle

L'accessibilité, la capacité d'un produit à être utilisé par des personnes aux capacités les plus variées, handicap compris, n'est plus une ligne isolée. Elle se retrouve dans deux sous-caractéristiques de la capacité d'interaction :

- l'**inclusivité** : le produit peut être utilisé par des personnes d'horizons, d'âges et de capacités variés ;
- l'**assistance à l'utilisateur** : le produit aide l'utilisateur à atteindre son objectif, par exemple avec des alternatives aux contenus visuels ou sonores.

ISO 25010 dit **quoi** viser, pas **comment** le vérifier. Pour l'accessibilité du web, les règles vérifiables sont dans d'autres référentiels : les WCAG au niveau international, et en France le RGAA, obligatoire pour une partie des organisations.

## 6 - ISO/IEC 25010 et ISO/IEC 27001 : ne pas confondre

Les deux normes sont souvent citées ensemble, mais elles ne portent pas sur la même chose :

| | ISO/IEC 25010 | ISO/IEC 27001 |
|---|---|---|
| Objet | la qualité d'un **produit** | le **système de management** de la sécurité de l'information d'une **organisation** |
| Contenu | un modèle : caractéristiques et sous-caractéristiques | des exigences d'organisation : analyse des risques, politique, mesures de sécurité, amélioration continue |
| Certification | pas directement : c'est un cadre de référence, sur lequel s'appuient quelques rares certifications de produit | oui : une entreprise peut être certifiée par un organisme accrédité |
| Ce que dit « on la respecte » | « nos exigences couvrent ces caractéristiques, et on les mesure » | « notre organisation gère ses risques de sécurité selon cette norme, audit à l'appui » |

Une entreprise certifiée ISO 27001 peut livrer un logiciel peu maintenable ou inaccessible : la certification porte sur sa façon de gérer la sécurité, pas sur la qualité de chacun de ses produits. À l'inverse, la caractéristique *security* d'ISO 25010 décrit la sécurité **du produit**.

## Récap

- ISO/IEC 25010:2023 découpe la qualité d'un produit en **neuf caractéristiques** : adéquation fonctionnelle, efficacité des performances, compatibilité, capacité d'interaction, fiabilité, sécurité, maintenabilité, flexibilité, innocuité.
- C'est un **vocabulaire et un cadre**, pas une norme d'exigences : on s'en sert pour écrire des exigences complètes, classer des défauts et choisir quoi mesurer.
- *Security* protège les données contre un attaquant ; *safety*, nouvelle en 2023, protège les personnes contre le système.
- L'**accessibilité** se répartit entre inclusivité et assistance à l'utilisateur ; les règles vérifiables sont dans les WCAG et le RGAA.
- **ISO 27001** certifie le management de la sécurité d'une organisation ; ISO 25010 décrit la qualité d'un produit.

> 📖 [ISO/IEC 25010:2023 sur le site de l'ISO](https://www.iso.org/standard/78176.html) · [ISO/IEC 27001:2022 sur le site de l'ISO](https://www.iso.org/standard/27001) · [Les normes ISO/IEC 25000, portail iso25000.com (en anglais)](https://iso25000.com/en/iso-25000-standards/iso-25010)

```quiz
title: Le modèle de qualité ISO/IEC 25010
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Une fonction de 400 lignes, copiée en trois endroits, que personne n'ose modifier. Quelle caractéristique d'ISO 25010 est touchée ?
    answers:
      - "La fiabilité"
      - "La maintenabilité"
      - "L'adéquation fonctionnelle"
      - "La flexibilité"
    correct: 2
    explanation: |
      Modularité, réutilisabilité, analysabilité et modifiabilité sont des sous-caractéristiques
      de la maintenabilité. Le code peut très bien marcher : c'est son évolution qui coûte.
  - prompt: Le logiciel d'un robot industriel ne s'arrête pas quand un capteur de présence humaine tombe en panne. Quelle caractéristique est en cause ?
    answers:
      - "La sécurité (security)"
      - "La compatibilité"
      - "L'innocuité (safety)"
      - "L'efficacité des performances"
    correct: 3
    explanation: |
      Il ne s'agit pas d'un attaquant mais d'un danger pour les personnes : c'est la safety,
      ajoutée en 2023, avec la sous-caractéristique « sécurité en cas de défaillance ».
  - prompt: Dans l'édition 2023, que devient l'utilisabilité (usability) de l'édition 2011 ?
    answers:
      - "Elle est supprimée"
      - "Elle devient la capacité d'interaction (interaction capability)"
      - "Elle est fusionnée dans la fiabilité"
      - "Elle part dans ISO 27001"
    correct: 2
    explanation: |
      Usability devient interaction capability, et l'accessibilité y est répartie entre
      inclusivité et assistance à l'utilisateur.
  - prompt: Une entreprise annonce « nous sommes certifiés ISO 27001 ». Que pouvez-vous en conclure sur son logiciel ?
    answers:
      - "Qu'il respecte les neuf caractéristiques d'ISO 25010"
      - "Qu'il est accessible au sens du RGAA"
      - "Rien de direct : la certification porte sur la façon dont l'organisation gère la sécurité de l'information, pas sur la qualité de chaque produit"
      - "Qu'il ne contient aucune faille de sécurité"
    correct: 3
    explanation: |
      ISO 27001 certifie un système de management. La qualité d'un produit, sécurité du
      produit comprise, relève d'un autre cadre, comme ISO 25010.
  - prompt: À quoi ISO 25010 ne sert-il pas ?
    answers:
      - "À vérifier qu'une exigence n'oublie pas une dimension de la qualité"
      - "À classer un incident selon ce qu'il a touché"
      - "À choisir quoi mesurer dans un projet"
      - "À certifier l'organisation qui développe le logiciel"
    correct: 4
    explanation: |
      La norme décrit la qualité d'un produit. Certifier une organisation, par exemple sur sa
      gestion de la sécurité, relève d'autres normes comme ISO 27001.
```
