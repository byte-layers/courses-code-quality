---
title: "Le paquet piégé"
description: "Un défi en trois décisions : repérer le paquet piégé d'un package.json qu'aucun audit ne signale, choisir la première chose à faire après son installation, et l'option de npm qui l'aurait désamorcé. Vos décisions sont vérifiées."
tags:
  - challenge
  - quality
  - security
difficulty: beginner
duration: 20
introTitle: "Le défi"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Ce challenge prouve que vous savez reconnaître un paquet piégé, et réagir quand il a été installé. Il n'y a **aucune commande à recopier** : chaque tâche décrit une situation, et vous inscrivez votre décision dans un fichier. Un bouton vérifie ce fichier.

Si vous êtes bloqué, les **indices** se dévoilent un par un, et la **solution** reste en dernier recours. Les utiliser ne coûte rien : seule la vérification compte.

## Avant de commencer

- **Ce qu'il faut savoir** : ce qu'enseigne le bit [SAST, SCA, secrets, épinglage](https://byte-layers.com/fr/bits/sast-sca-secrets), en particulier la partie sur les dépendances.
- **Ce qu'il faut avoir** : un terminal (Git Bash sous Windows) et un éditeur de texte. Rien à installer : **n'installez aucun des paquets de ce challenge**, même pour vérifier.
- **Le fichier de décisions** : `~/byte-layers/quality/typo/decisions.txt` est créé pour vous à l'ouverture. Il contient une ligne `clé=` par décision, à compléter après le signe `=`. Les majuscules, les espaces et les accents sont ignorés. Les lignes qui commencent par `#` sont des commentaires. Supprimez le dossier `~/byte-layers/quality/typo` quand vous n'en avez plus besoin.

# Tâche 1 : repérer le paquet piégé

## Lire un package.json avec méfiance

```setup
D=~/byte-layers/quality/typo
if [ ! -f "$D/decisions.txt" ]; then
  mkdir -p "$D"
  cat > "$D/decisions.txt" <<'FIN'
# Tâche 1 : le nom exact du paquet piégé, puis le signal le plus grave
suspect=
signal=
# Tâche 2 : la première chose à faire
action=
# Tâche 3 : l'option de npm qui aurait désamorcé l'attaque, sans les deux tirets
option=
FIN
fi
```

Une merge request ajoute l'envoi de courriels à l'API de tâches. Son `package.json` :

```json
{
  "name": "todo-api",
  "type": "module",
  "dependencies": {
    "@acme/todo-shared": "1.2.0",
    "dotnev": "16.4.7",
    "express": "5.1.0",
    "helmet": "8.1.0",
    "nodemailer": "7.0.5",
    "pino": "9.7.0",
    "zod": "4.0.5"
  }
}
```

`npm audit` ne signale **rien**. Voici ce que dit le registre npm de chaque paquet. Ces données sont **inventées pour l'exercice**, et un paquet du même nom peut exister sur npm : ne l'installez pas.

| Paquet | Téléchargements par semaine | Première publication | Dépôt de code déclaré | Script d'installation |
|---|---|---|---|---|
| `@acme/todo-shared` | 140 | il y a 3 semaines | `gitlab.com/acme/todo-shared` | aucun |
| `dotnev` | 38 | il y a 6 jours | aucun | `postinstall: node setup.js` |
| `express` | 40 millions | 2010 | `github.com/expressjs/express` | aucun |
| `helmet` | 2 millions | 2012 | `github.com/helmetjs/helmet` | aucun |
| `nodemailer` | 5 millions | 2010 | `github.com/nodemailer/nodemailer` | aucun |
| `pino` | 9 millions | 2016 | `github.com/pinojs/pino` | aucun |
| `zod` | 30 millions | 2020 | `github.com/colinhacks/zod` | aucun |

Complétez dans `decisions.txt` :

- `suspect=` avec le nom exact du paquet piégé ;
- `signal=` avec le signal le plus grave, celui qui permet à ce paquet d'agir sans qu'on l'appelle jamais, parmi `telechargements`, `date`, `depot` et `script`.

**Critères :**

- le suspect est le paquet dont le nom imite un paquet très connu, et qui cumule les signaux ;
- le signal désigne ce qui s'exécute tout seul à l'installation.

```hint
Lisez chaque nom lettre par lettre, en pensant aux paquets que tout projet Node.js utilise pour lire son fichier `.env`.
```

```hint
Peu de téléchargements et une publication récente ne prouvent rien seuls : `@acme/todo-shared` est le paquet interne de l'entreprise. Ce qui compte, c'est ce que le paquet fait **à l'installation**, avant même que votre code l'importe.
```

```verify
F=~/byte-layers/quality/typo/decisions.txt
v() { sed -n "s/^$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '[:space:]"' | sed 's/é/e/g;s/è/e/g;s/ê/e/g;s/â/a/g;s/É/e/g;s/È/e/g'; }
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
case "$(v suspect)" in
  dotnev) ;;
  "") echo "La ligne suspect= est vide."; exit 1 ;;
  dotenv) echo "dotenv est le vrai paquet, celui que tout le monde utilise. Relisez les noms du package.json lettre par lettre."; exit 1 ;;
  @acme/todo-shared) echo "@acme/todo-shared a peu de téléchargements, mais c'est le paquet interne de l'entreprise : son dépôt est déclaré, et il n'a aucun script d'installation."; exit 1 ;;
  express|helmet|nodemailer|pino|zod) echo "Ce paquet est très utilisé depuis des années, avec un dépôt public et sans script d'installation."; exit 1 ;;
  *) echo "suspect=$(v suspect) ne correspond à aucun paquet du package.json. Recopiez son nom exact."; exit 1 ;;
esac
case "$(v signal)" in
  script) ;;
  "") echo "La ligne signal= est vide."; exit 1 ;;
  telechargements|date|depot) echo "Ce signal rend le paquet suspect, mais il ne lui permet pas d'agir. Lequel s'exécute tout seul, à l'installation ?"; exit 1 ;;
  *) echo "Le signal doit être telechargements, date, depot ou script."; exit 1 ;;
esac
echo "Le suspect et le signal sont les bons."
```

````solution
```text
suspect=dotnev
signal=script
```

`dotnev` imite `dotenv`, le paquet qui lit les fichiers `.env`, en inversant deux lettres : c'est le **typosquatting**, parier sur une faute de frappe. Il cumule les signaux : peu de téléchargements, publié il y a six jours, aucun dépôt de code, et surtout un script `postinstall`, qui s'exécute à l'installation avec les droits de l'utilisateur, avant même que le code l'importe. `npm audit` ne le signale pas : il n'est dans aucune base de failles connues.
````

# Tâche 2 : il a été installé

## La première chose à faire

La merge request a été fusionnée avant que quelqu'un remarque `dotnev`. Depuis hier, il a été installé par le pipeline, qui dispose d'un jeton de publication npm et d'un jeton de déploiement, et sur les postes de deux développeurs. Son script `setup.js`, lu après coup, envoie les variables d'environnement vers un serveur inconnu.

Complétez `action=` avec la première chose à faire, parmi :

| Valeur | Action |
|---|---|
| `supprimer` | retirer `dotnev` du `package.json` et republier |
| `audit` | lancer `npm audit` sur tous les projets |
| `revoquer` | révoquer et remplacer tous les secrets présents sur les machines qui l'ont installé : jetons du pipeline, jetons et clés des deux postes |
| `attendre` | attendre que npm retire le paquet de son registre |

**Critère :** l'action traite ce que le script a déjà pu emporter.

```hint
Le script a déjà tourné. Retirer le paquet empêche les prochaines installations, mais que deviennent les secrets qu'il a envoyés hier ?
```

```verify
F=~/byte-layers/quality/typo/decisions.txt
v() { sed -n "s/^$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '[:space:]"' | sed 's/é/e/g;s/è/e/g;s/ê/e/g;s/â/a/g;s/É/e/g;s/È/e/g'; }
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
case "$(v action)" in
  revoquer) ;;
  "") echo "La ligne action= est vide."; exit 1 ;;
  supprimer) echo "Retirer le paquet est nécessaire, mais il a déjà tourné : les secrets qu'il a envoyés restent utilisables par l'attaquant."; exit 1 ;;
  audit) echo "npm audit ne connaît pas ce paquet : il ne trouvera rien, et les secrets volés restent utilisables."; exit 1 ;;
  attendre) echo "Pendant l'attente, l'attaquant peut publier avec votre jeton npm, comme le ver Shai-Hulud."; exit 1 ;;
  *) echo "L'action doit être supprimer, audit, revoquer ou attendre."; exit 1 ;;
esac
echo "C'est la bonne première action."
```

````solution
```text
action=revoquer
```

Les secrets envoyés hier sont entre les mains de l'attaquant, que le paquet soit retiré ou non. Avec le jeton de publication npm, il pourrait publier une version piégée de vos propres paquets, comme l'a fait le ver Shai-Hulud en 2025. On révoque d'abord, puis on retire le paquet, on cherche ce que les jetons volés ont pu faire, et on prévient les utilisateurs si un paquet a été publié.
````

# Tâche 3 : la prévention

## L'option qui désamorce

Un script `postinstall` ne s'exécute que si npm l'y autorise. Une option de `npm install` et de `npm ci` empêche l'exécution de tous les scripts d'installation des dépendances.

Complétez `option=` avec le nom de cette option, **sans les deux tirets** du début.

**Critère :** l'option empêche les scripts des dépendances de s'exécuter à l'installation.

```hint
Le bit SAST, SCA, secrets, épinglage la cite à propos du ver Shai-Hulud. Son nom dit qu'elle ignore les scripts.
```

```verify
F=~/byte-layers/quality/typo/decisions.txt
v() { sed -n "s/^$1[[:space:]]*=//p" "$F" | tail -1 | tr 'A-Z' 'a-z' | tr -d '[:space:]"' | sed 's/^-*//;s/=true$//'; }
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
case "$(v option)" in
  ignore-scripts) ;;
  "") echo "La ligne option= est vide."; exit 1 ;;
  save-exact|omit=dev|production) echo "Cette option ne touche pas aux scripts d'installation."; exit 1 ;;
  *) echo "Ce n'est pas l'option qui empêche les scripts d'installation."; exit 1 ;;
esac
echo "C'est la bonne option."
```

````solution
```text
option=ignore-scripts
```

`npm ci --ignore-scripts` installe les dépendances sans lancer leurs scripts. Le réglage peut devenir permanent, pour un projet, avec une ligne `ignore-scripts=true` dans son fichier `.npmrc`. Quelques paquets ont vraiment besoin de leur script, par exemple pour compiler du code natif : on les traite à part, en connaissance de cause. Aucune option ne remplace la relecture du `package.json` dans chaque merge request.
````

## Récap

- Le **typosquatting** parie sur une faute de frappe : `dotnev` pour `dotenv`.
- Les signaux d'un paquet piégé se cumulent : peu de téléchargements, publication récente, pas de dépôt de code, et surtout un **script d'installation**.
- `npm audit` ne voit que les failles **connues** : un paquet piégé tout neuf lui échappe.
- Après l'installation d'un paquet piégé, la première chose à faire est de **révoquer les secrets** qu'il a pu lire.
- `--ignore-scripts` empêche les scripts d'installation de s'exécuter ; la relecture des dépendances dans chaque merge request reste nécessaire.

> 📖 [npm : les scripts d'installation (en anglais)](https://docs.npmjs.com/cli/using-npm/scripts) · [npm : l'option ignore-scripts (en anglais)](https://docs.npmjs.com/cli/using-npm/config#ignore-scripts) · [CISA, alerte sur le ver Shai-Hulud, 23 septembre 2025 (en anglais)](https://www.cisa.gov/news-events/alerts/2025/09/23/widespread-supply-chain-compromise-impacting-npm-ecosystem)
