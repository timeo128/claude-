---
name: conseil-projet
description: Conseil d'analystes e-commerce professionnels inspiré de karpathy/llm-council. Utiliser quand l'utilisateur présente une idée de produit, de boutique ou de projet e-commerce (dropshipping, marque, produit physique) et demande s'il est bon, viable, s'il vaut le coup, ou veut un avis ou un débat. Plusieurs analystes jugent le projet sur 4 critères (douleur, cible, plan économique, simplicité), se critiquent de façon anonyme, puis un président rend un verdict argumenté. Déclencheurs : « conseil », « débat », « est-ce que ce projet est bien », « ton avis sur ce produit », /conseil-projet.
---

# Conseil projet : analystes e-commerce, débat et verdict

Inspiré de [karpathy/llm-council](https://github.com/karpathy/llm-council), qui fonctionne en 3 étapes :
avis indépendants → critique croisée anonyme → synthèse par un président.
Ici, les membres du conseil sont des sous-agents (outil Agent) qui jouent chacun le rôle
d'un **analyste e-commerce professionnel**.

Toujours répondre **en français**.

## La grille : les 4 critères obligatoires

Tout le conseil juge le projet sur ces 4 critères. Chaque critère reçoit une note sur 10.

1. **Douleur aiguë et récurrente.** Le produit résout une douleur ressentie *souvent* et *fort* :
   pas une gêne vague qu'on oublie, mais une douleur qui revient chaque semaine, voire chaque jour,
   et qui pourrit la vie quand elle arrive.
   Question test : « Cette personne pense-t-elle à cette douleur au moins une fois par semaine ? »
2. **Cible identitaire et émotionnellement engagée.** La cible se décrit en une phrase précise.
   « Les femmes » n'est pas une cible ; « la maman de 35 ans qui vient d'accoucher et n'arrive plus
   à dormir » en est une. Plus c'est précis et identitaire, plus ça marche.
   Question test : « Si je croise cette personne dans la rue, est-ce que je peux la reconnaître ? »
3. **Plan économique solide.** Prix de vente entre **40 € et 70 €**, et **marge x4 minimum**
   (prix de vente ≥ 4 × coût du produit livré). En dessous de 40 €, on ne peut pas payer les pubs ;
   au-dessus de 70 €, les gens réfléchissent trop avant d'acheter ; sans x4, il ne reste rien
   pour payer le reste.
   Question test : « Si je paye 15 € à Meta pour une vente, combien il me reste pour vivre ? »
   → Toujours faire le calcul : prix − coût produit − livraison − 15 € de pub − frais de paiement (~3 %).
4. **Produit simple**, avec 4 simplicités :
   - simple à livrer : pas fragile, pas énorme, pas réglementé ;
   - simple à utiliser : on comprend en 5 secondes ;
   - simple à marketer : effet « wow » visible en vidéo ;
   - effet visible : avant/après évident.

   Question test : « Si je montre une photo du produit à ma grand-mère, comprend-elle à quoi ça sert
   en 5 secondes ? »

**Note du projet** = moyenne des 4 critères. **Règle éliminatoire** : un critère à 3/10 ou moins
plafonne le verdict à FAIBLE, quelle que soit la moyenne.

## Étape 0 : Comprendre le projet

1. Rassembler le matériel : idée, lien produit (AliExpress, Amazon, boutique), photos, prix,
   dépôt de code, etc.
2. Écrire une **fiche projet** de 15 à 30 lignes : produit, problème résolu, cible, prix de vente prévu,
   coût d'achat et de livraison, canal d'acquisition (Meta, TikTok…), budget et temps de l'auteur,
   questions ouvertes.
3. Si le produit ou le prix sont impossibles à deviner, poser UNE question à l'utilisateur
   (de préférence : prix de vente et coût d'achat). Sinon, avancer en signalant les hypothèses.
4. Optionnel, si les outils `mcp__trendtrack__*` sont disponibles : lancer 1 ou 2 recherches
   (`search_products`, `search_ads`) pour voir si le produit est déjà vendu et à quel prix,
   et ajouter les résultats à la fiche. Rester économe en crédits.

## Étape 1 : Avis indépendants (en parallèle)

Lancer **en un seul message** 5 sous-agents (outil Agent, `subagent_type: general-purpose`),
chacun avec la fiche projet, la grille des 4 critères ci-dessus (copiée en entier) et son rôle :

| Membre | Rôle |
|---|---|
| A. L'Analyste Douleur | Spécialiste du critère 1 : la douleur est-elle vraiment aiguë et hebdomadaire ? |
| B. L'Analyste Cible | Spécialiste du critère 2 : la cible tient-elle en une phrase reconnaissable ? |
| C. L'Analyste Finance | Spécialiste du critère 3 : fait le calcul complet de marge et de rentabilité avec 15 € de pub. |
| D. L'Analyste Produit | Spécialiste du critère 4 : vérifie les 4 simplicités une par une. |
| E. L'Avocat du diable | Cherche activement pourquoi le produit va échouer (concurrence, saturation, retours, SAV, réglementation, fournisseur). Dur mais juste. |

Consigne à donner à chaque membre :

> Tu es un analyste e-commerce professionnel, membre d'un conseil qui évalue un projet.
> Ton rôle : <rôle>. Fiche projet : <fiche>. Grille des 4 critères : <grille>.
> Réponds en français, en 250 mots maximum :
> 1. Analyse détaillée de ton critère de spécialité (ou, pour l'avocat du diable, les risques majeurs)
> 2. Une note sur 10 pour CHACUN des 4 critères, avec une justification d'une ligne
> 3. Note globale sur 10 et verdict en une phrase : BON / PROMETTEUR SOUS CONDITIONS / FAIBLE
> N'invente rien : si une information manque (prix, coût…), dis-le et indique ton hypothèse.

## Étape 2 : Critique croisée anonyme

Anonymiser les 5 avis (« Avis 1 » à « Avis 5 », dans un ordre **mélangé**, sans nom de rôle),
les écrire dans un fichier du scratchpad, puis lancer en parallèle 3 sous-agents relecteurs
(analystes e-commerce eux aussi) avec la fiche projet, la grille et ce fichier :

> Voici 5 avis anonymes sur un projet e-commerce, jugé sur 4 critères. Pour chacun : ce qui est
> juste, ce qui est faux, exagéré ou non prouvé (vérifie les calculs de marge). Ajoute tout argument
> important que personne n'a vu.
> Termine EXACTEMENT par :
> CLASSEMENT FINAL:
> 1. Avis X
> 2. Avis Y
> ...
> (du plus pertinent au moins pertinent). En français, 300 mots maximum.

Calculer le rang moyen de chaque avis (comme `calculate_aggregate_rankings` dans llm-council).

## Étape 3 : Verdict du président

C'est toi (l'agent principal) le président. Pour chaque critère, retenir une note en donnant plus de
poids au spécialiste du critère et aux avis bien classés, pas à la majorité. Refaire toi-même le
calcul de marge. Répondre à l'utilisateur avec ce format :

```
## Verdict du conseil : <BON | PROMETTEUR SOUS CONDITIONS | FAIBLE> (X/10)

**En une phrase :** ...

### Les 4 critères
| Critère | Note | Verdict |
|---|---|---|
| 1. Douleur aiguë et récurrente | X/10 | ✅ / ⚠️ / ❌ + une ligne |
| 2. Cible identitaire | X/10 | ... |
| 3. Plan économique (40-70 €, marge x4) | X/10 | ... |
| 4. Produit simple | X/10 | ... |

### Le calcul
Prix de vente − coût produit − livraison − 15 € pub − frais = ce qu'il reste par vente.

### Le débat
- L'Analyste Douleur (X/10) : ...
- L'Analyste Cible (X/10) : ...
- L'Analyste Finance (X/10) : ...
- L'Analyste Produit (X/10) : ...
- L'Avocat du diable (X/10) : ...

### Points de désaccord (et qui a raison selon le président, et pourquoi)
### Pourquoi le projet peut marcher
### Pourquoi il peut échouer
### Les 3 actions prioritaires pour l'améliorer
```

Règles :
- Être honnête : ne pas flatter. Un projet faible doit être dit faible, avec des raisons.
- Citer des éléments concrets (prix, coût, cible, caractéristiques du produit), pas des généralités.
- Proposer, si possible, une version améliorée qui passe mieux les 4 critères
  (cible plus précise, bundle pour monter le prix, autre fournisseur…).
- Signaler les limites : tous les membres sont le même modèle avec des rôles différents,
  contrairement à llm-council qui utilise plusieurs modèles différents.
- Version rapide : si l'utilisateur demande un avis « rapide », sauter l'étape 2 et lancer 3 membres
  (Analyste Finance, Analyste Produit, Avocat du diable), chacun notant les 4 critères.
