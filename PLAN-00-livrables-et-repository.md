# PLAN-00 — Plan détaillé des livrables et du repository cible

**Produit** : ITIL Autonomous Incident Analyzer (code projet : **IAA**)
**Document** : `IAA-PLAN-00` · v0.1 · statut : **pour validation**
**Amont** : `ITILANALYZER-DA-V0` (DAv0 fourni) — décisions DEC-00 à DEC-08, exigences EF-01→10 / ENF-01→10, risques R-01→11, questions ouvertes Q-01→07
**Aval conditionné** : aucun document de la phase 1 n'est rédigé avant validation du présent plan
**Répertoire cible** : `D:\Programmation\ITIL Analyzer\ItilAnalyzer`

---

## 0. Cadre de travail

### 0.1 Règle absolue de séquencement

```
Spec Driven Architecture  →  Spec Driven Design  →  Spec Driven Development
   (phases 1 à 3)               (phase 4)              (phases 5 à 7)
```

Aucune ligne de code applicative n'est produite tant que les phases 1, 2, 3 et 4 ne sont pas produites **et** validées. Une porte de validation explicite (`GATE-n`) sépare chaque phase. Une porte non franchie bloque la phase suivante — sans exception, par cohérence avec la règle de séquencement des lots du DAv0.

### 0.2 Positionnement Core Platform / Context Packs

Tout livrable est classé dès sa création dans l'une de ces trois colonnes. Un livrable non classé est un livrable refusé en revue.

| Colonne | Contenu | Propriété | Versionnement |
|---|---|---|---|
| **Core Platform** | Machine à états, superviseur, contrats JSON, port LLM, moteur RAG, observabilité | Éditeur du socle | SemVer du socle |
| **Context Pack — client** | Cartographie, vocabulaire, matrice de priorité, seuils, politique de sécurité, souveraineté | Livrable au client | SemVer du pack |
| **Context Pack — projet** | Taxonomie, runbooks, catalogue N2, liste blanche d'actions, mapping d'états | Livrable au client | SemVer du pack |
| **Adaptateurs** | ITSM, documentaire, IAM, canaux | Selon contrat | SemVer par adaptateur |

Contrôle appliqué en revue (parade R-08) : toute chaîne de caractères métier client trouvée dans le Core est un défaut bloquant. Le pilote Orange (Genergy N1/N2, Oceane N3) n'apparaît **jamais** ailleurs que dans les packs et les adaptateurs, y compris dans les exemples de documentation.

### 0.3 Nomenclature et identifiants de traçabilité

| Préfixe | Objet | Exemple | Introduit en |
|---|---|---|---|
| `DEC-xx` | Décision d'architecture actée en DAv0 | DEC-06 | DAv0 |
| `EF-xx` / `ENF-xx` | Exigence fonctionnelle / non fonctionnelle | ENF-07 | DAv0, étendu en phase 3 |
| `Q-xx` | Question ouverte bloquante | Q-01 | DAv0, étendu |
| `R-xx` | Risque | R-07 | DAv0, étendu |
| `ADR-xxx` | Architecture Decision Record | ADR-003 | Phase 1 |
| `BM-xx` | Besoin métier | BM-04 | Phase 2 |
| `UC-xx` | Use case | UC-12 | Phase 2 |
| `US-xxx` | User story | US-104 | Phase 2 |
| `DOM-xx` | Concept du domaine | DOM-Ticket | Phase 2 |
| `EVT-xx` | Événement métier | EVT-TriageDecided | Phase 2 |
| `CTR-xx` | Contrat (API ou événement) | CTR-API-Triage | Phase 4 |
| `SFD-xx` / `STD-xx` | Spéc. fonctionnelle / technique détaillée | SFD-03 | Phase 4 |
| `KPI-xx` | Indicateur | KPI-IA-04 | Phase 3 |
| `TST-xx` | Cas de test / d'évaluation | TST-EVAL-017 | Phase 4 |

Règle de traçabilité : **tout** livrable de phase N porte un en-tête `Amont:` listant les identifiants de phase N-1 qu'il instruit, et la matrice `docs/traceability/matrice-tracabilite.md` est régénérée à chaque porte. Un identifiant orphelin (aucun aval) ou non rattaché (aucun amont) est un défaut de revue.

### 0.4 Statuts de document

`brouillon` → `pour revue` → `pour validation` → `validé (vX.Y)` → `figé (baseline)` → `obsolète (remplacé par …)`

### 0.5 Definition of Done d'un livrable documentaire

1. En-tête complet : id, version, statut, auteur, amont, aval, date
2. Traçabilité bidirectionnelle renseignée
3. Tous les schémas fournis dans les **trois formats** exigés : Markdown (texte structuré), PlantUML (`.puml`), Draw.io (`.drawio`, XML)
4. Aucune décision implicite : toute option écartée l'est avec un critère écrit et, si structurante, une matrice pondérée
5. Aucune dépendance à une question ouverte non tranchée sans mention explicite de repli
6. Revue passée (grille de revue en `docs/quality/grille-revue.md`)

---

## 1. Vue d'ensemble de la chaîne de livrables

| Phase | Intitulé | Livrables majeurs | Porte | Conditionne |
|---|---|---|---|---|
| **P0** | Plan & cadrage | Ce document, charte de traçabilité, glossaire v0 | GATE-0 | Tout |
| **P1** | Architecture First | DAv1, 15 ADR, urbanisation, 3 cartographies, 9 vues d'architecture, jeu de diagrammes complet | GATE-1 | P2, P4 |
| **P2** | Analyse fonctionnelle | Besoins, UC, US, processus ITIL, BPMN, domain model, event storming, dictionnaire métier | GATE-2 | P4 |
| **P3** | Exigences non fonctionnelles | 11 chapitres ENF chiffrés, budgets d'erreur, SLO/SLI, KPI | GATE-3 | P4, P6, P7 |
| **P4** | Spec Driven Development | SFD, STD, OpenAPI, AsyncAPI, JSON Schema, modèles de données, séquences, états, plan de tests & d'évaluation | **GATE-4 (porte de code)** | P5 |
| **P5** | Implémentation | Backend, frontend, couche IA/RAG, infra Docker/K8s/Helm | GATE-5 | P6 |
| **P6** | DevSecOps | CI/CD, qualité, SAST/DAST, secrets, versionnement, supply chain | GATE-6 | P7 |
| **P7** | Déploiement & exploitation | Ansible 4 environnements, runbooks d'exploitation, plan de bascule, réversibilité | GATE-7 | Mise en service |

**Ce que la porte GATE-4 signifie concrètement** : c'est la seule porte qui autorise l'écriture de code applicatif. Avant elle, le seul code admis est celui du tooling documentaire (génération de diagrammes, validation de schémas JSON, génération de la matrice de traçabilité) — isolé dans `/tools`, hors `/src`.

### 1.1 Correspondance avec les 4 lots du DAv0

Le plan documentaire et la trajectoire projet sont deux axes distincts qui se croisent :

| Lot DAv0 | Preuve de sortie | Phases documentaires requises avant démarrage |
|---|---|---|
| Lot 1 — Socle et mesure | Matrice de confusion ≥ 85 % | P1, P2, P3 complets ; P4 sur le périmètre Intake + Triage + Superviseur |
| Lot 2 — Connaissance & observation | Accord mesuré sur trafic réel | P4 sur ingestion de connaissance et RAG |
| Lot 3 — Assistance au technicien | Temps gagné mesuré | P4 sur console, Investigateur N2, Préparateur N3 |
| Lot 4 — Automatisation encadrée | Zéro incident imputé | P4 sur Résolveur N1 et écritures ; P6 et P7 complets |

Conséquence : **P4 est produite par incréments alignés sur les lots**, pas en un bloc monolithique. P1 à P3, en revanche, sont produites intégralement avant le lot 1.

---

## 2. Phase 1 — Architecture First

### 2.1 DAv1 — Dossier d'architecture détaillé

Document maître, `docs/architecture/DAv1/`. Découpé en fichiers pour rester revisable, assemblé par un sommaire.

| Id | Chapitre | Contenu | Amont |
|---|---|---|---|
| DAv1-01 | Introduction, périmètre, références | Rappel du périmètre, ce qui est hors périmètre de façon permanente | DAv0 §Périmètre |
| DAv1-02 | Contexte et parties prenantes | Acteurs génériques (N1/N2/N3, responsable de service, RSSI, exploitant) + instanciation pilote en annexe | DAv0 §Parties prenantes |
| DAv1-03 | Principes directeurs | 6 principes + ITIL v4, HITL, XAI, Auditability First, Security by Design, DDD, EDA, hexagonal | DAv0 §Principes |
| DAv1-04 | **Architecture logique** | 5 couches, 7 agents, contrats, arbre de décision à 4 portes, court-circuit permanent | DEC-00, DAv0 §Archi logique |
| DAv1-05 | **Architecture applicative** | Modules, ports & adaptateurs, registre d'agents, machine à états, moteur de fusion de packs | DEC-08 |
| DAv1-06 | **Architecture technique** | Runtimes, passerelle modèles, index, stockages, files, caches | DEC-04, DEC-08 |
| DAv1-07 | **Architecture de déploiement** | Topologies dev/qualif/préprod/prod, K8s, dimensionnement, multi-tenant vs mono-tenant | ENF-08, Q-03 |
| DAv1-08 | **Architecture de sécurité** | Zonage, IAM, secrets, cloisonnement par pack, anti-injection, chiffrement, RGPD/AI Act | ENF-07, ENF-08, R-06, R-07 |
| DAv1-09 | **Architecture des données** | Modèle canonique, identifiants externes multiples, cycle de vie, rétention, minimisation, souveraineté | EF-07, R-11, Q-06 |
| DAv1-10 | **Architecture RAG** | Chaîne d'ingestion 6 étapes, corpus 2 étages, index hybride, reranking, citation obligatoire, versionnement et validité documentaire | DEC-03 |
| DAv1-11 | **Architecture du superviseur** | 7 contrôles, fail-closed, verdicts, garde-fous absolus, modes dégradés | DEC-06 |
| DAv1-12 | **Curseur d'autonomie** | 5 niveaux, granularité client × projet × catégorie × action, montée/descente pilotées par KPI | DEC-05 |
| DAv1-13 | **Architecture observabilité** | Traces, métriques, logs, trace d'audit rejouable, 4 dashboards métier + technique, banc d'évaluation | ENF-01, ENF-02 |
| DAv1-14 | **Architecture DevSecOps** | Chaîne de build, environnements, promotion d'artefacts, gestion des versions de prompts et de packs | ENF-09 |
| DAv1-15 | Réversibilité et portabilité | Sortie fournisseur modèle, ITSM, index, hébergeur ; plan de réversibilité contractuelle | ENF-09 |
| DAv1-16 | Risques et trajectoire | R-01→R-11 étendus, plan de mitigation, lots | DAv0 §Risques |
| DAv1-17 | Annexes | Instanciation pilote (Genergy/Oceane) **isolée en annexe**, jamais dans le corps | DEC-01 |

### 2.2 ADR — Architecture Decision Records

Format : MADR simplifié (contexte, options, critères pondérés, décision, conséquences, statut, amont/aval). Un fichier par ADR dans `docs/adr/`.

| Id | Sujet | Amont | Priorité |
|---|---|---|---|
| ADR-001 | Vision agentique : pipeline multi-agents orchestré par code déterministe | DEC-00 | P1 |
| ADR-002 | Stratégie LLM : port LLM + SDK d'abstraction multi-fournisseur, classes S/M/L | DEC-04, DEC-08 | P1 |
| ADR-003 | Architecture RAG : corpus à deux étages, index hybride, reranking, citations | DEC-03 | P1 |
| ADR-004 | Stratégie d'observabilité et d'auditabilité rejouable | ENF-01, ENF-02 | P1 |
| ADR-005 | Human-in-the-loop et curseur d'autonomie | DEC-05 | P1 |
| ADR-006 | Architecture des adaptateurs (ITSM, documentaire, IAM, canaux) | DEC-07 | P1 |
| ADR-007 | Socle + packs : frontière normative et moteur de fusion | DEC-01 | P1 |
| ADR-008 | Modèle canonique aligné ITIL, surface terrain, mapping déclaratif | DEC-02 | P1 |
| ADR-009 | Choix de la stack backend : Python/FastAPI **ou** Java 21/Spring Boot 3.5 — matrice pondérée, décision unique et motivée | Demande §5 | P1 |
| ADR-010 | Monolithe modulaire au démarrage, frontières de modules explicites | DEC-08 | P1 |
| ADR-011 | Orchestration codée vs BPM, et condition d'introduction ultérieure d'un BPM | DEC-08 | P2 |
| ADR-012 | Stockage : trace d'audit, état de la machine, index vectoriel, corpus | DAv1-09 | P2 |
| ADR-013 | Modèle de multi-tenance et d'étanchéité (index, secrets, chiffrement) | ENF-08 | P1 |
| ADR-014 | Stratégie d'événements : bus, garanties de livraison, idempotence, sagas | EDA | P2 |
| ADR-015 | Frontend Angular : architecture, state management, design system, accessibilité | Demande §5 | P2 |
| ADR-016 | Conformité AI Act et RGPD : qualification du système, transparence, rétention | Q-06 | P1 |
| ADR-017 | Stratégie d'évaluation et de non-régression IA (jeux, seuils, gating CI) | DAv0 §Mesure | P1 |
| ADR-018 | Réversibilité fournisseur et souveraineté des données | ENF-09, Q-03 | P2 |

> ADR-009 est volontairement traité comme une décision à part entière et non comme une préférence : le DAv0 laisse le choix ouvert (Python pressenti pour l'écosystème IA, Java pertinent si le SI cible l'impose). La matrice tranchera sur critères, et la décision sera unique pour le moteur métier, avec autorisation explicite de services périphériques dans un autre langage derrière contrats HTTP/événementiels.

### 2.3 Urbanisation et cartographies

| Livrable | Contenu | Emplacement |
|---|---|---|
| Urbanisation | Zones, quartiers, îlots ; règles d'urbanisme (qui a le droit d'appeler qui) ; flux inter-zones | `docs/architecture/urbanisation/` |
| Cartographie fonctionnelle | Capacités métier, décomposition en fonctions, couverture par le produit, écarts | `docs/architecture/cartographie/fonctionnelle/` |
| Cartographie applicative | Applications, modules, interfaces, flux, criticité, propriétaire | `docs/architecture/cartographie/applicative/` |
| Cartographie technique | Composants d'infrastructure, dépendances, zones réseau, technologies, obsolescence | `docs/architecture/cartographie/technique/` |
| Matrice de couverture | Capacité × module × exigence | `docs/traceability/` |

### 2.4 Diagrammes — triple format obligatoire

Chaque diagramme existe en trois exemplaires cohérents : `NN-nom.md` (description structurée + lecture du schéma), `NN-nom.puml`, `NN-nom.drawio`.

| Jeu | Diagrammes | Format |
|---|---|---|
| C4 | Contexte (C1), Conteneurs (C2), Composants (C3) sur 4 conteneurs clés, Code (C4) sur le superviseur | PlantUML C4 + Draw.io |
| Flux | Pipeline agentique bout en bout ; arbre de décision à 4 portes ; chaîne d'ingestion de connaissance | Activité |
| Séquence | Intake, Triage, Résolution N1, Investigation N2, Escalade N3, Verdict superviseur, Montée/descente d'autonomie | Séquence |
| État | Cycle de vie du ticket canonique ; cycle de vie d'un document du corpus ; cycle de vie d'un pack | États |
| Déploiement | 4 environnements, topologie K8s, zones réseau | Déploiement |
| Sécurité | Zonage, flux de données, points de contrôle, surface d'attaque | Flux + zonage |
| Données | Modèle canonique, modèle physique, flux de données et rétention | Classes / ERD |
| Urbanisation | Vue en zones, flux inter-quartiers | Draw.io principalement |

Un script de vérification (`tools/diagrams/`) contrôle à chaque porte que les trois formats existent et portent le même identifiant. Un diagramme disponible dans moins de trois formats bloque la revue.

**Sortie de GATE-1** : DAv1 complet et validé, 18 ADR rédigés dont 12 en statut `accepté`, cartographies produites, jeu de diagrammes complet en triple format.

---

## 3. Phase 2 — Analyse fonctionnelle

| Id | Livrable | Contenu | Emplacement |
|---|---|---|---|
| F-01 | Besoins métier | BM-xx : énoncé, valeur, acteur, mesure de succès, priorité MoSCoW | `docs/functional/besoins/` |
| F-02 | Use cases | UC-xx : acteurs, préconditions, scénario nominal, alternatifs, exceptions, postconditions, règles applicables | `docs/functional/use-cases/` |
| F-03 | User stories | US-xxx groupées en épopées, critères d'acceptation Gherkin, rattachement UC | `docs/functional/user-stories/` |
| F-04 | Processus ITIL v4 | Incident (cœur), demande de service (cœur, branche séparée), problème (détection seule), changement (point de sortie), connaissance (alimente), SLM (contrainte) | `docs/functional/itil/` |
| F-05 | BPMN | Processus incident, processus demande, processus d'escalade N3, processus de promotion de connaissance, processus de changement de niveau d'autonomie | `docs/functional/bpmn/` |
| F-06 | Domain Model (DDD) | Contextes bornés, agrégats, entités, objets valeur, invariants, langage ubiquitaire | `docs/functional/domain/` |
| F-07 | Event Storming | Événements de domaine, commandes, politiques, points chauds, vues de lecture | `docs/functional/event-storming/` |
| F-08 | Dictionnaire métier | Terme, définition, synonymes terrain, correspondance ITIL, porteur (socle/pack), pièges | `docs/functional/glossaire/` |
| F-09 | Règles métier | BR-xx : règles de triage, de bascule, de promotion de corpus, de calcul de priorité — exprimées **hors code** | `docs/functional/regles/` |
| F-10 | Matrice RACI | Par processus et par étape, incluant le propriétaire du corpus normatif (Q-05) | `docs/functional/raci/` |

**Contextes bornés pressentis** (à confirmer en F-06) : Ingestion & Normalisation · Triage & Routage · Connaissance (corpus, ingestion, promotion) · Résolution & Exécution d'actions · Supervision & Conformité · Gouvernance & Contexte (packs, autonomie) · Observabilité & Évaluation · Intégration (adaptateurs).

**Sortie de GATE-2** : couverture démontrée — chaque EF du DAv0 (étendue) est couverte par au moins un UC, chaque UC par au moins une US, chaque US par au moins un critère d'acceptation testable.

---

## 4. Phase 3 — Exigences non fonctionnelles

Un fichier par chapitre dans `docs/nfr/`. Chaque exigence porte : identifiant, énoncé, **valeur cible chiffrée**, méthode de mesure, seuil d'alerte, conséquence en cas de dépassement.

| Chapitre | Contenu attendu | Points durs hérités |
|---|---|---|
| NFR-01 Disponibilité | SLO par service, fenêtres de maintenance, dépendances externes, budget d'erreur | Fail-closed : indisponibilité = file d'attente, jamais action approximative (ENF-10) |
| NFR-02 Performance | Latence par classe de modèle (S < 2 s, M < 10 s, L < 90 s async), bout en bout triage < 60 s (ENF-03), débit | Budget jetons et coût par ticket (ENF-04) |
| NFR-03 Scalabilité | Dimensionnement par volumétrie, élasticité, points de contention (index, passerelle modèles) | **Volumétrie à confirmer** — voir §8 |
| NFR-04 Sécurité | Authentification, autorisation, secrets, chiffrement, anti-injection (0 échec toléré, ENF-07), étanchéité par construction (ENF-08), moindre privilège | R-06, R-07 |
| NFR-05 Observabilité | Traces distribuées, corrélation ticket ↔ appels modèles, métriques, journalisation, coût | KPI IA |
| NFR-06 Auditabilité | 100 % des décisions tracées (ENF-01), rejeu à l'identique (EF-09), déterminisme du routage (ENF-02), rétention | Q-06 |
| NFR-07 Résilience | Modes dégradés par composant, files, reprises, idempotence, circuit breakers, plan de continuité | ENF-10 |
| NFR-08 Maintenabilité | Modularité, couverture de tests, dette, versionnement des prompts et packs, documentation vivante | R-05 |
| NFR-09 Portabilité | Abstraction fournisseur modèle, ITSM, index, hébergeur (ENF-09) | ADR-018 |
| NFR-10 Réversibilité fournisseur | Plan de sortie par fournisseur, formats d'export, délai de bascule, test de réversibilité | ENF-09 |
| NFR-11 Souveraineté | Localisation traitement/index/trace, classes de modèles autorisées par pack client, sous-traitance | Q-03 |
| NFR-12 Qualité IA | Accord ≥ 85 % (ENF-05), calibration écart < 10 pts (ENF-06), taux d'hallucination détectée, seuils de gating | DAv0 §Mesure |
| NFR-13 Conformité | RGPD, AI Act, accessibilité RGAA du frontend, exigences contractuelles | ADR-016 |

**Livrable transverse** : `docs/nfr/kpi/` — catalogue complet des KPI demandés (métier, IA, exploitation), avec pour chacun : définition formelle, formule, source de donnée, fréquence, cible, seuil d'alerte, dashboard d'affichage, et **usage dans le pilotage du curseur d'autonomie**.

**Sortie de GATE-3** : aucune exigence sans valeur chiffrée ni méthode de mesure. Une exigence non mesurable est refusée.

---

## 5. Phase 4 — Spec Driven Development (porte de code)

Produite **par incréments alignés sur les lots**. Chaque incrément est complet pour son périmètre avant tout code correspondant.

| Id | Livrable | Contenu | Emplacement |
|---|---|---|---|
| S-01 | Spécifications fonctionnelles détaillées | SFD par module : comportement exhaustif, règles, cas limites, messages, états d'erreur, comportement en mode dégradé | `docs/technical/specs-fonctionnelles/` |
| S-02 | Spécifications techniques détaillées | STD par module : structure interne, dépendances, algorithmes, configuration, points d'extension | `docs/technical/specs-techniques/` |
| S-03 | Contrats API | OpenAPI 3.1 : API d'ingestion, API console technicien, API de gouvernance (packs, autonomie), API d'administration du corpus, API d'observabilité | `docs/technical/api/openapi/` |
| S-04 | Contrats d'événements | AsyncAPI 3 + JSON Schema par événement, conventions de nommage, garanties, versionnement, compatibilité ascendante | `docs/technical/events/` |
| S-05 | Contrats d'agents | Les 7 contrats du DAv0 formalisés : `ticket-canonique`, `decision-triage`, `plan-action`, `categorie-n2`, `diagnostic`, `dossier-escalade`, `verdict` — champs `confiance`, `justification`, `sources` obligatoires partout | `docs/technical/contracts/` |
| S-06 | Modèles de données | Modèle logique, modèle physique par stockage, indexation, partitionnement par pack, rétention, migrations | `docs/technical/data-model/` |
| S-07 | JSON Schema | Schémas versionnés de tous les contrats + schémas des **packs** (client, projet) + schéma de la trace d'audit | `docs/technical/schemas/` |
| S-08 | Diagrammes de séquence | Un par cas d'usage et par mode dégradé | `docs/technical/sequences/` |
| S-09 | Diagrammes d'état | Ticket, document du corpus, pack, niveau d'autonomie, action | `docs/technical/etats/` |
| S-10 | Spécification du superviseur | Les 7 contrôles spécifiés individuellement : entrée, algorithme, verdict, motif, journalisation, cas de test | `docs/technical/superviseur/` |
| S-11 | Spécification RAG | Stratégie de découpage, embeddings, index hybride, pondération, reranking, filtrage par pack, format de citation, cycle de validité | `docs/technical/rag/` |
| S-12 | Registre de prompts | Prompts versionnés, distincts du code, avec entrées/sorties attendues et jeu d'évaluation attaché | `docs/technical/prompts/` |
| S-13 | Plan de tests | Stratégie, pyramide, tests de contrat, tests d'intégration, tests de sécurité (injection), tests de rejeu | `docs/technical/tests/` |
| S-14 | Plan d'évaluation IA | Jeux de cas annotés (composition du DAv0 : 8 pauvres / 6 bien rédigés / 3 limites / 2 doublons / 1 injection, puis extension), matrice de confusion, calibration, gating CI | `evaluation/` |
| S-15 | Spécification frontend | Parcours, écrans, états, composants, design system, accessibilité, affichage obligatoire des sources et de la mention IA | `docs/technical/frontend/` |

**Sortie de GATE-4** : pour le périmètre de l'incrément — schémas JSON validés par outillage, OpenAPI/AsyncAPI validant sans erreur, chaque règle métier associée à au moins un cas de test, chaque exigence non fonctionnelle associée à au moins une mesure automatisée. **Seule cette porte ouvre `/src`.**

---

## 6. Phases 5 à 7 — Implémentation, DevSecOps, Déploiement

### 6.1 Phase 5 — Implémentation

| Bloc | Contenu | Conditionné par |
|---|---|---|
| Backend | Stack tranchée par ADR-009 (Python/FastAPI ou Java 21/Spring Boot 3.5) ; architecture hexagonale stricte ; modules = contextes bornés de F-06 | GATE-4 |
| Couche IA | Registre d'agents, port LLM, routage par classe S/M/L, budget et plafonds, sorties structurées imposées | ADR-002 |
| RAG | Ingestion, anonymisation, clustering, synthèse, promotion, index hybride, reranking, citations | ADR-003 |
| Superviseur | 7 contrôles, fail-closed, verdicts tracés | ADR-005, S-10 |
| Frontend | Angular (dernière version) : console technicien, validation en un clic, dashboards, administration des packs et du curseur | ADR-015 |
| Infrastructure applicative | Dockerfiles, images durcies, Helm charts, manifestes K8s | ADR-010 |

Organisation de `/src` : par contexte borné, pas par couche technique. Le socle, les packs et les adaptateurs sont physiquement séparés — la frontière DEC-01 est visible dans l'arborescence, donc contrôlable en revue.

### 6.2 Phase 6 — DevSecOps

Pipelines (GitHub Actions ou GitLab CI — décision en ADR dédiée), SonarQube et qualité, SAST, SCA, secrets scanning, DAST, signature d'artefacts et SBOM, stratégie de branches et de versionnement (SemVer socle + SemVer packs + version de prompts), promotion d'artefacts entre environnements, **gating IA** (une régression d'évaluation bloque la livraison et redescend le curseur d'autonomie).

### 6.3 Phase 7 — Déploiement et exploitation

Ansible pour dev / qualification / préproduction / production : inventaires, variables (avec chiffrement des secrets), rôles, playbooks, procédures de bascule et de retour arrière. Documentation d'exploitation : runbooks, modes dégradés, astreinte, sauvegarde et restauration, gestion des incidents de la plateforme elle-même, procédure de réversibilité fournisseur.

---

## 7. Repository cible

Structure conforme à la demande, étendue là où le contenu l'exige. À créer à la racine de `D:\Programmation\ITIL Analyzer\ItilAnalyzer`.

```
ItilAnalyzer/
├── README.md
├── CHANGELOG.md
├── CONTRIBUTING.md
├── LICENSE
├── .editorconfig
├── .gitignore
├── .gitattributes
│
├── docs/
│   ├── README.md                      # carte de navigation documentaire
│   ├── 00-plan/
│   │   ├── PLAN-00-livrables-et-repository.md      # ce document
│   │   ├── charte-tracabilite.md
│   │   ├── conventions-nommage.md
│   │   └── portes-de-validation.md
│   │
│   ├── architecture/
│   │   ├── DAv0/                      # document source fourni, figé
│   │   ├── DAv1/
│   │   │   ├── 00-sommaire.md
│   │   │   ├── 01-introduction-perimetre.md
│   │   │   ├── 02-contexte-parties-prenantes.md
│   │   │   ├── 03-principes-directeurs.md
│   │   │   ├── 04-architecture-logique.md
│   │   │   ├── 05-architecture-applicative.md
│   │   │   ├── 06-architecture-technique.md
│   │   │   ├── 07-architecture-deploiement.md
│   │   │   ├── 08-architecture-securite.md
│   │   │   ├── 09-architecture-donnees.md
│   │   │   ├── 10-architecture-rag.md
│   │   │   ├── 11-superviseur.md
│   │   │   ├── 12-curseur-autonomie.md
│   │   │   ├── 13-architecture-observabilite.md
│   │   │   ├── 14-architecture-devsecops.md
│   │   │   ├── 15-reversibilite-portabilite.md
│   │   │   ├── 16-risques-trajectoire.md
│   │   │   └── annexes/
│   │   │       └── A1-instanciation-pilote.md      # SEUL endroit citant le client pilote
│   │   ├── urbanisation/
│   │   ├── cartographie/
│   │   │   ├── fonctionnelle/
│   │   │   ├── applicative/
│   │   │   └── technique/
│   │   └── vues/                      # vues transverses et fiches de synthèse
│   │
│   ├── adr/
│   │   ├── README.md                  # index, statuts, graphe de dépendances
│   │   ├── template-adr.md
│   │   └── ADR-001..ADR-018-*.md
│   │
│   ├── functional/
│   │   ├── besoins/
│   │   ├── use-cases/
│   │   ├── user-stories/
│   │   ├── itil/
│   │   ├── bpmn/
│   │   ├── domain/
│   │   ├── event-storming/
│   │   ├── regles/
│   │   ├── raci/
│   │   └── glossaire/
│   │
│   ├── nfr/
│   │   ├── 01-disponibilite.md  ...  13-conformite.md
│   │   ├── slo-sli.md
│   │   ├── budgets-erreur.md
│   │   └── kpi/
│   │       ├── kpi-metier.md
│   │       ├── kpi-ia.md
│   │       ├── kpi-exploitation.md
│   │       └── catalogue-kpi.yaml     # source unique, consommée par les dashboards
│   │
│   ├── technical/
│   │   ├── specs-fonctionnelles/
│   │   ├── specs-techniques/
│   │   ├── api/openapi/
│   │   ├── events/                    # AsyncAPI
│   │   ├── contracts/                 # contrats d'agents
│   │   ├── schemas/                   # JSON Schema versionnés
│   │   ├── data-model/
│   │   ├── sequences/
│   │   ├── etats/
│   │   ├── superviseur/
│   │   ├── rag/
│   │   ├── prompts/
│   │   ├── frontend/
│   │   └── tests/
│   │
│   ├── drawio/                        # tous les .drawio, miroir de l'arbo docs
│   │   ├── architecture/
│   │   ├── functional/
│   │   └── technical/
│   │
│   ├── plantuml/                      # tous les .puml, miroir de l'arbo docs
│   │   ├── architecture/
│   │   ├── functional/
│   │   └── technical/
│   │
│   ├── traceability/
│   │   ├── matrice-tracabilite.md     # généré
│   │   ├── couverture-exigences.md    # généré
│   │   └── registre-decisions.md      # DEC-xx ↔ ADR-xxx
│   │
│   ├── quality/
│   │   ├── grille-revue.md
│   │   └── definition-of-done.md
│   │
│   └── exploitation/
│       ├── runbooks/
│       ├── modes-degrades.md
│       ├── astreinte.md
│       └── reversibilite.md
│
├── src/                               # VERROUILLÉ jusqu'à GATE-4
│   ├── core/                          # socle générique — ne connaît ni client ni ITSM
│   │   ├── domain/
│   │   ├── application/
│   │   │   ├── orchestration/         # machine à états
│   │   │   ├── agents/                # registre d'agents
│   │   │   ├── supervision/           # 7 contrôles
│   │   │   ├── knowledge/             # RAG : exploitation
│   │   │   └── governance/            # packs, curseur d'autonomie
│   │   ├── ports/                     # ports hexagonaux (LLM, ITSM, doc, IAM, store)
│   │   └── contracts/                 # schémas partagés
│   ├── adapters/
│   │   ├── itsm/
│   │   │   ├── mock/
│   │   │   ├── mail/
│   │   │   └── <itsm-client>/
│   │   ├── documentaire/
│   │   ├── iam/
│   │   ├── llm/
│   │   └── observability/
│   ├── packs/
│   │   ├── _templates/                # gabarits de pack client et pack projet
│   │   ├── client-<nom>/
│   │   └── projet-<nom>/
│   ├── api/                           # exposition HTTP
│   ├── workers/                       # traitements asynchrones, ingestion
│   └── frontend/                      # Angular
│
├── tests/
│   ├── unit/
│   ├── integration/
│   ├── contract/
│   ├── e2e/
│   ├── security/                      # injection, étanchéité, fail-closed
│   └── fixtures/
│
├── evaluation/
│   ├── datasets/                      # cas annotés, anonymisés
│   ├── scenarios/
│   ├── metrics/                       # confusion, calibration, hallucination
│   ├── reports/
│   └── gating/                        # seuils bloquants CI
│
├── observability/
│   ├── dashboards/                    # métier ×4, technique, IA
│   ├── alerts/
│   ├── slo/
│   └── audit/                         # format et outillage de la trace rejouable
│
├── security/
│   ├── threat-model/                  # STRIDE + menaces spécifiques LLM
│   ├── policies/
│   ├── secrets/                       # conventions, jamais de secret
│   ├── compliance/                    # RGPD, AI Act, registre de traitement
│   └── pentest/
│
├── deployment/
│   ├── docker/
│   ├── helm/
│   ├── kubernetes/
│   └── ansible/
│       ├── inventories/
│       │   ├── dev/  qualif/  preprod/  prod/
│       ├── group_vars/
│       ├── host_vars/
│       ├── roles/
│       └── playbooks/
│
├── ci/                                # pipelines, politiques de qualité
│
└── tools/                             # outillage documentaire, autorisé avant GATE-4
    ├── diagrams/                      # vérification triple format, rendu
    ├── traceability/                  # génération de la matrice
    └── schema-validation/
```

**Trois règles d'arborescence non négociables**

1. `src/core/` ne contient aucune référence à un client, un ITSM ou un vocabulaire métier spécifique. Contrôle automatisé en CI (liste de termes interdits alimentée par les packs).
2. `docs/drawio/` et `docs/plantuml/` **miroitent** l'arborescence documentaire : un schéma est retrouvable par son chemin, dans n'importe quel format.
3. `src/` est verrouillé jusqu'à GATE-4 (fichier `src/.locked` + contrôle CI), afin que la règle absolue soit appliquée par l'outillage et non par la discipline.

---

## 8. Questions bloquantes avant d'attaquer la phase 1

### 8.1 Questions héritées du DAv0, toujours ouvertes

Q-01 (écriture via mail dans l'ITSM), Q-02 (historique des réaffectations extractible), Q-03 (classes de modèles autorisées : hébergement, localisation, fournisseurs), Q-04 (accès lecture aux logs applicatifs), Q-05 (propriétaire nommé du corpus normatif), Q-06 (rétention de la trace et base légale), Q-07 (validation des poids de la matrice de l'Arbitrage 1).

Le DAv1 est rédigeable sans leur réponse, **à condition** que chaque dépendance soit assortie d'un repli écrit. Q-03 et Q-06 sont les deux seules qui, sans réponse, laissent l'architecture de sécurité et l'architecture des données en hypothèse plutôt qu'en décision.

### 8.2 Questions nouvelles, nécessaires au dimensionnement

Aucune exigence de scalabilité ni de disponibilité n'est chiffrable sans ces éléments. Ils manquent au DAv0.

| Id | Question | Ce qu'elle bloque |
|---|---|---|
| Q-08 | Volumétrie : tickets/jour, pic horaire, saisonnalité, nombre de techniciens simultanés sur la console | NFR-02, NFR-03, dimensionnement, budget LLM |
| Q-09 | Volumétrie documentaire : nombre de documents et de tickets historiques à indexer, taux de renouvellement | Dimensionnement de l'index, coût d'embedding, NFR-03 |
| Q-10 | Cible de disponibilité et plage de service (24/7 ? heures ouvrées ? astreinte ?) | NFR-01, topologie de déploiement, coût d'infrastructure |
| Q-11 | Cible d'hébergement : cloud public, cloud souverain, on-premise, K8s existant ou à construire | DAv1-07, NFR-11, ADR-013 |
| Q-12 | Nombre de clients et de projets à servir à 12 / 24 mois | Mono-tenant vs multi-tenant (ADR-013), coût de maintenance |
| Q-13 | Budget cible par ticket et enveloppe mensuelle LLM | ENF-04, routage par classes, plafonds |
| Q-14 | Équipe : taille, compétences dominantes (Python vs Java), présence d'une compétence MLOps | ADR-009, ADR-011, faisabilité de la trajectoire |
| Q-15 | Contrainte de délai sur le premier jalon démontrable | Séquencement P1→P4, taille des incréments |

---

## 9. Séquence de production proposée

| Étape | Production | Volume indicatif |
|---|---|---|
| 1 | Validation du présent plan + réponses aux questions §8.2 | — |
| 2 | Squelette du repository + README + conventions + charte de traçabilité + templates (ADR, SFD, STD, diagrammes) | arborescence complète, fichiers d'amorçage |
| 3 | DAv1 chapitres 01→06 (introduction → architecture technique) + diagrammes C4 en triple format | bloc 1 |
| 4 | DAv1 chapitres 07→12 (déploiement, sécurité, données, RAG, superviseur, autonomie) + diagrammes associés | bloc 2 |
| 5 | DAv1 chapitres 13→17 + ADR-001 à ADR-010 | bloc 3 |
| 6 | ADR-011 à ADR-018 + urbanisation + 3 cartographies | bloc 4 |
| 7 | **GATE-1** — revue d'architecture | — |
| 8 | Phase 2 complète (analyse fonctionnelle) | 4 blocs |
| 9 | **GATE-2**, puis phase 3 (ENF + KPI) | 2 blocs |
| 10 | **GATE-3**, puis phase 4 incrément « lot 1 » | 4 blocs |
| 11 | **GATE-4** — ouverture de `/src` | — |

Chaque bloc est livré sous forme de fichiers prêts à déposer dans le repository, avec leur traçabilité renseignée.

---

## 10. Trois points sur lesquels j'attire votre attention dès maintenant

1. **Le chemin critique n'est pas documentaire.** Le DAv0 le dit sans détour : les runbooks n'existent nulle part sous forme exploitable, et leur formalisation conditionne toute résolution automatique. Le plan documentaire ci-dessus peut se dérouler intégralement sans qu'un seul runbook soit promu en corpus normatif — et le produit n'aura alors aucune valeur au-delà du niveau 1 d'autonomie. La chaîne d'ingestion (P4/S-11) et le rituel de promotion doivent démarrer en parallèle de la phase 1, pas après.

2. **ADR-009 (Python vs Java) a plus de conséquences qu'il n'y paraît.** Il conditionne l'écosystème d'orchestration agentique, la testabilité du port LLM et la disponibilité des bibliothèques d'évaluation. Je le traite en priorité P1, et je le tranche sur matrice pondérée avec le critère « compétences d'exploitation disponibles » explicitement pesé — c'est-à-dire que la réponse à Q-14 détermine largement la décision.

3. **Le volume documentaire demandé est considérable.** Neuf vues d'architecture, dix-huit ADR, quinze familles de spécifications, tout en triple format de schéma. Produit intégralement avant la moindre ligne de code, cela représente un effort qui doit être assumé comme tel. Si une contrainte de délai existe sur le premier jalon (Q-15), je recommande de réduire la **profondeur** de P4 au seul périmètre du lot 1 — jamais de réduire P1 à P3, qui sont ce qui rend le reste réutilisable.

---

*Fin du document PLAN-00. Aucune rédaction de document d'architecture n'est engagée avant validation de ce plan.*
