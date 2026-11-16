---
title: "Les outils d'analyse de code et leurs alternatives"
description: "ESLint, jscpd, Semgrep, SonarQube, SonarQube Cloud, GitLab SAST, CodeQL : ce que chacun mesure, s'il lui faut un serveur, où part votre code et ce qu'il coûte, pour choisir selon le projet."
tags:
  - bit
  - quality
difficulty: beginner
duration: 15
requires: "0.4.0"
---

Les labs de ce stack ont utilisé trois outils légers, ESLint, jscpd et Semgrep, sans serveur et sans compte. Ce n'est qu'un choix parmi d'autres : en entreprise, vous croiserez souvent SonarQube, ou les analyses intégrées à GitHub et GitLab. Ce bit les compare selon quatre questions simples, pour que vous sachiez choisir, et défendre votre choix.

**Pour qui** : un développeur qui a déjà lancé un analyseur de code, ou fait le lab [Mesurer la dette avec Semgrep, ESLint et jscpd](https://byte-layers.com/fr/labs/measure-debt).

> 📅 **Sources vérifiées le 16 novembre 2026.** Les offres gratuites et les prix changent souvent : vérifiez la page de tarifs de l'éditeur avant de décider.

# Les outils d'analyse de code et leurs alternatives

## 1 - Quatre questions à poser à un outil

Tous ces outils font de l'**analyse statique** : ils lisent le code sans l'exécuter. Ils diffèrent sur quatre points :

| Question | Pourquoi elle compte |
|---|---|
| **Que mesure-t-il ?** | la sécurité (failles, secrets), la maintenabilité (complexité, duplication, mauvaises pratiques), la couverture des tests ; peu d'outils font tout |
| **Lui faut-il un serveur ?** | un outil en ligne de commande tourne dans n'importe quel pipeline ; une plateforme garde un historique et un tableau de bord, mais il faut l'héberger ou s'y abonner |
| **Où part le code ?** | sur le poste, rien ne sort ; un service en ligne reçoit le code ou ses résultats, ce qui compte pour un code confidentiel ou soumis à des règles de souveraineté |
| **Que coûte-t-il ?** | gratuit, gratuit sous conditions (projet public, taille, nombre de contributeurs), ou payant par contributeur |

## 2 - Les outils en ligne de commande

Ce sont ceux des labs. Ils tournent sur le poste ou dans un job de pipeline, et rien ne quitte la machine.

| Outil | Ce qu'il mesure | Coût |
|---|---|---|
| **ESLint** et ses plugins (dont `eslint-plugin-sonarjs`, qui reprend des règles de SonarQube) | erreurs fréquentes, style, complexité cognitive ; JavaScript et TypeScript seulement | gratuit, open source |
| **jscpd** | duplication, dans plus de 200 langages et formats | gratuit, open source |
| **Semgrep**, le moteur | sécurité et règles maison, dans une trentaine de langages ; on écrit ses règles en quelques lignes de YAML | moteur gratuit, sous licence LGPL 2.1 |

Leur limite : chacun mesure une chose, et aucun ne garde d'historique. Pour savoir si la dette baisse d'un mois à l'autre, il faut conserver les rapports soi-même.

Semgrep propose aussi une **plateforme** en ligne, *Semgrep AppSec Platform*, avec des règles plus poussées et un tableau de bord. Elle est gratuite jusqu'à 10 contributeurs, puis payante par contributeur.

## 3 - SonarQube : une plateforme de qualité

**SonarQube**, de l'éditeur suisse SonarSource, est l'outil de référence en entreprise. Il réunit dans un seul tableau de bord les bugs, les failles, les « mauvaises odeurs » du code (*code smells*), la duplication, la complexité et la couverture des tests, avec un historique. La couverture n'est pas calculée par SonarQube : il lit le rapport produit par vos tests.

Il existe en deux formes :

| Forme | Ce que c'est | Coût |
|---|---|---|
| **SonarQube Server** | un serveur que vous hébergez, avec une base de données ; le code reste chez vous | l'édition gratuite, *Community Build*, n'analyse que la branche principale : l'analyse des branches et des merge requests demande une édition payante, à partir de la *Developer Edition* |
| **SonarQube Cloud** (anciennement SonarCloud) | le même outil en service en ligne, sans rien à héberger ; le code est analysé chez SonarSource | gratuit pour les projets publics ; pour les projets privés, une offre gratuite jusqu'à 50 000 lignes de code, limitée à la branche principale, puis payante |

Une analyse SonarQube se pilote par un **quality gate**, un ensemble de conditions (pas de nouvelle faille, couverture du nouveau code au-dessus d'un seuil…) qui rend un verdict, comme la barrière du lab [Une barrière qualité sur les merge requests](https://byte-layers.com/fr/labs/merge-request-quality-gate).

## 4 - Les analyses intégrées aux forges

GitHub et GitLab proposent leur propre analyse de sécurité, intégrée aux pull requests et aux merge requests.

| Outil | Forge | Ce qu'il mesure | Coût |
|---|---|---|---|
| **GitLab SAST** | GitLab | failles de sécurité ; s'active en incluant un modèle de job fourni par GitLab, et repose en grande partie sur un analyseur construit sur Semgrep | l'analyse tourne sur toutes les offres ; l'affichage des failles dans la merge request et les analyseurs avancés demandent l'offre Ultimate |
| **CodeQL** | GitHub | failles de sécurité ; le code est transformé en base de données, interrogée par des requêtes | gratuit pour les dépôts publics ; payant pour les dépôts privés, avec l'offre GitHub Code Security |

Leur force : rien à installer, et les résultats s'affichent là où l'équipe relit le code. Leur limite : ils vous lient à une forge, et mesurent la sécurité plus que la maintenabilité.

## 5 - Le comparatif

| Outil | Sécurité | Maintenabilité | Couverture | Serveur | Le code sort-il ? | Gratuit |
|---|---|---|---|---|---|---|
| ESLint + sonarjs | un peu | oui | non | non | non | oui |
| jscpd | non | duplication | non | non | non | oui |
| Semgrep (moteur) | oui | règles maison | non | non | non | oui |
| SonarQube Server | oui | oui | importée | à héberger | non | branche principale seulement |
| SonarQube Cloud | oui | oui | importée | en ligne | oui | projets publics, ou privés jusqu'à 50 000 lignes |
| GitLab SAST | oui | non | non | intégré à GitLab | vers votre GitLab | analyse oui, affichage en MR non |
| CodeQL | oui | peu | non | intégré à GitHub | vers GitHub | dépôts publics |

## 6 - Choisir selon le projet

Il n'y a pas de meilleur outil, seulement un outil adapté au contexte :

| Contexte | Un choix raisonnable |
|---|---|
| un projet open source, public | SonarQube Cloud ou CodeQL, gratuits pour les projets publics, avec historique et affichage dans les pull requests |
| une petite équipe, code privé, sans budget | les outils en ligne de commande dans le pipeline, comme dans les labs ; SonarQube Community Build si l'on veut un tableau de bord sur la branche principale |
| un code qui ne doit pas quitter l'entreprise (santé, défense, données sensibles) | des outils qui tournent chez vous : la ligne de commande, ou SonarQube Server hébergé en interne ; pas de service en ligne |
| une grande entreprise déjà sur une forge payante | l'analyse de la forge (GitLab Ultimate, GitHub Code Security) pour la sécurité, SonarQube pour la maintenabilité |

Ces outils se combinent. Le cas courant : un outil rapide dans chaque merge request pour arrêter les défauts tôt, et une plateforme pour suivre la tendance dans le temps.

## Récap

- Quatre questions pour comparer : **que mesure-t-il, lui faut-il un serveur, où part le code, que coûte-t-il**.
- ESLint, jscpd et Semgrep tournent **sans serveur** et sans faire sortir le code, mais sans historique.
- **SonarQube** réunit sécurité, maintenabilité et couverture dans un tableau de bord ; l'édition gratuite auto-hébergée se limite à la branche principale, **SonarQube Cloud** est gratuit pour les projets publics.
- **GitLab SAST** et **CodeQL** intègrent l'analyse de sécurité aux forges ; l'affichage complet est payant pour le code privé.
- Le choix dépend du **contexte** : public ou privé, budget, confidentialité du code.

> 📖 [SonarQube Server : l'analyse des branches (en anglais)](https://docs.sonarsource.com/sonarqube-server/analyzing-source-code/branch-analysis/introduction) · [SonarQube Cloud : les offres (en anglais)](https://docs.sonarsource.com/sonarqube-cloud/administering-sonarcloud/managing-subscription/subscription-plans) · [Semgrep : questions fréquentes, licence et offres (en anglais)](https://docs.semgrep.dev/faq/overview) · [GitLab SAST (en anglais)](https://docs.gitlab.com/user/application_security/sast/) · [GitHub : CodeQL (en anglais)](https://codeql.github.com/)

```quiz
title: Les outils d'analyse de code
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Un hôpital veut analyser le code de son logiciel de dossiers patients, qui ne doit pas quitter son réseau. Quel choix est cohérent ?
    answers:
      - "SonarQube Cloud, pour éviter d'héberger un serveur"
      - "CodeQL sur un dépôt GitHub public"
      - "Des outils qui tournent en interne : ESLint, Semgrep en ligne de commande, ou SonarQube Server hébergé chez lui"
      - "Aucun outil : l'analyse statique est interdite dans la santé"
    correct: 3
    explanation: |
      Un service en ligne reçoit le code. Les outils en ligne de commande et un SonarQube
      auto-hébergé gardent le code dans le réseau de l'hôpital.
  - prompt: Avec l'édition gratuite de SonarQube Server, la Community Build, que ne pouvez-vous pas faire ?
    answers:
      - "Analyser les branches et les merge requests : seule la branche principale est analysée"
      - "Mesurer la duplication"
      - "Héberger le serveur vous-même"
      - "Définir un quality gate"
    correct: 1
    explanation: |
      L'analyse des branches et des merge requests commence à la Developer Edition, payante.
      La Community Build analyse la branche principale, avec quality gate et historique.
  - prompt: Quel outil ne demande ni serveur ni compte, et laisse écrire ses propres règles en YAML ?
    answers:
      - "SonarQube Cloud"
      - "CodeQL"
      - "GitLab SAST"
      - "Semgrep, le moteur en ligne de commande"
    correct: 4
    explanation: |
      Le moteur de Semgrep est gratuit et tourne sur le poste ou dans un job. Ses règles s'écrivent
      en YAML, avec des motifs qui ressemblent au code cherché.
  - prompt: Que fait SonarQube de la couverture des tests ?
    answers:
      - "Il la calcule en exécutant lui-même les tests"
      - "Il lit le rapport de couverture produit par vos tests, et l'affiche avec le reste"
      - "Il l'ignore : ce n'est pas de la qualité"
      - "Il l'estime à partir du nombre de fichiers de test"
    correct: 2
    explanation: |
      SonarQube n'exécute pas le code. Le pipeline lance les tests avec un outil de couverture,
      puis SonarQube importe le rapport.
  - prompt: Un projet open source public sur GitHub veut une analyse de sécurité affichée dans ses pull requests, sans budget. Quelle option convient ?
    answers:
      - "GitLab Ultimate"
      - "SonarQube Server Developer Edition"
      - "CodeQL, gratuit pour les dépôts publics"
      - "Aucune : tout est payant"
    correct: 3
    explanation: |
      CodeQL est gratuit pour les dépôts publics de GitHub. SonarQube Cloud, gratuit pour les
      projets publics, serait aussi un bon choix, en ajoutant la maintenabilité.
```
