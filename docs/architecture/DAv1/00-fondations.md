# DAv1 — Dossier d'Architecture détaillé · Fondations (ch. 00 à 05)

**Produit** : ITIL Autonomous Incident Analyzer (IAA)
**Référence** : `IAA-DA-V1` · v0.1 · statut : **pour revue**
**Amont** : `ITILANALYZER-DA-V0` (DEC-00→08, EF-01→10, ENF-01→10, R-01→11, Q-01→07), `PLAN-00`, `PLAN-00-A1`, `ADR-009 v0.2`
**Aval** : phases P2 (analyse fonctionnelle), P3 (ENF), P4 (spécifications)

---

## 00. Sommaire du DAv1

| Fichier | Chapitres |
|---|---|
| `00-fondations.md` | 01 Périmètre · 02 Contexte · 03 Principes · 04 Architecture logique · 05 Architecture applicative |
| `06-technique-deploiement.md` | 06 Architecture technique · 07 Architecture de déploiement |
| `08-securite-donnees.md` | 08 Architecture de sécurité · 09 Architecture des données |
| `10-rag-superviseur-autonomie.md` | 10 Architecture RAG · 11 Superviseur · 12 Curseur d'autonomie |
| `13-observabilite-devsecops.md` | 13 Observabilité · 14 DevSecOps · 15 Réversibilité · 16 Risques · 17 Annexes |
| `annexes/A1-instanciation-pilote.md` | Instanciation du client pilote — **seul** document citant l'ITSM pilote |

---

## 01. Périmètre

### 01.1 Ce que le produit fait

| Id | Capacité | Niveau concerné |
|---|---|---|
| CAP-01 | Réceptionner un ticket par canal abstrait et le normaliser en modèle canonique | Transverse |
| CAP-02 | Scorer la complétude et générer au plus 3 questions de retour au demandeur | N1 |
| CAP-03 | Qualifier : type ITIL, catégorie, impact, urgence, priorité, équipe, niveau cible | N1 |
| CAP-04 | Détecter un doublon d'un incident déjà ouvert | N1 |
| CAP-05 | Détecter un ticket mal dirigé par écart entre chemin déclaré et contenu réel | N1 |
| CAP-06 | Rechercher la connaissance associée, avec citations obligatoires | Transverse |
| CAP-07 | Appliquer un runbook normatif sur actions en liste blanche | N1 |
| CAP-08 | Produire un diagnostic argumenté et sourcé, en lecture seule | N2 |
| CAP-09 | Constituer un dossier d'escalade complet | N3 |
| CAP-10 | Superviser toute sortie avant effet de bord | Transverse |
| CAP-11 | Produire une trace d'audit rejouable et versionnée | Transverse |
| CAP-12 | Mesurer la performance métier, IA et d'exploitation | Transverse |
| CAP-13 | Construire et entretenir le corpus de connaissance | Transverse |
| CAP-14 | Gouverner le niveau d'autonomie par couple catégorie × action | Transverse |

### 01.2 Ce que le produit ne fera jamais

Hors périmètre de façon permanente, quelle que soit l'évolution du produit :

- Devenir l'ITSM : le produit ne remplace aucun outil de ticketing, il s'y branche.
- Clôturer un ticket sans validation humaine.
- Exécuter une action non réversible sans accord explicite.
- Décider de façon autonome sur un ticket sécurité, confidentiel ou à impact vital.
- Gouverner ITIL (CAB, rôles, cycle de vie du service) : le produit consomme ITIL, il ne l'administre pas.

### 01.3 Principe de généricité — Core Platform + Context Packs

| Élément | Core Platform | Pack client | Pack projet | Adaptateur |
|---|---|---|---|---|
| Machine à états, superviseur, contrats, port LLM, moteur RAG | ✔ | | | |
| Garde-fous absolus, détection d'injection | ✔ (définition) | durcissement seul | durcissement seul | |
| Cartographie applicative, vocabulaire, entités | | ✔ | compléments | |
| Politique de sécurité, seuils, niveaux d'autonomie | valeurs par défaut | ✔ | durcissement seul | |
| Taxonomie, runbooks, catalogue N2, liste blanche | gabarits | | ✔ | |
| Intégration ITSM / documentaire / IAM | interface | configuration | | ✔ implémentation |

**Règle de fusion des packs** — appliquée au démarrage, échec bloquant en cas de conflit :

1. Permissions : **intersection**. Un pack ne peut jamais élargir un droit.
2. Seuils de prudence : **maximum**. Le plus strict gagne.
3. Vocabulaire : le plus spécifique gagne.
4. Conflit non résolvable : blocage explicite au démarrage, jamais de résolution silencieuse.

**Contrôle automatisé (parade R-08)** : la CI échoue si un terme du vocabulaire client, un identifiant d'ITSM nommé ou une valeur de taxonomie projet apparaît dans `src/core/`. La liste des termes interdits est dérivée des packs chargés.

---

## 02. Contexte et parties prenantes

### 02.1 Acteurs génériques

| Acteur | Attente | Critère d'acceptation vérifiable |
|---|---|---|
| Technicien N1 | Moins de tri manuel, contexte prêt à l'emploi | Proposition affichée, contestable en un clic, traçée |
| Technicien N2 | Un dossier déjà corrélé à l'ouverture | Hypothèse sourcée, aucune action subie |
| Expert N3 | Dossier d'escalade complet | Tous les identifiants externes présents |
| Responsable de service | Tenue du SLA, aucun incident imputé à l'IA | Curseur d'autonomie sous contrôle et tracé |
| RSSI / client | Auditabilité, maîtrise des données | Trace rejouable, mention IA systématique |
| Éditeur du socle | Réutilisation sur d'autres comptes | Part de socle réutilisée mesurée au 2ᵉ projet |
| Exploitant | Système observable, dégradation maîtrisée | Dashboards technique + métier disponibles |
| Demandeur | Savoir qu'une IA est intervenue | **Contrainte non négociable, portée par l'architecture** |

### 02.2 Contrainte de transparence

Toute communication sortante vers un demandeur porte la mention qu'elle a été produite ou assistée par une IA. Cette mention est **injectée par le composant d'émission**, pas par un prompt : elle est structurellement non contournable, et son absence est un défaut bloquant détecté par le superviseur (contrôle n° 2, garde-fou absolu).

---

## 03. Principes directeurs

| Principe | Traduction architecturale concrète | Vérifiable par |
|---|---|---|
| **ITIL v4** | Modèle canonique aligné ITIL à l'intérieur, vocabulaire terrain en surface, mapping déclaratif porté par les packs | Schéma du modèle canonique |
| **Human In The Loop** | Aucune transition à effet de bord sans point de reprise humain ; le niveau d'autonomie est un paramètre lu à chaque décision | Tests de bout en bout par niveau |
| **Explainable AI** | `confiance`, `justification` et `sources` sont obligatoires dans **tous** les contrats d'agents | Validation de schéma |
| **Auditability First** | Trace écrite **avant** l'effet de bord, jamais après ; rejeu à l'identique garanti | Test de rejeu sur 100 % des cas |
| **Security By Design** | Cloisonnement par pack appliqué à l'index et au stockage, pas au prompt ; anonymisation avant tout appel de modèle | Test d'étanchéité inter-packs |
| **Spec Driven Development** | Contrats et schémas produits et validés avant le code | Porte GATE-4 |
| **DevSecOps** | Qualité, SAST, SCA, DAST et gating d'évaluation IA bloquants en CI | Pipeline |
| **Architecture hexagonale** | Le domaine ne dépend d'aucun fournisseur ; tout accès externe passe par un port | Test d'architecture (import linter) |
| **Domain Driven Design** | Modules = contextes bornés, langage ubiquitaire dans le dictionnaire métier | Revue d'architecture |
| **Event Driven Architecture** | Les transitions d'état émettent des événements de domaine versionnés | Catalogue AsyncAPI |

### 03.1 Principe surplombant

> Ce qui départage les architectures n'est pas la qualité de la réponse produite, mais la capacité à expliquer une décision six mois plus tard, en audit, et à la rejouer à l'identique.

Conséquence directe et non négociable : **aucune transition d'état n'est décidée par un appel de modèle**. Les modèles produisent des sorties structurées ; le code déterministe décide, à partir des champs de ces sorties. ENF-02 (déterminisme du routage) est ainsi garanti par construction, pas par réglage.

---

## 04. Architecture logique

### 04.1 Découpage en couches

| Couche | Responsabilité | Ce qu'elle isole | Rythme de changement |
|---|---|---|---|
| **L1 — Canaux & adaptateurs** | Recevoir, publier, traduire les formats externes | Les outils clients et modalités d'intégration | Rapide |
| **L2 — Processus métier** | Porter les états, règles de routage, reprises humaines | Le métier vis-à-vis des technologies IA | Lent |
| **L3 — IA & RAG** | Analyser, rechercher, citer, produire des sorties structurées | Modèles, prompts, index, fournisseurs | Rapide |
| **L4 — Supervision & actions** | Évaluer le risque, autoriser, exécuter, auditer | La génération vis-à-vis des effets de bord | Très lent |
| **L5 — Gouvernance** | Versionner, mesurer, évaluer, améliorer | Le cycle de vie vis-à-vis du runtime | Moyen |

Règle de dépendance : **L2 ne dépend que d'interfaces**. Un changement d'ITSM ou de fournisseur de modèle ne touche que L1 ou L3, jamais L2 ni L4.

### 04.2 Les sept agents

| Agent | Rôle | Entrée | Contrat de sortie | Droit d'écriture | Classe modèle | Condition de bascule |
|---|---|---|---|---|---|---|
| **Intake** | Normalise, déduplique, score la complétude | Message de canal brut, cartographie | `ticket-canonique` | **Oui** — commentaire | S (→ M si ambigu) | Complétude < 0,5 → 3 questions |
| **Triage** | Type, catégorie, impact, urgence, priorité, équipe, niveau cible | Ticket canonique, taxonomie, chemin déclaré | `decision-triage` | Aucun | M (→ L une fois si cas limite) | Confiance < seuil → proposition affichée |
| **Résolveur N1** | Applique un runbook normatif, rédige la réponse | Corpus normatif, liste blanche | `plan-action` | **Oui** — liste blanche seule | M | Aucun runbook applicable → N2 |
| **Classifieur N2** | Le sujet est-il dans le catalogue N2 traitable ? | Catalogue N2, historique | `categorie-n2` | Aucun | S | Hors catalogue → N3 |
| **Investigateur N2** | Corrèle logs et historique, formule une hypothèse | Logs, cas similaires, corpus indicatif | `diagnostic` | Aucun (lecture seule) | L | Non concluant → N3 |
| **Préparateur N3** | Constitue le dossier d'escalade | Historique complet du ticket | `dossier-escalade` | Aucun | M | Dépôt, identifiants externes conservés |
| **Superviseur** | Contrôle toute sortie avant effet de bord | Sortie d'agent, garde-fous, contexte fusionné | `verdict` | Aucun — **bloque** | Code + S | Un contrôle en échec → blocage motivé |

**Deux agents seulement écrivent.** C'est une propriété d'architecture, vérifiée par test : tout autre agent tentant un accès en écriture au port ITSM lève une erreur de conception, détectée en CI.

### 04.3 Arbre de décision — quatre portes

```
Ticket entrant
   │
   ├─ Court-circuit permanent ──► marqueur sécurité | confidentiel | impact vital | injection
   │                              → sortie immédiate du chemin automatique, quel que soit le niveau
   │
   ▼
Porte 1 — Complétude ≥ 0,5 ?
   non → 3 questions max au demandeur (auto, question seule)
   oui ▼
Porte 2 — Catégorie sûre ET confiance ≥ seuil ?
   non → proposition affichée au technicien, jamais appliquée
   oui ▼
Porte 3 — Un runbook NORMATIF couvre-t-il ce cas ?
   non → Investigateur N2 (lecture seule) → dossier N3 si non concluant
   oui ▼
Porte 4 — Action en liste blanche ET réversible ?
   non → validation humaine, action préparée, validée en un clic
   oui ▼
Résolution N1 automatique (sous réserve du niveau d'autonomie du couple catégorie × action)
```

Chaque « non » est une **sortie utile** : le ticket reste traité, par un humain, avec un contexte déjà constitué. Toutes les conditions sont évaluées par du code déterministe à partir des champs des contrats JSON.

### 04.4 Points de reprise humaine

| Point | Nature | Niveau minimal requis |
|---|---|---|
| Retour au demandeur (questions) | Automatique, question seule, sans effet de bord métier | 3 |
| Proposition de triage | Le technicien décide | 1 |
| Diagnostic N2 | Le technicien exécute | 1 |
| Action hors liste blanche | Validation explicite obligatoire | 2, jamais plus |
| Clôture | Humain uniquement, sans exception | 0 par garde-fou absolu |

---

## 05. Architecture applicative

### 05.1 Contextes bornés (DDD)

| Contexte | Responsabilité | Agrégat racine | Module `src/core/` |
|---|---|---|---|
| **Ingestion** | Réception, normalisation, déduplication, complétude | `Ticket` | `ingestion` |
| **Triage** | Classification, priorisation, routage, détection de mal-dirigé | `DecisionTriage` | `triage` |
| **Résolution** | Application de runbook, plan d'action, rédaction de réponse | `PlanAction` | `resolution` |
| **Investigation** | Corrélation, hypothèse, diagnostic | `Diagnostic` | `investigation` |
| **Escalade** | Constitution et dépôt du dossier | `DossierEscalade` | `escalation` |
| **Connaissance** | Ingestion, indexation, recherche, promotion, validité | `Document`, `Runbook` | `knowledge` |
| **Supervision** | Sept contrôles, verdicts, garde-fous | `Verdict` | `supervision` |
| **Gouvernance** | Packs, fusion, niveaux d'autonomie, seuils | `Pack`, `NiveauAutonomie` | `governance` |
| **Observabilité** | Trace d'audit, métriques, évaluation | `Trace`, `RunEvaluation` | `observability` |

### 05.2 Ports (hexagonal)

| Port | Opérations | Adaptateurs prévus |
|---|---|---|
| `ITSMPort` | `list`, `read`, `comment`, `route`, `escalate` | `mock` (mémoire), `mail`, `<itsm>` REST, `robot` (dernier recours) |
| `LLMPort` | `complete(prompt, contexte, schéma_sortie, classe)` | abstraction multi-fournisseur, modèle auto-hébergé |
| `EmbeddingPort` | `embed(texts)` | selon Q-03 |
| `SearchPort` | `search_lexical`, `search_vector`, `rerank` | index hybride |
| `DocumentPort` | `list`, `fetch`, `metadata` | GED, wiki, partage de fichiers |
| `IAMPort` | `authenticate`, `authorize`, `roles` | annuaire d'entreprise |
| `TracePort` | `append`, `replay`, `query` | stockage append-only |
| `NotificationPort` | `send(destinataire, contenu, mention_IA)` | mail, canal interne |

**Règle vérifiée en CI** : `src/core/domain` et `src/core/application` n'importent aucun module de `src/adapters`. Contrôle par analyseur d'imports, bloquant.

### 05.3 Machine à états

États du ticket : `RECU → NORMALISE → EN_ATTENTE_DEMANDEUR | TRIE → EN_RESOLUTION_N1 | EN_INVESTIGATION_N2 → EN_ESCALADE_N3 → TRANSMIS | RESOLU_PROPOSE → CLOS(humain)`, plus `BLOQUE_SUPERVISEUR` et `EN_FILE_MODE_DEGRADE` accessibles depuis tout état.

Propriétés exigées :

- **Transitions explicites et exhaustives** : toute transition non déclarée est une erreur, pas un cas par défaut.
- **Idempotence** : rejouer une transition avec le même identifiant d'exécution ne produit aucun effet supplémentaire.
- **Persistance avant effet** : l'état et la trace sont écrits avant l'appel à un adaptateur à effet de bord.
- **Reprise** : une panne en cours de transition reprend au dernier état persisté, jamais au milieu.

### 05.4 Registre d'agents

Les agents sont enfichables et remplaçables isolément. Chaque agent déclare : identifiant, version, contrat d'entrée, contrat de sortie, classe de modèle nominale, règle d'escalade, budget de jetons, comportement de repli. Le registre est chargé au démarrage et figé pour la durée d'exécution ; sa composition est inscrite dans la trace.

### 05.5 Organisation du code

```
src/
├── core/
│   ├── domain/          # entités, objets valeur, invariants — zéro dépendance externe
│   ├── application/     # cas d'usage, orchestration, agents, supervision, gouvernance
│   ├── ports/           # interfaces abstraites
│   └── contracts/       # modèles Pydantic générés depuis les JSON Schema
├── adapters/            # implémentations des ports
├── packs/               # packs déclaratifs, jamais de code
├── api/                 # FastAPI : routes, dépendances, sécurité
├── workers/             # tâches asynchrones : ingestion corpus, classe L, évaluation
└── frontend/            # Angular
```
