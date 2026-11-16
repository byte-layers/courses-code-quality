---
title: "Ariane 5 vol 501 : un logiciel réutilisé sans ses hypothèses"
description: "Le 4 juin 1996, la première Ariane 5 perd son guidage 37 secondes après l'allumage, puis se désintègre. La cause : un logiciel repris d'Ariane 4, juste pour Ariane 4, jamais testé avec la trajectoire d'Ariane 5. À quelle phase est né le problème ?"
tags:
  - bit
  - quality
difficulty: beginner
duration: 10
requires: "0.4.0"
---

Le 4 juin 1996, le premier vol d'Ariane 5 s'achève par l'autodestruction du lanceur, moins de quarante secondes après le décollage. Aucun matériel n'était en panne : un logiciel a fait exactement ce pour quoi il avait été écrit, mais pour une autre fusée. Ce bit raconte l'enchaînement, d'après le rapport de la commission d'enquête, puis pose la question qui sert à tous les incidents de ce layer : **à quelle phase est né le problème, et à quelle phase aurait-on pu l'attraper ?**

**Pour qui** : tout développeur. Aucune connaissance en spatial n'est nécessaire.

> 📅 **Sources vérifiées le 16 novembre 2026.**

# Ariane 5 vol 501 : un logiciel réutilisé sans ses hypothèses

## 1 - Ce qui s'est passé

Le lanceur connaît sa position et son orientation grâce à deux **centrales inertielles**, appelées SRI. Elles sont identiques, matériel et logiciel : l'une travaille, l'autre attend en secours.

| Instant | Événement |
|---|---|
| 36,7 s après l'allumage du moteur principal (30 s après le décollage) | la centrale de secours s'arrête sur une erreur logicielle ; environ 0,05 seconde plus tard, la centrale active s'arrête pour la même raison |
| juste après | le calculateur de bord lit les données de diagnostic envoyées par la centrale comme si c'étaient des données de vol ; il braque les tuyères au maximum |
| moins de 40 s après le décollage | le lanceur dévie brutalement, les liaisons se rompent, l'autodestruction se déclenche comme prévu |

## 2 - La chaîne des causes

Le rapport remonte la chaîne maillon par maillon :

1. **Un calcul inutile en vol.** L'erreur se produit dans une fonction d'**alignement** de la centrale, qui ne sert qu'avant le décollage. Sur Ariane 4, on la laissait tourner une quarantaine de secondes après le décollage, pour pouvoir relancer vite un compte à rebours interrompu. Ce besoin n'existait pas sur Ariane 5, mais le logiciel a été repris tel quel.
2. **Une valeur trop grande.** Cette fonction calcule une valeur liée à la vitesse horizontale, appelée BH. Le début de la trajectoire d'Ariane 5 est différent de celui d'Ariane 4 : la vitesse horizontale y est beaucoup plus élevée.
3. **Une conversion non protégée.** Le code convertit BH d'un nombre à virgule flottante sur 64 bits vers un entier signé sur 16 bits. La valeur dépasse ce qu'un entier de 16 bits peut contenir : c'est une erreur d'opérande. D'autres conversions, au même endroit, étaient protégées. Pour celle-ci, les partenaires du projet avaient jugé la valeur physiquement limitée, ou dotée d'une grande marge, sans le vérifier avec des données de trajectoire.
4. **Une réaction prévue pour le matériel.** La spécification disait : en cas d'exception, signaler l'erreur, l'enregistrer, et **arrêter le processeur**. Cette règle suppose une panne matérielle aléatoire, que la centrale de secours rattrape. Mais face à une erreur de logiciel, la centrale de secours, qui fait tourner le même logiciel, tombe de la même façon.
5. **Des tests qui ne pouvaient pas la voir.** Les données de trajectoire d'Ariane 5 ne figuraient pas dans les exigences de la centrale. Les simulations du vol complet utilisaient un modèle logiciel de la centrale, pas la vraie : on la considérait validée par ses propres essais et par les vols d'Ariane 4.

## 3 - À quelle phase est né le problème ?

Un incident a rarement une seule cause. On distingue la phase où le défaut **naît** et celles où il aurait pu être **attrapé**. Les phases utilisées dans tout ce layer : exigences, conception, codage, test, déploiement, exploitation, et dépendances (le code qu'on n'a pas écrit soi-même).

| Phase | Ce qui s'y est passé |
|---|---|
| **Exigences** | le défaut naît ici : les exigences de la centrale ne contiennent pas la trajectoire d'Ariane 5, et gardent une exigence d'Ariane 4 devenue inutile |
| Conception | la règle « toute exception arrête le processeur » est pensée pour le matériel ; deux centrales identiques ne protègent pas d'une erreur de logiciel |
| Conception et codage | une conversion laissée sans protection, par une décision prise en commun, sur une hypothèse jamais vérifiée ni écrite dans le code |
| Test | aucun test avec la vraie centrale et la trajectoire d'Ariane 5 : le défaut, présent depuis le début, n'est jamais déclenché |

La phase de naissance, c'est l'**exigence** : un composant réutilisé emporte avec lui les hypothèses du système pour lequel il a été écrit. La conception, le codage et le test sont les endroits où on aurait pu l'attraper.

## 4 - Ce que ça change pour vous

- **Réutiliser, c'est hériter d'hypothèses.** Une bibliothèque, un module copié d'un autre projet, un extrait proposé par un agent de code : chacun a été écrit pour un contexte. Le reprendre, c'est vérifier que ce contexte est encore le vôtre.
- **Écrire les hypothèses dans le code.** « Cette valeur ne dépasse jamais 32 767 » doit devenir un test, ou une vérification qui échoue proprement.
- **Tester le système avec les vraies données.** Un composant validé seul ne prouve rien sur son comportement dans un nouvel ensemble.
- **Une redondance identique ne protège pas d'un bug.** Deux copies du même logiciel tombent ensemble.

## Récap

- Ariane 501 a été détruite le 4 juin 1996, sans aucune panne matérielle.
- Une fonction d'alignement héritée d'Ariane 4, inutile en vol, a reçu une valeur trop grande pour une conversion vers un entier de 16 bits.
- La règle « exception = arrêt du processeur » a éteint les deux centrales, qui faisaient tourner le même logiciel.
- Le défaut est né dans les **exigences** : la trajectoire d'Ariane 5 n'y figurait pas. Le **test** aurait pu l'attraper, avec la vraie centrale et les vraies données.

> 📖 [Rapport de la commission d'enquête Ariane 501, ESA, juillet 1996 (PDF, en anglais)](https://esamultimedia.esa.int/docs/esa-x-1819eng.pdf) · [Communiqué de l'ESA sur le rapport (en anglais)](https://www.esa.int/Newsroom/Press_Releases/Ariane_501_-_Presentation_of_Inquiry_Board_report)

```quiz
title: Ariane 5 vol 501
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Qu'est-ce qui a déclenché l'erreur dans la centrale inertielle ?
    answers:
      - "Une panne électrique de la centrale de secours"
      - "La conversion d'une valeur flottante sur 64 bits, trop grande pour un entier signé sur 16 bits"
      - "Un virus introduit dans le logiciel de vol"
      - "Une erreur de saisie de la trajectoire par un opérateur"
    correct: 2
    explanation: |
      La valeur BH, liée à la vitesse horizontale, dépassait ce qu'un entier de 16 bits peut
      contenir. La conversion n'était pas protégée.
  - prompt: Pourquoi la centrale de secours n'a-t-elle servi à rien ?
    answers:
      - "Elle n'était pas branchée"
      - "Elle faisait tourner le même logiciel, et a rencontré la même erreur"
      - "Elle a été désactivée par l'équipe au sol"
      - "Elle n'existait pas sur Ariane 5"
    correct: 2
    explanation: |
      La redondance protège d'une panne matérielle aléatoire, pas d'une erreur de logiciel :
      deux copies du même code tombent ensemble.
  - prompt: À quelle phase est né le problème ?
    answers:
      - "Exigences : la trajectoire d'Ariane 5 n'y figurait pas, et une exigence d'Ariane 4 devenue inutile y restait"
      - "Déploiement : la mauvaise version du logiciel a été chargée"
      - "Exploitation : l'équipe au sol a réagi trop tard"
      - "Dépendances : une bibliothèque tierce était compromise"
    correct: 1
    explanation: |
      Le logiciel a été repris d'Ariane 4 avec ses hypothèses. Les exigences de la centrale
      ne disaient rien de la trajectoire d'Ariane 5.
  - prompt: Quel test aurait le plus probablement révélé le défaut avant le vol ?
    answers:
      - "Un test de charge du calculateur de bord"
      - "Une simulation du vol complet avec la vraie centrale et la trajectoire d'Ariane 5"
      - "Une relecture de l'orthographe des commentaires"
      - "Un test d'interface graphique"
    correct: 2
    explanation: |
      Les simulations du vol utilisaient un modèle de la centrale. Avec la vraie centrale et
      les vraies données, la valeur trop grande serait apparue.
  - prompt: Vous reprenez une fonction d'un ancien projet, ou un extrait proposé par un agent de code. Quelle leçon d'Ariane 5 s'applique ?
    answers:
      - "Le code a déjà servi ailleurs, il est donc validé"
      - "Vérifier les hypothèses du contexte d'origine, et les écrire sous forme de tests"
      - "Doubler le serveur pour avoir un secours"
      - "Désactiver les exceptions"
    correct: 2
    explanation: |
      Un code réutilisé emporte les hypothèses du système pour lequel il a été écrit. Les
      rendre explicites, par des tests, est la protection.
```
