---
title: "01 - Publier le projet sur gitlab.com"
description: "Créer un projet vide, et y envoyer les branches main, quality/agent-export et clean-export"
---

Le projet `todo-api` vient de GitHub. Pour que GitLab CI le contrôle, il faut une copie sur gitlab.com. Toutes les commandes de cette étape sont fournies : elle prépare le terrain.

## 01.1 - Créer le projet

Sur gitlab.com, cliquez sur **New project**, puis **Create blank project**. Nommez-le `todo-api-quality`, choisissez la visibilité **Public**, et **décochez** *Initialize repository with a README* : le projet doit être vide, puisque vous allez y envoyer un historique existant. Cliquez sur **Create project**.

Sur la page du projet, le bouton **Code** donne son adresse SSH, de la forme `git@gitlab.com:votre-nom/todo-api-quality.git`. Enregistrez-la dans un fichier : modifiez d'abord la commande ci-dessous pour remplacer l'adresse d'exemple par la vôtre, puis lancez-la. Les vérifications de ce lab liront ce fichier.

```bash
echo 'git@gitlab.com:votre-nom/todo-api-quality.git' > ~/byte-layers/quality/gitlab-url
```

## 01.2 - Envoyer les branches

Votre projet local a déjà un dépôt distant, `origin`, sur GitHub. On en ajoute un second, nommé `gitlab`, puis on y envoie les trois branches utiles :

```exec
cd ~/byte-layers/quality/todo-api && git remote add gitlab "$(cat ../gitlab-url)" && git push gitlab main quality/agent-export clean-export
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git remote add gitlab <adresse>` | déclare un second dépôt distant, nommé `gitlab` |
> | `"$(cat ../gitlab-url)"` | remplace ce morceau par le contenu du fichier `gitlab-url` : l'adresse enregistrée à la section précédente |
> | `git push gitlab main quality/agent-export clean-export` | envoie les trois branches sur gitlab.com |

Au premier envoi vers gitlab.com, SSH peut demander `Are you sure you want to continue connecting (yes/no/[fingerprint])?` : tapez `yes`. Si Git répond `remote gitlab already exists`, le dépôt distant est déjà déclaré d'un essai précédent : lancez seulement `git push gitlab main quality/agent-export clean-export` dans le projet. Si Git répond `Permission denied (publickey)`, votre clé SSH n'est pas enregistrée sur gitlab.com : ajoutez-la dans **Preferences**, **SSH Keys**.

Rechargez la page du projet : le menu des branches propose `main`, `quality/agent-export` et `clean-export`.

> 📖 [git remote (en anglais)](https://git-scm.com/docs/git-remote) · [GitLab : utiliser des clés SSH (en anglais)](https://docs.gitlab.com/user/ssh/)

```verify
C=$(cat ~/byte-layers/quality/gitlab-url 2>/dev/null)
[ -n "$C" ] || { echo "L'adresse du projet GitLab n'est pas enregistrée dans ~/byte-layers/quality/gitlab-url : lancez la commande de la section 01.1."; exit 1; }
echo "$C" | grep -q "votre-nom" && { echo "Le fichier ~/byte-layers/quality/gitlab-url contient encore l'adresse d'exemple : remplacez votre-nom par votre nom d'utilisateur GitLab."; exit 1; }
case "$C" in
  git@*) H="https://$(echo "$C" | sed -E 's#git@([^:]+):.*#\1#')"; P=$(echo "$C" | sed -E 's#git@[^:]+:##') ;;
  *) H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#'); P=$(echo "$C" | sed -E 's#^https?://[^/]+/##') ;;
esac
A="$H/api/v4/projects/$(echo "$P" | sed -E 's#\.git$##; s#/#%2F#g')"
for b in main quality%2Fagent-export clean-export; do
  curl -fsS "$A/repository/branches/$b" >/dev/null 2>&1 || { echo "La branche $(echo $b | sed 's#%2F#/#') n'est pas sur le projet GitLab, ou le projet n'est pas public."; exit 1; }
done
exit 0
```
