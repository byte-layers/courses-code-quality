---
title: "04 - Semgrep et les règles du projet"
description: "Constater que les règles génériques ne voient pas tout, puis écrire deux règles propres au projet"
---

ESLint et jscpd mesurent des défauts généraux. Certains défauts sont propres à **votre** projet : ici, un jeton écrit en dur, ou lu dans l'adresse. **Semgrep** cherche dans le code des **motifs** qui ressemblent à du code, et chacun peut écrire ses propres règles en quelques lignes de YAML.

Semgrep est un programme Python. Le plus simple est de le lancer dans un conteneur Docker, avec l'image officielle `semgrep/semgrep`, sans rien installer.

## 04.1 - Les règles génériques

Semgrep publie des ensembles de règles prêtes à l'emploi. `p/javascript` en contient quelques dizaines pour JavaScript. Passez-le sur le code de l'agent :

```exec
cd ~/byte-layers/quality && docker run --rm -v "$PWD:/src" semgrep/semgrep:1.184.0 semgrep scan --config p/javascript --metrics=off --no-git-ignore measure/agent/src
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `docker run --rm` | lance un conteneur, et le supprime à la fin |
> | `-v "$PWD:/src"` | rend le dossier courant visible dans le conteneur, sous `/src`, le dossier où l'image lance Semgrep |
> | `semgrep/semgrep:1.184.0` | l'image officielle, dans une version fixée |
> | `semgrep scan --config p/javascript` | analyse avec l'ensemble de règles `p/javascript`, téléchargé depuis le registre de Semgrep |
> | `--metrics=off` | n'envoie pas de statistiques d'usage à Semgrep |
> | `--no-git-ignore` | n'interroge pas Git pour savoir quels fichiers ignorer : dans un worktree, `.git` est un fichier qui pointe vers un chemin de votre poste, que le conteneur ne voit pas |
> | `measure/agent/src` | le dossier analysé |

**Sous Windows (Git Bash)**, préfixez la commande par `MSYS_NO_PATHCONV=1 ` : sans cela, Git Bash transforme le chemin `/src` en chemin Windows. **Sans Docker**, avec Semgrep installé, la même analyse s'écrit `semgrep scan --config p/javascript --metrics=off --no-git-ignore measure/agent/src`.

Résultat : `Findings: 0`. Les règles génériques cherchent des failles connues (injection, fonctions dangereuses) ; elles ne savent pas que `EXPORT_TOKEN = 'exp_…'` est un secret, ni que ce projet interdit les jetons dans l'adresse.

## 04.2 - À vous d'écrire deux règles

Une règle Semgrep a un identifiant, un langage, une gravité, un message, et un **motif** : un morceau de code où un nom en majuscules précédé de `$`, comme `$NAME`, remplace n'importe quelle expression. Cette commande crée le dossier `.semgrep` et le fichier des règles du projet, avec deux `TODO` à compléter :

```exec
cd ~/byte-layers/quality && mkdir -p .semgrep && cat > .semgrep/rules.yml <<'FIN'
rules:
  - id: hardcoded-token
    languages: [javascript]
    severity: ERROR
    message: "Jeton écrit en dur dans le code : le lire depuis une variable d'environnement"
    patterns:
      - pattern: TODO
      - metavariable-regex:
          metavariable: $NAME
          regex: (?i).*(token|secret|password|api_?key).*
  - id: secret-in-query-string
    languages: [javascript]
    severity: WARNING
    message: "Secret lu dans l'URL : il finit dans les journaux du serveur et l'historique du navigateur"
    pattern: TODO
FIN
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `mkdir -p .semgrep` | crée le dossier `.semgrep`, sans erreur s'il existe déjà ; le point au début le rend caché dans le Finder et l'Explorateur |
> | `cat > .semgrep/rules.yml <<'FIN'` … `FIN` | écrit les lignes jusqu'à `FIN` dans le fichier, comme pour `eslint.config.js` à l'étape 02 |

Ouvrez ensuite `~/byte-layers/quality/.semgrep/rules.yml` dans votre éditeur, et remplacez chaque `TODO` par un motif :

| Règle | Ce qu'elle doit trouver | La forme du motif |
|---|---|---|
| `hardcoded-token` | une constante dont le **nom** évoque un secret et dont la **valeur** est un texte écrit en dur : `const EXPORT_TOKEN = 'exp_…';` | une déclaration `const`, avec `$NAME` pour le nom et `'$VALUE'` pour un texte quelconque ; la partie `metavariable-regex`, fournie, ne garde que les noms qui contiennent `token`, `secret`, `password` ou `api_key` |
| `secret-in-query-string` | la lecture d'un paramètre `token` dans l'adresse : `params.get('token')` | un appel de la méthode `get` avec le texte `'token'`, sur n'importe quel objet : `$PARAMS` |

```hint
Un motif Semgrep s'écrit comme le code qu'il cherche, en remplaçant les parties variables par des `$MAJUSCULES`. Pour la première règle : `const $NAME = '$VALUE';`. Pour la seconde : `$PARAMS.get('token')`.
```

````solution
```yaml
rules:
  - id: hardcoded-token
    languages: [javascript]
    severity: ERROR
    message: "Jeton écrit en dur dans le code : le lire depuis une variable d'environnement"
    patterns:
      - pattern: const $NAME = '$VALUE';
      - metavariable-regex:
          metavariable: $NAME
          regex: (?i).*(token|secret|password|api_?key).*
  - id: secret-in-query-string
    languages: [javascript]
    severity: WARNING
    message: "Secret lu dans l'URL : il finit dans les journaux du serveur et l'historique du navigateur"
    pattern: $PARAMS.get('token')
```
````

```verify
F=~/byte-layers/quality/.semgrep/rules.yml
[ -f "$F" ] || { echo "Le fichier ~/byte-layers/quality/.semgrep/rules.yml n'existe pas : lancez la commande de la section 04.2."; exit 1; }
grep -q "pattern: TODO" "$F" && { echo "Un motif vaut encore TODO dans .semgrep/rules.yml : complétez-le."; exit 1; }
for id in hardcoded-token secret-in-query-string; do grep -q "id: $id" "$F" || { echo "La règle $id manque dans .semgrep/rules.yml."; exit 1; }; done
exit 0
```

## 04.3 - Passer les règles sur les trois versions

Passez vos règles sur les trois versions, avec l'option `--error` qui transforme les résultats en verdict :

```exec
cd ~/byte-layers/quality && for v in main agent clean; do echo "== $v"; docker run --rm -v "$PWD:/src" semgrep/semgrep:1.184.0 semgrep scan --config .semgrep/rules.yml --metrics=off --no-git-ignore --error --quiet measure/$v/src; echo "code de sortie : $?"; done
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `--config .semgrep/rules.yml` | utilise vos règles au lieu d'un ensemble du registre |
> | `--error` | rend un code de sortie `1` dès qu'une règle trouve quelque chose : c'est ce qui permettra d'arrêter un pipeline |
> | `--quiet` | n'affiche que les résultats, sans le bandeau de progression |
> | le reste | comme à la section 04.1, dans une boucle sur les trois versions |

`agent` donne deux résultats, la constante de la ligne 7 et la lecture de la ligne 27, et le code de sortie 1. `main` et `clean` n'en donnent aucun, code de sortie 0. Si `clean` en donne un, votre version lit encore le jeton dans l'adresse, ou l'écrit en dur.

> 📖 [Semgrep : la syntaxe des règles (en anglais)](https://semgrep.dev/docs/writing-rules/rule-syntax) · [Semgrep : la syntaxe des motifs (en anglais)](https://semgrep.dev/docs/writing-rules/pattern-syntax) · [L'image Docker semgrep/semgrep (en anglais)](https://hub.docker.com/r/semgrep/semgrep)

```verify
cd ~/byte-layers/quality 2>/dev/null || { echo "Le dossier ~/byte-layers/quality n'existe pas."; exit 1; }
F=.semgrep/rules.yml
[ -f "$F" ] || { echo "Le fichier .semgrep/rules.yml n'existe pas : lancez la commande de la section 04.2."; exit 1; }
grep -q "pattern: TODO" "$F" && { echo "Un motif vaut encore TODO dans .semgrep/rules.yml : complétez-le."; exit 1; }
for id in hardcoded-token secret-in-query-string; do grep -q "id: $id" "$F" || { echo "La règle $id manque dans .semgrep/rules.yml."; exit 1; }; done
command -v semgrep >/dev/null 2>&1 || docker info >/dev/null 2>&1 || { echo "Ni Semgrep ni Docker ne répondent : lancez Docker Desktop (ou installez Semgrep), puis réessayez."; exit 1; }
export MSYS_NO_PATHCONV=1
sg() { if command -v semgrep >/dev/null 2>&1; then semgrep "$@"; else docker run --rm -v "$PWD:/src" semgrep/semgrep:1.184.0 semgrep "$@"; fi; }
N=$(sg scan --config "$F" --metrics=off --no-git-ignore --json measure/agent/src 2>/dev/null | grep -o '"check_id": *"[^"]*"' | sort -u | wc -l | tr -d ' ')
[ "$N" -ge 2 ] || { echo "Vos règles trouvent $N sorte(s) de problème dans measure/agent au lieu de 2 : vérifiez les deux motifs (voir l'indice)."; exit 1; }
sg scan --config "$F" --metrics=off --no-git-ignore --error --quiet measure/main/src >/dev/null 2>&1 || { echo "Vos règles trouvent un problème dans measure/main, qui n'a pas d'export : un motif est trop large."; exit 1; }
sg scan --config "$F" --metrics=off --no-git-ignore --error --quiet measure/clean/src >/dev/null 2>&1 || { echo "Vos règles trouvent un problème dans measure/clean : votre version garde un jeton en dur ou lu dans l'adresse, ou un motif est trop large."; exit 1; }
exit 0
```
