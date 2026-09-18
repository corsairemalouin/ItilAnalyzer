# Matrice de traçabilité — DAv0 → DAv1

## Décisions DAv0

| DEC | Énoncé | Chapitre DAv1 | ADR | Artefact vérifiable |
|---|---|---|---|---|
| DEC-00 | Pipeline multi-agents, code déterministe | 03.1, 04.2, 04.3 | ADR-001 | `04-etats-ticket.puml` |
| DEC-01 | Socle + packs déclaratifs | 01.3, 05.1 | ADR-007 | `pack-client.schema.json`, `pack-projet.schema.json`, test anti-contamination CI |
| DEC-02 | Noyau ITIL, surface terrain | 09.1 | ADR-008 | `decision-triage.schema.json` (`type`, `impact`, `urgence`, `priorite`) |
| DEC-03 | Corpus à deux étages | 10.2, 10.3 | ADR-003 | `source.schema.json` (`etage`), `plan-action.schema.json` (`etage: normatif`) |
| DEC-04 | Trois classes de modèles | 06.2 | ADR-002 | `pack-client.schema.json` → `classes_modeles_autorisees` |
| DEC-05 | Curseur 0→4 par catégorie × action | 12 | ADR-005 | `pack-projet.schema.json` → `niveaux_autonomie`, `PATCH /gouvernance/autonomie` |
| DEC-06 | Superviseur 7 contrôles, fail-closed | 11 | ADR-004 | `verdict.schema.json` (7 contrôles, `minItems: 7`) |
| DEC-07 | Entrée sans intégration, écriture progressive | 05.2, 07 | ADR-006 | `niveau_risque` de la liste blanche |
| DEC-08 | Port LLM, orchestration codée, monolithe modulaire | 05, 06.1 | ADR-010, ADR-011 | `02-conteneurs-c2.puml` |

## Exigences fonctionnelles

| EF | Chapitre | Contrat | Test |
|---|---|---|---|
| EF-01 Ingérer sans API et normaliser | 05.2, 07 | `ticket-canonique` | Adaptateur mail + mock |
| EF-02 Scorer complétude, ≤ 3 questions | 04.3 Porte 1 | `ticket-canonique.questions_demandeur` (`maxItems: 3`) | Schéma |
| EF-03 Triage structuré avec confiance et justification | 04.2 | `decision-triage` | Schéma + banc |
| EF-04 Détecter un doublon | 04.2 | `ticket-canonique.doublon` | Jeu d'évaluation : 2 cas |
| EF-05 Appliquer un runbook sur liste blanche | 04.3 Porte 3-4 | `plan-action` | Contrôle 3 du superviseur |
| EF-06 Diagnostic N2 sourcé, lecture seule | 04.2 | `diagnostic` | Aucun droit d'écriture, test d'architecture |
| EF-07 Dossier N3 avec tous les identifiants | 09.2 | `dossier-escalade.identifiants_externes` (`minItems: 1`) | Test EF-07 |
| EF-08 Exposer toute proposition avant effet de bord | 11.1 | `verdict` | Adaptateur refuse sans verdict |
| EF-09 Rejouer une décision à l'identique | 13.2 | `Trace` | `POST /audit/traces/{id}/rejouer` |
| EF-10 Mesurer l'écart chemin déclaré / contenu | 04.2 | `decision-triage.mal_dirige` | KPI mal-dirigé résiduel |

## Exigences non fonctionnelles

| ENF | Cible | Chapitre | Mécanisme |
|---|---|---|---|
| ENF-01 Auditabilité | 100 % | 13.2 | Trace écrite avant effet, versions complètes |
| ENF-02 Déterminisme du routage | 100 % | 03.1 | Aucune transition décidée par un modèle |
| ENF-03 Latence triage | < 60 s | 06.3 | Budget par étape, contractuel, repli déclaré |
| ENF-04 Coût par ticket | budget figé | 06.2 | Plafond de jetons, escalade plafonnée |
| ENF-05 Accord annotation humaine | ≥ 85 % | 13.5 | Matrice de confusion, gating CI |
| ENF-06 Calibration | < 10 pts | 13.5 | Courbe de calibration, gating CI |
| ENF-07 Résistance à l'injection | 0 échec | 08.1 M-01/M-02, 11.2 | Contrôle 7, isolation du contenu, cas d'injection |
| ENF-08 Étanchéité des contextes | par construction | 07.3, 08.1 M-03 | Filtrage à l'index, mono-tenant v0, test d'étanchéité |
| ENF-09 Portabilité fournisseur | couche d'abstraction | 15 | Ports + test de réversibilité |
| ENF-10 Dégradation maîtrisée | fail-closed | 06.4 | Modes dégradés par composant |

## Questions ouvertes bloquantes

| Q | Bloque | Effet si non tranchée |
|---|---|---|
| Q-01 | Écriture ITSM | Niveaux 0-1 livrables sans elle — non bloquant pour le DAv1 |
| Q-02 | Mesure du mal-dirigé | Gain non démontrable |
| **Q-03** | **DAv1-06, DAv1-07, ADR-018** | **Classe L et dimensionnement GPU en hypothèse (R-14)** |
| Q-04 | Investigateur N2 | Valeur du N2 fortement réduite |
| Q-05 | Promotion du corpus | Étage 2 jamais alimenté (R-01) |
| **Q-06** | **DAv1-09, ADR-016** | **Rétention et base légale en hypothèse** |
| Q-09 | ADR-012 | Choix du moteur d'index non figé |
| Q-10, Q-12, Q-13, Q-15 | NFR-01, ADR-013, ENF-04, séquencement | Phase 3 partiellement en hypothèse |
