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

## Phase 2 — Couverture fonctionnelle (GATE-2)

| Livrable | Fichier | Couvre |
|---|---|---|
| F-01 Besoins métier | `functional/01-besoins-metier.md` | BM-01 à BM-17, dérivés de DAv0 §Besoin métier |
| F-02 Use cases | `functional/02-use-cases.md` | UC-01 à UC-10, couvrent EF-01 à EF-10 (table de couverture en fin de document) |
| F-03 User stories | `functional/03-user-stories.md` | US-101 à US-901, 9 épopées, chaque UC couvert par au moins 2 US avec critères Gherkin |
| F-04 Processus ITIL | `functional/04-itil-processus.md` | DEC-02, table de correspondance ITIL ↔ terrain |
| F-05 BPMN | `functional/05-bpmn.md` + `plantuml/functional/01-03` | Processus incident, promotion, autonomie |
| F-06 Domain model | `functional/06-domain-model.md` | 9 contextes bornés, agrégats des 5 contextes structurants |
| F-07 Event storming | `functional/07-event-storming.md` | 3 flux, 3 points chauds identifiés, base du catalogue AsyncAPI (S-04) |
| F-08 Dictionnaire métier | `functional/08-dictionnaire-metier.md` | 26 termes canoniques, fait autorité sur le vocabulaire |
| F-09 Règles métier | `functional/09-regles-metier.md` | BR-01 à BR-31, chacune rattachée à un UC et une exigence amont |
| F-10 RACI | `functional/10-raci.md` | 21 activités, introduit le rôle Propriétaire du corpus (répond à Q-05 côté gouvernance) |

**Critère de sortie GATE-2** : chaque EF est couverte par au moins un UC (vérifié) ; chaque UC par au moins une US (vérifié) ; chaque règle métier par un UC amont (vérifié). Reste à produire avant clôture effective de la porte : les cas de test associés à chaque BR (S-13, phase 4) — la couverture fonctionnelle est complète, la couverture de test ne l'est pas encore, ce qui est attendu à ce stade.

**Point chaud non résolu, à trancher avant S-01** : stabilité du score de complétude en cas de complément asynchrone du ticket (F-07). N'est bloquant ni pour GATE-2 ni pour la suite immédiate, mais doit être tranché avant la rédaction de la spécification fonctionnelle détaillée de l'Intake.

## Phase 3 — Exigences non fonctionnelles chiffrées (GATE-3)

| Chapitre NFR | Fichier | Nombre d'exigences chiffrées | Statut |
|---|---|---|---|
| NFR-01 Disponibilité | `nfr/01-disponibilite-performance-scalabilite.md` | 6 | Complet — ferme Q-10 |
| NFR-02 Performance | idem | 7 | Complet sauf NFR-02.7 (budget/ticket) |
| NFR-03 Scalabilité | idem | 4 | Complet, requalifié (débit non dimensionnant) |
| NFR-04 Sécurité | `nfr/02-securite-souverainete-conformite.md` | 10 | Complet |
| NFR-11 Souveraineté | idem | 5 | 2 sur 5 bloquées par Q-03 |
| NFR-13 Conformité | idem | 8 | 2 sur 8 bloquées par Q-06, 2 hors périmètre technique |
| NFR-05 Observabilité | `nfr/03-observabilite-auditabilite.md` | 5 | Complet |
| NFR-06 Auditabilité | idem | 7 | 1 sur 7 en valeur par défaut proposée (Q-06) |
| NFR-07 Résilience | `nfr/04-resilience-maintenabilite-portabilite.md` | 5 | Complet |
| NFR-08 Maintenabilité | idem | 6 | Complet |
| NFR-09 Portabilité | idem | 3 | Complet |
| NFR-10 Réversibilité | idem | 5 | Complet |
| NFR-12 Qualité IA | `nfr/05-qualite-ia.md` | 11 | Complet |
| Catalogue KPI | `nfr/kpi/catalogue-kpi.md` | 25 KPI | Complet, source unique pour les dashboards |

**Décision actée en phase 3, absente du DAv0** : hors fenêtre de service (heures ouvrées, Q-10), l'exécution automatique de niveau 3-4 est suspendue par défaut ; seules l'ingestion, le triage et la préparation continuent 24/7, sans effet de bord. Un 8ᵉ cas de dégradation contextuelle s'ajoute au superviseur (NFR-01, DAv1-06.4).

**Critère de sortie GATE-3** : sur ~72 exigences chiffrées, 4 restent partiellement bloquées par Q-03 ou Q-06, avec repli explicite dans chaque cas (fail-closed pour Q-03, valeur par défaut de 3 ans proposée pour Q-06). Aucune exigence n'est laissée sans méthode de mesure.

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
