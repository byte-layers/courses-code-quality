---
title: "05 - Tout mesurer d'une commande"
description: "Écrire un script qui passe les trois outils sur une version et rend un verdict, puis comparer ce que les outils trouvent et ce que la relecture avait trouvé"
---

Trois outils, trois commandes, trois versions : neuf lancements à chaque fois. Un petit script les regroupe, affiche un tableau lisible, et rend un **code de sortie** : `0` si tout passe, `1` sinon. C'est ce code qui permettra au lab suivant d'arrêter un pipeline.

## 05.1 - À vous d'écrire le script

Cette commande crée le script `measure.sh` avec trois `TODO`, et le rend exécutable :

```exec
cd ~/byte-layers/quality && cat > measure.sh <<'FIN'
#!/usr/bin/env bash
# Mesure chaque version demandée avec ESLint, jscpd et Semgrep.
# Exemple : ./measure.sh main agent clean
cd "$(dirname "$0")" || exit 1
export MSYS_NO_PATHCONV=1

semgrep_scan() {
  if command -v semgrep >/dev/null 2>&1; then semgrep "$@"
  else docker run --rm -v "$PWD:/src" semgrep/semgrep:1.184.0 semgrep "$@"; fi
}

verdict() {
  if "$@" >/dev/null 2>&1; then echo OK; else echo KO; fi
}

[ $# -gt 0 ] || set -- main agent clean
status=0
printf '%-8s %-8s %-8s %-8s\n' version eslint jscpd semgrep
for v in "$@"; do
  src="measure/$v/src"
  e=$(verdict TODO)
  j=$(verdict TODO)
  s=$(verdict TODO)
  printf '%-8s %-8s %-8s %-8s\n' "$v" "$e" "$j" "$s"
  [ "$e$j$s" = "OKOKOK" ] || status=1
done
exit $status
FIN
chmod +x measure.sh
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `cat > measure.sh <<'FIN'` … `FIN` | écrit le script ; les apostrophes autour de `FIN` empêchent le terminal de remplacer les `$` à l'écriture |
> | `cd "$(dirname "$0")"` | dans le script : se place dans le dossier du script, d'où qu'on le lance |
> | `semgrep_scan` | une fonction qui lance Semgrep installé, ou à défaut l'image Docker de l'étape 04 |
> | `verdict` | une fonction qui lance la commande reçue sans rien afficher, puis écrit `OK` si elle a réussi, `KO` sinon |
> | `[ $# -gt 0 ] \|\| set -- main agent clean` | sans argument, mesure les trois versions |
> | `$(verdict …)` | garde le mot `OK` ou `KO` dans une variable |
> | `printf '%-8s …'` | aligne les colonnes sur 8 caractères |
> | `[ "$e$j$s" = "OKOKOK" ] \|\| status=1` | colle les trois verdicts ; si ce n'est pas `OKOKOK`, un outil a échoué |
> | `exit $status` | le script échoue si un seul outil a échoué sur une seule version |
> | `chmod +x measure.sh` | rend le script exécutable |

Ouvrez `~/byte-layers/quality/measure.sh` dans votre éditeur, et remplacez chaque `TODO` par la commande de l'outil, appliquée au dossier `"$src"` :

| Ligne | Outil | Reprenez la commande de |
|---|---|---|
| `e=` | ESLint | la section 02.4, sans la boucle |
| `j=` | jscpd | la section 03.1, avec `--reporters silent` au lieu de `console` |
| `s=` | Semgrep | la section 04.3, avec la fonction `semgrep_scan` au lieu de `docker run … semgrep` |

```hint
La fonction `verdict` reçoit la commande en arguments : `e=$(verdict npx eslint "$src")`. Même forme pour les deux autres lignes, en commençant par `npx jscpd` et par `semgrep_scan scan`.
```

````solution
```bash
  e=$(verdict npx eslint "$src")
  j=$(verdict npx jscpd --min-tokens 30 --threshold 5 --reporters silent "$src")
  s=$(verdict semgrep_scan scan --config .semgrep/rules.yml --metrics=off --no-git-ignore --error --quiet "$src")
```
````

Lancez le script sans argument : il mesure les trois versions, puis affichez son code de sortie.

```exec
cd ~/byte-layers/quality && ./measure.sh; echo "code de sortie : $?"
```

Le tableau attendu, en une vingtaine de secondes (Semgrep démarre un conteneur à chaque version) :

```text
version  eslint   jscpd    semgrep
main     OK       OK       OK
agent    KO       KO       KO
clean    OK       OK       OK
code de sortie : 1
```

Le code de sortie vaut `1` parce que `agent` échoue. Lancé sur une seule version, `./measure.sh clean` rend `0`.

```verify
cd ~/byte-layers/quality 2>/dev/null || { echo "Le dossier ~/byte-layers/quality n'existe pas."; exit 1; }
[ -x measure.sh ] || { echo "Le script measure.sh n'existe pas ou n'est pas exécutable : lancez la commande de la section 05.1."; exit 1; }
command -v semgrep >/dev/null 2>&1 || docker info >/dev/null 2>&1 || { echo "Ni Semgrep ni Docker ne répondent : lancez Docker Desktop (ou installez Semgrep), puis réessayez."; exit 1; }
grep -q "verdict TODO" measure.sh && { echo "Une ligne de measure.sh contient encore TODO : complétez-la."; exit 1; }
OUT=$(./measure.sh main agent clean 2>&1)
echo "$OUT" | grep -qE '^main +OK +OK +OK' || { echo "$OUT"; echo "main devrait être OK pour les trois outils (tableau ci-dessus)."; exit 1; }
echo "$OUT" | grep -qE '^agent +KO +KO +KO' || { echo "$OUT"; echo "agent devrait être KO pour les trois outils (tableau ci-dessus) : une commande ne mesure pas le dossier \"\$src\", ou un outil n'est pas lancé."; exit 1; }
echo "$OUT" | grep -qE '^clean +OK +OK +OK' || { echo "$OUT"; echo "clean devrait être OK pour les trois outils (tableau ci-dessus)."; exit 1; }
./measure.sh clean >/dev/null 2>&1 || { echo "./measure.sh clean devrait rendre le code de sortie 0."; exit 1; }
./measure.sh agent >/dev/null 2>&1 && { echo "./measure.sh agent devrait rendre le code de sortie 1."; exit 1; }
exit 0
```

## 05.2 - Ce que les outils trouvent, et ce qu'ils ne trouvent pas

Reprenez votre relecture du lab précédent, `~/byte-layers/quality/review.md`, et comparez-la à ce que les outils ont signalé sur `agent` :

| Défaut relevé à la relecture | Trouvé par |
|---|---|
| `==` et `!=` | ESLint, `eqeqeq` |
| `formatDate` jamais appelée | ESLint, `no-unused-vars` |
| conditions imbriquées, fonction trop longue | ESLint, `sonarjs/cognitive-complexity` |
| l'échappement CSV copié trois fois | jscpd |
| le jeton écrit en dur | Semgrep, votre règle `hardcoded-token` |
| le jeton lu dans l'adresse | Semgrep, votre règle `secret-in-query-string` |
| les tâches de même titre retirées de l'export | **aucun outil** |
| `startsWith` qui accepte `/todos/exportation` | **aucun outil** |
| les noms `data`, `tmp`, `x` | **aucun outil** |
| l'ancienne version laissée en commentaire | **aucun outil** avec cette configuration |
| `EXPORT_TOKEN` absent de `.env.example` | **aucun outil** |

Les outils trouvent vite, à chaque fois, et sans fatigue, les défauts **de forme** : six sur onze. Ils ne savent pas ce que le code **doit** faire : le bug du dédoublonnage et celui de `startsWith` ne se voient qu'avec un humain qui comprend le besoin, ou un test qui l'exprime. Un outil ne remplace pas la relecture ; il la libère des défauts mécaniques, pour qu'elle se concentre sur le sens.

```acknowledge
J'ai comparé ce que les outils trouvent avec ce que ma relecture avait trouvé.
```

## 05.3 - Bonus : mesurer le code de vos propres agents

Si vous avez fait le lab [Spécifier ou improviser](https://byte-layers.com/fr/labs/spec-vs-vibe) du stack des agents de code, vos branches `export-vibe` et `export-spec` attendent dans `~/byte-layers/ai/todo-api`. Sortez-les à côté des trois autres, et mesurez-les :

```exec
cd ~/byte-layers/ai/todo-api && git worktree add --detach ~/byte-layers/quality/measure/vibe export-vibe && git worktree add --detach ~/byte-layers/quality/measure/spec export-spec && ~/byte-layers/quality/measure.sh vibe spec
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git worktree add --detach ~/byte-layers/quality/measure/vibe export-vibe` | sort la branche improvisée dans `measure/vibe`, comme à l'étape 01 |
> | la deuxième commande `git worktree add` | même chose pour la branche spécifiée, dans `measure/spec` |
> | `~/byte-layers/quality/measure.sh vibe spec` | mesure ces deux versions seulement |

Si Git répond `invalid reference`, la branche n'existe pas dans votre projet : sautez ce bonus. Sinon, le résultat dépend de ce que votre agent a écrit, et le code de sortie vaut `1` dès qu'une des deux versions échoue. Les règles Semgrep ne cherchent que les deux défauts de l'agent de ce lab : un `KO` vient plus souvent d'ESLint (complexité, `==`) ou de jscpd.

## Récap

- **ESLint** et **sonarjs** mesurent des erreurs fréquentes et la **complexité cognitive** ; **jscpd** mesure la **duplication** ; **Semgrep** applique des **règles propres au projet**, écrites en quelques lignes de YAML.
- Face à une dette existante, on fixe le seuil au niveau actuel, puis on l'abaisse : c'est le **cliquet**.
- Les ensembles de règles génériques ne connaissent pas votre projet : une règle maison attrape ce qu'ils laissent passer.
- Un script qui rend un code de sortie transforme des mesures en **verdict**, prêt pour un pipeline.
- Les outils trouvent les défauts de forme, pas les bugs de sens : ils complètent la relecture et les tests, ils ne les remplacent pas.

> 📖 [ESLint (en anglais)](https://eslint.org/docs/latest/) · [eslint-plugin-sonarjs (en anglais)](https://www.npmjs.com/package/eslint-plugin-sonarjs) · [jscpd (en anglais)](https://github.com/kucherenko/jscpd) · [Semgrep (en anglais)](https://semgrep.dev/docs/)
