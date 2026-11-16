---
title: "02 - Planter une dépendance vulnérable et un secret"
description: "Proposer une merge request qui ajoute une vieille version de lodash et un secret écrit en dur, et la voir arrêtée"
---

Une fonctionnalité anodine : prévenir un service extérieur quand une tâche change, par un appel HTTP signé d'un secret. Le développeur, pressé, ajoute une bibliothèque qu'il connaît dans une vieille version, et écrit le secret dans le code. Il ne s'en sert pas encore dans ce premier jet, mais la dépendance est déjà déclarée, donc installée partout. C'est exactement ce que les deux nouveaux jobs doivent arrêter.

## 02.1 - La merge request fautive

Cette commande crée la branche, ajoute la dépendance et le fichier, puis envoie le tout avec une merge request. Le secret est **faux**, inventé pour ce lab :

```exec
cd ~/byte-layers/quality/todo-api && git switch main && git pull gitlab main && git switch -c security/planted && npm install --save-exact lodash@4.17.20 && cat > src/notify.js <<'FIN'
const WEBHOOK_URL = 'https://hooks.example.com/todo-api';
const WEBHOOK_SECRET = 'nt_7Fq9Lm2Xv8Rt4Kp1Zc6Bn3Hd5Wj0Ys';

export async function notify(message) {
  await fetch(WEBHOOK_URL, {
    method: 'POST',
    headers: { 'X-Webhook-Secret': WEBHOOK_SECRET },
    body: JSON.stringify({ message }),
  });
}
FIN
git add -A && git commit -m "Ajoute les notifications" && git push -u gitlab security/planted -o merge_request.create -o merge_request.target=main -o merge_request.title="Ajoute les notifications"
```

> 🔍 **Comprendre la commande**
>
> | Morceau | Rôle |
> |---|---|
> | `git switch main && git pull gitlab main` | revient sur `main`, et y récupère la fusion de l'étape précédente |
> | `git switch -c security/planted` | crée la branche à partir de `main`, et s'y place |
> | `npm install --save-exact lodash@4.17.20` | ajoute lodash, une bibliothèque d'utilitaires très répandue, dans une version de 2020 ; `--save-exact` note la version exacte dans `package.json`, sans `^` |
> | `cat > src/notify.js <<'FIN'` … `FIN` | écrit le module de notification, avec son secret en dur |
> | `git add -A && git commit -m "…"` | enregistre tous les changements dans un commit |
> | `git push -u gitlab security/planted -o …` | envoie la branche et ouvre la merge request, avec les options d'envoi du lab précédent |

Ouvrez la merge request. Au bout de trois minutes :

| Job | Verdict | Pourquoi |
|---|---|---|
| `tests`, `eslint`, `duplication` | ✅ | le code est simple, et rien n'est copié |
| `audit` | ❌ | lodash 4.17.20 a une faille grave connue : le journal affiche `lodash  <=4.17.23`, `Severity: high` |
| `secrets` | ❌ | gitleaks trouve le secret dans `src/notify.js`, avec la règle `generic-api-key`, et masque sa valeur : `REDACTED` |
| `semgrep` | ❌ | votre règle `hardcoded-token` reconnaît une constante nommée `…SECRET` |

Le secret est arrêté deux fois, par deux outils différents : Semgrep lit le code actuel, gitleaks lit l'historique. Cette différence compte à l'étape suivante.

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
S=$(curl -fsS "$A/merge_requests/$M/pipelines" 2>/dev/null | grep -o '"status":"[a-z_]*"' | sort -u | tr '\n' ' ')
echo "$S" | grep -q '"status":"failed"' || { echo "Aucun pipeline de la merge request n'a échoué ($S) : attendez la fin du pipeline, ou vérifiez que les jobs audit et secrets sont bien dans .gitlab-ci.yml sur main."; exit 1; }
exit 0
```

> 📖 [npm install --save-exact (en anglais)](https://docs.npmjs.com/cli/commands/npm-install) · [gitleaks : les règles par défaut (en anglais)](https://github.com/gitleaks/gitleaks/blob/master/config/gitleaks.toml)
