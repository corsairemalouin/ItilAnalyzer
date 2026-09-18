# ADR-009 — Stack du moteur métier

**Statut** : accepté · v0.2 (révise la v0.1 qui retenait Java)
**Date** : 2026-09-18
**Amont** : `PLAN-00`, `PLAN-00-A1` §3, DAv0 §Architecture logicielle
**Aval** : DAv1-05, DAv1-06, ADR-002, ADR-011, ADR-017, ADR-019 (annulé)

## Contexte

Le DAv0 laisse le choix ouvert entre Python et Java, en notant que l'écosystème IA est nettement plus riche côté Python et que Java s'impose si le SI cible ou les compétences l'exigent.

Une première instruction (A1 §3) avait retenu Java 21 / Spring Boot 3.5 sur la base d'une compétence d'équipe déclarée Java. Cette instruction avait identifié un risque majeur qui en découlait — R-12 : impossibilité de porter en Java, à maturité équivalente, la chaîne d'ingestion de connaissance et le banc d'évaluation, avec pour conséquence l'impossibilité de prouver ENF-05 et ENF-06, donc de faire monter le curseur d'autonomie. La parade était une architecture bi-langage (noyau Java + satellite Python), au prix d'une seconde chaîne de build et d'une frontière à défendre en permanence.

L'information de compétence a été corrigée : **des compétences Python existent en interne**. Le critère qui avait fait pencher la balance vers Java disparaît.

## Options

| Option | Description |
|---|---|
| A | Java 21 / Spring Boot 3.5, tout Java |
| B | Java 21 noyau + satellite Python d'analyse |
| **C** | **Python 3.12 / FastAPI, stack unique** |

## Matrice pondérée révisée

| Critère | Poids | A | B | C |
|---|---|---|---|---|
| Écosystème IA / RAG / évaluation / calibration | 25 % | 2 | 5 | 5 |
| Compétences disponibles en interne | 20 % | 5 | 4 | 4 |
| Simplicité d'exploitation (une chaîne, un artefact) | 15 % | 5 | 2 | 5 |
| Maintenabilité à 3 ans (typage, refactoring, tests) | 15 % | 5 | 4 | 4 |
| Capacité à prouver ENF-05 / ENF-06 (banc d'évaluation) | 15 % | 2 | 5 | 5 |
| Vitesse de mise en œuvre du moteur métier | 10 % | 4 | 3 | 4 |
| **Score pondéré** | 100 % | **3,60** | **4,05** | **4,55** |

## Décision

**Python 3.12 / FastAPI pour l'intégralité du moteur métier**, des agents, du superviseur, de la machine à états, de la chaîne d'ingestion de connaissance, du banc d'évaluation et des API.

**ADR-019 (frontière noyau/satellite) est annulé** : il n'a plus d'objet. Le risque R-12 est clos par construction.

## Conséquences

### Positives

- Une seule chaîne de build, un seul artefact, un seul profil de durcissement, une seule compétence d'exploitation.
- La chaîne d'ingestion (clustering par similarité de cause, synthèse, anonymisation) et le banc d'évaluation (matrice de confusion, calibration, détection d'hallucination) sont écrits dans l'écosystème où ces problèmes sont résolus. C'est ce qui rend ENF-05 (accord ≥ 85 %) et ENF-06 (calibration < 10 pts) réellement mesurables plutôt que déclaratifs.
- L'observabilité LLM (traces, coûts, jetons) dispose d'outillage mature, ce qui sert directement les KPI IA exigés.
- Le port LLM est implémentable derrière une abstraction multi-fournisseur mature, ce qui sert ENF-09 (réversibilité).

### Négatives, et parades

| Conséquence | Parade rendue obligatoire |
|---|---|
| Typage dynamique : risque de sortie non conforme non détectée | **Pydantic v2 en frontière de tout contrat**, validation stricte, `strict=True`, aucune coercition silencieuse. Une sortie non conforme est une exception, jamais un cas rattrapé (exigence DAv0 sur les contrats). |
| Refactoring moins sûr qu'en langage compilé | `mypy --strict` bloquant en CI sur `src/core/`, sans exception |
| Performance d'exécution | Non dimensionnant : < 0,01 req/s au pic (A1 §2.1). L'I/O asynchrone de FastAPI couvre très largement le besoin. |
| Gestion des dépendances et supply chain | Verrouillage strict (`uv` + lockfile), SBOM, scan SCA bloquant en CI |
| Tentation d'écrire du code dynamique « pratique » dans le socle | Règles de lint (`ruff`) interdisant `Any` non justifié, `eval`, import dynamique, dans `src/core/` |

### Impacts sur les autres décisions

| Décision | Effet |
|---|---|
| ADR-002 (port LLM) | Abstraction multi-fournisseur réévaluée sur l'écosystème Python — tranche en faveur d'une couche d'abstraction plutôt que d'un SDK propriétaire |
| ADR-011 (orchestration codée vs BPM) | L'orchestration codée reste retenue ; la machine à états est implémentée en code pur, sans dépendance à un framework d'agents |
| ADR-017 (évaluation) | Le harnais devient un livrable natif du lot 1, sans coût de portage |
| R-12 | **Clos** |
| PLAN-00 §6.1 | `src/` : projet Python unique, modules = contextes bornés |
