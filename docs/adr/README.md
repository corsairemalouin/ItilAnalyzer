# Registre des ADR

Format : MADR simplifié. Chaque ADR porte `Amont:` et `Aval:`. Un ADR n'est jamais modifié une fois accepté — il est **remplacé** par un nouvel ADR qui le référence.

| Id | Sujet | Décision | Statut |
|---|---|---|---|
| ADR-001 | Vision agentique | Pipeline multi-agents orchestré par **code déterministe**. Aucune transition décidée par un appel de modèle. Le RAG est une brique, pas l'architecture. | accepté (DEC-00) |
| ADR-002 | Stratégie LLM | Port LLM + couche d'abstraction multi-fournisseur. Trois classes S/M/L affectées par étape, escalade plafonnée à une par étape, déclenchée par le code. Classe autorisée = paramètre du pack client. | accepté (DEC-04) |
| ADR-003 | Architecture RAG | Corpus à deux étages (indicatif automatique / normatif promu par humain nommé), index hybride, reranking, citation obligatoire, filtrage par pack appliqué à l'index. | accepté (DEC-03) |
| ADR-004 | Observabilité et auditabilité | Trois piliers séparés : trace d'audit rejouable, observabilité technique, banc d'évaluation. Trace écrite **avant** l'effet de bord. | accepté |
| ADR-005 | Human In The Loop | Curseur 0→4 par couple catégorie × action, démarrage systématique à 0, montée sous 5 conditions cumulatives, descente automatique par le code. | accepté (DEC-05) |
| ADR-006 | Adaptateurs | Interfaces uniques par famille (ITSM, documentaire, IAM, canaux). Un changement d'outil = un nouvel adaptateur, jamais une réécriture des agents. | accepté (DEC-07) |
| ADR-007 | Socle + packs | Frontière normative socle/pack client/pack projet. Fusion : intersection des permissions, maximum des seuils, conflit = blocage au démarrage. | accepté (DEC-01) |
| ADR-008 | Modèle canonique | Noyau ITIL restreint (incident, demande, problème) à l'intérieur, vocabulaire terrain en surface, mapping déclaratif porté par les packs. | accepté (DEC-02) |
| **ADR-009** | **Stack du moteur métier** | **Python 3.12 / FastAPI, stack unique. Pydantic strict en frontière de contrat, `mypy --strict` bloquant sur le socle.** | **accepté v0.2** |
| ADR-010 | Découpage de déploiement | Monolithe modulaire à frontières explicites. Microservices écartés : aucune charge ne les justifie (< 0,01 req/s au pic). | accepté (DEC-08) |
| ADR-011 | Orchestration | Orchestration codée pour le cœur agentique. Machine à états en code pur, sans framework d'agents. BPM réservé à une éventuelle phase ultérieure pour les tâches humaines. | accepté |
| ADR-012 | Stockage | PostgreSQL transactionnel + trace append-only + index hybride colocalisé en v0. `SearchPort` abstrait pour permettre un moteur dédié. | **à figer après Q-09** |
| ADR-013 | Multi-tenance | **Mono-tenant en v0** (isolation physique = ENF-08 par construction), cloisonnement logique par pack néanmoins implémenté et testé pour rendre le multi-tenant possible plus tard. | accepté, à réviser au 2ᵉ client |
| ADR-014 | Événements | Événements de domaine versionnés sur les transitions d'état, idempotence par identifiant d'exécution, catalogue AsyncAPI. | proposé |
| ADR-015 | Frontend | Angular, console technicien centrée sur la proposition contestable en un clic, affichage obligatoire des sources et de la confiance, accessibilité. | proposé |
| ADR-016 | Conformité RGPD / AI Act | Anonymisation avant stockage et avant tout appel ; mention IA injectée par le composant d'émission ; supervision humaine documentée par le superviseur lui-même. | **bloqué par Q-06** |
| ADR-017 | Évaluation et non-régression IA | Banc d'évaluation livrable du lot 1. Gating CI bloquant sur ENF-05, ENF-06, ENF-07. Tout changement de modèle, prompt ou pack invalide les évaluations et redescend le curseur. | accepté |
| ADR-018 | Réversibilité et souveraineté | Test de réversibilité par axe (modèle, ITSM, index, hébergeur, données). Localisation vérifiée au démarrage à partir du pack client. | **bloqué par Q-03** |
| ~~ADR-019~~ | ~~Frontière noyau Java / satellite Python~~ | **Annulé** — sans objet depuis ADR-009 v0.2. | annulé |
