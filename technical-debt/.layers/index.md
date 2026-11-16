---
title: "La dette technique et le quadrant de Fowler"
description: "Ce qu'est la dette technique selon Ward Cunningham, ses types, qui la crée et qui la paie, comment l'identifier, et le quadrant de Martin Fowler pour distinguer une dette choisie d'un désordre."
tags:
  - bit
  - quality
difficulty: beginner
duration: 15
requires: "0.4.0"
---

« On n'a pas le temps de bien faire, on corrigera plus tard. » Cette phrase crée de la **dette technique**. Parfois, c'est une bonne décision, prise en connaissance de cause ; souvent, c'est un désordre qui s'accumule sans que personne ne l'ait choisi. Ce bit donne la définition d'origine, les formes que prend la dette, qui la crée et qui la paie, comment la repérer, et une grille pour juger chaque dette : le quadrant de Martin Fowler.

**Pour qui** : un développeur qui connaît les caractéristiques de qualité d'un logiciel, présentées dans [Le modèle de qualité ISO/IEC 25010](https://byte-layers.com/fr/bits/iso-25010-quality-model). La dette touche d'abord la **maintenabilité**.

> 📅 **Sources vérifiées le 16 novembre 2026.**

# La dette technique et le quadrant de Fowler

## 1 - Une métaphore financière

**D'où vient le terme.** Ward Cunningham, l'un des auteurs du Manifeste agile, l'a employé en 1992 dans un retour d'expérience présenté à la conférence OOPSLA. Son idée : livrer un premier code imparfait, c'est s'endetter. Un peu de dette accélère le développement, à condition de la rembourser vite en réécrivant. Le danger vient de la dette qu'on ne rembourse jamais : chaque minute passée sur un code « pas tout à fait juste » en paie les intérêts.

La métaphore a deux termes, qu'il faut garder distincts :

| Terme | Dans le code | Exemple |
|---|---|---|
| **Capital** | le travail qu'il faudrait pour remettre le code au propre | deux jours pour fusionner trois copies d'une même fonction |
| **Intérêts** | le surcoût payé à **chaque** modification tant que la dette reste | chaque correction doit être faite trois fois, et une copie est souvent oubliée |

Le calcul qui en découle : une dette dans un code qu'on ne touche jamais ne coûte presque rien ; la même dette dans un module modifié chaque semaine coûte cher. On rembourse d'abord là où les intérêts sont les plus lourds.

## 2 - Les formes de la dette

La dette ne se limite pas au code mal écrit :

| Type | À quoi on la reconnaît |
|---|---|
| **Code** | duplication, fonctions trop longues, noms trompeurs, complexité inutile |
| **Conception et architecture** | des modules trop liés, une base de données partagée par tous, un choix technique qui ne tient plus la charge |
| **Tests** | peu de tests, des tests lents ou instables, des parties que personne n'ose modifier faute de filet |
| **Documentation** | une procédure d'installation fausse, des décisions jamais écrites, un savoir qui ne vit que dans une tête |
| **Dépendances** | des bibliothèques en retard de plusieurs versions majeures, ou plus maintenues |
| **Infrastructure et livraison** | un déploiement manuel, un serveur configuré à la main que personne ne sait reconstruire |

Un agent de code ajoute une source : il écrit vite du code qui passe les tests, mais qui duplique l'existant ou ignore les conventions du projet. Ce risque est détaillé dans [Les risques du code généré par un agent](https://byte-layers.com/fr/bits/generated-code-risks).

## 3 - Qui la crée, qui la paie, qui décide

| Acteur | Son rôle face à la dette |
|---|---|
| Développeur, agent de code | la crée, souvent sans le vouloir, et la voit en premier |
| Tech lead, architecte | la repère, l'estime, propose quand la rembourser |
| Product owner, métier | arbitre entre nouvelles fonctionnalités et remboursement, souvent sans voir la dette |
| Direction | fixe les délais qui poussent à s'endetter, et le budget qui permet de rembourser |
| Utilisateurs, clients | paient les intérêts sans le savoir : fonctionnalités plus lentes à venir, bugs qui reviennent |

Le décalage est là : ceux qui voient la dette ne sont pas ceux qui décident du planning. Rendre la dette visible et chiffrée, en temps perdu ou en risque, est le premier travail de l'équipe technique.

## 4 - Identifier la dette

**Les signes, avant les outils.** Une modification simple qui prend des jours ; un module que tout le monde évite ; des bugs qui reviennent toujours au même endroit ; un nouvel arrivant qui met des semaines à livrer sa première modification.

**Les mesures.** Des outils d'analyse de code en mesurent une partie :

| Mesure | Ce qu'elle repère |
|---|---|
| complexité cyclomatique ou cognitive | des fonctions difficiles à comprendre et à tester |
| duplication | du code copié-collé |
| couverture des tests | les parties sans filet |
| dépendances obsolètes | les mises à jour en retard |
| effort de remédiation | une estimation, en temps, du travail pour corriger les problèmes détectés ; SonarQube la présente comme la dette du projet |

Ces mesures ne voient pas la dette d'architecture ni celle de documentation. Elles se complètent d'un **registre de la dette** : une liste, dans le gestionnaire de tickets, de chaque dette connue, avec son coût estimé et ses intérêts. Le stack mesure le projet fil rouge avec ces outils dans un lab dédié.

## 5 - Le quadrant de Martin Fowler

Toutes les dettes ne se valent pas. En 2009, Martin Fowler propose de les classer selon deux questions : la dette a-t-elle été prise **délibérément** ou **par inadvertance** ? Et la décision était-elle **prudente** ou **imprudente** ?

| | **Imprudente** | **Prudente** |
|---|---|---|
| **Délibérée** | « Pas le temps de concevoir, on code. » L'équipe sait qu'elle fait mal, et ne prévoit pas de rembourser. | « Il faut livrer maintenant, on assume et on reprendra ensuite. » Une dette choisie, avec un plan. |
| **Par inadvertance** | « Une architecture en couches, c'est quoi ? » L'équipe ne sait pas qu'elle crée de la dette. | « Maintenant, on sait comment on aurait dû faire. » Ce qu'on apprend en construisant révèle une meilleure conception. |

Ce que chaque case appelle :

| Case | La réponse |
|---|---|
| délibérée et prudente | la seule dette vraiment **choisie** : l'écrire dans le registre, avec la date où on la rembourse |
| délibérée et imprudente | un problème de pilotage : la rendre visible et chiffrée à ceux qui fixent les délais |
| par inadvertance et imprudente | un problème de compétences : formation, revue de code, binômes |
| par inadvertance et prudente | inévitable, même dans une très bonne équipe : prévoir du temps pour remanier une fois qu'on a compris |

Fowler insiste sur un point : un code en désordre, écrit sans savoir-faire, n'est pas une dette au sens de Cunningham. La métaphore suppose une décision et un remboursement prévu ; le désordre n'a ni l'un ni l'autre.

## 6 - Rembourser

- **La règle du boy-scout** : laisser chaque fichier un peu plus propre qu'on l'a trouvé, à chaque modification.
- **Un budget fixe** : réserver une part de chaque itération au remboursement, au lieu d'attendre un « grand nettoyage » qui n'arrive jamais.
- **Rembourser avant de construire** : remanier le module que la prochaine fonctionnalité va toucher, juste avant de la coder.
- **Ne pas rembourser partout** : un code qui va disparaître, ou qu'on ne modifie jamais, peut garder sa dette.

## Récap

- La dette technique, selon Ward Cunningham (1992), a un **capital**, le travail pour corriger, et des **intérêts**, le surcoût payé à chaque modification.
- Elle prend plusieurs formes : code, architecture, tests, documentation, dépendances, infrastructure.
- Ceux qui la voient, les développeurs, ne sont pas ceux qui décident du planning : il faut la rendre **visible et chiffrée**.
- Les outils mesurent une partie de la dette ; un **registre** suit le reste.
- Le **quadrant de Fowler** croise délibérée ou par inadvertance, prudente ou imprudente. Seule la dette délibérée et prudente est un vrai choix.

> 📖 [Ward Cunningham, « The WyCash Portfolio Management System », OOPSLA 1992 (en anglais)](https://c2.com/doc/oopsla92.html) · [Martin Fowler, « Technical Debt Quadrant », 2009 (en anglais)](https://martinfowler.com/bliki/TechnicalDebtQuadrant.html) · [Martin Fowler, « Technical Debt » (en anglais)](https://martinfowler.com/bliki/TechnicalDebt.html)

```quiz
title: La dette technique et le quadrant de Fowler
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Dans la métaphore de la dette technique, que sont les « intérêts » ?
    answers:
      - "Le surcoût payé à chaque modification du code tant que la dette n'est pas remboursée"
      - "Le prix des licences des outils d'analyse"
      - "Le temps passé à écrire les tests"
      - "Le salaire des développeurs"
    correct: 1
    explanation: |
      Le capital est le travail pour remettre le code au propre ; les intérêts, le surcoût de
      chaque modification. D'où la priorité aux zones souvent modifiées.
  - prompt: Une équipe livre volontairement une version simplifiée pour tenir un salon, et note dans le registre la refonte à faire le mois suivant. Où se place cette dette dans le quadrant de Fowler ?
    answers:
      - "Délibérée et imprudente"
      - "Par inadvertance et imprudente"
      - "Délibérée et prudente"
      - "Par inadvertance et prudente"
    correct: 3
    explanation: |
      La dette est choisie, en connaissance de cause, avec un plan de remboursement : c'est
      la seule case qui correspond pleinement à la métaphore de Cunningham.
  - prompt: Après six mois de projet, l'équipe comprend enfin le métier et voit qu'une autre conception aurait été bien meilleure. De quelle dette s'agit-il ?
    answers:
      - "Délibérée et imprudente"
      - "Par inadvertance et prudente"
      - "Ce n'est pas de la dette"
      - "Délibérée et prudente"
    correct: 2
    explanation: |
      Personne n'a choisi cette dette, et personne n'a été négligent : c'est en construisant
      qu'on apprend. Elle est inévitable ; on prévoit du temps pour remanier.
  - prompt: Quelle dette les outils d'analyse de code mesurent-ils mal ?
    answers:
      - "La duplication"
      - "La complexité des fonctions"
      - "La dette d'architecture et de documentation"
      - "Les dépendances obsolètes"
    correct: 3
    explanation: |
      Les outils voient le code ligne à ligne. Une architecture inadaptée ou une procédure
      d'installation fausse se suivent dans un registre de la dette.
  - prompt: Un module truffé de duplications n'a pas été modifié depuis trois ans et sera supprimé l'an prochain. Faut-il rembourser cette dette en priorité ?
    answers:
      - "Oui, toute dette doit être remboursée au plus vite"
      - "Non : ses intérêts sont presque nuls, puisque personne ne le modifie ; on rembourse d'abord là où le code change souvent"
      - "Oui, car un outil d'analyse la signale"
      - "Non, car la duplication n'est pas une dette"
    correct: 2
    explanation: |
      Une dette coûte par ses intérêts, payés à chaque modification. Un code qu'on ne touche
      plus, et qui va disparaître, peut la garder.
```
