# DAv1 — RAG, superviseur et curseur d'autonomie (ch. 10 à 12)

**Amont** : DEC-03, DEC-05, DEC-06, R-01, R-07, `PLAN-00-A1` §2.3

---

## 10. Architecture RAG

### 10.1 Chaîne d'ingestion de la connaissance

```
1. Collecte      → historique résolu, formulaire, doc projet, wiki, GED, logs
2. Anonymisation → détection et masquage PII + rejet si secret, AVANT tout stockage
3. Clustering    → regroupement des tickets par similarité de cause
4. Synthèse      → brouillon de runbook par cluster, avec citations, classe L
5. Revue humaine → un technicien nommé approuve, corrige ou rejette
6. Publication   → versionné, signé, daté, propriétaire identifié, date d'expiration
```

Étapes 1 à 4 : automatiques, asynchrones, hors chemin critique. Étape 5 : le goulot réel du projet. Étape 6 : seule porte d'entrée du corpus normatif.

### 10.2 Corpus à deux étages

| | **Étage 1 — indicatif** | **Étage 2 — normatif** |
|---|---|---|
| Contenu | Tickets résolus anonymisés, cas similaires, fils de discussion, extraits de doc | Runbooks approuvés, catalogue N2, liste blanche d'actions, réponses types |
| Origine | Automatique, réindexé en continu | Généré puis **validé par un humain nommé** |
| Usage autorisé | Aide au diagnostic N2, cas similaires, détection de doublon, contexte affiché à l'humain | Résolution N1 automatique, réponse au demandeur, toute action à effet de bord |
| Usage **interdit** | Déclencher une action, justifier une réponse envoyée au demandeur | Tout contenu non signé ou dont la revue a expiré |

Le point de bascule est l'**effet de bord** : une connaissance non validée peut informer un humain, elle ne peut jamais autoriser une action. C'est le principe du superviseur appliqué au corpus.

### 10.3 Règles de promotion étage 1 → étage 2

Conditions cumulatives, vérifiées par le code avant publication :

1. Le brouillon est généré à partir d'au moins **3 tickets concordants**.
2. Il cite ses sources, **une par étape** du runbook.
3. Un technicien **nommé** approuve ; son identité est inscrite dans la version.
4. Toutes les actions citées **existent en liste blanche** ; sinon le runbook reste en proposition.
5. Une date d'expiration de revue est fixée. **Sans revue à échéance, retour automatique à l'étage 1.**

Le retour à l'étage 1 est automatique et ne demande aucune validation : c'est du code, pas un processus.

### 10.4 Stratégie de recherche

| Élément | Décision | Justification |
|---|---|---|
| **Indexation hybride** | Lexical (BM25) + vectoriel, fusion des scores | Les codes d'erreur et noms d'applications se retrouvent mal en vectoriel seul |
| **Filtrage par pack** | Appliqué à **l'index**, jamais au prompt | L'étanchéité doit être structurelle (ENF-08), un filtre de prompt se contourne |
| **Filtrage par étage** | La requête déclare l'étage autorisé selon l'usage | Un agent à effet de bord ne peut interroger que l'étage 2 |
| **Reranking** | Modèle de reclassement avant injection | Moins de contexte, plus de pertinence, moins de jetons |
| **Citation obligatoire** | Toute affirmation issue du corpus porte sa source et sa version | Une sortie sans source est non actionnable, par construction |
| **Validité documentaire** | Chaque document porte version, propriétaire, date de publication, date d'expiration | Obsolescence détectable et mesurée (KPI : âge du corpus normatif) |
| **Confiance documentaire** | Score par document, dérivé de l'étage, de la fraîcheur, du nombre de sources concordantes | Alimente la confiance de la décision |

### 10.5 Isolation du contenu récupéré

Le contenu issu de l'index est injecté dans le contexte **comme donnée balisée**, jamais comme instruction. Toute directive présente dans un document récupéré est ignorée par construction, et sa présence est un signal remonté au superviseur (contrôle n° 7). C'est la contre-mesure à M-02 (injection indirecte).

### 10.6 Ce qui est socle et ce qui est spécifique

| Chantier | Nature | Porté par |
|---|---|---|
| **Exploitation** du corpus : index hybride, reranking, citation, filtrage, validité | Générique | Core Platform |
| **Construction** initiale du corpus : identification des sources, qualification, gouvernance documentaire | Spécifique | Pack projet + effort client |

Chez un client disposant déjà d'une base documentaire, l'effort se concentre sur la qualification et l'intégration. Chez un client dont le corpus est inexistant, la construction est le chemin critique — et doit être planifiée comme un chantier à part entière, pas comme un paramétrage.

---

## 11. Le superviseur

### 11.1 Position

**Aucune sortie d'agent n'atteint un effet de bord sans passer par le superviseur.** Ce n'est pas une politique, c'est une propriété du code : les adaptateurs à effet de bord n'acceptent d'appel que porteur d'un verdict valide et non expiré. Un appel sans verdict lève une erreur.

### 11.2 Les sept contrôles

| N° | Contrôle | Entrée | Échec → |
|---|---|---|---|
| 1 | **Schéma de sortie valide** | Sortie d'agent, JSON Schema versionné | Blocage : sortie non conforme |
| 2 | **Aucun garde-fou absolu touché** | Sortie, liste des 6 garde-fous | Blocage : garde-fou, motif nommé |
| 3 | **Action présente en liste blanche** | Actions du plan, liste blanche du pack projet | Blocage : action non autorisée |
| 4 | **Confiance au-dessus du seuil** | `confiance`, seuil du couple catégorie × action | Dégradation : proposition au lieu d'exécution |
| 5 | **Niveau d'autonomie respecté** | Niveau du couple catégorie × action, environnement | Dégradation : validation humaine requise |
| 6 | **Compatible avec le SLA restant** | Échéance du ticket, durée estimée de l'action | Dégradation ou escalade prioritaire |
| 7 | **Aucune instruction injectée** | Contenu du ticket, contenu récupéré du corpus | Blocage : court-circuit permanent, alerte sécurité |

Un seul contrôle en échec suffit à produire un verdict de blocage ou de dégradation. Le verdict porte toujours un **motif lisible par un humain** — un blocage silencieux est un défaut.

### 11.3 Verdicts possibles

| Verdict | Effet | Trace |
|---|---|---|
| `AUTORISE` | L'action s'exécute | Contrôles, seuils appliqués, versions |
| `BLOQUE` | Rien ne s'exécute, motif affiché au technicien | Contrôle en échec, valeur observée vs attendue |
| `VALIDATION_HUMAINE` | L'action est préparée, présentée, validée en un clic | Contrôle ayant déclenché la dégradation |
| `MODE_DEGRADE` | Fonctionnement réduit déclaré, confiance minorée | Composant indisponible, repli appliqué |

### 11.4 Biais assumé

Le superviseur est **délibérément trop prudent**. Un faux blocage coûte quelques minutes de technicien ; une action non désirée en production coûte beaucoup plus. En cas d'hésitation : bloquer et expliquer.

Corollaire mesuré : le taux de blocage est un KPI suivi. Au-delà de 25 %, on analyse les motifs — non pour assouplir les contrôles, mais pour corriger l'agent ou le pack qui les déclenche.

### 11.5 Court-circuit permanent

Un ticket portant l'un de ces marqueurs sort du chemin automatique **quel que soit le niveau d'autonomie configuré** : sécurité · confidentiel · impact vital · tentative d'injection détectée. Ce court-circuit est prioritaire sur toute autre règle et n'est désactivable par aucun pack.

---

## 12. Curseur d'autonomie

### 12.1 Les cinq niveaux

| Niveau | Nom | Comportement |
|---|---|---|
| **0** | Observation | Journalisé, invisible pour le technicien |
| **1** | Suggestion | Le technicien voit et accepte ou non |
| **2** | Validation humaine obligatoire | Action préparée, validée en un clic |
| **3** | Exécution automatique réversible | Exécuté seul, annulable |
| **4** | Autonomie avancée | Exécuté seul, sans filet |

### 12.2 Granularité

Le niveau est une propriété du **couple catégorie × action**, paramétrable par client, par projet, par catégorie de ticket et par action. Ce n'est pas une propriété du projet.

**Tout projet démarre au niveau 0, sur toutes les catégories, sans exception.**

Le niveau 4 n'est pas un objectif. Sur la majorité des catégories, le niveau 2 est la cible raisonnable et suffisante.

### 12.3 Montée — conditions cumulatives

1. 200 décisions observées sur la catégorie concernée
2. 30 jours d'observation minimum
3. 95 % d'accord avec la décision humaine
4. Zéro incident grave imputé
5. Validation conjointe service + client

### 12.4 Descente — automatique, sans validation

| Déclencheur | Effet |
|---|---|
| Accord sous 90 % sur 7 jours glissants | Descente d'un cran |
| Incident imputable au pipeline | Descente immédiate |
| Changement de modèle ou de version sans réévaluation | Descente sur les catégories concernées |
| Changement de pack sans réévaluation | Descente sur les catégories concernées |

La descente est déclenchée par le code. Elle ne demande l'accord de personne.

### 12.5 Conséquence du dimensionnement sur le rythme de montée

À 2 000 tickets/mois répartis sur une vingtaine de catégories, une catégorie courante voit 4 à 10 tickets/jour : **20 à 50 jours pour atteindre 200 décisions**. Les catégories de queue de distribution n'y parviendront jamais.

Trois conséquences à acter :

1. Le **niveau 2** est la cible réaliste à l'horizon du pilote. Les niveaux 3 et 4 ne concerneront qu'une poignée de catégories à fort volume.
2. La **granularité de la taxonomie devient un paramètre d'architecture** : une taxonomie trop fine rend l'autonomie statistiquement inatteignable. À arbitrer en phase 2 avec un seuil chiffré — proposition : aucune catégorie de la taxonomie d'évaluation ne doit représenter moins de 3 % du volume.
3. Une **agrégation de catégories pour l'évaluation**, distincte de la taxonomie de triage, est nécessaire. Elle est portée par le pack projet.

### 12.6 Nature architecturale du curseur

Le niveau d'autonomie est un **paramètre lu par le superviseur à chaque décision**, pas une consigne donnée à une équipe. Il est versionné, tracé, et tout changement est un événement d'audit avec auteur, date, motif et valeur précédente.

Un score de confiance n'est jamais une preuve de justesse : il se combine à la qualité des sources, à la criticité, à la nouveauté du cas, aux contrôles de sécurité et à la réversibilité de l'action.
