---
title: "Laravel-Lang, mai 2026 : 700 anciennes versions réécrites en une nuit"
description: "Un jeton GitHub volé, des étiquettes de version recréées sur un code malveillant, et des projets PHP qui installent un voleur d'identifiants en croyant installer des traductions. À quelle phase est né le problème ?"
tags:
  - bit
  - quality
difficulty: beginner
duration: 10
requires: "0.4.0"
---

Les paquets `laravel-lang` fournissent les traductions d'une application Laravel dans plus de soixante-dix langues : messages de validation, statuts HTTP, noms de champs. Les 22 et 23 mai 2026, un attaquant réécrit des centaines de leurs anciennes versions. Un simple `composer update` suffit alors à installer, dans le projet, un programme qui vole les identifiants du serveur. Ce bit suit la déclaration des mainteneurs et l'analyse de Snyk, et pose la question : **à quelle phase est né le problème ?**

**Pour qui** : un développeur qui a déjà installé un paquet avec un gestionnaire de dépendances (Composer, npm). Les phases utilisées sont présentées dans [Ariane 5 vol 501](https://byte-layers.com/fr/bits/ariane-5-failure).

> 📅 **Sources vérifiées le 16 novembre 2026.**

# Laravel-Lang, mai 2026 : 700 anciennes versions réécrites en une nuit

## 1 - D'où vient une version Composer

Composer, le gestionnaire de dépendances de PHP, trouve les paquets sur **Packagist**. Packagist ne stocke pas le code : il lit les étiquettes Git du dépôt GitHub de chaque paquet. L'étiquette `v15.0.0` devient la version 15.0.0.

Conséquence : si quelqu'un supprime une étiquette et la recrée sur un autre commit, la version 15.0.0 désigne désormais un autre code. Le numéro de version ne garantit pas le contenu.

## 2 - Ce qui s'est passé

| Date | Événement |
|---|---|
| avant le 22 mai | un jeton d'accès personnel GitHub d'un membre de l'équipe Laravel-Lang tombe entre les mains de l'attaquant ; les mainteneurs pensent qu'il vient d'une fuite de données récente chez GitHub |
| 22 mai 2026 | l'attaquant supprime les étiquettes de quatre dépôts (`laravel-lang/lang`, `http-statuses`, `attributes`, `actions`) et les recrée, avec les mêmes noms, sur ses propres commits. Les premiers relevés comptent environ 233 versions empoisonnées |
| 22 et 23 mai | les réécritures continuent par vagues ; le total dépasse 700 versions. Les mainteneurs suppriment les étiquettes, qui réapparaissent « sous leurs yeux » |
| pour arrêter l'attaque | l'équipe retire les droits d'écriture de tous ses membres, ce qui stoppe les réécritures ; elle révoque ensuite les jetons et les clés SSH, et désactive GitHub Actions |
| 23 mai | Packagist retire les versions malveillantes et masque temporairement les quatre paquets, rétablis après nettoyage |

Le code malveillant tenait dans un nouveau fichier, `src/helpers.php`, déclaré dans le `composer.json` du paquet :

```json
"autoload": {
    "files": ["src/helpers.php"]
}
```

L'entrée `autoload.files` demande à Composer de charger ce fichier **à chaque exécution** de l'application : chaque requête web, chaque commande, chaque tâche de fond. Le fichier téléchargeait un second programme, qui cherchait les clés de cloud, les secrets Kubernetes, les jetons de CI, les clés SSH et les fichiers `.env`, puis les envoyait à l'attaquant. Le commit d'origine des dépôts n'a pas été modifié : tout passait par les étiquettes.

Les paquets `laravel-lang` sont maintenus par une communauté indépendante : ils ne font pas partie du framework Laravel officiel.

## 3 - La chaîne des causes

1. **Un jeton trop puissant, et volé.** Un jeton personnel permettait de supprimer et de recréer les étiquettes de plusieurs dépôts.
2. **Des étiquettes modifiables.** Rien n'empêchait de recréer une ancienne étiquette sur un autre commit.
3. **Des projets qui font confiance au numéro de version.** Toute installation qui résout la version à ce moment-là, un `composer update` ou une installation sans fichier de verrouillage, reçoit le code de l'attaquant.
4. **Un code exécuté dès le chargement.** Avec `autoload.files`, il n'est même pas nécessaire d'appeler une fonction du paquet.

## 4 - À quelle phase est né le problème ?

| Phase | Chez qui | Ce qui s'y est passé |
|---|---|---|
| Exploitation | les mainteneurs | un jeton personnel aux droits étendus, volé |
| **Dépendances** | les projets utilisateurs | une dépendance réinstallée à partir d'un numéro de version, sans vérifier que le contenu était le même qu'avant |

Pour une équipe qui développe une application Laravel, le problème est né dans ses **dépendances**. Elle n'a écrit aucune ligne fautive, et le paquet n'est même pas du code métier : des traductions.

## 5 - Ce que ça change pour vous

- **Versionner le fichier de verrouillage.** `composer.lock`, comme `package-lock.json` pour npm, enregistre le commit exact et, selon la source, une empreinte de chaque dépendance. `composer install` installe ce qui est verrouillé ; `composer update` résout à nouveau les versions. Relire le diff du fichier de verrouillage dans chaque merge request.
- **Un numéro de version n'est pas un contenu.** Après un incident comme celui-ci, les versions ont été rattachées de nouveau au bon code : le numéro seul ne dit pas si un projet a été touché, il faut regarder la date d'installation et le commit.
- **Limiter les jetons.** Un jeton à droits fins, limité à un dépôt et à une durée, aurait réduit les dégâts.
- **Limiter les sorties réseau** des serveurs de CI et de production : le second programme n'aurait pas pu être téléchargé depuis un serveur qui ne parle qu'aux adresses autorisées.

## Récap

- Les 22 et 23 mai 2026, environ 700 anciennes versions de quatre paquets `laravel-lang` sont recréées sur du code malveillant, avec un jeton GitHub volé.
- Le code s'exécutait à chaque requête grâce à `autoload.files`, et volait les secrets du serveur.
- Le numéro de version restait le même : seul le commit avait changé.
- Pour les projets utilisateurs, le problème est né dans les **dépendances** ; un fichier de verrouillage versionné et relu est la première protection.

> 📖 [Laravel-Lang, déclaration des mainteneurs, mai 2026 (en anglais)](https://habr.com/en/articles/1038568/) · [Snyk, « Laravel-Lang Supply Chain Attack », mai 2026 (en anglais)](https://snyk.io/blog/laravel-lang-supply-chain-advisory) · [Composer, le fichier composer.lock (en anglais)](https://getcomposer.org/doc/01-basic-usage.md#commit-your-composer-lock-file-to-version-control)

```quiz
title: Laravel-Lang, mai 2026
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Comment l'attaquant a-t-il pu modifier d'anciennes versions sans toucher au code d'origine ?
    answers:
      - "En piratant les serveurs de Packagist"
      - "En supprimant les étiquettes Git et en les recréant, avec les mêmes noms, sur ses propres commits"
      - "En modifiant le framework Laravel officiel"
      - "En publiant un paquet au nom proche"
    correct: 2
    explanation: |
      Packagist lit les versions dans les étiquettes Git. Recréer une étiquette sur un autre
      commit change le code derrière le même numéro de version.
  - prompt: Pourquoi le code malveillant s'exécutait-il sans que l'application appelle une fonction du paquet ?
    answers:
      - "Il était déclaré dans `autoload.files`, que Composer charge à chaque exécution"
      - "Il remplaçait le fichier index.php"
      - "Il s'exécutait seulement dans les tests"
      - "Il demandait une confirmation à l'utilisateur"
    correct: 1
    explanation: |
      Composer charge chaque fichier de `autoload.files` dès le démarrage de l'application :
      chaque requête, commande ou tâche de fond.
  - prompt: Pour une équipe qui utilise ces paquets, à quelle phase est né le problème ?
    answers:
      - "Exigences"
      - "Dépendances : un paquet tiers, résolu par son numéro de version, a apporté un code malveillant"
      - "Codage de son application"
      - "Conception de sa base de données"
    correct: 2
    explanation: |
      L'équipe n'a écrit aucune ligne fautive. Le risque venait d'une dépendance, installée
      d'après un numéro de version dont le contenu avait changé.
  - prompt: Quelle commande résout à nouveau les versions, et peut donc récupérer une étiquette réécrite ?
    answers:
      - "`composer install` avec un `composer.lock` versionné"
      - "`composer update`"
      - "`git status`"
      - "`php -v`"
    correct: 2
    explanation: |
      `composer install` suit le fichier de verrouillage, qui enregistre le commit exact.
      `composer update` cherche les versions à nouveau.
  - prompt: Côté mainteneurs, quelle mesure aurait le plus réduit les dégâts ?
    answers:
      - "Traduire les paquets dans plus de langues"
      - "Un jeton à droits fins, limité à un dépôt et à une durée"
      - "Changer le nom des paquets"
      - "Publier plus souvent"
    correct: 2
    explanation: |
      Le jeton volé permettait de réécrire les étiquettes de plusieurs dépôts. Moins de droits,
      c'est moins de dégâts quand il fuite.
```
