# F-09 — Règles métier

**Amont** : F-02 (use cases), DAv1-04, DAv1-11, DAv1-12 · **Aval** : S-01 (SFD), S-13 (tests, une règle = au moins un cas de test)

Exprimées **hors code**, en langage métier vérifiable, pour rester lisibles en revue par un non-développeur. Chaque règle référence l'UC qui la mobilise et l'exigence amont qui la justifie.

## Ingestion (BR-01 à BR-02, BR-10)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-01 | Un ticket dont le score de complétude est inférieur à 0,5 ne peut pas progresser au-delà de l'état `EN_ATTENTE_DEMANDEUR` | UC-01 | EF-02 |
| BR-02 | Le nombre de questions générées vers le demandeur ne peut jamais dépasser 3 | UC-01 | EF-02 |
| BR-10 | Tout document ou message contenant un secret détecté est rejeté du traitement automatique, jamais masqué et poursuivi | UC-01 | ENF-07/08 |

## Triage (BR-03 à BR-05)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-03 | La priorité est toujours calculée par la matrice impact × urgence du pack client ; elle n'est jamais assignée directement par un agent | UC-02 | DAv0 §Formulaire guidé |
| BR-04 | Une décision de triage dont la confiance est inférieure au seuil du pack est présentée en proposition, jamais appliquée automatiquement | UC-02 | ENF-06 |
| BR-05 | L'escalade vers la classe de raisonnement L pour une même décision de triage ne peut se produire qu'une seule fois | UC-02 | DAv1-06.2 (règle 3) |

## Résolution N1 (BR-06 à BR-08, BR-11)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-06 | Une action ne peut figurer dans un plan d'action que si elle existe dans la liste blanche du pack projet en vigueur au moment de l'exécution | UC-03 | Garde-fou absolu 2 |
| BR-07 | Un plan d'action ne peut s'appuyer que sur un runbook de l'étage normatif ; l'étage indicatif ne peut jamais justifier une action | UC-03 | DEC-03 |
| BR-08 | Toute réponse envoyée au demandeur porte la mention qu'elle a été produite ou assistée par une IA, injectée par le composant d'émission | UC-03 | Garde-fou absolu 3 |
| BR-11 | Une action non réversible ne peut jamais être exécutée sans validation humaine explicite, quel que soit le niveau d'autonomie configuré | UC-03 | Garde-fou absolu 1 |

## Investigation et escalade (BR-09, BR-12)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-09 | L'agent Investigateur N2 n'exécute jamais d'action d'écriture, quel que soit le résultat de son analyse | UC-04 | Contrat `diagnostic` : accès lecture seule |
| BR-12 | Un dossier d'escalade ne peut être déposé que s'il porte à la fois l'identifiant du système d'origine et l'identifiant du système cible | UC-05 | EF-07 |

## Supervision (BR-13 à BR-19)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-13 | Les 7 contrôles du superviseur sont **tous** évalués, même après l'échec du premier, afin de produire un motif complet | UC-06 | DAv1-11.2 |
| BR-14 | Un seul contrôle en échec suffit à produire un verdict autre que AUTORISE | UC-06 | DEC-06 |
| BR-15 | Aucun adaptateur à effet de bord n'accepte d'appel sans verdict valide et non expiré | UC-06 | DAv1-11.1 |
| BR-16 | Un ticket portant un marqueur de court-circuit sort du chemin automatique quel que soit le niveau d'autonomie configuré | UC-06 | DAv1-11.5 |
| BR-17 | Aucune élévation de droits n'est exécutée, quelle que soit la demande formulée dans le contenu d'un ticket | UC-06 | Garde-fou absolu 5 |
| BR-18 | Aucune sortie d'agent n'est produite sans trace correspondante écrite avant l'effet de bord | UC-06 | Garde-fou absolu 6 |
| BR-19 | En cas d'indisponibilité d'un composant nécessaire à un contrôle, le verdict est dégradé et le chemin automatique est fermé (fail-closed) | UC-06 | ENF-10 |

## Gouvernance de l'autonomie (BR-20 à BR-24)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-20 | Tout projet démarre au niveau d'autonomie 0, sur toutes les catégories, sans exception | UC-07 | DEC-05 |
| BR-21 | Une montée de niveau exige simultanément : 200 décisions observées, 30 jours d'observation, 95 % d'accord, zéro incident grave, double validation service + client | UC-07 | DAv0 §Curseur |
| BR-22 | Une descente de niveau est déclenchée automatiquement par le système, sans validation requise | UC-07 | DAv0 §Curseur |
| BR-23 | Tout changement de version de modèle, de prompt ou de pack invalide les évaluations en cours et entraîne une descente automatique sur les catégories concernées | UC-07 | R-05 |
| BR-24 | Le niveau d'autonomie maximal atteignable est plafonné par l'environnement de déploiement, indépendamment de la configuration du pack | DAv1-07.1 | ENF-08 |

## Connaissance (BR-25 à BR-29)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-25 | Un brouillon de runbook ne peut être généré qu'à partir d'au moins 3 tickets concordants | UC-08 | DEC-03 |
| BR-26 | Chaque étape d'un runbook promu cite une source | UC-08 | DEC-03 |
| BR-27 | La promotion d'un runbook exige l'identité d'un technicien nommé, inscrite dans la version publiée | UC-08 | DEC-03 |
| BR-28 | Un runbook ne peut être publié que si toutes les actions qu'il cite existent dans la liste blanche du pack projet | UC-08 | DEC-03 |
| BR-29 | Un runbook normatif dont la date d'expiration de revue est dépassée retourne automatiquement à l'étage indicatif | UC-08 | DEC-03 |

## Console et audit (BR-30 à BR-31)

| Id | Règle | UC | Amont |
|---|---|---|---|
| BR-30 | Toute contestation d'une proposition par un technicien est conservée comme annotation de référence, utilisée dans le calcul du taux d'accord | UC-09 | ENF-05 |
| BR-31 | Le rejeu d'une décision passée à partir de sa trace doit produire une décision strictement identique à l'originale | UC-10 | EF-09, ENF-02 |

## Couverture

Chaque règle de ce catalogue est associée, en S-13, à au moins un cas de test dédié. Une règle sans cas de test associé est un défaut de revue à la porte GATE-4.
