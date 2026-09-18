# F-05 — BPMN

**Amont** : F-04 (processus ITIL) · **Aval** : S-08 (séquences), implémentation de la machine à états

## Diagrammes produits

| Id | Fichier PlantUML | Processus couvert |
|---|---|---|
| BPMN-01 | `docs/plantuml/functional/01-bpmn-incident.puml` | Gestion des incidents, chemin nominal complet (les 4 portes) |
| BPMN-02 | `docs/plantuml/functional/02-bpmn-promotion-connaissance.puml` | Promotion étage 1 → étage 2 du corpus |
| BPMN-03 | `docs/plantuml/functional/03-bpmn-changement-autonomie.puml` | Montée et descente du curseur d'autonomie |

**Note de format** : ces diagrammes sont fournis en PlantUML (notation activité), équivalente au contenu d'un BPMN standard (couloirs implicites par note, passerelles exclusives = `if/then/else`, tâches humaines marquées `<<humain>>`). L'export Draw.io XML de ces trois diagrammes est à produire dans un second temps dédié — voir note de fin de document.

## BPMN-01 — Lecture du processus de gestion des incidents

Le diagramme matérialise l'arbre à 4 portes de DAv1-04 comme un processus BPMN à part entière. Points de lecture :

- Le **court-circuit permanent** est modélisé comme une passerelle exclusive évaluée en tout premier, avant toute autre étape — conforme à sa priorité absolue sur le reste du pipeline.
- Chaque **« non »** d'une porte est une branche vers une tâche humaine (orange), jamais une branche d'erreur : c'est une propriété du processus, pas un traitement d'exception.
- Le superviseur est la dernière passerelle avant tout effet visible à l'extérieur : aucun chemin du diagramme n'atteint « Exécution » ou « Réponse au demandeur » sans passer par ses 7 contrôles.

## BPMN-02 — Lecture de la promotion de connaissance

Trois passerelles d'arrêt anticipé (secret détecté, moins de 3 tickets concordants, actions non couvertes par la liste blanche) modélisent les trois façons dont un brouillon **n'atteint jamais** l'étage normatif sans validation complète. La boucle de fin (expiration de revue → retour à l'étage 1) est représentée comme un cycle de vie continu, pas comme une fin de processus : un document normatif reste sous surveillance après sa publication.

## BPMN-03 — Lecture du changement d'autonomie

Modélisé en `fork/fork again` : les quatre conditions (trois de descente, une de montée) sont évaluées **en parallèle et en continu** par le système, pas séquentiellement. Une condition de descente prend toujours le pas — le diagramme n'a pas de chemin où montée et descente sont évaluées comme mutuellement exclusives dans le temps, ce qui reflète la réalité : le système surveille en permanence, sur toutes les catégories à la fois.

## Couloirs (acteurs) par processus

| Processus | Couloir « Système / agents » | Couloir « Superviseur » | Couloir « Humain » |
|---|---|---|---|
| Incident | Intake, Triage, Résolveur N1, Investigateur N2, Préparateur N3 | Superviseur | Demandeur, Technicien N1/N2, Expert N3 |
| Promotion | Collecte, clustering, synthèse | — (pas d'effet de bord tant que non publié) | Propriétaire du corpus |
| Autonomie | Calcul continu des métriques | — | Responsable de service, Client |

## Reste à produire

Export Draw.io XML des 3 diagrammes ci-dessus (exigence de triple format du repository), à traiter en fin de phase 2 avec les diagrammes de F-06 et F-07, dans un lot dédié aux exports graphiques plutôt qu'en flux continu avec la rédaction — pour limiter le risque d'erreur de structure XML non détectée.
