---
title: "SAST, SCA, secrets, épinglage : sécuriser la chaîne logicielle"
description: "Les quatre portes par lesquelles une faille entre dans un logiciel (votre code, vos dépendances, vos secrets, vos outils de build) et le contrôle automatique qui garde chacune : SAST, SCA, détection de secrets et épinglage des versions."
tags:
  - bit
  - quality
  - security
difficulty: beginner
duration: 15
requires: "0.4.0"
---

Un logiciel ne se résume pas au code que votre équipe écrit. Il embarque des centaines de dépendances, se construit avec des images Docker et des outils de pipeline, et manipule des secrets. Chacun de ces éléments est une porte d'entrée pour une faille, ou pour un attaquant. Ce bit présente les quatre portes, et le contrôle automatique qui garde chacune.

**Pour qui** : un développeur qui a déjà écrit un pipeline, ou fait le lab [Une barrière qualité sur les merge requests](https://byte-layers.com/fr/labs/merge-request-quality-gate).

> 📅 **Sources vérifiées le 16 novembre 2026.**

# SAST, SCA, secrets, épinglage : sécuriser la chaîne logicielle

## 1 - Les quatre portes

La **chaîne logicielle** (*software supply chain*) désigne tout ce qui entre dans le logiciel livré : le code, ses dépendances, les outils qui le construisent, et les accès qu'il utilise.

| Porte | Le risque | Le contrôle |
|---|---|---|
| **votre code** | une faille écrite par l'équipe, ou par un agent : injection, secret en dur, contrôle d'accès oublié | **SAST** : analyse statique de sécurité |
| **vos dépendances** | un paquet avec une faille connue, ou un paquet piégé | **SCA** : analyse de la composition |
| **vos secrets** | un jeton ou un mot de passe publié dans le dépôt | **détection de secrets** |
| **vos outils de build** | une image Docker ou une action de pipeline modifiée par un attaquant | **épinglage** des versions |

Le bit [Trivy, l'outil de sécurité compromis](https://byte-layers.com/fr/bits/trivy-compromise) montre la quatrième porte en action, et [Laravel Lang](https://byte-layers.com/fr/bits/laravel-lang-compromise) la deuxième.

## 2 - SAST : chercher les failles dans votre code

Le **SAST** (*Static Application Security Testing*) lit le code source sans l'exécuter, et cherche des motifs dangereux : une requête SQL construite en collant une entrée utilisateur, un `eval` sur une donnée reçue, un secret écrit en dur. C'est ce que fait Semgrep, avec ses règles génériques ou vos règles maison ; CodeQL et GitLab SAST font de même, intégrés aux forges.

Sa limite : il signale ce qui **ressemble** à une faille. Il produit des **faux positifs**, des alertes sur du code sûr, et des **faux négatifs**, des failles qu'aucune règle ne décrit. Le lab [Mesurer la dette](https://byte-layers.com/fr/labs/measure-debt) l'a montré : les règles génériques de Semgrep n'ont rien vu dans le code de l'agent, deux règles écrites pour le projet ont trouvé le jeton.

## 3 - SCA : connaître ses dépendances

Un projet Node.js de taille moyenne installe plusieurs centaines de paquets : quelques dizaines déclarés dans `package.json`, et toutes leurs propres dépendances, dites **transitives**. La **SCA** (*Software Composition Analysis*) compare cette liste aux bases de **vulnérabilités connues**, chacune identifiée par un numéro **CVE** (*Common Vulnerabilities and Exposures*) ou un avis de sécurité.

| Outil | Ce qu'il fait |
|---|---|
| `npm audit` | compare les versions de `package-lock.json` à la base d'avis de sécurité de GitHub ; `--audit-level=high` n'échoue qu'à partir des failles graves |
| Dependabot, Renovate | ouvrent automatiquement une merge request quand une dépendance a une nouvelle version, ou un correctif de sécurité |
| `npm sbom` | produit un **SBOM** (*Software Bill of Materials*), l'inventaire complet des composants, au format CycloneDX ou SPDX |

Le SBOM devient une obligation pour beaucoup de logiciels vendus dans l'Union européenne, avec le règlement européen sur la cyberrésilience : le bit [Les normes et référentiels de la qualité en France](https://byte-layers.com/fr/bits/french-quality-frameworks) en donne le calendrier.

La SCA ne voit que les failles **connues**. Un paquet piégé tout neuf n'est dans aucune base : en septembre 2025, le ver **Shai-Hulud** a infecté plus de 500 paquets npm. Il s'exécutait à l'installation, par un script `postinstall`, volait les jetons de publication des développeurs, et s'en servait pour publier des versions piégées de leurs propres paquets. Deux habitudes limitent ce risque : installer avec `npm ci`, qui respecte exactement `package-lock.json`, et désactiver les scripts d'installation quand on n'en a pas besoin (`npm ci --ignore-scripts`).

## 4 - Les secrets : ne jamais les publier

Un secret publié dans un dépôt est **compromis**, même s'il est supprimé au commit suivant : il reste dans l'historique de Git, et des robots parcourent en permanence les dépôts publics à la recherche de clés. La seule réponse sûre est de le **révoquer** et d'en créer un nouveau ; nettoyer l'historique vient ensuite.

| Contrôle | Où |
|---|---|
| un fichier `.env` ignoré par Git, et un `.env.example` sans valeurs réelles | dans le projet, dès sa création |
| **gitleaks**, outil libre qui cherche des secrets dans les fichiers et l'historique de Git | sur le poste, avant l'envoi, et dans le pipeline |
| la détection de secrets des forges | GitLab analyse les pipelines sur toutes ses offres ; GitHub bloque par défaut, sur les dépôts publics, l'envoi de nombreux types de clés connues |
| les **variables masquées** du pipeline | pour donner un secret à un job sans l'écrire dans le dépôt |

## 5 - L'épinglage : savoir exactement ce qui tourne

Un nom de version comme `node:24` ou `v1` est une **étiquette** : son propriétaire peut la déplacer vers un autre contenu, et un attaquant qui a volé ses accès aussi. C'est ce qui s'est passé avec Trivy : des étiquettes de version de ses actions GitHub ont été déplacées vers du code piégé. **Épingler**, c'est désigner un contenu par son **empreinte**, qui change si le moindre octet change :

| Élément | Étiquette, déplaçable | Épinglé |
|---|---|---|
| une dépendance npm | `"lodash": "^4.17.0"` dans `package.json` | la version exacte et son empreinte dans `package-lock.json`, installées par `npm ci` |
| une image Docker | `node:24-bookworm-slim` | `node:24-bookworm-slim@sha256:…`, l'empreinte de l'image |
| une action GitHub | `uses: actions/checkout@v4` | `uses: actions/checkout@<empreinte du commit>` |

L'épinglage a un coût : les correctifs n'arrivent plus tout seuls. On le compense par un outil comme Renovate, qui propose les mises à jour par merge request : on garde le contrôle, sans rester figé.

## Récap

- Une faille entre par **quatre portes** : votre code, vos dépendances, vos secrets, vos outils de build.
- Le **SAST** cherche des motifs dangereux dans votre code ; il produit des faux positifs et ne voit que ce que ses règles décrivent.
- La **SCA** compare vos dépendances aux failles **connues** ; un paquet piégé tout neuf lui échappe, d'où `npm ci` et `--ignore-scripts`.
- Un **secret** publié est compromis : on le révoque ; gitleaks et la détection des forges évitent de le publier.
- **Épingler** par empreinte une dépendance, une image ou une action garantit ce qui tourne ; Renovate garde les versions à jour.

> 📖 [CISA, « Widespread Supply Chain Compromise Impacting npm Ecosystem », 23 septembre 2025 (en anglais)](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem) · [npm audit (en anglais)](https://docs.npmjs.com/cli/commands/npm-audit) · [npm sbom (en anglais)](https://docs.npmjs.com/cli/commands/npm-sbom) · [gitleaks (en anglais)](https://github.com/gitleaks/gitleaks) · [OWASP, Software Supply Chain Security Cheat Sheet (en anglais)](https://cheatsheetseries.owasp.org/cheatsheets/Software_Supply_Chain_Security_Cheat_Sheet.html)

```quiz
title: SAST, SCA, secrets, épinglage
tracksProgress: true
submitMode: last-only
questions:
  - prompt: "`npm audit` ne signale rien sur votre projet. Que pouvez-vous en conclure ?"
    answers:
      - "Que vos dépendances ne contiennent aucun code malveillant"
      - "Qu'aucune de vos dépendances n'a de faille connue dans la base consultée ; un paquet piégé récent peut encore lui échapper"
      - "Que votre propre code est sûr"
      - "Que vos secrets sont protégés"
    correct: 2
    explanation: |
      La SCA compare vos versions aux failles connues. Un paquet piégé tout neuf, comme ceux
      du ver Shai-Hulud à ses débuts, n'est dans aucune base, et votre propre code relève du SAST.
  - prompt: Un développeur a poussé une clé d'API dans un commit, puis l'a supprimée au commit suivant. Que faut-il faire en premier ?
    answers:
      - "Rien : le fichier ne la contient plus"
      - "Rendre le dépôt privé"
      - "Ajouter le fichier au .gitignore"
      - "Révoquer la clé et en créer une nouvelle : elle reste dans l'historique, et doit être considérée comme compromise"
    correct: 4
    explanation: |
      Supprimer le fichier ne retire pas la clé de l'historique de Git. Des robots parcourent
      les dépôts publics en quelques minutes : seule la révocation la rend inutilisable.
  - prompt: Pourquoi écrire `node:24-bookworm-slim@sha256:…` plutôt que `node:24-bookworm-slim` dans un pipeline ?
    answers:
      - "Parce que l'empreinte désigne exactement un contenu, alors qu'une étiquette peut être déplacée vers une autre image"
      - "Pour télécharger l'image plus vite"
      - "Parce que GitLab refuse les étiquettes"
      - "Pour obtenir automatiquement les correctifs de sécurité"
    correct: 1
    explanation: |
      C'est l'inverse de la dernière réponse : une image épinglée ne change plus. On garde les
      correctifs en mettant l'empreinte à jour, par exemple avec Renovate.
  - prompt: Quel contrôle trouve une requête SQL construite en collant une entrée utilisateur ?
    answers:
      - "La SCA"
      - "La détection de secrets"
      - "Le SAST, l'analyse statique de sécurité du code"
      - "L'épinglage"
    correct: 3
    explanation: |
      L'injection est un défaut de votre code : c'est le domaine du SAST, avec Semgrep, CodeQL
      ou GitLab SAST.
  - prompt: Le ver Shai-Hulud s'exécutait à l'installation des paquets. Quelle habitude limite ce risque ?
    answers:
      - "Mettre à jour les dépendances le plus souvent possible, sans relire"
      - "Installer avec npm ci, qui respecte le fichier de verrouillage, et désactiver les scripts d'installation inutiles avec --ignore-scripts"
      - "Supprimer package-lock.json pour toujours avoir les dernières versions"
      - "Lancer npm audit une fois par an"
    correct: 2
    explanation: |
      Le fichier de verrouillage fige les versions vérifiées, et --ignore-scripts empêche un
      script postinstall de s'exécuter. Supprimer le verrouillage fait l'inverse.
```
