# Exigences non fonctionnelles — Résilience, Maintenabilité, Portabilité, Réversibilité (NFR-07 à 10)

**Amont** : ENF-09, ENF-10, ADR-009, DAv1-15, R-05

---

## NFR-07 — Résilience

| Id | Énoncé | Cible | Méthode de mesure | Seuil d'alerte |
|---|---|---|---|---|
| NFR-07.1 | Dégradation maîtrisée (ENF-10) | Fail-closed sur tous les composants critiques listés en DAv1-06.4 | Test d'injection de panne par composant | Tout chemin automatique restant ouvert malgré une panne critique : défaut bloquant |
| NFR-07.2 | Idempotence des transitions | 100 % — rejouer une transition avec le même identifiant d'exécution ne produit aucun effet supplémentaire | Test d'idempotence automatisé | Tout effet dupliqué détecté : défaut bloquant |
| NFR-07.3 | Reprise après panne en cours de transition | Reprise au dernier état persisté, jamais au milieu | Test de coupure en cours de traitement | Perte d'état détectée : défaut bloquant |
| NFR-07.4 | Capacité de la file d'attente en mode dégradé | Absorbe 48 h de flux nominal sans perte (≈ 190 tickets à 2 000/mois) | Test de charge en mode dégradé | Débordement de file |
| NFR-07.5 | Test de continuité | Exercice de reprise complet, 1 fois par trimestre | Rapport d'exercice | Non tenu à la date prévue |

## NFR-08 — Maintenabilité

| Id | Énoncé | Cible | Méthode de mesure |
|---|---|---|---|
| NFR-08.1 | Couverture de tests sur `src/core/` | ≥ 85 % | Rapport de couverture CI, bloquant sous ce seuil |
| NFR-08.2 | Typage strict sur `src/core/` | 0 erreur `mypy --strict` | Pipeline CI, bloquant |
| NFR-08.3 | Dette technique (complexité cyclomatique) | Seuil d'alerte à 10 par fonction, refus au-delà de 15 | SonarQube |
| NFR-08.4 | Documentation vivante | Chaque agent du registre auto-documente son contrat, sa classe, sa règle d'escalade | Vérification de complétude au chargement du registre |
| NFR-08.5 | Versionnement des prompts, hors du code applicatif | 100 % des prompts en production référencés dans le registre versionné | Audit du registre |
| NFR-08.6 | Délai de correction d'une régression détectée par le banc d'évaluation | < 5 jours ouvrés | Suivi de ticket | > 10 jours ouvrés |

## NFR-09 — Portabilité

| Id | Énoncé | Cible | Méthode de mesure |
|---|---|---|---|
| NFR-09.1 | Changement de fournisseur de modèle sans réécrire les agents (ENF-09) | Aucune modification de `src/core/` | Test de réversibilité : basculer le banc d'évaluation sur un second fournisseur | 
| NFR-09.2 | Changement d'ITSM sans réécrire le pipeline | Nouvel adaptateur seulement | Exécution complète du pipeline sur l'adaptateur `mock`, sans modification du domaine |
| NFR-09.3 | Portabilité du frontend | Angular sans dépendance à un service propriétaire non substituable | Revue d'architecture frontend |

## NFR-10 — Réversibilité fournisseur

| Id | Énoncé | Cible | Méthode de mesure | Fréquence |
|---|---|---|---|---|
| NFR-10.1 | Réversibilité du fournisseur de modèle | Bascule testée et documentée | Exécution du test de réversibilité (NFR-09.1) | 1 fois par semestre |
| NFR-10.2 | Réversibilité de l'ITSM | Export et réimport testés | Exécution sur adaptateur `mock` | 1 fois par semestre |
| NFR-10.3 | Réversibilité de l'hébergeur | Déploiement complet testé sur un second environnement cible | Exercice de portabilité | 1 fois par an |
| NFR-10.4 | Réversibilité des données | Export documenté (trace, corpus, packs) dans des formats ouverts, réimport vérifié | Test d'export/réimport | 1 fois par an |
| NFR-10.5 | Délai de bascule fournisseur en cas d'urgence contractuelle | < 30 jours ouvrés, sur la base des exercices ci-dessus | Estimation documentée, révisée après chaque exercice | — |
