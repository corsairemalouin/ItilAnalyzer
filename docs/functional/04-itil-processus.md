# F-04 — Processus ITIL v4

**Amont** : DEC-02, DAv0 §Arbitrage 2 · **Aval** : F-05 (BPMN), F-06 (domain model)

## Périmètre ITIL retenu

| Pratique ITIL | Statut v0 | Traitement dans le pipeline |
|---|---|---|
| Gestion des incidents | Cœur | Chemin nominal Intake → Triage → N1 → N2 → N3 |
| Gestion des demandes de service | Cœur | Branche séparée : catalogue de services, pas de diagnostic, SLA propre |
| Gestion des problèmes | Détection seule | Le pipeline détecte les récurrences et propose un enregistrement de problème ; il ne le traite pas |
| Gestion des changements | Point de sortie | Toute action requérant un changement est routée vers l'humain, jamais exécutée |
| Gestion des configurations (CMDB) | Consommée | La cartographie applicative du pack projet joue ce rôle en v0 |
| Gestion des connaissances | Alimentée | Runbooks et cas résolus — voir corpus à deux étages, DAv1-10 |
| Gestion des niveaux de service | Contrainte | Le SLA restant est un des 7 contrôles du superviseur |

**Limite assumée** : l'alignement ITIL porte sur le modèle de données et le vocabulaire, pas sur la gouvernance ITIL (CAB, rôles, cycle de vie du service complet). Toute demande fonctionnelle qui pousse vers un CAB, une gestion de changement complète ou une CMDB pleine est un signal du risque R-10 (glissement de périmètre) et doit être refusée ou reportée hors du produit.

## Processus 1 — Gestion des incidents (chemin nominal)

**Déclencheur** : ouverture d'un ticket de type `incident`.

**Étapes** :
1. Identification et enregistrement (UC-01)
2. Catégorisation et priorisation (UC-02)
3. Diagnostic initial et résolution N1 (UC-03) ou investigation N2 (UC-04)
4. Escalade fonctionnelle si nécessaire (UC-05)
5. Résolution et clôture — **clôture toujours humaine**

**SLA** : porté par le pack client, dérivé de la matrice impact × urgence.

## Processus 2 — Gestion des demandes de service

**Déclencheur** : ouverture d'un ticket de type `demande`.

**Différence structurante avec l'incident** : pas de diagnostic, pas d'investigation N2. Le pipeline vérifie l'éligibilité au catalogue de services du pack projet, applique un runbook de traitement de demande standard (même mécanisme de liste blanche et de supervision), et route vers validation humaine si l'action requise n'est pas dans le catalogue.

**SLA propre**, distinct de celui des incidents, porté par le pack client.

**Règle de non-confusion** (justifiée par le DAv0) : confondre incident et demande de service est la principale source de faux N2. Le Triage doit déterminer le `type` ITIL **avant** toute étape de diagnostic ; une demande ne doit jamais entrer dans le chemin d'investigation N2.

## Processus 3 — Détection de problème (sans traitement)

**Déclencheur** : récurrence détectée par corrélation de cause entre plusieurs incidents (fonction portée par la classe L, en tâche de fond asynchrone).

**Sortie** : une proposition d'enregistrement de problème, présentée au responsable de service. Le pipeline **ne traite jamais** le problème lui-même — c'est un point de sortie explicite vers le processus de gestion des problèmes de l'organisation, hors périmètre du produit.

## Processus 4 — Point de sortie changement

**Déclencheur** : une action requise par une résolution implique une modification de configuration, de code ou d'infrastructure au sens ITIL du changement.

**Comportement** : le pipeline ne propose jamais une telle action en liste blanche (contrainte portée par le pack projet à la conception de la liste blanche elle-même — aucune action de type changement ne doit y figurer). Si un agent identifie qu'un changement serait nécessaire, il le formule comme recommandation dans le diagnostic ou le dossier d'escalade, sans tenter de l'exécuter.

## Processus 5 — Promotion de connaissance (transverse)

Décrit en détail dans DAv1-10 et UC-08. Rattaché à la pratique « gestion des connaissances » d'ITIL : c'est le mécanisme par lequel l'expérience opérationnelle redevient un actif utilisable.

## Table de correspondance ITIL ↔ terrain

Portée par le pack projet, jamais codée en dur :

| Concept ITIL (interne) | Surface terrain (ce que voit l'équipe) |
|---|---|
| `type` : incident / demande / problème | Nature du ticket telle que libellée dans l'ITSM du client |
| `impact` × `urgence` → `priorité` | Matrice de priorité du contrat de service |
| `catégorie` hiérarchique | Chemins du formulaire guidé, exportés |
| `groupe_support` | Équipes N1/N2/N3 nommées côté client |
| `état` canonique | Statuts natifs de l'ITSM N1/N2, statuts natifs du système N3 |

Le **formulaire guidé amont** n'est pas un détail d'ergonomie : ses chemins sont la taxonomie déjà écrite, à exporter, pas à réinventer en atelier. C'est aussi la source du signal de mal-dirigé (écart chemin déclaré / contenu réel).
