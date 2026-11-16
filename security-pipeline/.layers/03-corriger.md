---
title: "03 - Corriger, jusqu'à l'historique"
description: "Mettre à jour la dépendance, sortir le secret du code, puis réécrire la branche pour que le secret disparaisse de l'historique"
---

Deux défauts à corriger, de natures différentes : une dépendance se met à jour, un secret publié ne se « corrige » pas, il se **révoque**.

## 03.1 - À vous de corriger la dépendance et le code

Sur la branche `security/planted` :

1. Trouvez la version de lodash qui corrige la faille, et installez-la, toujours avec `--save-exact`. `npm audit` sur votre poste vous la donne : `fix available via npm audit fix --force`, suivi de `Will install lodash@…`. Ne lancez pas `npm audit fix --force`, qui peut changer d'autres dépendances sans prévenir : installez vous-même la version indiquée.
2. Dans `src/notify.js`, lisez le secret dans l'environnement, avec `process.env.WEBHOOK_SECRET`, comme le projet le fait déjà pour ses autres réglages.
3. Déclarez ce réglage, sans valeur réelle, dans `.env.example` : une ligne `WEBHOOK_SECRET=change-me`.

Puis vérifiez sur votre poste, et commitez :

```exec
cd ~/byte-layers/quality/todo-api && npm audit --omit=dev --audit-level=high && npm test && git add -A && git commit -m "Lit le secret dans l'environnement, met à jour lodash"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `npm audit --omit=dev --audit-level=high` | doit répondre `found 0 vulnerabilities` |
> | `npm test` | les tests passent toujours |
> | `git add -A && git commit -m "…"` | enregistre la correction dans un second commit |

```hint
La ligne `Will install lodash@…` de `npm audit` donne la version ; `npm view lodash version` donne aussi la dernière version publiée. Dans `src/notify.js`, la ligne du secret devient `const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET;`.
```

````solution
```bash
cd ~/byte-layers/quality/todo-api
npm install --save-exact lodash@4.18.1
sed -i.bak "s/^const WEBHOOK_SECRET = .*/const WEBHOOK_SECRET = process.env.WEBHOOK_SECRET;/" src/notify.js && rm src/notify.js.bak
echo 'WEBHOOK_SECRET=change-me' >> .env.example
```

lodash 4.18.1 est la dernière version au moment où ce lab est écrit ; une version plus récente convient aussi, tant que `npm audit` ne signale rien.
````

```verify
cd ~/byte-layers/quality/todo-api 2>/dev/null || { echo "Le dossier ~/byte-layers/quality/todo-api n'existe pas."; exit 1; }
grep -q "nt_7Fq9" src/notify.js 2>/dev/null && { echo "src/notify.js contient encore le secret écrit en dur."; exit 1; }
grep -q "process.env.WEBHOOK_SECRET" src/notify.js 2>/dev/null || { echo "src/notify.js doit lire le secret avec process.env.WEBHOOK_SECRET."; exit 1; }
grep -q "^WEBHOOK_SECRET=" .env.example || { echo "WEBHOOK_SECRET n'est pas déclaré dans .env.example."; exit 1; }
npm audit --omit=dev --audit-level=high >/dev/null 2>&1 || { echo "npm audit signale encore une faille grave : mettez lodash à jour."; exit 1; }
exit 0
```

## 03.2 - Le secret est encore là

Envoyez la correction :

```exec
cd ~/byte-layers/quality/todo-api && git push gitlab security/planted
```

Le nouveau pipeline de la merge request rend `audit` et `semgrep` verts. Mais `secrets` reste rouge : gitleaks trouve toujours le secret, dans le **premier commit** de la branche. Le fichier ne le contient plus, l'historique si. Et cet historique est public : n'importe qui peut lire le premier commit.

Dans un vrai projet, la règle est absolue : un secret publié est **compromis**. La première chose à faire est de le **révoquer** auprès du service qui l'a émis, et d'en créer un nouveau, rangé dans une variable masquée du pipeline ou dans le `.env` du serveur. Ici, le secret est faux : il n'y a rien à révoquer, mais c'est la première étape à ne jamais oublier.

## 03.3 - Réécrire la branche

Une fois le secret révoqué, on le retire de l'historique de la branche, pour qu'il ne parte pas dans `main`. Tant que la branche n'est pas fusionnée, c'est simple : on remplace ses deux commits par un seul, propre, puis on remplace la branche sur gitlab.com.

```exec
cd ~/byte-layers/quality/todo-api && git fetch gitlab && git reset --soft "$(git merge-base gitlab/main HEAD)" && git commit -m "Ajoute les notifications" && git push --force-with-lease gitlab security/planted
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git fetch gitlab` | met à jour la copie locale des branches de gitlab.com |
> | `git merge-base gitlab/main HEAD` | trouve le commit de `main` d'où part la branche |
> | `git reset --soft "$(…)"` | ramène la branche à ce commit, en gardant tous les changements, prêts à être commités : les deux commits disparaissent, leur contenu final reste |
> | `git commit -m "…"` | enregistre ce contenu final en un seul commit, qui n'a jamais contenu le secret |
> | `git push --force-with-lease` | remplace la branche sur gitlab.com par la nouvelle, mais refuse si quelqu'un d'autre l'a modifiée entre-temps |

Le pipeline repart, et cette fois les six jobs sont verts. Fusionnez la merge request, **sans** cocher *Squash commits* : la branche n'a déjà plus qu'un commit.

Attention à ce que cette réécriture ne fait pas : GitLab garde les anciennes versions d'une merge request, visibles dans l'onglet **Changes**, et l'ancien commit reste lisible par son identifiant. Le secret ne partira pas dans `main`, mais il n'a pas disparu de gitlab.com : c'est pour cela que la révocation passe en premier.

Si la branche avait déjà été fusionnée, le secret serait dans l'historique de `main` : le retirer demanderait de réécrire `main` avec un outil comme `git filter-repo`, et de prévenir toute l'équipe. D'où l'intérêt d'arrêter le secret dans la merge request, avant la fusion, et mieux encore sur le poste, avant l'envoi : gitleaks tourne aussi en local.

```verify
C=$(cat ~/byte-layers/quality/gitlab-url 2>/dev/null)
[ -n "$C" ] || { echo "L'adresse du projet GitLab n'est pas enregistrée dans ~/byte-layers/quality/gitlab-url : elle vient du lab Une barrière qualité sur les merge requests (section 01.1)."; exit 1; }
echo "$C" | grep -q "votre-nom" && { echo "Le fichier ~/byte-layers/quality/gitlab-url contient encore l'adresse d'exemple : remplacez votre-nom par votre nom d'utilisateur GitLab."; exit 1; }
case "$C" in
  git@*) H="https://$(echo "$C" | sed -E 's#git@([^:]+):.*#\1#')"; P=$(echo "$C" | sed -E 's#git@[^:]+:##') ;;
  *) H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#'); P=$(echo "$C" | sed -E 's#^https?://[^/]+/##') ;;
esac
A="$H/api/v4/projects/$(echo "$P" | sed -E 's#\.git$##; s#/#%2F#g')"
M=$(curl -fsS "$A/merge_requests?source_branch=security%2Fplanted&state=all" 2>/dev/null | grep -o '"iid":[0-9]*' | head -1 | cut -d: -f2)
[ -n "$M" ] || { echo "Aucune merge request depuis la branche security/planted : lancez la commande de la section 02.1."; exit 1; }
curl -fsS "$A/merge_requests/$M" 2>/dev/null | grep -q '"state":"merged"' || { echo "La merge request de security/planted n'est pas fusionnée : attendez que son pipeline réussisse, puis cliquez sur Merge."; exit 1; }
N=$(curl -fsS "$A/merge_requests/$M/commits" 2>/dev/null | grep -o '"short_id"' | wc -l | tr -d ' ')
[ "$N" = 1 ] || { echo "La merge request contient $N commits au lieu d'un seul : le premier commit, avec le secret, est peut-être resté dans l'historique (section 03.3). Si vous avez coché Squash commits à la fusion, GitLab en ajoute un : la branche devait déjà n'en avoir qu'un."; exit 1; }
curl -fsS "$A/repository/files/src%2Fnotify.js/raw?ref=main" 2>/dev/null | grep -q "process.env.WEBHOOK_SECRET" || { echo "src/notify.js sur main ne lit pas le secret dans l'environnement."; exit 1; }
S=$(curl -fsS "$A/pipelines?ref=main&per_page=1" 2>/dev/null | grep -o '"status":"[a-z_]*"' | head -1)
[ "$S" = '"status":"success"' ] || { echo "Le dernier pipeline de main n'a pas réussi ($S) : attendez qu'il se termine."; exit 1; }
exit 0
```

> 📖 [git reset (en anglais)](https://git-scm.com/docs/git-reset) · [git push --force-with-lease (en anglais)](https://git-scm.com/docs/git-push#Documentation/git-push.txt---force-with-leaseltrefnamegt) · [git filter-repo (en anglais)](https://github.com/newren/git-filter-repo)
