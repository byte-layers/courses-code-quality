---
title: "04 - Deux merge requests, deux verdicts"
description: "Proposer le code de l'agent et le voir arrêté, puis proposer votre version et la fusionner"
---

La barrière est en place. On la met à l'épreuve avec les deux versions de l'export : celle de l'agent, puis la vôtre.

## 04.1 - Proposer le code de l'agent

Une merge request lance le pipeline décrit dans **sa propre branche**. Or la branche `quality/agent-export` a été créée avant le pipeline : elle n'a pas de `.gitlab-ci.yml`. On crée donc une branche `agent-mr` à partir du code de l'agent, on y fusionne `main` pour récupérer les outils et le pipeline, puis on l'envoie en demandant à GitLab d'ouvrir la merge request :

```exec
cd ~/byte-layers/quality/todo-api && git switch -c agent-mr quality/agent-export && git merge --no-edit main && git push -u gitlab agent-mr -o merge_request.create -o merge_request.target=main -o merge_request.title="Ajoute l'export des tâches"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch -c agent-mr quality/agent-export` | crée la branche `agent-mr` à partir du code de l'agent, et s'y place |
> | `git merge --no-edit main` | y fusionne `main` (outils, configuration, pipeline) avec le message de commit par défaut ; les deux branches ne touchent pas les mêmes fichiers, donc pas de conflit |
> | `git push -u gitlab agent-mr` | envoie la branche sur gitlab.com, et la relie à sa copie distante |
> | `-o merge_request.create` | une **option d'envoi** de GitLab : ouvre une merge request pour cette branche |
> | `-o merge_request.target=main` | la merge request vise `main` |
> | `-o merge_request.title="…"` | son titre |

Git affiche l'adresse de la merge request créée : ouvrez-la. Au bout de deux à trois minutes, le pipeline se termine :

| Job | Verdict | Pourquoi |
|---|---|---|
| `tests` | ✅ réussi | les 8 tests de l'agent passent, dont son test complaisant |
| `eslint` | ❌ échoué | `==`, `formatDate` jamais appelée, complexité cognitive de 91 |
| `duplication` | ❌ échoué | environ 11 % de code dupliqué, au-dessus de 5 % |
| `semgrep` | ❌ échoué | le jeton écrit en dur, et le jeton lu dans l'adresse |

La merge request affiche que la fusion est bloquée tant que le pipeline n'a pas réussi : le bouton **Merge** est indisponible. S'il est disponible, la case **Pipelines must succeed** de la section 03.3 n'est pas cochée : cochez-la. Ouvrez ensuite l'onglet **Reports**, puis **Code quality** (selon la version de GitLab, ce rapport peut aussi apparaître dans un bloc **Code Quality** de l'onglet **Overview**) : GitLab y liste les défauts trouvés par ESLint, chacun avec son fichier, sa ligne et sa gravité. Les journaux des jobs `duplication` et `semgrep` donnent le détail de leurs propres défauts : cliquez sur le job dans l'onglet **Pipelines** de la merge request.

```verify
C=$(cat ~/byte-layers/quality/gitlab-url 2>/dev/null)
[ -n "$C" ] || { echo "L'adresse du projet GitLab n'est pas enregistrée dans ~/byte-layers/quality/gitlab-url : lancez la commande de la section 01.1."; exit 1; }
case "$C" in
  git@*) H="https://$(echo "$C" | sed -E 's#git@([^:]+):.*#\1#')"; P=$(echo "$C" | sed -E 's#git@[^:]+:##') ;;
  *) H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#'); P=$(echo "$C" | sed -E 's#^https?://[^/]+/##') ;;
esac
A="$H/api/v4/projects/$(echo "$P" | sed -E 's#\.git$##; s#/#%2F#g')"
M=$(curl -fsS "$A/merge_requests?source_branch=agent-mr&state=opened" 2>/dev/null | grep -o '"iid":[0-9]*' | head -1 | cut -d: -f2)
[ -n "$M" ] || { echo "Aucune merge request ouverte depuis la branche agent-mr : lancez la commande de la section 04.1."; exit 1; }
S=$(curl -fsS "$A/merge_requests/$M" 2>/dev/null | grep -o '"head_pipeline":{[^}]*' | grep -o '"status":"[a-z_]*"' | head -1)
case "$S" in
  '"status":"failed"') exit 0 ;;
  '"status":"success"') echo "Le pipeline de la merge request a réussi : la barrière laisse passer le code de l'agent. Vérifiez les commandes des jobs, et l'option --error de Semgrep."; exit 1 ;;
  '') echo "La merge request n'a pas de pipeline : la branche agent-mr contient-elle .gitlab-ci.yml ? Relancez git merge --no-edit main sur agent-mr, puis git push."; exit 1 ;;
  *) echo "Le pipeline de la merge request n'est pas terminé ($S) : attendez quelques minutes."; exit 1 ;;
esac
```

## 04.2 - Proposer votre version, et la fusionner

Même démarche pour votre branche `clean-export` : on y fusionne `main`, puis on l'envoie avec une merge request.

```exec
cd ~/byte-layers/quality/todo-api && git switch clean-export && git merge --no-edit main && git push -u gitlab clean-export -o merge_request.create -o merge_request.target=main -o merge_request.title="Ajoute l'export des tâches, version relue"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch clean-export` | se place sur votre branche corrigée |
> | `git merge --no-edit main` | y fusionne les outils et le pipeline de `main` |
> | `git push -u gitlab clean-export -o …` | envoie la branche et ouvre la merge request, avec les mêmes options d'envoi qu'à la section 04.1 |

Ouvrez la merge request : les quatre jobs sont verts, avec 11 tests réussis. L'onglet **Reports**, **Code quality**, n'annonce aucun nouveau défaut. Avant de fusionner, décochez **Delete source branch** si la case est cochée : la branche `clean-export` reste ainsi sur GitLab. Cliquez sur **Merge** : votre export rejoint `main`, et un dernier pipeline tourne sur `main`.

Si un job échoue, votre version diffère de la solution du lab précédent : lisez le journal du job, corrigez sur `clean-export`, commitez, puis `git push` ; le pipeline de la merge request repart tout seul.

```verify
C=$(cat ~/byte-layers/quality/gitlab-url 2>/dev/null)
[ -n "$C" ] || { echo "L'adresse du projet GitLab n'est pas enregistrée dans ~/byte-layers/quality/gitlab-url : lancez la commande de la section 01.1."; exit 1; }
case "$C" in
  git@*) H="https://$(echo "$C" | sed -E 's#git@([^:]+):.*#\1#')"; P=$(echo "$C" | sed -E 's#git@[^:]+:##') ;;
  *) H=$(echo "$C" | sed -E 's#^(https?://)([^/@]*@)?([^/]+)/.*#\1\3#'); P=$(echo "$C" | sed -E 's#^https?://[^/]+/##') ;;
esac
A="$H/api/v4/projects/$(echo "$P" | sed -E 's#\.git$##; s#/#%2F#g')"
curl -fsS "$A/merge_requests?source_branch=clean-export&state=merged" 2>/dev/null | grep -q '"iid"' || { echo "La merge request de clean-export n'est pas fusionnée : attendez que son pipeline réussisse, puis cliquez sur Merge."; exit 1; }
curl -fsS "$A/repository/files/src%2Fserver.js/raw?ref=main" 2>/dev/null | grep -q "todos/export" || { echo "La branche main du projet GitLab n'a pas encore la route d'export."; exit 1; }
S=$(curl -fsS "$A/pipelines?ref=main&per_page=1" 2>/dev/null | grep -o '"status":"[a-z_]*"' | head -1)
[ "$S" = '"status":"success"' ] || { echo "Le dernier pipeline de main n'a pas réussi ($S) : attendez qu'il se termine."; exit 1; }
exit 0
```

> 📖 [GitLab : les options d'envoi de Git (en anglais)](https://docs.gitlab.com/topics/git/commit/#push-options) · [GitLab : les rapports d'une merge request (en anglais)](https://docs.gitlab.com/user/project/merge_requests/reports/)
