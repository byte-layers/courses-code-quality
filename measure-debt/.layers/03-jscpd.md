---
title: "03 - jscpd et la duplication"
description: "Détecter le code copié-collé, et fixer un seuil de duplication"
---

La duplication est la dette la plus facile à mesurer : un bloc copié trois fois devra être corrigé trois fois. **jscpd** (*copy/paste detector*) compare le code par **jetons**, les plus petits morceaux qu'il reconnaît : un mot-clé, un nom, un opérateur, une parenthèse. Deux suites de jetons identiques assez longues forment un **clone**, même si les espaces ou les retours à la ligne diffèrent.

## 03.1 - Mesurer les trois versions

jscpd est déjà installé depuis l'étape précédente. Lancez-le sur chaque version :

```exec
cd ~/byte-layers/quality && for v in main agent clean; do echo "== $v"; npx jscpd --min-tokens 30 --threshold 5 --reporters console measure/$v/src; echo "code de sortie : $?"; done
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `for v in main agent clean; do … done` | répète la mesure pour chaque version |
> | `--min-tokens 30` | un clone doit compter au moins 30 jetons identiques : en dessous, on trouverait des répétitions banales, comme deux `return res.end(…)` |
> | `--threshold 5` | échoue si plus de 5 % du code est dupliqué |
> | `--reporters console` | affiche le résultat dans le terminal |
> | `measure/$v/src` | le dossier analysé |
> | `echo "code de sortie : $?"` | affiche le code de retour de jscpd : `0` si le seuil est respecté, `1` sinon |

| Version | Ce que trouve jscpd |
|---|---|
| `main` | 0 clone, code de sortie 0 |
| `agent` | 2 clones dans `server.js` : les trois blocs qui échappent un titre pour le CSV ; environ 11 % de duplication, au-dessus du seuil ; code de sortie 1 |
| `clean` | 0 clone, code de sortie 0 |

jscpd a retrouvé le défaut DRY de votre relecture, sans rien savoir du CSV.

## 03.2 - Le choix des deux réglages

Les deux nombres sont des choix d'équipe, pas des vérités :

| Réglage | Trop bas | Trop haut |
|---|---|---|
| `--min-tokens` | des centaines de « clones » sans intérêt : deux appels de fonction qui se ressemblent | seuls les gros copier-coller sont vus |
| `--threshold` | le moindre clone fait échouer la mesure | un projet très dupliqué passe |

Sur un petit projet comme celui-ci, quelques clones suffisent à dépasser 5 %. Sur un gros projet, on part souvent du niveau actuel, comme pour la complexité, puis on abaisse.

**Essayez** : relancez la commande de la section 03.1 en remplaçant `30` par `15`. Sur ce petit projet, `main` et `clean` n'ont toujours aucun clone ; sur un projet plus grand, des dizaines de petites répétitions apparaîtraient. Les étapes suivantes gardent 30.

> 📖 [jscpd (en anglais)](https://github.com/kucherenko/jscpd)

```verify
cd ~/byte-layers/quality 2>/dev/null || { echo "Le dossier ~/byte-layers/quality n'existe pas."; exit 1; }
[ -f node_modules/.bin/jscpd ] || { echo "jscpd n'est pas installé : lancez la commande de la section 02.1 de l'étape précédente."; exit 1; }
npx jscpd --min-tokens 30 --threshold 5 --reporters silent measure/agent/src >/dev/null 2>&1 && { echo "jscpd ne trouve pas la duplication du code de l'agent : vérifiez que measure/agent contient bien la branche quality/agent-export."; exit 1; }
npx jscpd --min-tokens 30 --threshold 5 --reporters silent measure/clean/src >/dev/null 2>&1 || { echo "jscpd trouve plus de 5 % de duplication dans measure/clean : votre version garde des blocs copiés, regroupez-les dans une fonction."; exit 1; }
exit 0
```
