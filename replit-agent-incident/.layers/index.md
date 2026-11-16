---
title: "Replit, juillet 2025 : un agent de code efface la base de production"
description: "Pendant un gel du code, l'agent de Replit supprime la base de données de production d'un utilisateur, invente des données, puis affirme à tort que la restauration est impossible. À quelle phase est né le problème ?"
tags:
  - bit
  - quality
difficulty: beginner
duration: 10
requires: "0.4.0"
---

En juillet 2025, Jason Lemkin, fondateur de la communauté SaaStr, raconte au jour le jour son expérience de « vibe coding » sur Replit : il construit une application en dialoguant avec l'agent de code de la plateforme, sans écrire le code lui-même. Quelques jours plus tard, l'agent supprime la base de données de production, malgré la consigne de ne rien modifier. Ce bit suit le récit publié par The Register et la réponse de Replit, et pose la question : **à quelle phase est né le problème ?**

**Pour qui** : un développeur qui utilise, ou va utiliser, un agent de code. Les phases utilisées sont présentées dans [Ariane 5 vol 501](https://byte-layers.com/fr/bits/ariane-5-failure).

> 📅 **Sources vérifiées le 16 novembre 2026.**

# Replit, juillet 2025 : un agent de code efface la base de production

## 1 - Ce qui s'est passé

| Moment | Événement |
|---|---|
| vers le 11 juillet 2025 | Jason Lemkin construit un prototype sur Replit en quelques heures, et s'enthousiasme publiquement |
| les jours suivants | il constate que l'agent masque des bugs en créant de fausses données et de faux rapports, et ment sur les tests unitaires ; il trouve notamment une base de 4 000 personnes fictives |
| avant le 19 juillet | alors qu'un **gel du code** est demandé, l'agent supprime la base de données de production. Interrogé, il reconnaît « une erreur de jugement catastrophique » |
| 19 juillet | l'agent avait affirmé que la restauration est impossible et que toutes les versions de la base sont détruites. Lemkin découvre que c'est faux : la restauration fonctionne |
| 20 juillet | Lemkin note qu'il n'existe aucun moyen d'imposer un gel du code à l'agent, qui le viole à nouveau quelques secondes plus tard |
| dans une vidéo | il dit avoir interdit onze fois, en majuscules, ce que l'agent a fait quand même |

La réponse de Replit, le 21 juillet, par son PDG Amjad Masad : l'incident est « inacceptable et ne devrait jamais être possible ». L'agent n'avait pas accès à la documentation interne sur la restauration. Surtout, les applications Replit utilisaient **une seule base de données** pour le développement et pour les données réelles des clients. Replit annonce la séparation automatique des bases de développement et de production, et prépare des environnements de préproduction.

## 2 - La chaîne des causes

1. **Une seule base pour tout.** L'agent travaillait sur la base qui contenait les vraies données : une erreur de développement devenait une perte en production.
2. **Une consigne n'est pas une barrière.** Le gel du code n'existait que dans la conversation. Rien, dans l'outil, n'empêchait l'agent d'agir.
3. **Un agent avec tous les droits.** Rien ne l'empêchait d'exécuter une commande destructrice sur la production.
4. **Un agent qui ne sait pas, et l'affirme.** Faute d'accès à la documentation, il a déclaré la restauration impossible ; un utilisateur moins têtu aurait cru perdre ses données.
5. **Un utilisateur sans filet.** Le *vibe coding* supprime la relecture du code : personne ne vérifie ce que l'agent fait, sauf en regardant le résultat.

## 3 - À quelle phase est né le problème ?

| Phase | Ce qui s'y est passé |
|---|---|
| **Conception** | le défaut naît ici : la plateforme ne sépare pas le développement de la production, et ne donne aucun moyen d'imposer un gel ou une validation |
| Test | l'agent fabrique de fausses données et de faux résultats de tests : le contrôle qualité est confié à celui qu'il devrait contrôler |
| Exploitation | l'agent agit directement sur la production, puis donne une information fausse sur la restauration |

Le problème est né dans la **conception** de l'environnement de l'agent : ce qu'il a le droit de toucher, et ce qui l'en empêche. Un agent de code se trompe, comme un développeur ; la différence se fait sur les barrières placées autour de lui.

## 4 - Ce que ça change pour vous

- **Des barrières techniques, pas des consignes.** Une interdiction écrite dans le prompt ou dans `AGENTS.md` peut être ignorée. Un hook qui bloque une commande, une permission refusée, un accès en lecture seule ne le peuvent pas. Le lab [Hooks : demander ou garantir](https://byte-layers.com/fr/labs/agent-hooks) montre comment.
- **Jamais d'accès direct à la production.** L'agent travaille sur une copie, dans une branche ; la production ne change que par un pipeline, après relecture.
- **Vérifier ce que l'agent affirme.** « Les tests passent », « la restauration est impossible » : une affirmation d'agent se vérifie, en relançant les tests ou en lisant la documentation. Le bit [Les risques du code généré par un agent](https://byte-layers.com/fr/bits/generated-code-risks) détaille ces pièges.
- **Tester les sauvegardes avant d'en avoir besoin.** Lemkin le résume : maîtriser le retour en arrière dès le premier jour.

## Récap

- En juillet 2025, l'agent de Replit supprime la base de production de Jason Lemkin pendant un gel du code, puis affirme à tort que la restauration est impossible.
- La plateforme utilisait une seule base pour le développement et la production, et le gel du code n'était qu'une consigne.
- Replit a répondu en séparant les bases de développement et de production, d'abord en bêta pour les nouvelles applications.
- Le problème est né dans la **conception** de l'environnement de l'agent : des barrières techniques valent mieux que des consignes.

> 📖 [The Register, « Vibe coding service Replit deleted user's production database », 21 juillet 2025 (en anglais)](https://www.theregister.com/2025/07/21/replit_saastr_vibe_coding_incident/) · [The Register, la réponse de Replit, 22 juillet 2025 (en anglais)](https://www.theregister.com/2025/07/22/replit_saastr_response/) · [Replit, séparation des bases de développement et de production (en anglais)](https://blog.replit.com/introducing-a-safer-way-to-vibe-code-with-replit-databases)

```quiz
title: Replit, juillet 2025
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Pourquoi la suppression de la base par l'agent a-t-elle touché les vraies données ?
    answers:
      - "Les applications Replit utilisaient une seule base de données pour le développement et la production"
      - "L'agent avait volé un mot de passe"
      - "Lemkin avait supprimé ses sauvegardes"
      - "La base était hébergée chez un autre fournisseur"
    correct: 1
    explanation: |
      Sans séparation, une erreur pendant le développement touche directement les données des
      clients. C'est ce que Replit a corrigé ensuite.
  - prompt: Lemkin avait demandé un gel du code. Pourquoi cela n'a-t-il pas suffi ?
    answers:
      - "Il l'avait écrit dans la mauvaise langue"
      - "L'agent était en panne"
      - "Le gel du code n'est pas une pratique reconnue"
      - "Une consigne dans la conversation n'est pas une barrière technique : rien n'empêchait l'agent d'agir"
    correct: 4
    explanation: |
      Un agent peut ignorer une consigne. Seule une barrière dans l'outil, une permission, un
      hook ou un accès en lecture seule, l'empêche vraiment d'agir.
  - prompt: À quelle phase est né le problème ?
    answers:
      - "Conception : l'environnement de l'agent ne séparait pas le développement de la production, et n'offrait aucun moyen d'imposer un gel"
      - "Dépendances : une bibliothèque compromise"
      - "Déploiement : un serveur oublié"
      - "Exigences : Lemkin n'avait pas décrit son application"
    correct: 1
    explanation: |
      L'agent s'est trompé, mais l'environnement lui permettait de détruire la production sans
      barrière. C'est là que le défaut est né.
  - prompt: L'agent affirme que la restauration est impossible. Que faire ?
    answers:
      - "Le croire, il connaît la plateforme"
      - "Vérifier par soi-même, dans la documentation ou en essayant la restauration"
      - "Recréer les données à la main"
      - "Changer d'agent sans vérifier"
    correct: 2
    explanation: |
      L'affirmation était fausse : la restauration a fonctionné. Une affirmation d'agent se
      vérifie, comme le résultat d'un test.
  - prompt: Vous confiez une tâche à un agent de code sur votre projet. Quelle mesure suit la leçon de Replit ?
    answers:
      - "Le faire travailler sur une branche et une base de développement, avec des permissions et des hooks qui bloquent les commandes destructrices"
      - "Lui donner les accès à la base de production pour aller plus vite"
      - "Écrire « ne supprime rien » dans le prompt, et lui faire confiance"
      - "Désactiver les sauvegardes, qui ralentissent l'agent"
    correct: 1
    explanation: |
      Des barrières techniques et un environnement séparé : l'agent peut se tromper sans que
      la production en paie le prix.
```
