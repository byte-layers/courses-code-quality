---
title: "La septième entreprise"
description: "Un défi en trois décisions sur une entreprise que vous n'avez pas encore vue : les textes qui s'appliquent aujourd'hui, celui qu'une nouvelle fonction rend applicable, et les mesures à lancer en priorité. Vos décisions sont vérifiées."
tags:
  - challenge
  - quality
difficulty: intermediate
duration: 25
introTitle: "Le défi"
requires: "0.4.0"
options:
  allowManualProgress: false
  autoCompleteOnNext: unverifiable
---

Ce challenge vous permet de prouver que vous savez dire quels textes s'appliquent à une entreprise, et ce qu'ils changent dans son travail de développement. Le bit [Six entreprises, six cadres de qualité](https://byte-layers.com/fr/bits/quality-obligations-by-company) en présentait six ; voici la septième, gardée pour ce défi. Il n'y a **aucune commande à recopier** : chaque tâche pose une question sur l'entreprise, et vous inscrivez votre décision dans un fichier. Un bouton vérifie ce fichier.

Si vous êtes bloqué, les **indices** se dévoilent un par un, et la **solution** reste en dernier recours. Les utiliser ne coûte rien : seule la vérification compte.

## Avant de commencer

- **Ce qu'il faut savoir** : ce qu'enseignent [Normes, règlements et référentiels qualité en France](https://byte-layers.com/fr/bits/french-quality-frameworks) et [Six entreprises, six cadres de qualité](https://byte-layers.com/fr/bits/quality-obligations-by-company).
- **Ce qu'il faut avoir** : un terminal (Git Bash sous Windows) et un éditeur de texte.
- **Le fichier de décisions** : `~/byte-layers/quality/obligations/decisions.txt` est créé pour vous à l'ouverture. Il contient une ligne `clé=` par décision, à compléter après le signe `=`. Les majuscules, les espaces et les accents sont ignorés. Les lignes qui commencent par `#` sont des commentaires. Supprimez le dossier `~/byte-layers/quality/obligations` quand vous n'en avez plus besoin.

> L'entreprise est inventée ; les textes sont réels. Comme le bit, ce défi donne des repères, pas un avis juridique.

# Tâche 1 : les textes qui s'appliquent aujourd'hui

## L'entreprise : Kinéo

```setup
D=~/byte-layers/quality/obligations
if [ ! -f "$D/decisions.txt" ]; then
  mkdir -p "$D"
  cat > "$D/decisions.txt" <<'FIN'
# Tâche 1 : les lettres des textes qui s'appliquent aujourd'hui, par exemple a,c
textes=
# Tâche 2 : le texte que la nouvelle fonction rend applicable, puis la norme de cycle de vie qui va avec
fonction=
norme=
# Tâche 3 : les lettres des trois mesures à lancer, par exemple b,d,f
mesures=
FIN
fi
```

**Kinéo** emploie 40 personnes et réalise 6 millions d'euros de chiffre d'affaires. Elle édite une application web et mobile pour les patients qui se rééduquent après une opération du genou :

- le patient suit chez lui un programme d'exercices en vidéo, et saisit après chaque séance son niveau de douleur et l'amplitude de son genou ;
- son kinésithérapeute consulte ces courbes et ajuste le programme ;
- l'abonnement, 9 euros par mois, se vend en ligne aux particuliers ; le paiement se fait sur la page du prestataire de paiement ;
- la production et les sauvegardes tournent aujourd'hui chez un hébergeur cloud généraliste, sans certification particulière ;
- l'application ne contient aucun système d'intelligence artificielle ;
- Kinéo la présente comme un outil au service du kinésithérapeute, qui seul décide du programme.

Voici les textes à examiner :

| Lettre | Texte |
|---|---|
| a | RGPD, article 9 (catégories particulières de données) |
| b | Code de la santé publique, article L.1111-8 : hébergement des données par un hébergeur certifié HDS |
| c | règlement sur les dispositifs médicaux (MDR) |
| d | DORA |
| e | directive sur l'accessibilité (European Accessibility Act), transposée |
| f | règlement sur l'IA (AI Act), système à haut risque |
| g | NIS2 |

Complétez `textes=` avec les lettres de **tous** les textes qui s'appliquent à Kinéo **aujourd'hui**, séparées par des virgules.

**Critères :**

- chaque lettre choisie correspond à un texte qui s'applique, d'après les données traitées, les clients et le secteur de Kinéo ;
- aucun texte qui s'applique n'est oublié.

```hint
Partez des données : la douleur et l'amplitude d'un genou opéré, suivies par un soignant, sont-elles des données ordinaires ?
```

```hint
Puis des clients : Kinéo vend en ligne à des particuliers. Est-elle une microentreprise, c'est-à-dire moins de 10 personnes et au plus 2 millions d'euros de chiffre d'affaires ? Pour les textes restants, demandez-vous si Kinéo est un acteur financier, si elle utilise de l'IA, et si elle est un hôpital ou son prestataire.
```

```verify
F=~/byte-layers/quality/obligations/decisions.txt
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
v() { tr 'A-Z' 'a-z' < "$F" | sed -n "s/^[[:space:]]*$1[[:space:]]*=//p" | tail -1 | tr -d '[:space:]"'; }
R=$(v textes)
[ -n "$R" ] || { echo "La ligne textes= est vide."; exit 1; }
echo "$R" | grep -qE '^[a-g](,[a-g])*$' || { echo "Écrivez des lettres de a à g séparées par des virgules, par exemple a,c."; exit 1; }
L=$(echo "$R" | tr -cd 'a-g' | fold -w1 | sort -u | tr -d '\n')
M=""
case "$L" in *c*) M="$M
- c : le MDR s'applique à un logiciel qui a une finalité médicale propre, comme décider d'une rééducation. Aujourd'hui, Kinéo présente l'application comme un outil du kinésithérapeute, et c'est lui qui ajuste le programme.";; esac
case "$L" in *d*) M="$M
- d : DORA vise le secteur financier ; Kinéo n'est pas un acteur financier, et elle confie le paiement à un prestataire.";; esac
case "$L" in *f*) M="$M
- f : l'application ne contient aucun système d'IA.";; esac
case "$L" in *g*) M="$M
- g : NIS2 vise les entités essentielles, comme les hôpitaux, et leurs prestataires ; Kinéo n'est ni l'un ni l'autre.";; esac
case "$L" in *a*) ;; *) M="$M
- il manque un texte sur les données : la douleur et l'amplitude d'un genou suivi par un soignant sont des données de santé.";; esac
case "$L" in *b*) ;; *) M="$M
- il manque un texte sur l'hébergement : qui doit être certifié pour héberger des données de santé ?";; esac
case "$L" in *e*) ;; *) M="$M
- il manque un texte sur les clients : Kinéo vend en ligne à des particuliers, et avec 40 salariés ce n'est pas une microentreprise.";; esac
[ -z "$M" ] || { echo "Votre liste ($L) n'est pas encore la bonne :$M"; exit 1; }
echo "Les trois textes sont les bons : RGPD article 9, certification HDS, directive sur l'accessibilité."
```

````solution
```text
textes=a,b,e
```

- **a** : la douleur et l'amplitude d'un genou opéré, suivies par un soignant, sont des données de santé, une catégorie particulière du RGPD.
- **b** : tout hébergeur de données de santé à caractère personnel doit être certifié HDS. Celui de Kinéo ne l'est pas : c'est un écart à corriger.
- **e** : Kinéo vend un abonnement en ligne à des particuliers, un service de commerce en ligne ; avec 40 salariés, elle n'est pas une microentreprise exemptée.
- **c** ne s'applique pas encore : ce qui compte est la finalité que le fabricant revendique, ici un outil au service du kinésithérapeute. **d**, **f** et **g** ne concernent pas Kinéo.
````

# Tâche 2 : la fonction qui fait changer Kinéo de catégorie

## Progression automatique

L'équipe produit prépare une nouvelle fonction, **Progression auto** : d'après la douleur et l'amplitude saisies, l'application **augmente ou réduit elle-même les exercices** du patient, sans attendre le kinésithérapeute. Les règles ont été écrites par des kinésithérapeutes ; ce n'est toujours pas de l'IA.

Complétez dans `decisions.txt` :

- `fonction=` avec le texte que cette fonction rend applicable, sous son sigle ;
- `norme=` avec le numéro de la norme habituellement utilisée pour le cycle de vie de ce type de logiciel.

**Critères :**

- le texte est celui qui encadre un logiciel ayant une finalité médicale, ici adapter la rééducation du patient ;
- la norme est celle du cycle de vie du logiciel dans ce cadre.

```hint
Comparez avec MédiSuivi, dans le bit des six entreprises : quelle fonction l'a fait changer de catégorie, et quel texte est alors devenu applicable ?
```

```verify
F=~/byte-layers/quality/obligations/decisions.txt
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
v() { tr 'A-Z' 'a-z' < "$F" | sed -n "s/^[[:space:]]*$1[[:space:]]*=//p" | tail -1 | tr -d '[:space:]"' | sed 's/é/e/g;s/è/e/g;s/É/e/g'; }
case "$(v fonction)" in
  mdr|*dispositif*medica*|*2017/745*) ;;
  "") echo "La ligne fonction= est vide."; exit 1 ;;
  *aiact*|*ia*|*intelligence*) echo "Progression auto n'utilise pas d'IA : ce sont des règles fixes. Ce qui change, c'est sa finalité : elle modifie elle-même la rééducation du patient."; exit 1 ;;
  *hds*|*rgpd*) echo "Ce texte s'appliquait déjà avant la fonction. Lequel devient applicable parce que le logiciel décide lui-même de la rééducation ?"; exit 1 ;;
  *) echo "Ce n'est pas ce texte. Quel texte est devenu applicable à MédiSuivi avec la proposition de dose d'insuline ?"; exit 1 ;;
esac
case "$(v norme)" in
  *62304*) ;;
  "") echo "La ligne norme= est vide."; exit 1 ;;
  *27001*) echo "ISO 27001 porte sur la gestion de la sécurité de l'information, pas sur le cycle de vie d'un logiciel de dispositif médical."; exit 1 ;;
  *13485*|*14971*) echo "Cette norme existe pour les dispositifs médicaux, mais elle porte sur le système qualité du fabricant ou sur la gestion des risques, pas sur le cycle de vie du logiciel."; exit 1 ;;
  *25010*) echo "ISO 25010 décrit les caractéristiques de qualité d'un logiciel, pas le cycle de vie d'un logiciel de dispositif médical."; exit 1 ;;
  *) echo "Ce n'est pas cette norme. Cherchez la norme IEC du cycle de vie des logiciels de dispositifs médicaux, citée pour MédiSuivi."; exit 1 ;;
esac
echo "Progression auto fait de Kinéo le fabricant d'un dispositif médical, avec le MDR et IEC 62304."
```

````solution
```text
fonction=MDR
norme=IEC 62304
```

Un logiciel qui adapte lui-même la rééducation d'un patient a une finalité médicale : c'est un **dispositif médical**, soumis au règlement 2017/745 (MDR), avec marquage CE et évaluation clinique. La norme **IEC 62304** décrit le cycle de vie de ce logiciel ; c'est elle qu'on utilise habituellement pour montrer la conformité. Que les règles soient écrites à la main ou apprises par une IA n'y change rien : c'est la finalité qui compte.
````

# Tâche 3 : les mesures à lancer

## Trois mesures prioritaires

Kinéo veut traduire ces obligations en mesures concrètes, dans son infrastructure et dans son pipeline, et ne peut en lancer que trois ce trimestre. Voici les propositions de l'équipe :

| Lettre | Proposition |
|---|---|
| a | migrer la production et les sauvegardes chez un hébergeur certifié HDS |
| b | copier chaque nuit la base de production en recette, pour tester sur des cas réels |
| c | relier chaque exigence de Progression auto à ses tests et à son code, version après version |
| d | lancer des tests d'accessibilité automatiques sur les pages d'abonnement et de compte client |
| e | ajouter un jeu de test qui mesure les biais du modèle de recommandation |
| f | conserver les numéros de carte, chiffrés, pour faciliter le réabonnement |

Complétez `mesures=` avec les lettres des **trois** propositions à retenir.

**Critères :**

- chaque mesure retenue répond à un texte des tâches 1 et 2 ;
- aucune proposition retenue ne crée un nouveau risque ou ne répond à un texte qui ne s'applique pas.

```hint
Deux propositions créent un risque au lieu d'en réduire un : où iraient des données de santé réelles ? Que gagnait Kinéo à faire payer sur la page du prestataire ?
```

```hint
Une proposition répond à un texte écarté à la tâche 1.
```

```verify
F=~/byte-layers/quality/obligations/decisions.txt
[ -f "$F" ] || { echo "Le fichier $F n'existe pas. Rechargez la page du challenge pour le recréer."; exit 1; }
v() { tr 'A-Z' 'a-z' < "$F" | sed -n "s/^[[:space:]]*$1[[:space:]]*=//p" | tail -1 | tr -d '[:space:]"'; }
R=$(v mesures)
[ -n "$R" ] || { echo "La ligne mesures= est vide."; exit 1; }
echo "$R" | grep -qE '^[a-f](,[a-f])*$' || { echo "Écrivez des lettres de a à f séparées par des virgules, par exemple b,d,f."; exit 1; }
L=$(echo "$R" | tr -cd 'a-f' | fold -w1 | sort -u | tr -d '\n')
M=""
case "$L" in *b*) M="$M
- b : aucune donnée réelle de patient ne doit servir en développement ni en test, comme chez MédiSuivi ; on teste avec des données fictives ou anonymisées.";; esac
case "$L" in *e*) M="$M
- e : Kinéo n'a pas de modèle d'IA ; les règles de Progression auto sont fixes.";; esac
case "$L" in *f*) M="$M
- f : en payant sur la page du prestataire, Kinéo ne voit jamais les numéros de carte. Les conserver élargirait son périmètre PCI DSS.";; esac
case "$L" in *a*) ;; *) M="$M
- il manque la mesure qui corrige l'écart de la tâche 1 sur l'hébergement.";; esac
case "$L" in *c*) ;; *) M="$M
- il manque la mesure qu'exige le cycle de vie d'un dispositif médical.";; esac
case "$L" in *d*) ;; *) M="$M
- il manque la mesure qui répond à la directive sur l'accessibilité.";; esac
[ -z "$M" ] || { echo "Votre choix ($L) n'est pas encore le bon :$M"; exit 1; }
echo "Les trois mesures sont les bonnes : hébergement HDS, traçabilité des exigences, tests d'accessibilité."
```

````solution
```text
mesures=a,c,d
```

| Lettre | Verdict |
|---|---|
| a | retenu : corrige l'écart sur la certification HDS |
| b | écarté : des données de santé réelles en recette, c'est exactement ce qu'il faut éviter ; on teste avec des données fictives ou anonymisées |
| c | retenu : la traçabilité exigences, tests, code qu'attend IEC 62304 |
| d | retenu : la directive sur l'accessibilité vise la partie commerce en ligne, l'abonnement et le compte, vérifiée par des tests automatiques en plus d'un audit RGAA |
| e | écarté : pas d'IA chez Kinéo |
| f | écarté : élargit le périmètre PCI DSS que le paiement délégué réduisait |
````

## Récap

- Les obligations se déduisent des **données** (santé), des **clients** (vente en ligne à des particuliers, taille de l'entreprise) et du **secteur**, pas de la technologie.
- Une **fonction** peut faire changer l'entreprise de catégorie : adapter soi-même la rééducation d'un patient fait d'un logiciel un dispositif médical, avec ou sans IA.
- Chaque texte se traduit en une **mesure** précise ; une mesure mal choisie peut créer un risque au lieu d'en réduire un.

> 📖 [Hébergement de données de santé, Agence du numérique en santé](https://esante.gouv.fr/produits-services/hds) · [Règlement (UE) 2017/745 sur les dispositifs médicaux, sur EUR-Lex](https://eur-lex.europa.eu/eli/reg/2017/745/oj?locale=fr) · [Accessibilité numérique, RGAA](https://accessibilite.numerique.gouv.fr/)
