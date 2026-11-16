---
title: "05 - SonarQube face aux outils légers"
description: "Comparer ce que SonarQube et les outils du lab précédent trouvent, voir comment brancher SonarQube sur un pipeline, et ranger"
---

SonarQube arrête maintenant le code de l'agent. Mais trouve-t-il les mêmes défauts que les outils légers du lab précédent ?

## 05.1 - Ce que chacun trouve

| Défaut du code de l'agent | ESLint, jscpd et Semgrep (lab précédent) | SonarQube, profil `todo-api` |
|---|---|---|
| complexité cognitive de 91 | ✅ | ✅ |
| `==` et `!=` | ✅ (`eqeqeq`) | ❌ règle absente du profil |
| `formatDate` jamais appelée | ✅ (`no-unused-vars`) | ❌ |
| l'échappement CSV copié trois fois | ✅ (jscpd, 30 jetons de code) | ❌ blocs trop courts pour son seuil de duplication, d'environ 100 jetons de code |
| le jeton écrit en dur, et lu dans l'adresse | ✅ (vos règles Semgrep) | ❌ |
| la couverture en baisse | ❌ | ✅ (68,5 %, sous 80 %) |
| `replace` avec une expression régulière au lieu de `replaceAll` | ❌ | ✅ |

Aucun outil ne voit tout. SonarQube apporte un tableau de bord, un historique et la couverture ; les outils légers vont plus loin là où on les règle finement, comme les règles maison de Semgrep. Les réglages font plus que l'outil : un SonarQube par défaut laissait tout passer, une configuration de dix minutes arrête le code de l'agent. On peut aussi activer d'autres règles dans le profil `todo-api`, par exemple celle des comparaisons strictes.

## 05.2 - Et dans un pipeline ?

Un SonarQube installé sur votre poste n'est joignable que depuis votre poste : les runners partagés de gitlab.com ne le voient pas. Pour l'utiliser dans un pipeline, il faut un runner installé sur la même machine, comme celui du lab [Un runner GitLab sur son poste](https://byte-layers.com/fr/labs/gitlab-runner-local), ou un SonarQube hébergé sur un serveur que les runners joignent. Il faut deux jobs : le premier lance les tests avec Node.js et garde le rapport de couverture en artefact, le second lance le scanner, dont l'image n'a pas Node.js. Le jeton est rangé dans une variable masquée du projet GitLab, nommée `SONAR_TOKEN` (**Settings**, **CI/CD**, **Variables**) ; si vous la marquez aussi *Protected*, elle n'est transmise qu'aux branches protégées.

```yaml
coverage:
  image: node:24-bookworm-slim
  script:
    - npm ci
    - node --test --experimental-test-coverage --test-reporter=lcov --test-reporter-destination=coverage/lcov.info
  artifacts:
    paths:
      - coverage/

sonarqube:
  needs: [coverage]
  image:
    name: sonarsource/sonar-scanner-cli:12.4.0.4321_8.4.0
    entrypoint: [""]
  variables:
    SONAR_HOST_URL: http://sonarqube:9000
    GIT_DEPTH: "0"
  script:
    - sonar-scanner -Dsonar.projectKey=todo-api -Dsonar.sources=src -Dsonar.tests=test -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info -Dsonar.qualitygate.wait=true
```

Ce pipeline n'est pas à lancer ici : il suppose un runner local et un SonarQube joignable sous le nom `sonarqube`. Avec le runner du lab cité plus haut, ce nom n'est connu que si les conteneurs des jobs rejoignent le réseau Docker `sonarqube-lab` : la ligne `network_mode = "sonarqube-lab"`, dans la section `[runners.docker]` de la configuration du runner, le demande. `needs: [coverage]` attend le premier job et récupère son artefact ; `GIT_DEPTH: "0"` clone tout l'historique, dont SonarQube a besoin pour dater les lignes : dans un pipeline, plus besoin de `sonar.scm.disabled`.

## 05.3 - Ranger

SonarQube consomme environ 2 Go de mémoire. Pour l'arrêter sans rien perdre : `docker stop sonarqube`, puis `docker start sonarqube` pour le relancer. Pour tout supprimer, conteneur, données, réseau, images, et les fichiers de ce lab :

```exec
cd ~ && docker rm -f sonarqube && docker volume rm sonarqube-data && docker network rm sonarqube-lab && docker image rm sonarqube:26.11.0.134175-community sonarsource/sonar-scanner-cli:12.4.0.4321_8.4.0 && rm -rf ~/byte-layers/quality/sonar-token ~/byte-layers/quality/sonar-scan.sh ~/byte-layers/quality/sonar-*.log ~/byte-layers/quality/measure/*/coverage
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `docker rm -f sonarqube` | arrête et supprime le conteneur |
> | `docker volume rm sonarqube-data` | supprime ses données : projets, analyses, profil et gate |
> | `docker network rm sonarqube-lab` | supprime le réseau de l'étape 01 |
> | `docker image rm …` | supprime les deux images, environ 1,5 Go |
> | `rm -rf …` | supprime le jeton, le script, les journaux et les rapports de couverture de ce lab |

```acknowledge
J'ai comparé ce que trouvent SonarQube et les outils légers, et compris comment SonarQube se branche sur un pipeline.
```

## Récap

- **SonarQube Community Build** tourne dans un conteneur ; le **scanner** analyse le code et envoie les résultats au serveur, avec un **jeton**.
- SonarQube **lit** la couverture dans un rapport (LCOV pour JavaScript) : il ne lance pas les tests.
- Par défaut, le profil *Sonar way core* et la gate *Sonar way* ne jugent que le **nouveau code** : une première analyse ne bloque rien.
- Un **profil** choisit les règles, une **quality gate** fixe les conditions du verdict ; tous deux se copient et se règlent pour le projet.
- `sonar.qualitygate.wait=true` transforme le verdict en code de sortie, prêt pour un pipeline ; un SonarQube sur le poste n'est joignable que par un runner local.

> 📖 [SonarQube : les profils de qualité (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/instance-administration/analysis-functions/quality-profiles/) · [SonarQube : les quality gates (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/instance-administration/analysis-functions/quality-gates/) · [SonarQube : la couverture JavaScript (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/test-coverage/javascript-typescript-test-coverage/) · [SonarQube et GitLab CI (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/devops-platform-integration/gitlab-integration/adding-analysis-to-gitlab-ci-cd)
