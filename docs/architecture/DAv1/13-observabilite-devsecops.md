# DAv1 — Observabilité, DevSecOps, réversibilité, risques (ch. 13 à 17)

**Amont** : ENF-01, ENF-02, ENF-05, ENF-06, ENF-09, R-01→R-11

---

## 13. Architecture observabilité

### 13.1 Trois piliers distincts

| Pilier | Objet | Destinataire | Rétention |
|---|---|---|---|
| **Trace d'audit** | Rejouer une décision à l'identique | Auditeur, RSSI, client | Longue, définie par le pack client (Q-06) |
| **Observabilité technique** | Santé, latence, coût, erreurs | Exploitant | Moyenne |
| **Banc d'évaluation** | Qualité des décisions IA | Équipe produit, gouvernance du curseur | Longue |

Ces trois piliers sont séparés. Confondre la trace d'audit avec les logs applicatifs est une erreur classique : la trace est un livrable contractuel, pas un artefact d'exploitation.

### 13.2 Trace d'audit — contenu obligatoire

Écrite **avant** l'effet de bord. Une trace absente bloque l'action.

| Champ | Contenu |
|---|---|
| Identifiants | Ticket (tous identifiants externes), exécution, corrélation |
| Entrées | Ticket canonique, contexte récupéré (références et versions, pas le contenu intégral) |
| Versions | Image applicative, pack client, pack projet, registre de prompts, prompt utilisé, modèle **et sa version exacte** |
| Décision | Contrat de sortie complet, avec confiance, justification, sources |
| Verdict | Les 7 contrôles, leur résultat, les seuils appliqués, le motif |
| Action | Ce qui a été exécuté, réversibilité, résultat |
| Humain | Identité du validateur, acceptation ou correction, horodatage |
| Intégrité | Empreinte, chaînage append-only |

**Test de rejeu** : une trace permet de reconstituer la décision à l'identique. Vérifié sur 100 % des cas du jeu d'évaluation, bloquant en CI (ENF-02, EF-09).

### 13.3 Dashboards

**Métier — quatre tableaux**

| Tableau | Indicateurs |
|---|---|
| Activité | Tickets analysés, traités, escaladés |
| Performance | Temps moyen de qualification, temps moyen de traitement, gain vs manuel, respect SLA, respect OLA |
| Qualité | Confiance moyenne, taux de validation des recommandations, taux de correction humaine, taux de réouverture, satisfaction |
| Résolution | Taux de résolution automatique, taux d'escalade |

**IA**

Confiance moyenne · coût par ticket · nombre d'appels LLM · jetons consommés · taux d'acceptation des recommandations · taux de correction humaine · **taux d'hallucination détectée** (affirmation non rattachable à une source citée).

**Exploitation**

Disponibilité par composant · temps de réponse par étape · taux d'échec par agent · consommation GPU (si auto-hébergement) · consommation LLM · **nombre de décisions bloquées par le superviseur**, par motif.

### 13.4 Seuils d'alerte

| Indicateur | Ce qu'il révèle | Seuil | Action automatique |
|---|---|---|---|
| Taux d'accord technicien | Dérive de qualité | < 90 % sur 7 jours glissants | **Descente du curseur** |
| Taux de blocage superviseur | Sur-prudence ou régression d'agent | > 25 % | Analyse des motifs |
| Écart de calibration | Confiance non prédictive | > 10 points | Blocage de montée du curseur |
| Âge du corpus normatif | Obsolescence des runbooks | Revue expirée | **Retour du document à l'étage 1** |
| Coût par ticket | Dérive de classe L | > budget | Alerte, plafonnement |
| Taux de mal-dirigé résiduel | Valeur réelle du triage | vs baseline historique | Suivi |

### 13.5 Banc d'évaluation

**Composition du jeu de départ — 20 cas** : 8 tickets pauvres ou ambigus · 6 bien rédigés, 1 par catégorie · 3 cas limites entre catégories · 2 doublons d'un même incident · 1 cas d'injection qui doit être refusé.

Un jeu composé uniquement de tickets bien rédigés ne mesure rien : c'est sur les tickets pauvres que le triage casse.

**Trois lectures** :

1. **Matrice de confusion** sur `categorie` et `niveau_cible`. Objectif d'entrée : 85 % d'accord (ENF-05).
2. **Calibration** : la confiance annoncée prédit-elle la justesse ? Une confiance haute sur une décision fausse est le défaut prioritaire (ENF-06, écart < 10 pts).
3. **Contexte manquant** : pour chaque écart, ce qui aurait permis de trancher. Cette lecture dit **quel pack enrichir**, pas quel prompt retoucher — c'est la différence entre corriger la cause et corriger le symptôme.

**Critères de passage en pilote** : accord ≥ 85 % · calibration < 10 pts · cas d'injection refusé à 100 % · rejeu à l'identique vérifié sur 100 % des cas · latence et coût dans le budget · zéro écriture non tracée.

---

## 14. Architecture DevSecOps

| Étape | Outillage | Blocant ? |
|---|---|---|
| Lint et format | `ruff` | Oui |
| Typage | `mypy --strict` sur `src/core/` | Oui |
| Tests unitaires et d'intégration | `pytest`, couverture minimale définie en P3 | Oui |
| Tests d'architecture | Analyseur d'imports : le domaine n'importe aucun adaptateur | Oui |
| Test de non-contamination du socle | Aucun terme client dans `src/core/` (R-08) | Oui |
| Qualité | SonarQube | Oui |
| SAST | Analyse statique de sécurité | Oui |
| SCA + SBOM | Dépendances, licences, vulnérabilités | Oui |
| Secrets scanning | Historique complet | Oui |
| Validation de schémas | JSON Schema, OpenAPI, AsyncAPI | Oui |
| DAST | Sur préproduction | Oui avant production |
| **Gating d'évaluation IA** | Rejeu du banc, seuils ENF-05/06/07 | **Oui — une régression bloque la livraison** |
| Signature d'artefact | Image signée, provenance | Oui |

### 14.1 Versionnement

Trois axes versionnés indépendamment, tous trois inscrits dans chaque trace :

| Axe | Schéma | Effet d'un changement |
|---|---|---|
| Socle (Core Platform) | SemVer | Rejeu complet du banc d'évaluation |
| Packs (client, projet) | SemVer par pack | **Descente du curseur** sur les catégories touchées, jusqu'à réévaluation |
| Registre de prompts | Version par prompt, hors du code applicatif | **Descente du curseur**, rejeu du banc |

---

## 15. Réversibilité et portabilité

| Axe | Mécanisme | Test de réversibilité |
|---|---|---|
| Fournisseur de modèle | `LLMPort` + abstraction multi-fournisseur ; aucune dépendance à une fonctionnalité propriétaire dans le domaine | Basculer le banc d'évaluation sur un second fournisseur |
| ITSM | `ITSMPort` ; le socle n'encode aucune spécificité d'outil | Exécuter le pipeline complet sur l'adaptateur `mock` |
| Index / moteur de recherche | `SearchPort` ; format de segment et métadonnées normalisés | Réindexer sur un second moteur |
| Hébergeur | Conteneurs + Helm, aucune dépendance à un service managé propriétaire non substituable | Déploiement complet sur un second environnement |
| Données | Export documenté de la trace, du corpus et des packs dans des formats ouverts | Export et réimport vérifiés |

**Frontière juridique = frontière technique** : la propriété du socle reste à l'éditeur, les packs sont livrables au client. C'est aussi ce qui limite le verrouillage : le socle n'encode aucune spécificité d'outil client, seul l'adaptateur en dépend.

---

## 16. Risques

| Id | Risque | P | I | Parade portée par l'architecture | Signal d'alerte |
|---|---|---|---|---|---|
| R-01 | Les runbooks ne se formalisent pas | haute | fort | Chaîne d'ingestion : la génération produit un brouillon sourcé, valider remplace rédiger | < 5 runbooks promus à la fin du lot 2 |
| R-02 | Aucune écriture possible dans l'ITSM | moyenne | fort | Les niveaux 0 et 1 n'exigent aucune écriture : la valeur d'assistance est livrable sans intégration | Q-01 défavorable |
| R-03 | Historique non extractible ou trop pauvre | moyenne | fort | Repli sur l'arbre du formulaire comme taxonomie + cas annotés à la main | Extraction non obtenue sous 2 semaines |
| R-04 | Rejet par les techniciens | haute | moyen | Niveau 0 invisible au démarrage, proposition toujours contestable, N2/N3 en lecture seule | Acceptation < 50 % |
| R-05 | Dérive après changement de modèle ou de pack | haute | moyen | Versions dans la trace, descente automatique du curseur | Accord < 90 % sur 7 jours |
| R-06 | Fuite de données inter-clients ou vers un fournisseur | faible | critique | Cloisonnement appliqué à l'index, anonymisation avant appel, classe de modèle par pack | Tout accès cross-pack : arrêt immédiat |
| R-07 | Injection conduisant à une action | moyenne | critique | Isolation du contenu, contrôle n° 7, cas d'injection dans le banc | Un seul échec : blocage du lot |
| R-08 | Contamination du socle par des spécificités client | haute | moyen | Frontière normative, contrôle automatisé en CI, mesure de réutilisation au 2ᵉ projet | Terme métier client dans `src/core/` |
| R-09 | Dérive de coût par appels de classe L | moyenne | faible | Routage par classe, escalade plafonnée à une par étape, budget contrôlé par le code | Coût par ticket hors budget |
| R-10 | Glissement vers un projet ITSM complet | moyenne | moyen | Sous-ensemble ITIL borné, pratiques non couvertes en points de sortie | CMDB, CAB ou gestion de changement au backlog |
| R-11 | Perte de la référence croisée à l'escalade | moyenne | moyen | Liste d'identifiants externes dans le modèle canonique | Dossier sans identifiant du système cible |
| ~~R-12~~ | ~~Écosystème IA pauvre en Java~~ | — | — | **Clos par ADR-009 v0.2 (Python)** | — |
| **R-13** | **Taxonomie trop fine : l'autonomie devient statistiquement inatteignable** | **haute** | **moyen** | Seuil de volume minimal par catégorie d'évaluation, agrégation distincte de la taxonomie de triage | Plus de 30 % des catégories sous 3 % du volume |
| **R-14** | **Q-03 impose un modèle auto-hébergé : dimensionnement et exploitation changent de nature** | **moyenne** | **fort** | Port LLM abstrait ; branche de dimensionnement GPU chiffrée séparément | Réponse RSSI excluant tout modèle hébergé en zone de confiance |

---

## 17. Annexes

- `annexes/A1-instanciation-pilote.md` — instanciation du client pilote. **Seul document du DAv1 citant un ITSM nommé.** Aucun terme de cette annexe ne doit apparaître ailleurs dans le dossier ni dans `src/core/`.
- `docs/adr/` — registre des ADR.
- `docs/technical/contracts/` — les sept contrats d'agents en JSON Schema.
- `docs/traceability/matrice-tracabilite.md` — couverture DEC/EF/ENF → chapitres → ADR.
