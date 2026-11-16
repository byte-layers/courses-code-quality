---
title: "03 - Écrire le pipeline"
description: "Un job par outil, un rapport de qualité pour ESLint, et la règle qui interdit de fusionner une merge request en échec"
---

Le pipeline se décrit dans le fichier `.gitlab-ci.yml`, à la racine du dépôt. On y met un **job** par contrôle : les tests, ESLint, jscpd et Semgrep. Des jobs séparés tournent en parallèle, et chacun affiche son propre verdict : on voit tout de suite quel outil a échoué.

## 03.1 - À vous de compléter le pipeline

Cette commande écrit le pipeline avec deux `TODO`, ceux des jobs `duplication` et `semgrep` :

```exec
cd ~/byte-layers/quality/todo-api && cat > .gitlab-ci.yml <<'FIN'
workflow:
  rules:
    - if: $CI_PIPELINE_SOURCE == "merge_request_event"
    - if: $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH

default:
  image: node:24-bookworm-slim

tests:
  script:
    - npm ci
    - npm test

eslint:
  script:
    - npm ci
    - npx eslint --format gitlab src
  artifacts:
    when: always
    reports:
      codequality: gl-codequality.json

duplication:
  script:
    - npm ci
    - TODO

semgrep:
  image: semgrep/semgrep:1.184.0
  script:
    - TODO
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > .gitlab-ci.yml <<'FIN'` … `FIN` | écrit les lignes jusqu'à `FIN` dans le fichier `.gitlab-ci.yml` ; les apostrophes autour de `FIN` gardent les `$` du pipeline tels quels |

Le fichier, bloc par bloc :

| Bloc | Rôle |
|---|---|
| `workflow: rules:` | crée un pipeline pour chaque merge request, et pour chaque envoi sur la branche par défaut, `main` ; pas pour les autres branches, ce qui évite un pipeline en double |
| `default: image:` | chaque job tourne dans un conteneur créé à partir de cette image Docker, ici Node.js 24 |
| `tests:`, `eslint:`… | le nom de chaque job, choisi librement |
| `script:` | les commandes du job ; le job échoue dès qu'une commande échoue |
| `npm ci` | installe exactement les versions de `package-lock.json`, sur une machine neuve ; plus strict que `npm install` |
| `--format gitlab` | ESLint écrit le fichier `gl-codequality.json` au format de GitLab, et affiche dans le journal du job chaque défaut avec un lien vers la ligne concernée |
| `artifacts: reports: codequality:` | transmet ce fichier à GitLab, qui l'affiche dans la merge request |
| `when: always` | garde le rapport même quand le job échoue : c'est justement là qu'il est utile |
| `image: semgrep/semgrep:1.184.0` | le job Semgrep remplace l'image par défaut par celle de Semgrep, comme à l'étape 04 du lab précédent |

Ouvrez `~/byte-layers/quality/todo-api/.gitlab-ci.yml` dans votre éditeur, et remplacez chaque `TODO` par la commande de l'outil, appliquée au dossier `src`. Reprenez les commandes du lab précédent, avec deux différences :

- pour jscpd, l'affichage par défaut suffit : pas besoin de `--reporters` ;
- pour Semgrep, pas de `docker run` (le job tourne déjà dans l'image de Semgrep), et pas de `--no-git-ignore` : dans le pipeline, le projet est un clone ordinaire, que Semgrep lit sans peine.

Gardez l'indentation : en YAML, les espaces en début de ligne font partie de la structure.

```hint
Le job `duplication` lance `npx jscpd --min-tokens 30 --threshold 5 src`. Le job `semgrep` lance `semgrep scan` avec vos règles, sans statistiques, et l'option qui transforme un résultat en échec : `--config .semgrep/rules.yml --metrics=off --error src`.
```

````solution
```yaml
duplication:
  script:
    - npm ci
    - npx jscpd --min-tokens 30 --threshold 5 src

semgrep:
  image: semgrep/semgrep:1.184.0
  script:
    - semgrep scan --config .semgrep/rules.yml --metrics=off --error src
```
````

```verify
F=~/byte-layers/quality/todo-api/.gitlab-ci.yml
[ -f "$F" ] || { echo "Le fichier .gitlab-ci.yml n'existe pas dans le projet : lancez la commande de la section 03.1."; exit 1; }
grep -q "TODO" "$F" && { echo "Le fichier .gitlab-ci.yml contient encore un TODO : complétez-le."; exit 1; }
grep -q "npx jscpd" "$F" || { echo "Le job duplication ne lance pas jscpd (npx jscpd …)."; exit 1; }
grep -q "semgrep scan" "$F" && grep -q -- "--error" "$F" || { echo "Le job semgrep doit lancer semgrep scan avec l'option --error, sinon il ne fera jamais échouer le pipeline."; exit 1; }
grep -q "docker run" "$F" && { echo "Le pipeline ne doit pas contenir docker run : le job semgrep tourne déjà dans l'image de Semgrep."; exit 1; }
exit 0
```

## 03.2 - Envoyer le pipeline sur main

Les outils, leur configuration et le pipeline partent dans un seul commit. Pour aller vite, vous l'envoyez directement sur `main`, en tant que propriétaire du projet ; en équipe, ce changement passerait lui aussi par une merge request.

```exec
cd ~/byte-layers/quality/todo-api && git add package.json package-lock.json eslint.config.js .semgrep .gitlab-ci.yml && git commit -m "Ajoute une barrière qualité : ESLint, jscpd et Semgrep" && git push gitlab main
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git add …` | prépare les quatre fichiers et le dossier `.semgrep` pour le commit |
> | `git commit -m "…"` | enregistre le commit sur `main` |
> | `git push gitlab main` | l'envoie sur gitlab.com, ce qui déclenche un pipeline |

Ouvrez **Build**, **Pipelines** dans le projet GitLab. Un pipeline tourne sur `main`, avec quatre jobs. Au bout de deux à trois minutes, les quatre sont verts. Ce pipeline compte : GitLab compare le rapport de qualité d'une merge request à celui de la branche visée, ici `main`.

Si un job échoue, cliquez dessus pour lire son journal : une erreur d'indentation du YAML est signalée avant même le lancement, sur la page des pipelines.

## 03.3 - Interdire la fusion d'une merge request en échec

Un pipeline rouge ne bloque rien tant que GitLab n'en tient pas compte. Dans le projet, ouvrez **Settings**, **Merge requests**, et dans la partie **Merge checks**, cochez **Pipelines must succeed**, puis cliquez sur **Save changes**. Ce réglage n'est pas vérifié automatiquement : à l'étape suivante, le bouton **Merge** indisponible prouvera qu'il est actif.

> 📖 [GitLab : le fichier .gitlab-ci.yml (en anglais)](https://docs.gitlab.com/ci/yaml/) · [GitLab : Code Quality (en anglais)](https://docs.gitlab.com/ci/testing/code_quality/) · [eslint-formatter-gitlab (en anglais)](https://www.npmjs.com/package/eslint-formatter-gitlab) · [GitLab : exiger un pipeline réussi (en anglais)](https://docs.gitlab.com/user/project/merge_requests/auto_merge/#require-a-successful-pipeline-for-merge)

```verify
C=$(cat ~/byte-layers/quality/gitlab-url 2>/dev/null)
[ -n "$C" ] || { echo "L'adresse du projet GitLab n'est pas enregistrée dans ~/byte-layers/quality/gitlab-url : lancez la commande de la section 01.1."; exit 1; }
case "$C" in
  git@*) H="https://$(echo "$C" | sed -E 's#git@([^:]+):.*#\1#')"; P=$(echo "$C" | sed -E 's#git@[^:]+:##') ;;
  *) H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#'); P=$(echo "$C" | sed -E 's#^https?://[^/]+/##') ;;
esac
A="$H/api/v4/projects/$(echo "$P" | sed -E 's#\.git$##; s#/#%2F#g')"
Y=$(curl -fsS "$A/repository/files/.gitlab-ci.yml/raw?ref=main" 2>/dev/null)
[ -n "$Y" ] || { echo "Le fichier .gitlab-ci.yml n'est pas sur la branche main du projet GitLab : lancez la commande de la section 03.2."; exit 1; }
for w in "npx eslint" "npx jscpd" "semgrep scan" "codequality"; do echo "$Y" | grep -q "$w" || { echo "Le pipeline publié sur main ne contient pas « $w »."; exit 1; }; done
S=$(curl -fsS "$A/pipelines?ref=main&per_page=1" 2>/dev/null | grep -o '"status":"[a-z_]*"' | head -1)
[ "$S" = '"status":"success"' ] || { echo "Le dernier pipeline de main n'a pas réussi ($S) : attendez qu'il se termine, ou lisez le journal du job en échec."; exit 1; }
exit 0
```
