---
title: "01 - Ajouter les contrôles de sécurité"
description: "Un job d'audit des dépendances, un job de détection de secrets, et des images épinglées par empreinte"
---

On part de `main`, qu'on met d'abord à jour avec la fusion de votre version corrigée, et on y ajoute deux jobs.

## 01.1 - Les deux nouveaux jobs

La branche `main` de votre poste n'a pas encore la fusion faite sur gitlab.com : on la récupère d'abord, puis on ajoute les deux jobs à la fin du fichier du pipeline :

```exec
cd ~/byte-layers/quality/todo-api && git switch main && git pull gitlab main && cat >> .gitlab-ci.yml <<'FIN'

audit:
  script:
    - npm ci
    - npm audit --omit=dev --audit-level=high

secrets:
  image:
    name: zricethezav/gitleaks:v8.30.1
    entrypoint: [""]
  script:
    - gitleaks git --redact --verbose .
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch main && git pull gitlab main` | se place sur `main`, et y récupère la fusion faite sur gitlab.com |
> | `cat >> .gitlab-ci.yml <<'FIN'` … `FIN` | **ajoute** les lignes à la fin du pipeline (`>>`), sans effacer ce qui existe (`>` effacerait) |

Les deux jobs, ligne par ligne :

| Ligne | Rôle |
|---|---|
| `npm audit --omit=dev` | ne regarde que les dépendances **de production**, celles qui partent avec l'application ; les outils de développement, comme ESLint, ont eux aussi des failles signalées, mais ils ne sont jamais livrés |
| `--audit-level=high` | n'échoue qu'à partir des failles graves (*high* et *critical*) ; sans `allow_failure`, ce job bloque la merge request |
| `image: name: zricethezav/gitleaks:v8.30.1` | l'image officielle de gitleaks, dans une version fixée |
| `entrypoint: [""]` | l'image lance `gitleaks` dès son démarrage ; le runner a besoin de démarrer un shell pour lancer le script : on remplace ce démarrage par rien |
| `gitleaks git --redact --verbose .` | cherche des secrets dans **tout l'historique** de Git disponible, pas seulement dans les fichiers actuels ; `--redact` masque les secrets trouvés dans le journal du job, qui est public |

## 01.2 - Lire l'empreinte des images

Le pipeline utilise trois images, désignées par une étiquette : `node:24-bookworm-slim`, `semgrep/semgrep:1.184.0` et `zricethezav/gitleaks:v8.30.1`. Cette commande télécharge chacune d'elles, et affiche son **empreinte** :

```exec
cd ~ && for i in node:24-bookworm-slim semgrep/semgrep:1.184.0 zricethezav/gitleaks:v8.30.1; do docker pull -q "$i" >/dev/null && echo "$i -> $(docker inspect --format '{{index .RepoDigests 0}}' "$i")"; done
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `for i in … ; do … done` | répète la suite pour chaque image |
> | `docker pull -q "$i" >/dev/null` | télécharge l'image, sans rien afficher |
> | `docker inspect --format '{{index .RepoDigests 0}}' "$i"` | affiche son empreinte, sous la forme `nom@sha256:…` |
> | `$( … )` | remplace ce morceau par le résultat de la commande |

Chaque ligne ressemble à `node:24-bookworm-slim -> node@sha256:5c1f…` : 64 caractères hexadécimaux après `sha256:`. Ces empreintes changent à chaque nouvelle publication de l'image : les vôtres seront différentes de celles d'un autre apprenant, et c'est normal.

## 01.3 - À vous d'épingler les images

Ouvrez `~/byte-layers/quality/todo-api/.gitlab-ci.yml` dans votre éditeur. Pour chacune des trois images, ajoutez à la suite de l'étiquette un `@` et l'empreinte affichée à la section précédente, en gardant l'étiquette : elle reste lisible pour un humain, et l'empreinte décide de ce qui tourne.

```hint
L'image par défaut devient `image: node:24-bookworm-slim@sha256:<les 64 caractères>`. Faites de même pour l'image du job `semgrep`, et pour la ligne `name:` du job `secrets`. Copiez la partie `sha256:…` affichée après le `@`, pas le nom qui la précède.
```

````solution
```yaml
default:
  image: node:24-bookworm-slim@sha256:<empreinte affichée pour node>

semgrep:
  image: semgrep/semgrep:1.184.0@sha256:<empreinte affichée pour semgrep>

secrets:
  image:
    name: zricethezav/gitleaks:v8.30.1@sha256:<empreinte affichée pour gitleaks>
    entrypoint: [""]
```
````

```verify
F=~/byte-layers/quality/todo-api/.gitlab-ci.yml
grep -q "gitleaks git" "$F" && grep -q "npm audit --omit=dev" "$F" || { echo "Les jobs audit et secrets manquent dans .gitlab-ci.yml : lancez la commande de la section 01.1."; exit 1; }
L=$(grep -E 'node:24|semgrep/semgrep:|zricethezav/gitleaks:' "$F" | grep -v '@sha256:[0-9a-f]\{64\}')
[ -z "$L" ] || { echo "Ces lignes n'ont pas d'empreinte complète (@sha256: suivi de 64 caractères) :"; echo "$L"; exit 1; }
exit 0
```

## 01.4 - Passer par une merge request

Ce changement du pipeline passe lui-même par une merge request : c'est la bonne pratique, et c'est obligatoire si vous avez protégé `main` comme le proposait la fin du lab précédent. Avant d'envoyer, essayez l'audit sur votre poste : il doit réussir, puisque `main` n'a aucune dépendance de production.

```exec
cd ~/byte-layers/quality/todo-api && npm audit --omit=dev --audit-level=high && git switch -c security/controls && git add .gitlab-ci.yml && git commit -m "Ajoute l'audit des dépendances, la détection de secrets, et épingle les images" && git push -u gitlab security/controls -o merge_request.create -o merge_request.target=main -o merge_request.title="Ajoute les contrôles de sécurité"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `npm audit --omit=dev --audit-level=high` | le même contrôle que le job, sur votre poste : `found 0 vulnerabilities` |
> | `git switch -c security/controls` | crée une branche pour ce changement, à partir de `main` |
> | `git add`, `git commit` | enregistre le pipeline modifié |
> | `git push -u gitlab security/controls -o …` | l'envoie et ouvre la merge request, avec les options d'envoi du lab précédent |

Ouvrez la merge request : son pipeline a maintenant six jobs, tous verts au bout de trois minutes environ. Le journal du job `secrets` se termine par `no leaks found`. Fusionnez la merge request : un pipeline tourne ensuite sur `main`.

> 📖 [npm audit (en anglais)](https://docs.npmjs.com/cli/commands/npm-audit) · [gitleaks (en anglais)](https://github.com/gitleaks/gitleaks) · [GitLab : choisir l'image d'un job (en anglais)](https://docs.gitlab.com/ci/yaml/#image) · [Docker : les empreintes d'images (en anglais)](https://docs.docker.com/reference/cli/docker/image/pull/#pull-an-image-by-digest-immutable-identifier)

```verify
C=$(cat ~/byte-layers/quality/gitlab-url 2>/dev/null)
[ -n "$C" ] || { echo "L'adresse du projet GitLab n'est pas enregistrée dans ~/byte-layers/quality/gitlab-url : elle vient du lab Une barrière qualité sur les merge requests (section 01.1)."; exit 1; }
echo "$C" | grep -q "votre-nom" && { echo "Le fichier ~/byte-layers/quality/gitlab-url contient encore l'adresse d'exemple : remplacez votre-nom par votre nom d'utilisateur GitLab."; exit 1; }
case "$C" in
  git@*) H="https://$(echo "$C" | sed -E 's#git@([^:]+):.*#\1#')"; P=$(echo "$C" | sed -E 's#git@[^:]+:##') ;;
  *) H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#'); P=$(echo "$C" | sed -E 's#^https?://[^/]+/##') ;;
esac
A="$H/api/v4/projects/$(echo "$P" | sed -E 's#\.git$##; s#/#%2F#g')"
Y=$(curl -fsS "$A/repository/files/.gitlab-ci.yml/raw?ref=main" 2>/dev/null)
echo "$Y" | grep -q "gitleaks git" && echo "$Y" | grep -q "npm audit" || { echo "Le pipeline de main sur gitlab.com n'a pas encore les jobs audit et secrets : lancez la commande de la section 01.4."; exit 1; }
echo "$Y" | grep -E 'node:24|semgrep/semgrep:|zricethezav/gitleaks:' | grep -qv '@sha256:' && { echo "Une image du pipeline publié n'est pas épinglée par empreinte."; exit 1; }
S=$(curl -fsS "$A/pipelines?ref=main&per_page=1" 2>/dev/null | grep -o '"status":"[a-z_]*"' | head -1)
[ "$S" = '"status":"success"' ] || { echo "Le dernier pipeline de main n'a pas réussi ($S) : attendez qu'il se termine, ou lisez le journal du job en échec."; exit 1; }
exit 0
```
