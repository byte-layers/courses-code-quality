---
title: "03 - Une première analyse, avec les réglages par défaut"
description: "Mesurer la couverture, analyser les trois versions, et constater que SonarQube laisse passer le code de l'agent"
---

SonarQube est prêt, le jeton aussi. On analyse les trois versions du projet sans rien régler, pour voir ce que donne une installation neuve.

## 03.1 - Mesurer la couverture des tests

SonarQube n'exécute pas les tests : il lit un **rapport de couverture**, produit avant l'analyse. Le lanceur de tests de Node.js sait l'écrire au format **LCOV**, le plus répandu. Cette commande le produit pour chaque version :

```exec
cd ~/byte-layers/quality && for v in main agent clean; do (cd measure/$v && mkdir -p coverage && node --test --experimental-test-coverage --test-reporter=lcov --test-reporter-destination=coverage/lcov.info >/dev/null && echo "$v : coverage/lcov.info écrit" || echo "$v : tests en échec, lancez npm test dans measure/$v pour voir lesquels"); done
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `for v in main agent clean; do … done` | répète la suite pour chaque version |
> | `( … )` | exécute les commandes entre parenthèses dans un sous-terminal : le `cd` ne change pas le dossier de la boucle |
> | `mkdir -p coverage` | crée le dossier du rapport ; Node.js ne le crée pas lui-même |
> | `node --test --experimental-test-coverage` | lance les tests en mesurant la couverture |
> | `--test-reporter=lcov --test-reporter-destination=coverage/lcov.info` | écrit le résultat au format LCOV, dans `coverage/lcov.info` |
> | `>/dev/null` | cache le reste de l'affichage |
> | `&& echo … \|\| echo …` | affiche une ligne si les tests ont réussi, une autre s'ils ont échoué |

Trois lignes `… : coverage/lcov.info écrit` s'affichent.

## 03.2 - Le script d'analyse

Le scanner est lui aussi une image Docker. Sa commande est longue, et vous la lancerez plusieurs fois : cette commande l'écrit une fois pour toutes dans un script, `sonar-scan.sh`, qui analyse les versions qu'on lui donne :

```exec
cd ~/byte-layers/quality && cat > sonar-scan.sh <<'FIN'
#!/usr/bin/env bash
# Analyse chaque version demandée avec SonarQube. Exemple : ./sonar-scan.sh main agent clean
cd "$(dirname "$0")" || exit 1
export MSYS_NO_PATHCONV=1
[ $# -gt 0 ] || set -- main agent clean
for v in "$@"; do
  docker run --rm --platform linux/amd64 --network sonarqube-lab \
    --user "$(id -u):$(id -g)" -e SONAR_USER_HOME=/tmp/.sonar \
    -v "$(pwd -W 2>/dev/null || pwd):/usr/src" \
    -e SONAR_HOST_URL=http://sonarqube:9000 -e SONAR_TOKEN="$(cat sonar-token)" \
    sonarsource/sonar-scanner-cli:12.4.0.4321_8.4.0 \
    -Dsonar.projectKey="todo-api-$v" \
    -Dsonar.projectBaseDir="/usr/src/measure/$v" \
    -Dsonar.sources=src -Dsonar.tests=test \
    -Dsonar.javascript.lcov.reportPaths=coverage/lcov.info \
    -Dsonar.scm.disabled=true \
    -Dsonar.working.directory=/tmp/.scannerwork \
    -Dsonar.qualitygate.wait=true > "sonar-$v.log" 2>&1
  code=$?
  echo "$v : $(grep -o 'QUALITY GATE STATUS: [A-Z]*' "sonar-$v.log" || echo 'analyse en échec, voir le fichier sonar-'"$v"'.log'), code de sortie $code"
done
FIN
chmod +x sonar-scan.sh
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > sonar-scan.sh <<'FIN'` … `FIN` | écrit le script ; `chmod +x` le rend exécutable |
> | `export MSYS_NO_PATHCONV=1` | sous Git Bash, empêche la conversion des chemins comme `/usr/src` en chemins Windows ; sans effet ailleurs |
> | `[ $# -gt 0 ] \|\| set -- main agent clean` | sans argument, analyse les trois versions |
> | `--platform linux/amd64` | l'image du scanner n'existe que pour les processeurs Intel et AMD ; sur un Mac à puce Apple, Docker l'émule, un peu plus lentement |
> | `--network sonarqube-lab` et `SONAR_HOST_URL=http://sonarqube:9000` | le scanner joint le serveur par le réseau de l'étape 01 |
> | `--user "$(id -u):$(id -g)"` et `SONAR_USER_HOME=/tmp/.sonar` | le scanner tourne avec votre identité, pour que les fichiers qu'il lit vous appartiennent, et garde son cache dans le conteneur |
> | `-v "$(pwd -W 2>/dev/null \|\| pwd):/usr/src"` | rend `~/byte-layers/quality` visible dans le conteneur, sous `/usr/src` ; `pwd -W` donne le chemin au format Windows sous Git Bash, `pwd` ailleurs |
> | `SONAR_TOKEN="$(cat sonar-token)"` | passe le jeton au scanner, sans l'écrire dans le script |
> | `-Dsonar.projectKey="todo-api-$v"` | un projet SonarQube par version : `todo-api-main`, `todo-api-agent`, `todo-api-clean` ; SonarQube le crée à la première analyse |
> | `-Dsonar.sources=src -Dsonar.tests=test` | le code à analyser, et le code des tests, compté à part |
> | `-Dsonar.javascript.lcov.reportPaths=…` | le rapport de couverture de la section 03.1 |
> | `-Dsonar.scm.disabled=true` | n'interroge pas Git : dans un worktree, `.git` pointe vers un chemin de votre poste que le conteneur ne voit pas ; un contournement propre à ce lab |
> | `-Dsonar.working.directory=/tmp/.scannerwork` | écrit les fichiers de travail du scanner dans le conteneur, pas dans vos worktrees |
> | `-Dsonar.qualitygate.wait=true` | attend le verdict de la quality gate, et rend un code de sortie différent de 0 si elle échoue |
> | `> "sonar-$v.log" 2>&1` | garde tout l'affichage du scanner dans un fichier, pour le relire en cas de problème |
> | `code=$?` | garde le code de sortie du scanner, affiché ensuite |

## 03.3 - Analyser les trois versions

Lancez le script. Chaque analyse prend environ une demi-minute :

```exec
cd ~/byte-layers/quality && ./sonar-scan.sh
```

Le résultat attendu, au bout d'une minute et demie environ :

```text
main : QUALITY GATE STATUS: PASSED, code de sortie 0
agent : QUALITY GATE STATUS: PASSED, code de sortie 0
clean : QUALITY GATE STATUS: PASSED, code de sortie 0
```

Ouvrez [http://localhost:9000](http://localhost:9000) : la page **Projects** liste les trois projets. Ouvrez `todo-api-agent`. Le code de l'agent, avec sa complexité cognitive de 91, son jeton écrit en dur et ses blocs copiés, affiche **0 problème** (*issues*) et une quality gate **Passed**. Seule la couverture, 68,5 %, plus basse que celle de `main` (83,7 %), trahit quelque chose.

## 03.4 - Pourquoi SonarQube ne voit rien

Deux réglages par défaut l'expliquent :

| Réglage par défaut | Ce qu'il fait | Pourquoi rien n'est arrêté |
|---|---|---|
| le **profil** JavaScript *Sonar way core* | environ 200 règles, celles sur lesquelles presque tout le monde s'accorde | la règle de complexité cognitive n'en fait pas partie, et le code de l'agent n'enfreint aucune des autres |
| la **quality gate** *Sonar way* | une seule condition : aucun nouveau problème dans le **nouveau code** | ici, aucune ligne n'est « nouvelle » : il n'y a rien à juger |

Ce second réglage applique la démarche que SonarSource appelle *Clean as You Code* : ne juger que le code ajouté ou modifié, pour ne pas noyer l'équipe sous la dette existante. C'est la même idée que le cliquet du lab précédent. SonarQube date chaque ligne grâce à Git, et compte comme nouvelles celles modifiées depuis la version précédente du projet. Dans ce lab, chaque version est un projet à part analysé une seule fois, et Git est désactivé : rien ne peut être nouveau. La démarche prend tout son sens quand SonarQube analyse le même projet à chaque changement, idéalement à chaque merge request, ce que fait l'édition payante.

Profils et quality gates se règlent dans l'interface : c'est l'objet de l'étape suivante.

> 📖 [SonarQube : les profils de qualité (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/instance-administration/analysis-functions/quality-profiles/) · [SonarQube : les quality gates (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/instance-administration/analysis-functions/quality-gates/) · [SonarScanner CLI (en anglais)](https://docs.sonarsource.com/sonarqube-community-build/analyzing-source-code/scanners/sonarscanner/)

```verify
cd ~/byte-layers/quality 2>/dev/null || { echo "Le dossier ~/byte-layers/quality n'existe pas."; exit 1; }
[ -x sonar-scan.sh ] || { echo "Le script sonar-scan.sh n'existe pas : lancez la commande de la section 03.2."; exit 1; }
T=$(cat sonar-token 2>/dev/null)
for v in main agent clean; do
  [ -f "measure/$v/coverage/lcov.info" ] || { echo "Le rapport de couverture de $v manque : lancez la commande de la section 03.1."; exit 1; }
  curl -s --max-time 10 -u "$T:" "http://localhost:9000/api/qualitygates/project_status?projectKey=todo-api-$v" | grep -Eq '"status":"(OK|ERROR)"' || { echo "Le projet todo-api-$v n'a pas encore d'analyse dans SonarQube : lancez ./sonar-scan.sh, puis lisez le fichier sonar-$v.log en cas d'échec."; exit 1; }
done
exit 0
```
