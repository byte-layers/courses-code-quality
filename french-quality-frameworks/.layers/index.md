---
title: "Normes, règlements et référentiels qualité en France"
description: "Distinguer norme, règlement, directive, référentiel et certification, puis connaître les référentiels qu'un projet numérique rencontre en France (RGAA et WCAG, RGESN, ANSSI, OWASP, DSFR) et ce qu'ils changent dans un pipeline."
tags:
  - bit
  - quality
  - security
difficulty: beginner
duration: 20
requires: "0.4.0"
---

Un client vous demande si son application est « conforme ». Conforme à quoi ? À une norme qu'il a choisie, à un règlement européen qui s'impose à lui, à un référentiel de l'État ? Ces textes n'ont ni la même force ni les mêmes conséquences. Ce bit les classe d'abord, puis présente ceux qu'un projet numérique croise le plus souvent en France, et ce que chacun change dans votre façon de travailler.

**Pour qui** : un développeur qui connaît les caractéristiques de qualité d'un logiciel, ce que présente [Le modèle de qualité ISO/IEC 25010](https://byte-layers.com/fr/bits/iso-25010-quality-model).

> 📅 **Sources vérifiées le 16 novembre 2026.** Plusieurs textes sont en cours d'évolution ; leur statut est daté dans le bit.

# Normes, règlements et référentiels qualité en France

## 1 - Six mots qui ne veulent pas dire la même chose

| Type de texte | Qui l'écrit | Obligatoire ? | Exemples |
|---|---|---|---|
| **Norme** | un organisme de normalisation : ISO, CEN en Europe, AFNOR en France | non, d'application volontaire ; elle le devient si un contrat ou un texte de loi y renvoie | ISO/IEC 25010, ISO/IEC 27001, EN 301 549 |
| **Règlement européen** | l'Union européenne | oui, directement, dans tous les pays de l'Union, sans loi nationale | RGPD, règlement sur l'IA (AI Act), Cyber Resilience Act |
| **Directive européenne** | l'Union européenne | oui, une fois **transposée** : chaque pays l'écrit dans sa propre loi, avec un délai | NIS2, directive sur l'accessibilité (European Accessibility Act) |
| **Loi et décret français** | le Parlement, le gouvernement | oui, en France | la loi de 2005 sur le handicap, la loi REEN de 2021 sur l'empreinte du numérique |
| **Référentiel** | une administration, une agence, une fondation | selon le cas : obligatoire quand une loi l'impose, sinon recueil de bonnes pratiques | RGAA, RGESN, DSFR, guides de l'ANSSI, OWASP |
| **Certification, qualification, attestation** | un tiers indépendant, après audit | volontaire, mais souvent exigée par un client ou par la loi pour un marché | certification ISO 27001, certification HDS pour l'hébergement de données de santé, qualification SecNumCloud de l'ANSSI |

Deux réflexes en découlent. Pour une **norme**, la question est « qui l'exige, et dans quel texte ? ». Pour une **directive**, c'est « a-t-elle été transposée, et par quelle loi ? » : tant qu'elle ne l'est pas, les entreprises restent en principe soumises aux anciennes règles nationales, même si le délai de transposition est dépassé.

## 2 - L'accessibilité : WCAG, EN 301 549, RGAA

**Les textes.**

| Texte | Type | Ce qu'il contient |
|---|---|---|
| WCAG 2.2 | recommandation internationale du W3C | les critères de succès de l'accessibilité du web, en trois niveaux : A, AA, AAA |
| EN 301 549 | norme européenne | les exigences d'accessibilité des produits et services numériques ; elle reprend pour le web les critères A et AA des WCAG 2.1 |
| RGAA 4.1.2 | référentiel de l'État français | une méthode de vérification : 106 critères et leurs tests, alignés sur les WCAG 2.1 niveau AA |

Le RGAA en vigueur est la version **4.1.2**. Une version 5, alignée sur les WCAG 2.2 et étendue aux applications mobiles, était en préparation à la date de vérification, avec une publication annoncée pour la fin de 2026.

**Qui est obligé.** En France, la loi de 2005 sur le handicap impose l'accessibilité numérique aux organismes publics, aux organismes chargés d'une mission de service public et aux entreprises dont le chiffre d'affaires dépasse 250 millions d'euros ; elle impose aussi de publier une **déclaration d'accessibilité**. Depuis le **28 juin 2025**, la directive européenne sur l'accessibilité, transposée par une loi de 2023, étend l'obligation à de nombreux produits et services du secteur privé, dont les sites et applications de commerce en ligne. Les microentreprises qui fournissent des services en sont exemptées. Pour ces services, la vérification s'appuie sur la norme EN 301 549 et sur le RGAA.

**Dans le pipeline.** Des outils comme axe-core ou Pa11y testent automatiquement une page dans la CI. Ils ne trouvent qu'**une partie** des défauts : un texte alternatif présent mais absurde, un ordre de lecture illogique, un parcours impossible au clavier demandent un contrôle humain. L'audit RGAA reste manuel ; l'automatisation sert de filet entre deux audits.

## 3 - L'écoconception : le RGESN

Le **RGESN**, référentiel général d'écoconception de services numériques, est publié par l'Arcep et l'Arcom avec l'ADEME. Il découle de la **loi REEN** de 2021, sur la réduction de l'empreinte environnementale du numérique. Sa version 2024 compte **78 critères**, posés sous forme de questions et répartis sur tout le cycle de vie d'un service, de la stratégie à l'hébergement, en passant par l'interface, le code et les algorithmes.

Le référentiel invite à publier une **déclaration d'écoconception**, qui dit quels critères le service respecte. Les obligations de la loi REEN visent surtout les acteurs publics et évoluent : pour un projet donné, vérifiez qui l'impose avant de le présenter comme obligatoire.

**Dans le pipeline.** Une partie se mesure : poids des pages, nombre de requêtes, taille des images, avec des outils comme EcoIndex ou Lighthouse. La plupart des critères, comme « ce service répond-il à un besoin réel ? », relèvent de la conception, pas d'un test.

## 4 - La sécurité : OWASP, ANSSI, et trois textes européens

**Les référentiels techniques.**

| Référentiel | Type | À quoi il sert |
|---|---|---|
| OWASP Top 10:2025 | classement d'une fondation internationale | un document de sensibilisation qui classe les dix catégories de risques les plus critiques pour les applications web ; l'édition 2025 ajoute les défaillances de la chaîne d'approvisionnement logicielle (A03) et la mauvaise gestion des situations exceptionnelles (A10) |
| OWASP ASVS | référentiel de vérification | des exigences de sécurité testables, par niveau, pour écrire et vérifier une application |
| Guides de l'ANSSI | recommandations de l'agence française de cybersécurité | le guide d'hygiène informatique et ses 42 mesures, des guides techniques par sujet |
| SecNumCloud | qualification délivrée par l'ANSSI | un niveau de sécurité pour les offres cloud ; l'État et ses opérateurs doivent l'exiger de leur hébergeur pour leurs données sensibles (loi SREN, 2024) |

**Les textes européens.**

| Texte | Type | Statut à la date de vérification |
|---|---|---|
| RGPD | règlement | en application depuis 2018 : protection des données personnelles, dès la conception |
| Cyber Resilience Act | règlement | en vigueur ; l'obligation de signaler les vulnérabilités activement exploitées s'applique depuis le 11 septembre 2026, l'essentiel des exigences de sécurité des produits à partir du 11 décembre 2027 |
| NIS2 | directive | sécurité des entités essentielles et importantes ; en France, le projet de loi qui la transpose, dit loi Résilience, était encore en discussion au Parlement |

**Dans le pipeline.** C'est le domaine le plus automatisable : analyse statique du code (SAST), analyse des dépendances (`npm audit`), détection des secrets commités. Les autres contenus de ce stack les mettent en place.

## 5 - Le design de l'État : le DSFR

Le **DSFR**, système de design de l'État, fournit les composants d'interface, les couleurs et la typographie des services publics, avec leur accessibilité déjà travaillée. Une circulaire du 7 juillet 2023 le rend **obligatoire pour les sites et applications de l'État**. Son code est publié sous licence ouverte Etalab 2.0, mais ses conditions d'utilisation l'interdisent hors des sites de l'État, sur les domaines `.gouv.fr` : une entreprise privée ne peut pas l'utiliser pour son propre site, puisqu'il porte la marque de l'État.

## 6 - Du texte au pipeline

Pour chaque référentiel qui s'applique à un projet, la même question : que peut-on vérifier automatiquement, et que faut-il garder pour un humain ?

| Référentiel | Caractéristique ISO 25010 | Vérifiable en CI | Reste humain |
|---|---|---|---|
| RGAA, WCAG | capacité d'interaction | une partie des critères : contrastes, attributs, structure | l'audit complet, les parcours, la pertinence des textes |
| RGESN | efficacité des performances, et au-delà | poids, requêtes, images | les choix de conception et de besoin |
| OWASP, ANSSI | sécurité | failles connues dans le code et les dépendances, secrets | la conception, les droits d'accès, l'analyse de risques |
| RGPD | sécurité, et au-delà | pas grand-chose : des tests sur les données exportées ou supprimées | la base légale, la durée de conservation, le registre des traitements |

## Récap

- Une **norme** est volontaire tant qu'aucun texte ne l'impose ; un **règlement** européen s'applique directement ; une **directive** doit être transposée ; un **référentiel** est obligatoire quand une loi le dit ; une **certification** est délivrée par un tiers après audit.
- **Accessibilité** : WCAG au niveau international, EN 301 549 en Europe, RGAA 4.1.2 en France ; obligation étendue au privé, commerce en ligne compris, depuis le 28 juin 2025.
- **Écoconception** : le RGESN, 78 critères, issu de la loi REEN.
- **Sécurité** : OWASP Top 10:2025 et ASVS, guides de l'ANSSI ; RGPD, Cyber Resilience Act et NIS2 côté européen.
- **DSFR** : obligatoire et réservé aux sites de l'État.
- Pour chaque référentiel, séparez ce que la CI vérifie de ce qu'un humain doit auditer.

> 📖 [RGAA, accessibilite.numerique.gouv.fr](https://accessibilite.numerique.gouv.fr/) · [WCAG 2.2 (en anglais)](https://www.w3.org/TR/WCAG22/) · [RGESN, Arcep](https://www.arcep.fr/demarches-et-services/professionnels/referentiel-general-ecoconception-services-numeriques.html) · [OWASP Top 10:2025 (en anglais)](https://owasp.org/Top10/2025/) · [Guide d'hygiène informatique, ANSSI](https://messervices.cyber.gouv.fr/guides/guide-dhygiene-informatique) · [Système de design de l'État](https://www.systeme-de-design.gouv.fr/)

```quiz
title: Normes, règlements et référentiels qualité en France
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Quelle est la différence entre un règlement européen et une directive européenne ?
    answers:
      - "Le règlement est facultatif, la directive obligatoire"
      - "Le règlement s'applique directement dans chaque pays ; la directive doit d'abord être transposée dans la loi de chaque pays"
      - "Aucune, ce sont deux noms du même texte"
      - "La directive ne concerne que les entreprises publiques"
    correct: 2
    explanation: |
      Le RGPD, un règlement, s'applique tel quel. NIS2, une directive, n'a d'effet en France
      qu'une fois transposée par une loi française.
  - prompt: Une norme ISO est-elle obligatoire ?
    answers:
      - "Oui, toujours, dès sa publication"
      - "Non, jamais"
      - "Non, elle est d'application volontaire, sauf si un contrat ou un texte de loi y renvoie"
      - "Oui, mais seulement pour les entreprises certifiées"
    correct: 3
    explanation: |
      Une norme devient contraignante par le texte qui la cite : un contrat, un appel d'offres,
      une loi ou un règlement.
  - prompt: Depuis le 28 juin 2025, une boutique en ligne d'une PME française doit-elle se soucier d'accessibilité numérique ?
    answers:
      - "Non, l'accessibilité ne concerne que les sites publics"
      - "Oui : la directive européenne sur l'accessibilité, transposée en France, couvre le commerce en ligne ; seules les microentreprises de services sont exemptées"
      - "Seulement si elle utilise le DSFR"
      - "Seulement si elle est certifiée ISO 27001"
    correct: 2
    explanation: |
      La directive (European Accessibility Act) étend l'obligation à de nombreux services privés,
      dont le commerce en ligne. En France, la vérification s'appuie sur la norme EN 301 549
      et sur le RGAA.
  - prompt: Un outil comme axe-core passe sans erreur dans la CI. Le site est-il conforme au RGAA ?
    answers:
      - "Oui, l'outil vérifie tous les critères"
      - "Non : les outils automatiques ne trouvent qu'une partie des défauts, et l'audit RGAA reste manuel"
      - "Oui, si le site utilise aussi le DSFR"
      - "Non, car axe-core ne sait tester que les applications mobiles"
    correct: 2
    explanation: |
      Pertinence d'un texte alternatif, ordre de lecture, parcours au clavier : ces points
      demandent un humain. L'outil sert de filet entre deux audits.
  - prompt: Une start-up veut utiliser le DSFR pour donner à son site un air officiel. Est-ce possible ?
    answers:
      - "Oui, son code est sous licence ouverte, donc utilisable par tous"
      - "Oui, à condition de citer l'État"
      - "Non : son usage est réservé aux sites et applications de l'État, puisqu'il porte la marque de l'État"
      - "Oui, s'il est combiné avec le RGESN"
    correct: 3
    explanation: |
      Le code est sous licence ouverte, mais les conditions d'utilisation réservent le DSFR aux
      sites de l'État.
```
