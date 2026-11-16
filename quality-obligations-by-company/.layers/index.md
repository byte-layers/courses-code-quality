---
title: "Six entreprises, six cadres de qualité"
description: "Six entreprises fictives (santé, finance, SaaS, objet connecté, prestataire d'une entité NIS2, commerce en ligne) : pour chacune, les normes, règlements et certifications qui s'appliquent, et ce qu'ils changent dans le pipeline de développement."
tags:
  - bit
  - quality
  - security
difficulty: intermediate
duration: 25
requires: "0.4.0"
---

Les textes de la qualité ne s'appliquent pas tous à tout le monde. Une même équipe, avec le même code, n'aura pas les mêmes obligations selon qu'elle travaille pour un hôpital, une banque ou une boutique en ligne. Ce bit présente six entreprises fictives. Pour chacune : ce qu'elle fait, les textes qui la concernent, et ce qu'ils changent concrètement dans le pipeline de ses développeurs.

**Pour qui** : un développeur qui sait distinguer une norme, un règlement, une directive et une certification, ce qu'explique [Normes, règlements et référentiels qualité en France](https://byte-layers.com/fr/bits/french-quality-frameworks).

**Comment le lire** : pour chaque entreprise, lisez la description, puis listez vous-même les textes qui vous paraissent s'appliquer **avant** de lire le tableau. Les écarts entre votre liste et le tableau sont ce que ce bit vous apprend.

> 📅 **Sources vérifiées le 16 novembre 2026.** Les entreprises sont inventées ; les textes sont réels. Ce bit donne des repères, pas un avis juridique : pour un vrai projet, l'analyse se fait avec un juriste ou le délégué à la protection des données.

# Six entreprises, six cadres de qualité

## 1 - Santé : MédiSuivi

**L'entreprise.** MédiSuivi édite une application web où les patients atteints de diabète saisissent leur glycémie ; leur médecin consulte les courbes. Une nouvelle fonction doit **proposer une dose d'insuline**.

| Texte | Type | Pourquoi il s'applique |
|---|---|---|
| RGPD, article 9 | règlement | les données de santé sont une catégorie particulière, au traitement strictement encadré |
| Certification HDS | certification, imposée par le Code de la santé publique (article L.1111-8) | tout hébergeur de données de santé à caractère personnel doit être certifié ; elle porte sur l'hébergeur, pas sur l'application |
| Règlement sur les dispositifs médicaux (MDR, 2017/745) | règlement | un logiciel qui propose une dose devient un **dispositif médical** : marquage CE, évaluation clinique |
| IEC 62304 | norme | le cycle de vie du logiciel d'un dispositif médical ; c'est la norme habituellement utilisée pour montrer la conformité au MDR |

**Dans le pipeline.**
- Production et sauvegardes chez un hébergeur certifié HDS uniquement.
- Aucune donnée réelle de patient en développement ni en test : des jeux de données fictifs ou anonymisés.
- Pour la fonction de dose : chaque exigence reliée à ses tests et à son code, traçable d'une version à l'autre ; une revue formelle avant chaque livraison.

## 2 - Finance : PayFlux

**L'entreprise.** PayFlux, établissement de paiement agréé, propose aux commerçants un terminal de paiement sur smartphone et un tableau de bord de leurs ventes.

| Texte | Type | Pourquoi il s'applique |
|---|---|---|
| DORA (règlement 2022/2554) | règlement | résilience opérationnelle numérique du secteur financier, applicable depuis le 17 janvier 2025 : gestion des risques informatiques, des incidents, tests de résilience, suivi des prestataires |
| DSP2 (directive 2015/2366) | directive, transposée | services de paiement ; impose l'**authentification forte** du client |
| PCI DSS | standard de l'industrie des cartes, exigé par contrat | protection des données de carte bancaire, avec des audits ou des auto-évaluations selon le volume |
| RGPD | règlement | données personnelles des commerçants et de leurs clients |

**Dans le pipeline.**
- Ne jamais stocker un numéro de carte en clair ; isoler la partie qui manipule les cartes, pour réduire le périmètre audité.
- Analyse de code et des dépendances bloquante sur chaque demande de fusion.
- Journalisation des incidents, et procédure de déclaration prête.
- Tests de reprise après sinistre, réalisés et documentés.

## 3 - SaaS : RHPilot

**L'entreprise.** RHPilot vend aux entreprises un logiciel en ligne de gestion des congés et des entretiens annuels. Un module récent **classe automatiquement les candidatures** reçues.

| Texte | Type | Pourquoi il s'applique |
|---|---|---|
| RGPD, article 28 | règlement | RHPilot traite les données des salariés **pour le compte** de ses clients : il est sous-traitant, avec un contrat qui fixe ses obligations |
| Règlement sur l'IA (AI Act, 2024/1689) | règlement | un système d'IA utilisé pour le recrutement est classé **à haut risque** : gestion des risques, qualité des données, documentation, contrôle humain |
| ISO/IEC 27001 | norme, certification | rien ne l'impose par la loi, mais les grands clients l'exigent souvent dans leurs appels d'offres |

**Dans le pipeline.**
- Séparation stricte des données de chaque client : un test automatique vérifie qu'un client ne voit jamais les données d'un autre.
- Chiffrement, journaux d'accès, suppression des données à la fin du contrat.
- Pour le module de tri : un jeu de test qui mesure les biais, une documentation du modèle, et une décision finale qui reste humaine.

## 4 - Objet connecté : Thermia

**L'entreprise.** Thermia fabrique un thermostat Wi-Fi vendu en grande surface, piloté par une application mobile et un service en ligne.

| Texte | Type | Pourquoi il s'applique |
|---|---|---|
| Directive sur les équipements radio et son règlement délégué 2022/30 | directive et règlement | depuis le 1er août 2025, un équipement radio connecté à internet doit respecter des exigences de cybersécurité ; la norme harmonisée EN 18031 les détaille |
| Cyber Resilience Act (règlement 2024/2847) | règlement | tout produit comportant des éléments numériques : signalement des vulnérabilités activement exploitées depuis le 11 septembre 2026, exigences de sécurité complètes à partir du 11 décembre 2027 |
| ETSI EN 303 645 | norme | une base de sécurité des objets connectés grand public, largement reprise |
| RGPD | règlement | les horaires de chauffage révèlent quand le logement est vide |

**Dans le pipeline.**
- Pas de mot de passe par défaut commun à tous les appareils.
- Une liste des composants logiciels, la **SBOM**, produite à chaque version, pour savoir quels appareils sont touchés quand une faille sort dans une bibliothèque.
- Des mises à jour de sécurité signées, livrées pendant toute la durée de support annoncée.
- Une adresse publique pour signaler une vulnérabilité, et une procédure pour la traiter.

## 5 - Prestataire d'une entité NIS2 : InfraSoin

**L'entreprise.** InfraSoin, entreprise de services numériques de 80 personnes, maintient le système d'information de plusieurs hôpitaux.

| Texte | Type | Pourquoi il s'applique |
|---|---|---|
| NIS2 (directive 2022/2555) | directive | les hôpitaux sont des entités essentielles ; ils doivent maîtriser la sécurité de leurs prestataires. Les fournisseurs de services numériques gérés peuvent eux-mêmes être visés. En France, la loi qui la transpose était encore en discussion au Parlement à la date de vérification |
| ISO/IEC 27001 | norme, certification | une façon reconnue de prouver aux hôpitaux que la sécurité est gérée |
| Certification HDS | certification | si InfraSoin héberge des données de santé |

**Dans le pipeline.**
- Authentification forte pour tout accès aux systèmes des clients.
- Comptes nominatifs, journalisés, retirés au départ de chaque salarié.
- Analyse des dépendances, et inventaire des logiciels livrés.
- Une procédure d'incident calée sur le calendrier de NIS2 : alerte précoce sous 24 heures, notification sous 72 heures, rapport final sous un mois.

## 6 - Commerce en ligne : Brindille

**L'entreprise.** Brindille, PME de 30 salariés, vend des plantes en ligne en France et en Belgique, sur son site et son application mobile.

| Texte | Type | Pourquoi il s'applique |
|---|---|---|
| Directive sur l'accessibilité (European Accessibility Act), transposée | directive | depuis le 28 juin 2025, les sites et applications de commerce en ligne doivent être accessibles ; la vérification s'appuie sur la norme EN 301 549 et sur le RGAA |
| RGPD et règles sur les traceurs | règlement et directive « ePrivacy », transposée | consentement aux cookies non essentiels, recommandations de la CNIL |
| DSP2 | directive | l'authentification forte s'applique aux paiements, via le prestataire de paiement |
| PCI DSS | standard, exigé par contrat | réduit au minimum si le paiement se fait sur la page du prestataire de paiement |

**Dans le pipeline.**
- Des tests d'accessibilité automatiques sur les pages clés, et un audit RGAA avant chaque refonte.
- Aucun traceur déposé avant le consentement, vérifié par un test de bout en bout.
- Le paiement confié aux champs ou à la page du prestataire, pour ne jamais voir un numéro de carte.

## Ce qui revient d'une entreprise à l'autre

| Constat | Exemple |
|---|---|
| le RGPD concerne presque tout le monde | les six entreprises traitent des données personnelles |
| une nouvelle fonction peut changer de catégorie | la dose d'insuline fait de MédiSuivi un fabricant de dispositif médical ; le tri des candidatures place RHPilot dans l'IA à haut risque |
| une certification est souvent exigée par le client, pas par la loi | ISO 27001 chez RHPilot et InfraSoin |
| chaque texte se traduit en contrôles dans le pipeline | tests d'isolement, SBOM, analyse des dépendances, tests d'accessibilité, procédure d'incident |

C'est le lien entre les normes et le travail quotidien : savoir quels textes s'appliquent dit **quels contrôles** mettre dans la CI, et **lesquels** garder pour un humain.

## Récap

- Les obligations dépendent du **secteur**, des **données** traitées et des **fonctions** du produit, pas de la technologie.
- **Santé** : RGPD article 9, certification HDS de l'hébergeur, MDR si le logiciel a une finalité médicale.
- **Finance** : DORA, DSP2, PCI DSS.
- **SaaS** : RGPD article 28 pour le sous-traitant, AI Act pour les usages à haut risque, ISO 27001 exigée par les clients.
- **Objet connecté** : règlement délégué RED et EN 18031, Cyber Resilience Act.
- **Prestataire d'une entité NIS2** : la sécurité de la chaîne de prestataires, et une procédure d'incident.
- **Commerce en ligne** : accessibilité depuis le 28 juin 2025, consentement aux traceurs, paiement délégué.

> 📖 [RGPD, texte et explications de la CNIL](https://www.cnil.fr/fr/reglement-europeen-protection-donnees) · [Hébergement de données de santé, Agence du numérique en santé](https://esante.gouv.fr/produits-services/hds) · [DORA, règlement (UE) 2022/2554 sur EUR-Lex](https://eur-lex.europa.eu/eli/reg/2022/2554/oj?locale=fr) · [AI Act, règlement (UE) 2024/1689 sur EUR-Lex](https://eur-lex.europa.eu/eli/reg/2024/1689/oj?locale=fr) · [Cyber Resilience Act, règlement (UE) 2024/2847 sur EUR-Lex](https://eur-lex.europa.eu/eli/reg/2024/2847/oj?locale=fr) · [NIS2, directive (UE) 2022/2555 sur EUR-Lex](https://eur-lex.europa.eu/eli/dir/2022/2555/oj?locale=fr)

```quiz
title: Six entreprises, six cadres de qualité
tracksProgress: true
submitMode: last-only
questions:
  - prompt: MédiSuivi ajoute une fonction qui propose une dose d'insuline. Quel texte devient applicable du fait de cette seule fonction ?
    answers:
      - "La directive sur l'accessibilité"
      - "DORA"
      - "Le règlement sur les dispositifs médicaux (MDR)"
      - "La directive sur les équipements radio"
    correct: 3
    explanation: |
      Un logiciel qui aide à une décision médicale devient un dispositif médical : marquage CE,
      évaluation clinique, et un cycle de développement traçable, souvent selon IEC 62304.
  - prompt: Sur quoi porte la certification HDS ?
    answers:
      - "Sur le code de l'application de santé"
      - "Sur le médecin qui consulte les données"
      - "Sur l'accessibilité de l'application"
      - "Sur l'hébergeur des données de santé"
    correct: 4
    explanation: |
      Le Code de la santé publique impose que l'hébergeur de données de santé à caractère
      personnel soit certifié. L'éditeur choisit donc un hébergeur certifié.
  - prompt: Pourquoi le module de tri des candidatures de RHPilot change-t-il ses obligations ?
    answers:
      - "Parce que l'AI Act classe les systèmes d'IA utilisés pour le recrutement à haut risque"
      - "Parce que tout logiciel RH doit être certifié ISO 27001"
      - "Parce que le recrutement relève de DORA"
      - "Parce qu'il collecte des données de santé"
    correct: 1
    explanation: |
      Haut risque veut dire gestion des risques, qualité des données, documentation et contrôle
      humain. Dans le pipeline : tests de biais et décision finale humaine.
  - prompt: Quel contrôle de pipeline répond le mieux aux exigences du Cyber Resilience Act pour le thermostat de Thermia ?
    answers:
      - "Un test de contraste des couleurs"
      - "Un test de consentement aux cookies"
      - "Une revue clinique"
      - "Une SBOM produite à chaque version, pour savoir quels appareils une nouvelle faille touche"
    correct: 4
    explanation: |
      Le CRA impose de gérer les vulnérabilités pendant toute la durée de support : savoir ce
      que contient chaque version est la base.
  - prompt: Brindille confie le paiement à la page de son prestataire de paiement. Qu'y gagne-t-elle ?
    answers:
      - "Elle ne voit jamais les numéros de carte, ce qui réduit au minimum son périmètre PCI DSS"
      - "Elle n'est plus soumise au RGPD"
      - "Elle échappe à l'obligation d'accessibilité"
      - "Elle n'a plus besoin d'authentification forte"
    correct: 1
    explanation: |
      Les données de carte ne passent pas par ses serveurs. L'authentification forte reste
      exigée, mais c'est le prestataire qui la met en œuvre.
```
