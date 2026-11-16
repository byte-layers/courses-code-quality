---
title: "04 - Un profil et une quality gate pour le projet"
description: "Choisir les règles et les conditions du verdict, pour que SonarQube arrête le code de l'agent sans bloquer main"
---

On veut le même verdict qu'au lab précédent : `main` et votre version passent, le code de l'agent échoue. Il faut pour cela deux réglages, à faire dans l'interface de SonarQube. Comme ce SonarQube n'analyse que la branche principale, la quality gate jugera **tout le code** (*overall code*), pas seulement le nouveau.

## 04.1 - À vous de régler le profil

Un profil intégré ne se modifie pas : on le **copie**, puis on change la copie. Dans le menu **Quality Profiles** :

1. Dans la partie **JavaScript**, ouvrez le menu de la ligne **Sonar way extended** (environ 250 règles, dont celle de la complexité cognitive), puis **Copy**, et nommez la copie `todo-api`.
2. Dans le profil `todo-api`, retrouvez la règle *Cognitive Complexity of functions should not be too high*, et changez son paramètre **threshold** : 15 par défaut, à vous de choisir la valeur qui accepte la dette d'origine de `main` (25 de complexité totale, mais 18 pour sa fonction la plus complexe) et votre version, comme au lab précédent.
3. Ouvrez le menu du profil `todo-api`, puis **Set as Default** : les trois projets l'utiliseront à leur prochaine analyse.

```hint
Dans le profil `todo-api`, cliquez sur le nombre de règles actives, cherchez « Cognitive Complexity » dans la zone de recherche, puis cliquez sur **Change** à côté de la règle pour modifier son paramètre. Toute valeur d'au moins 18 qui accepte aussi votre version convient ; la solution prend 20, comme pour ESLint au lab précédent.
```

## 04.2 - À vous d'écrire la quality gate

Dans le menu **Quality Gates**, cliquez sur **Create**, et nommez la gate `todo-api`. Ajoutez-lui des conditions avec **Add Condition**, en choisissant **On Overall Code**. Le verdict doit :

- échouer dès qu'il reste un **problème** (*issue*) dans le code, au sens du profil `todo-api` ;
- échouer si la **couverture** des tests descend sous 80 %, le seuil que le lab [Tester les tests avec la mutation](https://byte-layers.com/fr/labs/mutation-testing) a rappelé.

Selon la version, SonarQube fixe lui-même l'opérateur de chaque métrique, et signale que cette gate ne suit pas *Clean as You Code* : c'est voulu ici, puisqu'elle juge tout le code. Puis ouvrez le menu de la gate `todo-api`, et **Set as Default**.

```hint
Les deux métriques s'appellent **Issues** et **Coverage** dans la liste. Pour la première, l'opérateur est « is greater than » ; pour la seconde, « is less than ».
```

## 04.3 - Relancer les analyses

Mesurez à nouveau, avec le même script :

```exec
cd ~/byte-layers/quality && ./sonar-scan.sh
```

Le résultat attendu, si vos réglages sont justes :

```text
main : QUALITY GATE STATUS: PASSED, code de sortie 0
agent : QUALITY GATE STATUS: FAILED, code de sortie 3
clean : QUALITY GATE STATUS: PASSED, code de sortie 0
```

Si le verdict diffère, comparez vos réglages à la solution, puis relancez le script.

````solution
| Réglage | Valeur |
|---|---|
| profil JavaScript | `todo-api`, copie de *Sonar way extended*, par défaut |
| règle *Cognitive Complexity of functions should not be too high* | threshold : **20** |
| quality gate | `todo-api`, par défaut |
| condition 1, sur tout le code | **Issues** is greater than **0** |
| condition 2, sur tout le code | **Coverage** is less than **80.0** |
````

Ouvrez `todo-api-agent` dans l'interface. La gate est **Failed**, pour deux raisons : 4 problèmes (la complexité cognitive de 91, et trois appels à `replace` avec une expression régulière là où `replaceAll` suffit), et une couverture de 68,5 %. Le code de sortie 3 du scanner est ce qui arrêterait un pipeline.

```verify
cd ~/byte-layers/quality 2>/dev/null || { echo "Le dossier ~/byte-layers/quality n'existe pas."; exit 1; }
T=$(cat sonar-token 2>/dev/null)
st(){ curl -s --max-time 10 -u "$T:" "http://localhost:9000/api/qualitygates/project_status?projectKey=todo-api-$1"; }
st main | grep -q '"status":"OK"' || { echo "La gate échoue sur todo-api-main : le seuil de complexité est trop bas, ou une condition est trop stricte. Relancez ./sonar-scan.sh après chaque changement."; exit 1; }
st clean | grep -q '"status":"OK"' || { echo "La gate échoue sur todo-api-clean : relevez le seuil de complexité, ou lisez les problèmes du projet dans l'interface."; exit 1; }
A=$(st agent)
echo "$A" | grep -q '"projectStatus":{"status":"ERROR"' || { echo "La gate laisse passer todo-api-agent : vérifiez que le profil et la gate todo-api sont par défaut (Set as Default), puis relancez ./sonar-scan.sh."; exit 1; }
echo "$A" | grep -q '"metricKey":"violations"' || { echo "La gate de todo-api-agent n'a pas de condition sur les problèmes (Issues) : ajoutez-la, sur tout le code (On Overall Code)."; exit 1; }
echo "$A" | grep -q '"metricKey":"coverage"' || { echo "La gate de todo-api-agent n'a pas de condition sur la couverture (Coverage) : ajoutez-la, sur tout le code (On Overall Code)."; exit 1; }
exit 0
```
