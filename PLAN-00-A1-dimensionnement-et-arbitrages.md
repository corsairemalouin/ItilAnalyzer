# PLAN-00 / Addendum A1 — Dimensionnement, ADR-009 et point le plus risqué

**Document** : `IAA-PLAN-00-A1` · v0.1 · statut : **pour validation**
**Amont** : `IAA-PLAN-00`, `ITILANALYZER-DA-V0`
**Objet** : clôture de Q-08, Q-11, Q-14 ; conséquences sur les phases P1 à P5 ; instruction anticipée du risque principal

---

## 1. Réponses actées

| Id | Question | Réponse | Statut |
|---|---|---|---|
| Q-08 | Volumétrie de tickets | 500 à 2 000 / mois | **Close** |
| Q-11 | Cible d'hébergement | Cloud souverain / de confiance | **Close** |
| Q-14 | Compétence dominante de l'équipe | Java / Spring | **Close** |

Restent ouvertes et bloquantes : Q-03 (classes de modèles autorisées par le RSSI), Q-06 (rétention et base légale), Q-09 (volumétrie documentaire), Q-10 (plage de service), Q-12 (nombre de clients à 24 mois), Q-13 (budget par ticket), Q-15 (délai du premier jalon).

---

## 2. Dimensionnement dérivé de Q-08

### 2.1 Hypothèses de charge

Base retenue : **2 000 tickets/mois** (borne haute de la fourchette), 21 jours ouvrés.

| Grandeur | Valeur | Méthode |
|---|---|---|
| Tickets par jour ouvré | ~95 | 2 000 / 21 |
| Tickets par heure, moyenne | ~12 | sur 8 h |
| Pic horaire (facteur 2,5) | ~30 | ouverture de service, retour de week-end |
| Requêtes/s au pic | **< 0,01 req/s** | 30 / 3 600 |
| Techniciens simultanés sur la console | 5 à 15 | à confirmer |

### 2.2 Ce que ces chiffres invalident

| Hypothèse d'architecture | Verdict | Motif |
|---|---|---|
| Microservices | **Écarté** | Aucune charge ne justifie l'isolation ; confirme DEC-08 |
| Autoscaling applicatif | **Écarté** | Deux instances fixes suffisent, dimensionnées pour la disponibilité, pas pour le débit |
| Index vectoriel distribué (cluster dédié) | **Écarté en v0** | Volumétrie documentaire probablement < 10⁵ segments — à confirmer par Q-09 |
| GPU dédié pour l'inférence | **À instruire seulement si auto-hébergement** | Voir §4 |
| File d'attente asynchrone | **Conservé** | Non pour le débit, mais pour le fail-closed (ENF-10) et la classe L asynchrone |

### 2.3 Ce que ces chiffres rendent critique à la place

**Latence unitaire.** ENF-03 impose < 60 s de bout en bout sur le chemin triage. Avec un enchaînement Intake (classe S) → Triage (classe M) → recherche RAG → Superviseur, le budget se répartit ainsi :

| Étape | Budget | Marge |
|---|---|---|
| Normalisation + déduplication (S) | 2 s | |
| Recherche hybride + reranking | 3 s | |
| Triage (M) | 10 s | |
| Escalade éventuelle vers L, une seule fois | 30 s | plafonnée par le code |
| Superviseur (code + S) | 2 s | |
| **Total pire cas** | **47 s** | 13 s de marge |

La marge est étroite. Conséquence pour P4 : le budget de latence par étape est une **exigence contractuelle du contrat d'agent**, pas une cible d'optimisation. Un dépassement déclenche un repli, jamais une attente.

**Coût par ticket.** Répartition d'appels du DAv0 (60 % S, 35 % M, 5 % L), avec un contexte RAG typique de 4 à 8 k jetons :

| Classe | Appels / ticket | Jetons moyens | Part du coût |
|---|---|---|---|
| S | 3 à 5 | ~2 k | faible |
| M | 1 à 2 | ~8 k | dominante |
| L | 0,05 à 0,2 | ~30 k | variable, à plafonner |

À 2 000 tickets/mois, l'enveloppe LLM reste modeste en valeur absolue quel que soit le scénario. **Ce n'est donc pas le coût qui doit piloter le choix de modèle, mais la souveraineté et la qualité.** C'est un point important : il supprime l'argument économique en faveur d'un modèle petit et médiocre. Le plafond par ticket (ENF-04) reste nécessaire, mais comme garde-fou contre la dérive R-09, pas comme contrainte de conception.

**Rythme de montée du curseur d'autonomie.** Conditions cumulatives du DAv0 : 200 décisions observées **et** 30 jours **et** 95 % d'accord sur une catégorie donnée. À 95 tickets/jour répartis sur, disons, 15 à 25 catégories, une catégorie courante voit 4 à 10 tickets/jour — soit **20 à 50 jours pour atteindre 200 décisions**, et bien davantage pour les catégories de queue de distribution.

Conséquences, à acter :
- Le niveau 2 (validation humaine) est la cible atteignable à l'horizon du pilote ; les niveaux 3 et 4 ne concerneront qu'une poignée de catégories à fort volume.
- La granularité de la taxonomie devient un paramètre d'architecture : une taxonomie trop fine rend l'autonomie statistiquement inatteignable. À arbitrer en phase 2 (F-04), avec un critère chiffré.
- Il faut prévoir une **agrégation de catégories** pour l'évaluation, distincte de la taxonomie de triage.

---

## 3. ADR-009 tranché — Java 21 / Spring Boot 3.5

### 3.1 Matrice pondérée

| Critère | Poids | Python / FastAPI | Java 21 / Spring Boot 3.5 |
|---|---|---|---|
| Compétences d'exploitation disponibles (Q-14) | 25 % | 2 | 5 |
| Écosystème IA / RAG / évaluation | 20 % | 5 | 3 |
| Intégration au SI d'entreprise cible | 15 % | 3 | 5 |
| Maintenabilité à 3 ans, typage, refactoring | 15 % | 3 | 5 |
| Vitesse de mise en œuvre du moteur métier | 10 % | 4 | 4 |
| Empreinte et coût d'exécution | 10 % | 4 | 4 |
| Maturité des outils de supervision LLM | 5 % | 5 | 3 |
| **Score pondéré** | 100 % | **3,35** | **4,25** |

### 3.2 Décision

**Moteur métier, agents, superviseur, machine à états, API : Java 21 / Spring Boot 3.5.**

Le critère qui tranche est le premier. Une équipe Java qui écrit du Python produira du Python moyen, difficile à exploiter en production et à reprendre. Le DAv0 l'anticipait : *« Java s'intègre naturellement à un SI d'entreprise et aux équipes maîtrisant Spring »*, et la recommandation Python n'y était formulée que par l'écosystème IA, pas par une propriété du moteur métier.

Ce que ce choix **ne** résout **pas** : les briques réellement dépendantes de l'écosystème Python — clustering de tickets, synthèse de corpus, calcul de métriques de calibration, détection d'hallucination, harnais d'évaluation. C'est l'objet du §4.

### 3.3 Conséquences sur le plan

| Livrable | Impact |
|---|---|
| ADR-009 | Statut passe de « à instruire » à **accepté**, matrice ci-dessus intégrée |
| ADR-002 (stratégie LLM) | Le port LLM doit être spécifié en Java ; l'option « SDK d'abstraction multi-fournisseur » doit être réévaluée sur les offres Java |
| ADR-011 (orchestration) | La maturité de Spring pour les tâches longues et les reprises renforce l'orchestration codée |
| DAv1-05, DAv1-06 | Stack tranchée, plus d'option ouverte |
| `src/` | Arborescence Maven/Gradle multi-modules, un module par contexte borné |

---

## 4. Le point le plus risqué — et deux approches pour le traiter

### 4.1 Énoncé du risque

**R-12 — La chaîne IA (classe L, ingestion de connaissance, évaluation) est simultanément contrainte par la souveraineté et par un écosystème Java pauvre sur ces sujets.**

Trois faits se croisent :

1. **Souveraineté (Q-11)** — le catalogue de modèles accessibles est restreint à ce qui est hébergé dans une zone de confiance, ou auto-hébergé. La classe L du DAv0 (raisonnement, corrélation logs, hypothèse causale) est précisément celle où l'écart de qualité entre modèles est le plus marqué, et où les options souveraines sont les plus limitées.
2. **Java (Q-14)** — la chaîne d'ingestion du corpus (anonymisation, clustering par similarité de cause, synthèse de runbook, promotion) et le banc d'évaluation (matrice de confusion, calibration, détection d'hallucination) n'ont pas d'équivalent Java mature. Les réimplémenter est un effort réel et une source de défauts silencieux.
3. **Criticité** — ces deux briques ne sont pas périphériques. Sans classe L exploitable, l'Investigateur N2 perd l'essentiel de sa valeur. Sans banc d'évaluation, ENF-05, ENF-06 et tout le mécanisme de montée du curseur d'autonomie s'effondrent : on ne peut plus rien prouver, donc plus rien automatiser.

Probabilité : **haute**. Impact : **fort**. C'est, à ce stade, le premier risque du projet devant R-01 (formalisation des runbooks).

### 4.2 Approche A — Tout Java

Le moteur, les agents, la chaîne d'ingestion et le harnais d'évaluation sont écrits en Java, sur la pile Spring AI (abstraction de modèles, embeddings, vector stores, tool calling).

**Ce qui fonctionne bien.** Un seul langage, un seul build, une seule chaîne CI, une seule compétence d'exploitation. Le typage fort sert directement les contrats JSON du DAv0 : les 7 contrats d'agents deviennent des records Java validés à la compilation, ce qui est un vrai gain sur l'exigence « une sortie non conforme est un échec, pas un cas à rattraper ». Le déploiement se réduit à un artefact. L'équipe est immédiatement productive.

**Où ça casse.** Le clustering de tickets par similarité de cause et la synthèse de runbooks demandent des outils d'analyse qui n'existent pas en Java à maturité équivalente — on finit par les réimplémenter, mal, ou par les déléguer entièrement au LLM, ce qui coûte cher et dégrade la reproductibilité. Le harnais d'évaluation (calibration, matrices, analyse d'écart) est à construire de zéro : c'est plusieurs semaines d'effort sur une brique dont personne ne voit la valeur avant qu'elle manque. Enfin, l'écosystème d'observabilité LLM est en retard côté Java, ce qui touche directement les KPI IA demandés.

**Risque résiduel.** Le banc d'évaluation, non prioritaire aux yeux du projet, est sous-investi ou repoussé. Le curseur d'autonomie ne peut alors jamais monter faute de preuve, et le produit reste bloqué au niveau 1. Le risque ne se matérialise pas au lot 1 — il se matérialise au lot 3, quand il est coûteux de revenir en arrière.

### 4.3 Approche B — Noyau Java, satellite Python d'analyse

Le moteur métier, les agents, le superviseur, la machine à états et les API restent en Java/Spring — strictement conformes à ADR-009. Un service Python séparé, appelé derrière un contrat HTTP versionné, porte exclusivement les traitements analytiques **hors chemin critique** : chaîne d'ingestion du corpus (clustering, synthèse, anonymisation), banc d'évaluation, calcul des métriques de calibration et d'hallucination.

**Ce qui fonctionne bien.** Chaque brique est écrite dans le langage où elle a un écosystème. La frontière est nette et défendable : le satellite ne traite **aucun ticket en temps réel**, il n'a aucun droit d'écriture, il ne participe à aucune décision supervisée. Il travaille en batch sur du corpus déjà anonymisé et sur des jeux d'évaluation. Une panne du satellite dégrade l'ingestion de connaissance et l'évaluation — elle n'interrompt pas le traitement des tickets. La contrainte de latence ne s'applique pas à lui, donc son coût d'exploitation est faible.

**Où ça casse.** Deux chaînes de build, deux images, deux profils de sécurité à durcir, deux compétences à maintenir. Sur une équipe Java, le satellite risque de devenir orphelin. Le contrat HTTP doit être versionné et testé comme un contrat externe, sinon il dérive. Et la frontière demande de la discipline : la tentation d'y glisser « juste un petit traitement temps réel » est réelle, et c'est exactement ainsi que le risque revient.

**Risque résiduel.** Dérive du périmètre du satellite. Se contrôle par une règle d'architecture vérifiable en CI : le satellite n'expose aucune route appelée par le chemin de traitement d'un ticket, et n'a aucun accès en écriture aux adaptateurs ITSM.

### 4.4 Comparaison

| Critère | Poids | A — Tout Java | B — Java + satellite Python |
|---|---|---|---|
| Qualité du banc d'évaluation (conditionne ENF-05, ENF-06, curseur) | 30 % | 2 | 5 |
| Simplicité d'exploitation | 20 % | 5 | 3 |
| Adéquation aux compétences de l'équipe | 20 % | 5 | 3 |
| Qualité de la chaîne d'ingestion de connaissance (R-01) | 15 % | 2 | 5 |
| Latence et fiabilité du chemin critique | 10 % | 4 | 4 |
| Coût d'infrastructure | 5 % | 5 | 4 |
| **Score pondéré** | 100 % | **3,55** | **4,00** |

### 4.5 Recommandation

**Approche B**, avec une frontière écrite comme règle d'architecture non négociable :

> Le satellite d'analyse ne participe à aucun traitement de ticket en temps réel, n'a aucun droit d'écriture sur un système externe, et n'est jamais appelé par le superviseur. Il consomme du corpus anonymisé et des jeux d'évaluation, il produit des brouillons et des métriques.

Cette frontière n'est pas un arrangement : elle recoupe exactement la distinction du DAv0 entre ce qui a un effet de bord et ce qui n'en a pas. Le satellite est du côté sans effet de bord, par construction.

**Point de bascule vers A** : si l'équipe ne peut pas s'engager à maintenir une seconde chaîne, mieux vaut assumer A et **budgéter explicitement le banc d'évaluation en Java comme un livrable du lot 1**, pas comme une tâche implicite. Un banc d'évaluation médiocre est plus dangereux qu'un banc absent : il produit des chiffres auxquels on croit.

### 4.6 Ce qui reste à instruire d'urgence

La réponse du RSSI sur Q-03 (classes de modèles autorisées, localisation, fournisseurs) devient **le prérequis n°1**. Elle détermine si la classe L est servie par un modèle hébergé en zone de confiance ou par un modèle à poids ouverts auto-hébergé — ce dernier cas réintroduit une exigence GPU que le §2.2 avait écartée, et change le dimensionnement de l'infrastructure. Tant que Q-03 n'est pas tranchée, DAv1-06 et DAv1-07 restent en hypothèse.

---

## 5. Impacts à reporter dans PLAN-00

| Section PLAN-00 | Modification |
|---|---|
| §2.2 ADR-009 | Statut « accepté », matrice §3.1 intégrée |
| §2.2 | Ajout **ADR-019 — Frontière noyau Java / satellite d'analyse Python** (priorité P1) |
| §4 NFR-03 Scalabilité | Requalifiée : exigence de latence unitaire et de disponibilité, non de débit |
| §4 NFR-02 | Budget de latence par étape (§2.3) devient contractuel dans les contrats d'agents |
| §6.1 | Arborescence `src/` : multi-modules Maven/Gradle + `src/analytics/` (Python, isolé) |
| §8 | Q-08, Q-11, Q-14 closes ; **Q-03 passe en prérequis bloquant** de DAv1-06 et DAv1-07 |
| §8 | Ajout **R-12** (risque décrit en §4.1) au registre des risques |
| §10 | Le rythme de montée du curseur (§2.3) devient un point d'attention supplémentaire |

---

*Fin de l'addendum A1.*
