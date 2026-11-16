---
title: "Knight Capital : 460 millions de dollars perdus en 45 minutes"
description: "Le 1er août 2012, un déploiement manuel oublie un serveur sur huit. Un vieux code jamais supprimé se réveille et envoie des millions d'ordres de bourse. À quelle phase est né le problème ?"
tags:
  - bit
  - quality
difficulty: beginner
duration: 10
requires: "0.4.0"
---

Le 1er août 2012, Knight Capital, l'un des plus gros intermédiaires de la bourse américaine, perd plus de 460 millions de dollars en trois quarts d'heure. Pas de piratage, pas de panne : un serveur sur huit n'a pas reçu la nouvelle version du code, et un ancien code, que personne n'utilisait plus depuis neuf ans, s'est remis à tourner. Ce bit suit l'ordonnance de la SEC, l'autorité des marchés américaine, et pose la question : **à quelle phase est né le problème ?**

**Pour qui** : tout développeur ; aucune connaissance de la bourse n'est nécessaire. Les phases utilisées sont présentées dans [Ariane 5 vol 501](https://byte-layers.com/fr/bits/ariane-5-failure).

> 📅 **Sources vérifiées le 16 novembre 2026.**

# Knight Capital : 460 millions de dollars perdus en 45 minutes

## 1 - Ce qui s'est passé

Knight fait tourner un routeur d'ordres, appelé SMARS, sur huit serveurs. Il reçoit des ordres des autres systèmes de Knight, les « ordres parents », et les découpe en petits « ordres enfants » envoyés à la bourse.

| Date | Événement |
|---|---|
| 2003 | Knight cesse d'utiliser une ancienne fonction de SMARS, appelée **Power Peg**. Le code reste sur les serveurs, toujours appelable |
| 2005 | un remaniement déplace une partie du code : Power Peg ne sait plus compter les ordres déjà exécutés. Personne ne le reteste, puisqu'il ne sert plus |
| à partir du 27 juillet 2012 | Knight déploie un nouveau code pour un programme de la bourse de New York. Ce code **réutilise un indicateur** qui, autrefois, activait Power Peg |
| pendant ce déploiement | un technicien ne copie pas le nouveau code sur l'un des huit serveurs ; ce huitième serveur garde l'ancien code |
| 1er août, avant l'ouverture | le système envoie 97 courriels automatiques signalant une erreur « Power Peg disabled » ; ils ne sont pas conçus comme des alertes, personne ne les lit |
| 1er août, à l'ouverture (9 h 30) | les ordres portant l'indicateur arrivent sur le huitième serveur et réveillent Power Peg, qui envoie des ordres enfants sans s'arrêter |
| pendant 45 minutes | plus de 4 millions d'exécutions, sur 154 actions, pour plus de 397 millions d'actions achetées ou vendues |

En cherchant la panne, l'équipe désinstalle le nouveau code des sept serveurs sains. Résultat : ces serveurs retrouvent l'indicateur avec l'ancien code, et Power Peg se réveille aussi chez eux. La correction aggrave l'incident.

Bilan : plus de 460 millions de dollars de pertes. En 2013, la SEC inflige 12 millions de dollars d'amende à Knight, pour des contrôles insuffisants de son accès au marché.

## 2 - La chaîne des causes

1. **Du code mort laissé en production.** Power Peg ne servait plus depuis 2003, mais il restait sur les serveurs, appelable.
2. **Un remaniement sans test.** En 2005, le code de Power Peg est cassé par un déplacement de code, et jamais retesté.
3. **Un indicateur réutilisé.** Le nouveau code donne un nouveau sens à un indicateur qui avait déjà un sens pour l'ancien code.
4. **Un déploiement manuel, sans double vérification.** Pas de procédure écrite, pas de second technicien pour vérifier : un serveur oublié passe inaperçu.
5. **Des signaux ignorés.** Les 97 courriels du matin annonçaient le problème, mais n'étaient pas des alertes.
6. **Pas de plan d'urgence.** Aucune procédure de réponse à incident ; l'équipe a cherché la panne pendant que les ordres partaient.

## 3 - À quelle phase est né le problème ?

| Phase | Ce qui s'y est passé |
|---|---|
| Codage | le code mort n'est pas supprimé ; un indicateur existant est réutilisé avec un nouveau sens |
| Test | Power Peg n'est plus testé après le remaniement de 2005 ; le nouveau code n'est pas testé face à l'ancien |
| **Déploiement** | le déclencheur : un déploiement à la main oublie un serveur, et personne ne le vérifie |
| Exploitation | les courriels d'erreur ne sont pas des alertes ; la réaction en urgence désinstalle le bon code |

Le défaut était dormant dans le **code** depuis des années ; le **déploiement** l'a déclenché. Les deux sont des réponses défendables à la question, et c'est l'intérêt du cas : sans code mort, l'oubli d'un serveur aurait seulement retardé une fonctionnalité ; avec un déploiement automatisé et vérifié, le code mort serait resté dormant.

## 4 - Ce que ça change pour vous

- **Supprimer le code mort.** Git garde l'historique : un code qui ne sert plus se supprime, il ne se commente pas, il ne se laisse pas « au cas où ».
- **Ne jamais réutiliser un indicateur.** Un nouveau besoin mérite un nouveau nom de paramètre ou d'option.
- **Automatiser le déploiement, et le vérifier.** Un script ou un pipeline déploie partout la même version, et un contrôle confirme la version sur chaque serveur.
- **Une alerte est faite pour être lue.** Un message d'erreur que personne ne lit n'est pas de la surveillance.
- **Préparer la réponse à incident.** Savoir couper un système, ou revenir à la version précédente, avant d'en avoir besoin.

## Récap

- Le 1er août 2012, Knight Capital perd plus de 460 millions de dollars en 45 minutes.
- Un serveur sur huit n'a pas reçu le nouveau code ; un indicateur réutilisé y a réveillé Power Peg, un code abandonné en 2003 et cassé en 2005.
- La tentative de correction a étendu le problème aux sept autres serveurs.
- Le défaut dormait dans le **code** (code mort, indicateur réutilisé) ; le **déploiement** manuel, sans vérification, l'a déclenché.

> 📖 [SEC, ordonnance « In the Matter of Knight Capital Americas LLC », Release No. 34-70694, octobre 2013 (PDF, en anglais)](https://www.sec.gov/litigation/admin/2013/34-70694.pdf)

```quiz
title: Knight Capital
tracksProgress: true
submitMode: last-only
questions:
  - prompt: Pourquoi le huitième serveur a-t-il réveillé Power Peg ?
    answers:
      - "Un pirate y avait installé Power Peg"
      - "Il n'avait pas reçu le nouveau code, et l'indicateur réutilisé y activait encore l'ancien Power Peg"
      - "Il était plus rapide que les autres"
      - "Il avait reçu une version de test"
    correct: 2
    explanation: |
      Le nouveau code donnait un nouveau sens à un ancien indicateur. Sur le serveur oublié,
      l'indicateur gardait son ancien sens : activer Power Peg.
  - prompt: Qu'a provoqué la désinstallation du nouveau code sur les sept serveurs sains ?
    answers:
      - "Elle a arrêté l'incident"
      - "Elle n'a rien changé"
      - "Elle a réveillé Power Peg sur ces serveurs aussi, et aggravé l'incident"
      - "Elle a remboursé les pertes"
    correct: 3
    explanation: |
      Revenir en arrière n'est sûr que si l'ancienne version l'est. Ici, l'ancien code
      contenait encore Power Peg.
  - prompt: Quelle phase a déclenché l'incident le 1er août ?
    answers:
      - "Les exigences"
      - "Les dépendances"
      - "La conception de l'interface"
      - "Le déploiement : une copie manuelle a oublié un serveur, sans vérification"
    correct: 4
    explanation: |
      Le déploiement manuel, sans procédure écrite ni second technicien, a laissé l'ancien
      code sur un serveur.
  - prompt: Où le défaut dormait-il depuis des années ?
    answers:
      - "Dans le code : Power Peg, abandonné, n'avait jamais été supprimé, et ne fonctionnait plus correctement depuis 2005"
      - "Dans la base de données des clients"
      - "Dans la configuration du réseau"
      - "Dans une bibliothèque open source"
    correct: 1
    explanation: |
      Un code mort laissé en production, cassé par un remaniement jamais retesté : une
      dette de code qui attendait son déclencheur.
  - prompt: Quelle pratique aurait le plus simplement évité l'incident ?
    answers:
      - "Ajouter des serveurs"
      - "Envoyer plus de courriels d'erreur"
      - "Interdire les nouvelles fonctionnalités"
      - "Supprimer le code mort de Power Peg, et déployer avec un outil qui vérifie la version sur chaque serveur"
    correct: 4
    explanation: |
      Sans code mort, pas de Power Peg à réveiller ; avec un déploiement automatisé et
      vérifié, pas de serveur oublié.
```
