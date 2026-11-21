---
name: conseil-projet
description: Conseil de débat inspiré de karpathy/llm-council. Utiliser quand l'utilisateur montre un projet (dépôt GitHub, idée, business plan, code, site, produit) et demande s'il est bon, viable, s'il vaut le coup, ou veut un avis ou un débat sur ce projet. Plusieurs experts donnent un avis indépendant, se critiquent de façon anonyme, puis un président rend un verdict argumenté. Déclencheurs : « conseil », « débat », « est-ce que ce projet est bien », « ton avis sur ce projet », /conseil-projet.
---

# Conseil projet : débat et verdict

Inspiré de [karpathy/llm-council](https://github.com/karpathy/llm-council), qui fonctionne en 3 étapes :
avis indépendants → critique croisée anonyme → synthèse par un président.
Ici, les membres du conseil sont des sous-agents (outil Agent) qui jouent chacun un rôle d'expert.

Toujours répondre **en français**.

## Étape 0 : Comprendre le projet

1. Rassembler le matériel : cloner le dépôt (`git clone --depth 1` dans le scratchpad), lire le README,
   la structure, les fichiers clés ; ou lire le texte, le lien ou les fichiers fournis.
2. Écrire une **fiche projet** de 15 à 30 lignes : but, public visé, ce qui existe déjà (code, traction,
   état d'avancement), technologies, modèle économique s'il y en a un, questions ouvertes.
3. Si le but du projet est vraiment impossible à deviner, poser UNE question à l'utilisateur.
   Sinon, avancer.

## Étape 1 : Avis indépendants (en parallèle)

Lancer **en un seul message** 5 sous-agents (outil Agent, `subagent_type: general-purpose`),
chacun avec la fiche projet, le chemin du dépôt cloné s'il y en a un, et son rôle :

| Membre | Rôle |
|---|---|
| A. Le Technicien | Qualité du code, architecture, faisabilité technique, dette, sécurité, maintenabilité. |
| B. L'Investisseur | Marché, besoin réel, concurrence, modèle économique, potentiel de croissance. |
| C. L'Utilisateur | Expérience concrète : est-ce utile, simple, agréable ? Utiliserait-il ça ? |
| D. L'Avocat du diable | Cherche activement pourquoi le projet va échouer. Doit être dur mais juste. |
| E. Le Stratège | Faisabilité pour l'auteur (temps, compétences, coûts), priorités, prochaine étape. |

Consigne à donner à chaque membre :

> Tu es membre d'un conseil qui évalue un projet. Ton rôle : <rôle>.
> Fiche projet : <fiche>. Tu peux lire les fichiers à <chemin> si besoin.
> Réponds en français, en 250 mots maximum :
> 1. Points forts (avec preuves concrètes tirées du projet)
> 2. Points faibles et risques (avec preuves)
> 3. Note sur 10 et verdict en une phrase : BON / PROMETTEUR SOUS CONDITIONS / FAIBLE
> N'invente rien : si une information manque, dis-le.

## Étape 2 : Critique croisée anonyme

Anonymiser les 5 avis (« Avis 1 » à « Avis 5 », dans un ordre mélangé, sans nom de rôle).
Lancer en parallèle 3 sous-agents relecteurs avec la fiche projet et les 5 avis anonymes :

> Voici 5 avis anonymes sur un projet. Pour chacun : ce qui est juste, ce qui est faux,
> exagéré ou non prouvé. Ajoute tout argument important que personne n'a vu.
> Termine EXACTEMENT par :
> CLASSEMENT FINAL:
> 1. Avis X
> 2. Avis Y
> ...
> (du plus pertinent au moins pertinent). En français, 300 mots maximum.

Calculer le rang moyen de chaque avis (comme `calculate_aggregate_rankings` dans llm-council).

## Étape 3 : Verdict du président

C'est toi (l'agent principal) le président. Synthétiser en donnant plus de poids aux avis bien classés
et aux arguments appuyés par des preuves, pas à la majorité. Répondre à l'utilisateur avec ce format :

```
## Verdict du conseil : <BON | PROMETTEUR SOUS CONDITIONS | FAIBLE> (note moyenne X/10)

**En une phrase :** ...

### Le débat
- Le Technicien (X/10) : ...
- L'Investisseur (X/10) : ...
- L'Utilisateur (X/10) : ...
- L'Avocat du diable (X/10) : ...
- Le Stratège (X/10) : ...

### Points d'accord
### Points de désaccord (et qui a raison selon le président, et pourquoi)
### Pourquoi le projet peut marcher
### Pourquoi il peut échouer
### Les 3 actions prioritaires pour l'améliorer
```

Règles :
- Être honnête : ne pas flatter. Un projet faible doit être dit faible, avec des raisons.
- Citer des éléments concrets du projet (fichiers, fonctionnalités, chiffres), pas des généralités.
- Signaler les limites : tous les membres sont le même modèle avec des rôles différents,
  contrairement à llm-council qui utilise plusieurs modèles différents.
- Version rapide : si l'utilisateur demande un avis « rapide », sauter l'étape 2 et lancer 3 membres
  (Technicien, Investisseur, Avocat du diable).
