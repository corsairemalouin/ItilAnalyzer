# ITIL Autonomous Incident Analyzer (IAA)

Plateforme générique d'analyse et de traitement des tickets d'anomalie, construite sur le
principe **Core Platform + Context Packs** : un socle réutilisable qui ne connaît ni client
ni outil ITSM, contextualisé par des packs déclaratifs et des adaptateurs techniques.

## État du projet

Phase **P1 — Architecture First**. Aucune ligne de code applicative n'est produite avant la
porte GATE-4 (validation des spécifications). Voir `docs/00-plan/`.

## Navigation

| Chemin | Contenu |
|---|---|
| `docs/00-plan/` | Plan des livrables, portes de validation, dimensionnement, arbitrages |
| `docs/architecture/DAv0/` | Dossier d'architecture préliminaire, figé |
| `docs/architecture/DAv1/` | Dossier d'architecture détaillé, chapitres 01 à 17 |
| `docs/adr/` | Registre des décisions d'architecture |
| `docs/technical/contracts/` | Les 8 contrats JSON Schema des agents |
| `docs/technical/schemas/` | Schémas des packs client et projet |
| `docs/technical/api/` | OpenAPI 3.1 du socle |
| `docs/technical/prompts/` | Registre de prompts + prompt de reprise de contexte |
| `docs/plantuml/` | Diagrammes source PlantUML |
| `docs/traceability/` | Matrice de traçabilité DEC/EF/ENF → chapitres → ADR → contrats |

## Trois règles d'architecture vérifiées automatiquement

1. `src/core/` ne contient aucune référence à un client, un ITSM ou un vocabulaire métier
   spécifique. Contrôle CI bloquant.
2. Le domaine n'importe aucun adaptateur. Contrôle par analyseur d'imports, bloquant.
3. `src/` reste verrouillé jusqu'à la porte GATE-4.

## Pour reprendre le contexte ailleurs

`docs/technical/prompts/CONTEXTE-REPRISE.md` contient un prompt autoportant qui restitue
l'intégralité des décisions actées, des garde-fous, des exigences et de l'état d'avancement.
