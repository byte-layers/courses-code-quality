---
title: "01 - Démarrer SonarQube"
description: "Lancer SonarQube Community Build dans un conteneur, et attendre qu'il soit prêt"
---

SonarQube est un serveur : une application web, un moteur de recherche interne et une base de données. L'image Docker officielle contient tout. Toutes les commandes de cette étape sont fournies : elle prépare le terrain.

## 01.1 - Lancer le conteneur

Les conteneurs de ce lab se parlent par un **réseau Docker** dédié : le serveur SonarQube, puis l'outil d'analyse qui lui enverra ses résultats. On crée ce réseau, puis on lance le serveur :

```exec
cd ~ && { docker network inspect sonarqube-lab >/dev/null 2>&1 || docker network create sonarqube-lab; } && docker run -d --name sonarqube --network sonarqube-lab -p 9000:9000 -v sonarqube-data:/opt/sonarqube/data sonarqube:26.11.0.134175-community
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `docker network inspect sonarqube-lab … \|\| docker network create sonarqube-lab` | crée un réseau où les conteneurs se trouvent par leur nom, sauf s'il existe déjà |
> | `docker run -d --name sonarqube` | lance le serveur en arrière-plan, dans un conteneur nommé `sonarqube` |
> | `--network sonarqube-lab` | le branche sur ce réseau : les autres conteneurs le joindront à l'adresse `http://sonarqube:9000` |
> | `-p 9000:9000` | rend l'interface web accessible depuis votre navigateur, sur `http://localhost:9000` |
> | `-v sonarqube-data:/opt/sonarqube/data` | garde les données (projets, analyses, réglages) dans un **volume** Docker, qui survit à l'arrêt du conteneur |
> | `sonarqube:26.11.0.134175-community` | l'édition gratuite, *Community Build*, dans une version fixée |

Si Docker répond que le nom `sonarqube` est déjà utilisé, le conteneur existe d'un essai précédent : `docker start sonarqube` le relance. Le premier lancement télécharge environ 1 Go. Ce SonarQube garde ses données dans une base intégrée, prévue pour l'essai : en production, on lui associe une base PostgreSQL.

## 01.2 - Attendre qu'il soit prêt

Le serveur met une à deux minutes à démarrer. Cette commande interroge son état toutes les cinq secondes, et rend la main quand il répond `UP` :

```exec
cd ~ && until curl -s http://localhost:9000/api/system/status | grep -q '"status":"UP"'; do echo "SonarQube démarre…"; sleep 5; done; echo "SonarQube est prêt."
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `until … ; do … ; done` | répète le corps de la boucle **jusqu'à ce que** la condition réussisse |
> | `curl -s http://localhost:9000/api/system/status` | demande son état au serveur, sans afficher la progression |
> | `\| grep -q '"status":"UP"'` | réussit si la réponse contient `"status":"UP"` |
> | `echo …; sleep 5` | affiche un message, puis attend cinq secondes avant de réessayer |

Si la boucle tourne plus de cinq minutes, arrêtez-la avec Ctrl+C, vérifiez avec `docker ps -a` que le conteneur tourne encore, et lisez les journaux du serveur avec `docker logs sonarqube`. **Sous Linux**, une ligne `max virtual memory areas vm.max_map_count [65530] is too low` signifie que le système limite le moteur de recherche interne : lancez `sudo sysctl -w vm.max_map_count=524288`, puis `docker restart sonarqube`. Docker Desktop, sous macOS et Windows, n'est pas concerné.

> 📖 [SonarQube Community Build : installer avec Docker (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/try-out-sonarqube/) · [L'image Docker sonarqube (en anglais)](https://hub.docker.com/_/sonarqube)

```verify
curl -s --max-time 10 http://localhost:9000/api/system/status 2>/dev/null | grep -q '"status":"UP"' || { echo "SonarQube ne répond pas UP sur http://localhost:9000 : vérifiez qu'il tourne (docker ps), et attendez la fin du démarrage (section 01.2)."; exit 1; }
exit 0
```
