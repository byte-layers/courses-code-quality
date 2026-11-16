---
title: "02 - Se connecter et créer un jeton"
description: "Changer le mot de passe de l'administrateur, et créer le jeton qui autorise les analyses"
---

L'outil d'analyse, le **scanner**, envoie ses résultats au serveur. Le serveur ne les accepte que d'un utilisateur authentifié : le scanner présente un **jeton**, une longue chaîne secrète qui remplace le mot de passe.

## 02.1 - La première connexion

Ouvrez [http://localhost:9000](http://localhost:9000) dans votre navigateur. Connectez-vous avec l'identifiant `admin` et le mot de passe `admin`, ceux de toute installation neuve. SonarQube vous oblige aussitôt à choisir un nouveau mot de passe : choisissez-en un d'au moins 12 caractères, avec majuscules, minuscules, chiffres et caractères spéciaux, sinon SonarQube le refuse, et notez-le. Ce SonarQube ne tourne que sur votre poste, mais un mot de passe connu de tous reste une mauvaise habitude.

## 02.2 - Créer le jeton

Cliquez sur l'initiale de votre compte, en haut à droite, puis sur **My Account**, et sur l'onglet **Security**. Dans **Generate Tokens** :

| Champ | Valeur |
|---|---|
| **Name** | `todo-api` |
| **Type** | **Global Analysis Token** : il autorise l'analyse de tous les projets |
| **Expires in** | **30 days** |

Cliquez sur **Generate**. SonarQube affiche le jeton, qui commence par `sqa_` : **copiez-le maintenant**, il ne sera plus jamais affiché.

## 02.3 - Ranger le jeton

On range le jeton dans un fichier, lisible par vous seul, plutôt que de le recopier dans chaque commande. Lancez cette commande, puis collez le jeton quand elle le demande, et appuyez sur Entrée ; rien ne s'affiche pendant la saisie, c'est normal. Dans Git Bash, collez avec Maj+Inser ou un clic droit : Ctrl+V n'y colle pas.

```exec
cd ~/byte-layers/quality && printf 'Jeton : '; stty -echo 2>/dev/null; read -r T; stty echo 2>/dev/null; printf '%s' "$T" > sonar-token && chmod 600 sonar-token && echo && echo "Jeton enregistré."
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `printf 'Jeton : '` | affiche l'invite |
> | `stty -echo` … `stty echo` | coupe l'affichage de ce que vous tapez, puis le rétablit |
> | `read -r T` | lit la ligne collée, et la garde dans la variable `T` |
> | `printf '%s' "$T" > sonar-token` | écrit le jeton, sans retour à la ligne, dans le fichier `sonar-token` |
> | `chmod 600 sonar-token` | rend le fichier lisible et modifiable par vous seul |
> | `echo` | passe à la ligne, puisque la saisie masquée n'en a pas ajouté |

Ce fichier est en dehors du projet `todo-api` : il ne risque pas de partir dans Git.

```verify
F=~/byte-layers/quality/sonar-token
[ -s "$F" ] || { echo "Le fichier ~/byte-layers/quality/sonar-token est absent ou vide : lancez la commande de la section 02.3."; exit 1; }
curl -s --max-time 10 -u "$(cat "$F"):" http://localhost:9000/api/authentication/validate 2>/dev/null | grep -q '"valid":true' || { echo "SonarQube refuse le jeton enregistré : recopiez-le sans espace (section 02.3), ou créez-en un nouveau (section 02.2)."; exit 1; }
exit 0
```
