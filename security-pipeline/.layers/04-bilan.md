---
title: "04 - Ce que le pipeline garde, et ce qu'il laisse passer"
description: "Faire le point sur les quatre portes, les limites de chaque contrôle, et les gestes qui les complètent"
---

Votre pipeline garde maintenant les quatre portes du bit [SAST, SCA, secrets, épinglage](https://byte-layers.com/fr/bits/sast-sca-secrets). Chaque contrôle a ses angles morts.

## 04.1 - Les quatre portes, et leurs limites

| Porte | Contrôle dans le pipeline | Ce qu'il laisse passer | Ce qui le complète |
|---|---|---|---|
| votre code | Semgrep, vos règles | ce qu'aucune règle ne décrit | la relecture, les tests |
| vos dépendances | `npm audit` | un paquet piégé tout neuf, pas encore signalé ; les failles moyennes, et celles des outils de développement | `npm ci`, `--ignore-scripts`, Renovate pour les mises à jour |
| vos secrets | gitleaks | un secret qui ne ressemble à aucune règle (peu d'entropie, nom anodin) | `.env` ignoré par Git, variables masquées, gitleaks **sur le poste** avant l'envoi |
| vos outils de build | images épinglées | une image déjà piégée au moment où vous l'avez épinglée ; les correctifs, qui n'arrivent plus seuls | des images officielles, mises à jour par Renovate |

Deux réglages de ce lab sont des compromis assumés : `--omit=dev` ignore les failles des outils de développement, qui tournent pourtant dans le pipeline, et `--audit-level=high` laisse passer les failles moyennes. Une équipe les revoit avec le temps, comme les seuils de qualité : un cliquet.

## 04.2 - Garder les empreintes à jour

Une image épinglée ne reçoit plus les correctifs de sécurité. Renovate, un robot libre, lit le pipeline, repère les empreintes, et ouvre une merge request quand l'image a été republiée : le pipeline vérifie la nouvelle image, et un humain fusionne. Il fait de même pour les dépendances de `package.json`. Il s'installe sur gitlab.com comme une application, ou tourne dans un job planifié.

## 04.3 - La suite

Le challenge [Le paquet piégé](https://byte-layers.com/fr/challenges/typosquatting-challenge) vous met face à la porte la plus difficile : un `package.json` où l'un des paquets n'est pas celui qu'il prétend être, et qu'aucun audit ne signale.

```acknowledge
J'ai fait le point sur ce que chaque contrôle laisse passer, et sur les gestes qui le complètent.
```

## Récap

- `npm audit --omit=dev --audit-level=high` arrête une dépendance de production qui a une faille grave connue.
- **gitleaks** cherche les secrets dans **tout l'historique** de Git : supprimer le secret du fichier ne suffit pas.
- Un secret publié est **compromis** : on le révoque d'abord, puis on réécrit la branche (`git reset --soft`, un seul commit propre, `--force-with-lease`) pour qu'il ne parte pas dans `main` ; GitLab garde l'ancienne version de la merge request, d'où la révocation d'abord.
- Une image **épinglée** par empreinte ne change plus ; Renovate propose les mises à jour par merge request.
- Chaque contrôle a ses angles morts : ils se complètent, et complètent la relecture.

> 📖 [Renovate (en anglais)](https://docs.renovatebot.com/) · [GitLab : les variables masquées (en anglais)](https://docs.gitlab.com/ci/variables/#mask-a-cicd-variable) · [gitleaks : le hook pre-commit (en anglais)](https://github.com/gitleaks/gitleaks#pre-commit)
