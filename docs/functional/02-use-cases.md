# F-02 — Use cases

**Amont** : F-01, DAv1-04 (7 agents, arbre à 4 portes) · **Aval** : F-03 (user stories), S-01 (SFD)

Format complet : acteurs, préconditions, scénario nominal, alternatifs, exceptions, postconditions, règles applicables. Les UC ci-dessous sont développés intégralement pour les cas structurants ; les suivants sont listés avec renvoi au patron du premier de leur famille.

---

## UC-01 — Normaliser un ticket entrant

**Amont** : BM-01, BM-02, EF-01, EF-02 · **Acteur principal** : Agent Intake · **Acteurs secondaires** : Adaptateur canal, Superviseur

**Préconditions** : un message est disponible sur un canal configuré (mail, API, export).

**Scénario nominal**
1. L'adaptateur de canal transmet le message brut à l'Intake.
2. L'Intake anonymise le contenu (PII, secrets) avant tout traitement.
3. L'Intake extrait les champs du modèle canonique et calcule le score de complétude.
4. L'Intake vérifie la présence d'un doublon parmi les tickets ouverts.
5. L'Intake produit le contrat `ticket-canonique`.
6. Le Superviseur contrôle la sortie (contrôles 1, 2, 7).
7. Le ticket transite vers Triage (US-101 à suivre).

**Scénarios alternatifs**
- A1 (complétude < 0,5) : l'Intake génère au plus 3 questions ; le ticket passe à l'état `EN_ATTENTE_DEMANDEUR` ; les questions sont envoyées avec mention IA.
- A2 (doublon détecté) : le ticket est rattaché au ticket d'origine, un commentaire de rattachement est proposé (garde-fou : reste un commentaire réversible).

**Scénarios d'exception**
- E1 (marqueur de court-circuit détecté — sécurité, confidentiel, impact vital, injection) : le ticket passe directement à `BLOQUE_SUPERVISEUR`, reprise humaine immédiate.
- E2 (secret détecté dans le contenu) : le document/message est rejeté du traitement automatique, signalé, traité manuellement.
- E3 (passerelle modèles indisponible) : le ticket est mis en file (`EN_FILE_MODE_DEGRADE`), aucune normalisation devinée.

**Postconditions** : ticket canonique produit et tracé, ou ticket en attente du demandeur, ou ticket bloqué avec motif.

**Règles applicables** : BR-01, BR-02, BR-10 (voir F-09).

---

## UC-02 — Trier un ticket

**Amont** : BM-01, BM-07, EF-03, EF-10 · **Acteur principal** : Agent Triage

**Préconditions** : ticket canonique disponible, complétude ≥ 0,5.

**Scénario nominal**
1. Triage recherche le contexte pertinent (index hybride, borné au pack, étage autorisé).
2. Triage détermine type ITIL, catégorie, impact, urgence, priorité (calculée), équipe, niveau cible.
3. Triage évalue l'écart entre chemin déclaré et contenu réel (mal-dirigé).
4. Triage produit le contrat `decision-triage` avec confiance, justification, sources.
5. Le Superviseur contrôle la sortie.
6. Selon le verdict, le ticket est proposé au technicien ou route automatiquement (Porte 2).

**Scénarios alternatifs**
- A1 (confiance basse ou cas limite) : escalade à la classe L, **une seule fois**, puis décision finale au niveau atteint.
- A2 (confiance ≥ seuil et niveau d'autonomie suffisant) : routage automatique, sans affichage préalable, mais tracé.

**Scénarios d'exception**
- E1 (aucune catégorie du pack ne correspond) : catégorie `hors-referentiel`, niveau cible forcé à N2, confiance nulle, remontée au propriétaire de pack.

**Postconditions** : décision de triage tracée ; ticket route vers Résolution N1, Investigation N2, ou proposition technicien.

**Règles applicables** : BR-03, BR-04, BR-05.

---

## UC-03 — Résoudre un ticket N1 par runbook

**Amont** : BM-01, BM-04, EF-05 · **Acteur principal** : Agent Résolveur N1

**Préconditions** : niveau cible = N1, un runbook **normatif** existe pour la catégorie.

**Scénario nominal**
1. Le Résolveur recherche un runbook normatif applicable (étage 2 uniquement).
2. Il construit un plan d'action à partir des actions du runbook, toutes vérifiées présentes en liste blanche.
3. Il rédige la réponse au demandeur, avec mention IA.
4. Il produit le contrat `plan-action`.
5. Le Superviseur applique les 7 contrôles.
6. Selon verdict et niveau d'autonomie du couple catégorie × action : exécution automatique (niveau ≥ 3), validation en un clic (niveau 2), ou suggestion (niveau 1).

**Scénarios alternatifs**
- A1 (aucun runbook normatif applicable) : bascule vers UC-04 (Investigation N2).
- A2 (action hors liste blanche référencée par le runbook) : le runbook est retiré de l'étage 2 automatiquement, incident de gouvernance signalé (le pack projet doit être corrigé).

**Scénarios d'exception**
- E1 (contrôle 4, confiance sous seuil) : dégradation en `VALIDATION_HUMAINE` quel que soit le niveau d'autonomie configuré.

**Postconditions** : action exécutée (tracée, réversible) ou préparée pour validation.

**Règles applicables** : BR-06, BR-07, BR-08, BR-11 (garde-fous absolus).

---

## UC-04 — Investiguer un ticket N2

**Amont** : BM-02, BM-05, EF-06 · **Acteur principal** : Agent Investigateur N2

**Préconditions** : niveau cible = N2, sujet dans le catalogue N2 (UC préalable : classification N2).

**Scénario nominal**
1. L'Investigateur corrèle logs, historique et cas similaires (lecture seule, aucune écriture).
2. Il formule une ou plusieurs hypothèses, chacune sourcée, avec niveau de certitude et limites explicites.
3. Il produit le contrat `diagnostic`.
4. Le Superviseur contrôle (aucune écriture possible pour cet agent : contrôle 3 trivialement satisfait).
5. Le diagnostic est présenté au technicien N2, qui exécute.

**Scénarios alternatifs**
- A1 (hors catalogue N2) : bascule directe vers UC-05 (préparation d'escalade N3), sans passage par l'investigation.

**Scénarios d'exception**
- E1 (diagnostic non concluant) : bascule vers UC-05.
- E2 (accès aux logs indisponible ou hors périmètre — Q-04) : diagnostic produit avec `limites` mentionnant l'absence de logs, confiance réduite.

**Postconditions** : diagnostic tracé et sourcé, présenté en lecture seule.

**Règles applicables** : BR-09.

---

## UC-05 — Préparer un dossier d'escalade N3

**Amont** : BM-08, EF-07 · **Acteur principal** : Agent Préparateur N3

**Préconditions** : diagnostic non concluant, ou hors catalogue N2.

**Scénario nominal**
1. Le Préparateur rassemble historique, analyses réalisées, hypothèses, données collectées, documents consultés.
2. Il vérifie la présence des identifiants externes (origine **et** cible d'escalade) ; s'il n'existe aucun lien automatique, il dépose explicitement la référence d'origine dans le dossier.
3. Il produit le contrat `dossier-escalade` avec justification de l'escalade.
4. Le Superviseur contrôle la présence obligatoire des deux identifiants (critère d'acceptation de EF-07).
5. Le dossier est déposé côté système N3.

**Scénarios d'exception**
- E1 (identifiant cible non encore attribué au moment du dépôt) : dépôt différé, ticket en attente de bascule, alerte si délai dépassé.

**Postconditions** : dossier complet déposé, chaîne d'identifiants préservée.

**Règles applicables** : BR-12.

---

## UC-06 — Superviser une sortie d'agent

**Amont** : BM-09, BM-13, BM-14, DEC-06 · **Acteur principal** : Superviseur (code)

**Préconditions** : une sortie d'agent est disponible.

**Scénario nominal**
1. Le Superviseur évalue les 7 contrôles, tous, même après un premier échec (pour produire un motif complet).
2. Il produit le contrat `verdict`.
3. La trace est écrite **avant** toute notification du verdict à l'appelant.
4. Le verdict est retourné ; l'adaptateur à effet de bord vérifie sa validité et sa non-expiration avant d'agir.

**Scénarios d'exception**
- E1 (composant nécessaire à un contrôle indisponible) : verdict `MODE_DEGRADE`, fail-closed sur les contrôles concernés.
- E2 (court-circuit permanent détecté) : verdict `BLOQUE` immédiat, sans évaluation des autres contrôles inutile — mais tous les contrôles déjà évalués restent dans la trace.

**Postconditions** : verdict tracé, motif lisible par un humain en cas de blocage.

**Règles applicables** : les 6 garde-fous absolus (F-09), BR-13 à BR-19.

---

## UC-07 — Piloter le niveau d'autonomie

**Amont** : BM-15, DEC-05 · **Acteur principal** : Responsable de service · **Acteur secondaire** : le système (descente automatique)

**Scénario nominal (montée)**
1. Le système calcule en continu, par couple catégorie × action : nombre de décisions observées, ancienneté, taux d'accord, incidents.
2. Lorsque les 5 conditions sont réunies, une proposition de montée est soumise à validation conjointe service + client.
3. Sur double validation, le niveau change ; l'événement est tracé (auteur, date, motif, valeur précédente).

**Scénario nominal (descente)**
1. Le système détecte un déclencheur de descente (accord < 90 % sur 7 jours, incident imputable, changement de modèle/pack sans réévaluation).
2. Le niveau redescend automatiquement, sans validation requise.
3. L'événement est tracé.

**Règles applicables** : BR-20 à BR-24.

---

## UC-08 — Promouvoir un document du corpus indicatif au corpus normatif

**Amont** : BM-04, BM-06, DEC-03 · **Acteur principal** : Propriétaire du corpus (rôle nommé, Q-05)

**Scénario nominal**
1. Le système génère un brouillon de runbook à partir d'un cluster d'au moins 3 tickets concordants, avec une source par étape.
2. Le propriétaire du corpus revoit le brouillon : approuve, corrige, ou rejette.
3. Sur approbation, le système vérifie que toutes les actions citées existent en liste blanche.
4. Le document est publié : versionné, signé du nom de l'approbateur, daté, avec date d'expiration de revue fixée.

**Scénarios d'exception**
- E1 (action citée absente de la liste blanche) : le runbook reste en statut « proposition », non publiable jusqu'à correction du pack ou du brouillon.
- E2 (date d'expiration atteinte sans revue) : retour automatique à l'étage 1, sans validation.

**Règles applicables** : BR-25 à BR-29.

---

## UC-09 — Contester ou corriger une proposition

**Amont** : BM-03, BM-09, ENF-05 · **Acteur principal** : Technicien

**Scénario nominal**
1. Le technicien reçoit une proposition (triage, plan d'action, diagnostic).
2. Il l'accepte en un clic, ou la corrige, ou la rejette avec motif.
3. La contestation est enregistrée comme annotation de référence, utilisée par le banc d'évaluation et le calcul du taux d'accord.

**Règles applicables** : BR-30.

---

## UC-10 — Rejouer une décision passée

**Amont** : BM-10, EF-09, ENF-02 · **Acteur principal** : Auditeur

**Scénario nominal**
1. L'auditeur sélectionne une exécution par son identifiant.
2. Le système reconstitue la décision à partir de la trace : mêmes entrées, mêmes versions de prompts/packs/modèle.
3. Le système compare la décision rejouée à la décision d'origine et retourne un diff.

**Scénarios d'exception**
- E1 (diff non vide) : incident de traçabilité, remontée immédiate — ce cas ne doit jamais se produire en production (critère de passage en pilote : rejeu vérifié à 100 %).

**Règles applicables** : BR-31.

---

## Table de couverture EF → UC

| EF | UC couvrant |
|---|---|
| EF-01, EF-02 | UC-01 |
| EF-03, EF-10 | UC-02 |
| EF-04 | UC-01 (A2) |
| EF-05 | UC-03 |
| EF-06 | UC-04 |
| EF-07 | UC-05 |
| EF-08 | UC-06 |
| EF-09 | UC-10 |
