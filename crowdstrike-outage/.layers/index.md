---
title: "CrowdStrike, 19 juillet 2024 : une mise à jour de contenu, 8,5 millions de PC à l'arrêt"
description: "Un fichier de configuration attendu par 21 champs, un programme qui n'en fournit que 20, un validateur qui ne voit rien, et un envoi à tous les clients en même temps. À quelle phase est né le problème ?"
tags:
  - bit
  - quality
difficulty: beginner
duration: 10
requires: "0.4.0"
---

Le 19 juillet 2024, des millions d'ordinateurs Windows affichent un écran bleu et ne redémarrent plus : compagnies aériennes, hôpitaux, banques, chaînes de télévision. La cause n'est pas une attaque, mais une mise à jour de l'agent de sécurité Falcon, de l'éditeur CrowdStrike. Microsoft estime le nombre d'appareils touchés à 8,5 millions. Ce bit suit l'analyse de cause racine publiée par CrowdStrike et pose la question : **à quelle phase est né le problème ?**

**Pour qui** : tout développeur. Les phases utilisées sont présentées dans [Ariane 5 vol 501](https://byte-layers.com/fr/bits/ariane-5-failure).

> 📅 **Sources vérifiées le 16 novembre 2026.**

# CrowdStrike, 19 juillet 2024 : une mise à jour de contenu, 8,5 millions de PC à l'arrêt

## 1 - Deux sortes de mises à jour

L'agent Falcon tourne sur chaque poste, au cœur du système : un pilote chargé dès le démarrage de Windows. Il reçoit deux sortes de mises à jour :

| Mise à jour | Ce qu'elle contient | Rythme |
|---|---|---|
| **Le capteur** | le programme lui-même, avec ses « types de modèles » : des familles de détection, chacune définissant les données qu'elle examine | des versions espacées, testées avant publication |
| **Le contenu de réponse rapide** | des « instances » de ces modèles : des règles de détection, livrées dans des fichiers de canal | souvent, plusieurs fois par jour, pour suivre les nouvelles attaques |

Le contenu n'est pas du code : c'est de la configuration, lue par un **interpréteur** dans le capteur. Mais une configuration que le programme lit mal peut le faire planter.

## 2 - Ce qui s'est passé

| Date | Événement |
|---|---|
| février 2024 | la version 7.11 du capteur ajoute un modèle pour surveiller les échanges entre processus (IPC). Sa définition prévoit **21 champs** ; le code du capteur ne lui en fournit que **20** |
| les mois suivants | plusieurs livraisons d'instances de ce modèle se passent sans problème : elles utilisent un joker (« n'importe quelle valeur ») pour le 21e champ, que l'interpréteur ne lit donc jamais |
| 19 juillet 2024 | deux nouvelles instances sont livrées dans le fichier de canal 291. L'une compare vraiment le 21e champ à une valeur |
| juste après | l'interpréteur cherche le 21e champ, lit la mémoire au-delà de la fin du tableau, et le système plante. Le pilote étant chargé dès le démarrage, beaucoup de machines replantent à chaque redémarrage |

Le fichier fautif est livré à tous les postes connectés en même temps. Sur les machines bloquées, la réparation demande souvent une intervention manuelle, poste par poste.

## 3 - La chaîne des causes

L'analyse de CrowdStrike liste six constats :

1. **Un décompte non vérifié.** Le nombre de champs du modèle n'est pas vérifié à la compilation du capteur : 21 d'un côté, 20 de l'autre.
2. **Une lecture sans contrôle de bornes.** L'interpréteur lit le 21e élément sans vérifier que le tableau en contient 21.
3. **Des tests trop gentils.** Les tests du modèle utilisaient un joker sur le 21e champ : ils ne pouvaient pas déclencher la lecture fautive.
4. **Un validateur qui se trompe de référence.** Le validateur de contenu, qui contrôle chaque instance avant livraison, vérifiait qu'elle respectait la **définition** à 21 champs, pas ce que le capteur fournissait vraiment.
5. **Pas de test dans le vrai interpréteur.** Les nouvelles instances passaient le validateur, sans être exécutées dans un capteur réel avant la livraison.
6. **Pas de déploiement progressif.** Le contenu partait vers tous les clients d'un coup, sans premiers cercles de machines pour détecter un plantage.

## 4 - À quelle phase est né le problème ?

| Phase | Ce qui s'y est passé |
|---|---|
| **Conception et codage** | le défaut naît ici, en février : la définition annonce 21 champs, le code en fournit 20, et l'interpréteur ne contrôle pas les bornes |
| Test | les tests n'utilisent que des jokers sur le 21e champ ; le validateur contrôle la mauvaise référence |
| **Déploiement** | le contenu part partout en même temps : un défaut qui aurait touché quelques machines en touche des millions |

Le défaut est né en **conception et codage**, cinq mois avant l'incident. Le **déploiement** sans étapes a transformé un bug en panne mondiale. Les mesures prises par CrowdStrike visent chacune de ces phases : vérification du nombre de champs à la compilation, contrôle de bornes dans l'interpréteur, tests sans joker pour chaque champ, déploiement par cercles successifs avec retour en arrière, et contrôle laissé aux clients sur le moment de la mise à jour.

## 5 - Ce que ça change pour vous

- **La configuration est du code.** Un fichier YAML, JSON, un drapeau de fonctionnalité : s'il change le comportement d'un programme en production, il mérite des tests et un déploiement progressif, comme le code.
- **Un validateur doit vérifier la réalité.** Contrôler un fichier par rapport à une documentation ne prouve pas que le programme qui le lit l'accepte. Le test qui compte exécute le vrai programme.
- **Tester les cas limites, pas seulement le cas heureux.** Un test qui passe à côté d'un champ ne le teste pas.
- **Déployer par étapes.** D'abord quelques machines, puis un cercle plus large, avec un retour arrière prêt.

## Récap

- Le 19 juillet 2024, une mise à jour de contenu de CrowdStrike Falcon bloque environ 8,5 millions de PC Windows.
- Un modèle de détection défini avec 21 champs n'en recevait que 20 ; la première règle qui a lu le 21e a provoqué une lecture hors limites.
- Les tests et le validateur n'ont rien vu, et le contenu est parti vers tous les clients en même temps.
- Le défaut est né en **conception et codage** ; le **déploiement** sans étapes en a fait une panne mondiale.

> 📖 [CrowdStrike, « External Technical Root Cause Analysis — Channel File 291 », août 2024 (PDF, en anglais)](https://www.crowdstrike.com/wp-content/uploads/2024/08/Channel-File-291-Incident-Root-Cause-Analysis-08.06.2024.pdf) · [Microsoft, « Helping our customers through the CrowdStrike outage », juillet 2024 (en anglais)](https://blogs.microsoft.com/blog/2024/07/20/helping-our-customers-through-the-crowdstrike-outage/)

```quiz
title: CrowdStrike, 19 juillet 2024
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Qu'est-ce qui a fait planter les postes ?
    answers:
      - "Un virus déguisé en mise à jour"
      - "L'interpréteur a lu un 21e champ alors que le capteur n'en fournissait que 20 : une lecture hors des limites du tableau"
      - "Une coupure du réseau de CrowdStrike"
      - "Une mise à jour de Windows publiée le même jour"
    correct: 2
    explanation: |
      La définition du modèle annonçait 21 champs, le code en fournissait 20. La première règle
      qui a vraiment utilisé le 21e a déclenché la lecture fautive.
  - prompt: Pourquoi les tests n'ont-ils pas détecté le défaut ?
    answers:
      - "Il n'y avait aucun test"
      - "Les tests utilisaient un joker sur le 21e champ, si bien que l'interpréteur ne le lisait jamais"
      - "Les tests tournaient sur Linux"
      - "Les tests avaient été désactivés pour aller plus vite"
    correct: 2
    explanation: |
      Un test qui ne fait jamais lire le champ ne peut pas révéler un problème sur ce champ :
      il faut tester les cas limites.
  - prompt: À quelle phase est né le défaut ?
    answers:
      - "Exploitation, le 19 juillet"
      - "Conception et codage, en février : 21 champs définis, 20 fournis, et pas de contrôle de bornes"
      - "Dépendances : une bibliothèque tierce compromise"
      - "Exigences : le client n'avait pas demandé de détection IPC"
    correct: 2
    explanation: |
      Le défaut existait depuis la version 7.11 du capteur. Il attendait une règle qui lise
      le 21e champ.
  - prompt: Quelle phase a transformé un bug en panne mondiale ?
    answers:
      - "Le déploiement : le contenu est parti vers tous les clients en même temps"
      - "Les exigences"
      - "La documentation"
      - "La conception de l'interface"
    correct: 1
    explanation: |
      Un déploiement par cercles successifs aurait arrêté la livraison après les premières
      machines en panne.
  - prompt: Votre application lit un fichier de configuration modifiable sans redéployer le code. Quelle leçon tirer de CrowdStrike ?
    answers:
      - "La configuration n'est pas du code, elle ne mérite pas de tests"
      - "Traiter la configuration comme du code : la tester avec le vrai programme et la déployer par étapes"
      - "Ne jamais utiliser de fichier de configuration"
      - "Valider le fichier uniquement par rapport à sa documentation"
    correct: 2
    explanation: |
      Le contenu de CrowdStrike était de la configuration, et il a bloqué des millions de
      machines. Ce qui change un comportement en production se teste et se déploie avec soin.
```
