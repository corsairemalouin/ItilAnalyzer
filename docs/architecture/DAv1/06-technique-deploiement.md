# DAv1 — Architecture technique et de déploiement (ch. 06 à 07)

**Amont** : `00-fondations.md`, `ADR-009 v0.2`, `PLAN-00-A1` §2 (dimensionnement), DEC-04, DEC-08

---

## 06. Architecture technique

### 06.1 Stack retenue

| Couche | Technologie | Justification | ADR |
|---|---|---|---|
| Moteur métier, agents, superviseur, API | **Python 3.12 / FastAPI** | ADR-009 v0.2 | ADR-009 |
| Contrats et validation | **Pydantic v2, mode strict** | Une sortie non conforme est une exception, pas un cas rattrapé | ADR-009 |
| Typage | `mypy --strict` bloquant sur `src/core/` | Compense le typage dynamique | ADR-009 |
| Tâches asynchrones | File de tâches durable + workers dédiés | Classe L asynchrone, ingestion corpus, évaluation | ADR-014 |
| Persistance transactionnelle | **PostgreSQL** | État de la machine, packs, gouvernance, verdicts | ADR-012 |
| Trace d'audit | Stockage **append-only**, intégrité vérifiable | ENF-01, rejeu | ADR-012 |
| Index de connaissance | **Index hybride** : lexical (BM25) + vectoriel | Codes d'erreur et noms d'applications se retrouvent mal en vectoriel seul | ADR-003 |
| Passerelle modèles | Port LLM + couche d'abstraction multi-fournisseur | ENF-09 | ADR-002 |
| Frontend | **Angular** (dernière version LTS) | Demande §5 | ADR-015 |
| Observabilité | OpenTelemetry (traces, métriques, logs) + collecteur | KPI exploitation et IA | ADR-004 |
| Conteneurisation | Docker, images distroless, utilisateur non root | Sécurité | ADR-013 |
| Orchestration | Kubernetes + Helm | Demande §5 | ADR-010 |

**Note sur PostgreSQL et le vectoriel** : à la volumétrie attendue (Q-09 à confirmer, probablement < 10⁵ segments), une extension vectorielle sur PostgreSQL couvre le besoin et évite un composant supplémentaire. Le `SearchPort` reste néanmoins abstrait : le passage à un moteur dédié est un changement d'adaptateur, pas de conception. Décision à figer en ADR-012 une fois Q-09 répondue.

### 06.2 Les trois classes de modèles

| Classe | Usage | Latence cible | Part des appels | Sortie | Température |
|---|---|---|---|---|---|
| **S** | Extraction, classification fermée, détection (doublon, langue, PII, injection 1ʳᵉ passe) | < 2 s | ~60 % | structurée stricte | 0 |
| **M** | Décision et rédaction : triage, questions au demandeur, application de runbook, dossier N3 | < 10 s | ~35 % | structurée + citations | basse |
| **L** | Raisonnement : corrélation logs/historique, synthèse de runbook, détection de problème récurrent, arbitrage de cas limite | < 90 s, **asynchrone** | ~5 % | structurée + citations | basse |

**Règles transverses, non négociables** :

1. Le modèle et sa **version exacte** sont inscrits dans la trace de chaque appel.
2. Sortie structurée imposée partout — aucune prose libre ne circule dans le pipeline.
3. Escalade de classe **plafonnée à une fois par étape**, déclenchée par le code, jamais par le modèle (parade R-09).
4. Budget de jetons et d'appels par ticket connu à l'avance, contrôlé par le code, dépassement = repli et alerte.
5. La classe de modèle autorisée (hébergement, région, fournisseur) est un **paramètre du pack client**, jamais une constante du code (ENF-09, souveraineté).

### 06.3 Budget de latence sur le chemin critique (ENF-03 : < 60 s)

| Étape | Budget | Repli si dépassement |
|---|---|---|
| Normalisation + déduplication (S) | 2 s | File d'attente, jamais de normalisation devinée |
| Recherche hybride + reranking | 3 s | Recherche lexicale seule, marquée dégradée |
| Triage (M) | 10 s | Proposition non automatique au technicien |
| Escalade L éventuelle, une seule fois | 30 s | Décision au niveau M, confiance minorée |
| Superviseur (code + S) | 2 s | **Fail-closed** : blocage |
| **Total pire cas** | **47 s** | 13 s de marge |

Ces budgets sont **contractuels** : ils figurent dans la déclaration de chaque agent au registre et sont vérifiés en test de performance. Un dépassement déclenche le repli déclaré, jamais une attente.

### 06.4 Modes dégradés (ENF-10, fail-closed)

| Composant indisponible | Comportement | Chemin automatique |
|---|---|---|
| Superviseur | **Tout est bloqué** | Fermé |
| Passerelle modèles | Tickets mis en file, aucune décision | Fermé |
| Index de connaissance | Aucune réponse actionnable (pas de sources = non actionnable) | Fermé |
| Recherche vectorielle seule | Recherche lexicale, résultat marqué dégradé, confiance minorée | Ouvert, dégradé |
| Adaptateur ITSM en écriture | Propositions préparées et mises en attente | Ouvert sans écriture |
| Trace d'audit | **Tout est bloqué** — aucune sortie sans trace | Fermé |

Le principe est uniforme : en cas de doute, **bloquer et expliquer**. Un faux blocage coûte quelques minutes de technicien ; une action non désirée en production coûte infiniment plus.

### 06.5 Dimensionnement

Hypothèse : 2 000 tickets/mois, pic ~30 tickets/heure, < 0,01 req/s (voir `PLAN-00-A1` §2).

| Composant | Production | Justification |
|---|---|---|
| API + moteur | 2 répliques | Disponibilité, pas débit |
| Workers asynchrones | 2 répliques | Classe L + ingestion corpus |
| PostgreSQL | 1 instance managée + réplica | Sauvegarde et restauration testées |
| Index | colocalisé à la base en v0 | Volumétrie à confirmer (Q-09) |
| GPU | **aucun**, sauf auto-hébergement de modèle (Q-03) | Point de bascule majeur du dimensionnement |

**Point d'attention** : si la réponse à Q-03 impose un modèle à poids ouverts auto-hébergé pour la classe L, le dimensionnement change de nature (GPU, exploitation ML, capacité). Cette branche est chiffrée séparément dès que Q-03 est tranchée ; tant qu'elle ne l'est pas, ce chapitre est en hypothèse et non en décision.

---

## 07. Architecture de déploiement

### 07.1 Quatre environnements

| Environnement | Objet | Données | Modèles | Niveau d'autonomie max |
|---|---|---|---|---|
| **Développement** | Développement, tests unitaires | Jeux synthétiques uniquement | Adaptateur mock + modèle de test | 0 |
| **Qualification** | Tests d'intégration, évaluation IA | Jeux annotés anonymisés | Modèles réels, budget plafonné | 0 |
| **Préproduction** | Répétition de production, tests de charge et de sécurité | Copie anonymisée | Identiques à la production | 1 |
| **Production** | Service | Réelles | Selon pack client | Selon gouvernance, démarrage à 0 |

**Règle** : le niveau d'autonomie maximal atteignable est une propriété de l'environnement, plafonnée en dur. Aucune configuration de pack ne peut la dépasser.

### 07.2 Topologie cloud souverain

| Zone | Contenu | Exposition |
|---|---|---|
| **Zone publique** | Rien | — |
| **Zone d'entrée** | Ingress, WAF, terminaison TLS | Restreinte au réseau de l'entreprise |
| **Zone applicative** | API, moteur, workers | Interne uniquement |
| **Zone de données** | PostgreSQL, index, trace d'audit, coffre-fort de secrets | Accessible depuis la zone applicative seule |
| **Zone sortante** | Passerelle vers fournisseurs de modèles et systèmes externes | Sortie filtrée par liste blanche de destinations |

Contraintes liées à Q-11 (cloud souverain) :

- Localisation du traitement **et** de l'index définie par le pack client et vérifiée au démarrage : un pack déclarant une région non conforme empêche le démarrage.
- La sortie réseau est en liste blanche stricte : aucune destination non déclarée n'est joignable, ce qui limite structurellement l'exfiltration.
- Les données traversant la zone sortante sont anonymisées **avant** d'y entrer, pas à l'arrivée.

### 07.3 Isolation multi-client

| Option | Isolation | Coût | Statut |
|---|---|---|---|
| Mono-tenant : un déploiement complet par client | Maximale, physique | Élevé | **Retenue en v0** |
| Multi-tenant avec cloisonnement logique | Forte si bien faite, risque de défaut de configuration | Faible | À instruire si Q-12 révèle un grand nombre de clients |

Justification du mono-tenant en v0 : ENF-08 exige une étanchéité « garantie par construction ». Avec un seul client pilote et un volume faible, l'isolation physique donne cette garantie sans effort de conception. Le passage au multi-tenant est une décision à prendre au 2ᵉ ou 3ᵉ client, avec ADR-013 à réviser — et non une anticipation à faire maintenant.

**Même en mono-tenant**, le cloisonnement logique par pack est implémenté : filtrage de l'index par pack, clés de partitionnement, contrôle d'accès. Raison : c'est ce qui rend le passage au multi-tenant possible plus tard, et c'est ce qui est testé par le test d'étanchéité inter-packs.

### 07.4 Déploiement et promotion

- Artefact unique (image conteneur) promu d'un environnement à l'autre sans reconstruction.
- Configuration et packs injectés au déploiement, versionnés séparément de l'image.
- La version de l'image, la version de chaque pack et la version du registre de prompts sont inscrites dans la trace de chaque décision (ENF-01, parade R-05).
- Tout changement de version de modèle, de prompt ou de pack **invalide les évaluations en cours et redescend automatiquement le curseur d'autonomie** sur les catégories concernées, sans validation requise.
