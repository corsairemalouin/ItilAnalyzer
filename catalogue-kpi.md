# Catalogue KPI — Métier, IA, Exploitation

**Amont** : DAv0 §KPI requis, DAv1-13.3, NFR-01 à 12 · **Aval** : implémentation des dashboards (phase 5), `PATCH /gouvernance/autonomie` (usage en pilotage)

Format : identifiant, définition, formule, source, fréquence, cible, seuil d'alerte, dashboard, usage dans le pilotage du curseur.

## KPI métier

| Id | Nom | Formule | Source | Fréquence | Cible | Seuil d'alerte | Dashboard | Usage curseur |
|---|---|---|---|---|---|---|---|---|
| KPI-BM-01 | Temps moyen de traitement | Σ(clôture − réception) / n tickets | Trace | Quotidien | Baseline − 30 % à 6 mois | Dégradation > 10 % sur 7j | Performance | Non |
| KPI-BM-02 | Temps moyen de qualification | Σ(fin triage − réception) / n | Trace | Quotidien | < 5 min automatisé | > 15 min moyenne | Performance | Non |
| KPI-BM-03 | Taux de résolution automatique | tickets résolus niveau ≥3 / total N1 | Trace | Quotidien | Croissant, sans cible fixe imposée | Chute brutale > 20 % semaine | Résolution | **Oui — condition de montée** |
| KPI-BM-04 | Taux d'escalade | tickets escaladés N3 / total | Trace | Quotidien | Baseline − décroissant | Hausse > 15 % | Résolution | Non |
| KPI-BM-05 | Taux de réouverture | tickets rouverts / tickets clos | ITSM (via adaptateur) | Hebdomadaire | < baseline historique | > baseline | Qualité | Non |
| KPI-BM-06 | Respect SLA | tickets dans SLA / total | Trace + pack client | Quotidien | ≥ engagement contractuel | < engagement − 5 pts | Performance | **Oui — contrôle 6 du superviseur** |
| KPI-BM-07 | Respect OLA | idem, niveau opérationnel interne | Trace | Quotidien | ≥ cible interne | idem | Performance | Non |
| KPI-BM-08 | Taux de satisfaction | Enquête ou proxy (taux d'acceptation) | Enquête / Journal des décisions | Mensuel | Suivi | Baisse significative | Qualité | Non |
| KPI-BM-09 | Taux de mal-dirigé résiduel | tickets mal dirigés détectés / total | Décision de triage | Hebdomadaire | Décroissant vs baseline | Hausse | Résolution | Non |

## KPI IA

| Id | Nom | Formule | Source | Fréquence | Cible | Seuil d'alerte | Dashboard | Usage curseur |
|---|---|---|---|---|---|---|---|---|
| KPI-IA-01 | Confiance moyenne | moyenne(confiance) par agent | Trace | Quotidien | Suivi, corrélé à la calibration | Chute brutale | IA | Non directement |
| KPI-IA-02 | Coût par ticket | Σ coût d'appels / ticket | Passerelle modèles | Quotidien | Budget du pack (Q-13) | > 120 % budget sur 7j | IA | Non |
| KPI-IA-03 | Nombre d'appels LLM par ticket | Σ appels / ticket | Passerelle modèles | Quotidien | ~4 à 7 (S+M dominant) | Hausse anormale | IA | Non |
| KPI-IA-04 | Taux d'acceptation des recommandations | acceptées / (acceptées + corrigées + rejetées) | Journal des décisions | Hebdomadaire | ≥ 60 % en fin de pilote | < 50 % (R-04) | IA | **Oui — condition de montée** |
| KPI-IA-05 | Taux de correction humaine | corrigées / total propositions | Journal des décisions | Hebdomadaire | Décroissant | Hausse sur 4 semaines | IA | **Oui — condition de montée** |
| KPI-IA-06 | Taux d'hallucination détectée | sorties sans source valide / sorties avec affirmation | Banc d'évaluation + contrôle continu | Hebdomadaire | < 2 % | > 5 % | IA | **Oui — bloque la montée** |
| KPI-IA-07 | Taux d'accord technicien | décisions confirmées / décisions évaluées | Journal des décisions | Glissant 7 jours | ≥ 95 % pour montée | < 90 % | IA | **Oui — condition de montée et déclencheur de descente** |
| KPI-IA-08 | Écart de calibration | |confiance moyenne − justesse observée| | Banc d'évaluation | À chaque run | < 10 points | ≥ 10 points | IA | **Oui — bloque la montée** |
| KPI-IA-09 | Part d'appels par classe (S/M/L) | appels classe X / total | Passerelle modèles | Quotidien | ~60/35/5 | Dérive > 10 pts vs cible | IA | Non |

## KPI Exploitation

| Id | Nom | Formule | Source | Fréquence | Cible | Seuil d'alerte | Dashboard | Usage curseur |
|---|---|---|---|---|---|---|---|---|
| KPI-EX-01 | Disponibilité | temps up / temps fenêtre de service | Supervision technique | Mensuel | 99,5 % (NFR-01.1) | < 99,5 % | Exploitation | Non |
| KPI-EX-02 | Temps de réponse par étape | p95 par agent | Trace | Quotidien | Voir NFR-02 | Dépassement budget étape | Exploitation | Non |
| KPI-EX-03 | Taux d'échec par agent | échecs / appels | Trace | Quotidien | < 1 % | > 3 % | Exploitation | Non |
| KPI-EX-04 | Consommation GPU | si auto-hébergement (Q-03) | Infrastructure | Quotidien | Selon dimensionnement | > 80 % soutenu | Exploitation | Non |
| KPI-EX-05 | Consommation LLM (jetons) | Σ jetons / période | Passerelle modèles | Quotidien | Selon budget | Dérive vs budget | Exploitation | Non |
| KPI-EX-06 | Décisions bloquées par le superviseur | count(verdict=BLOQUE) par motif | Trace | Quotidien | < 25 % du total | > 25 % (DAv1-11.4) | Exploitation | Non, mais déclenche analyse |
| KPI-EX-07 | Âge du corpus normatif | date courante − date_publication, par document | Corpus | Quotidien | < date_expiration_revue | Revue expirée | Exploitation / IA | **Oui — retour automatique à l'étage 1** |

## Principe de gouvernance du catalogue

Ce catalogue est la **source unique** des définitions de KPI : un dashboard n'invente jamais un calcul, il consomme une définition d'ici. Toute modification de formule est un changement versionné du catalogue, avec effet rétroactif documenté sur l'historique déjà publié (les valeurs passées ne sont jamais recalculées silencieusement).

Les KPI marqués **« Oui »** en colonne « Usage curseur » sont ceux qui alimentent directement le mécanisme automatique de montée ou de descente décrit en DAv1-12.3/12.4 — ce sont les seuls dont le calcul doit être certifié par le banc d'évaluation avant toute mise en production, au même titre qu'un composant du chemin critique.
