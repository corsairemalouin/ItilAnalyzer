# F-06 — Domain Model (DDD)

**Amont** : DAv1-05 (contextes bornés esquissés), DAv1-09 (objets pivots) · **Aval** : S-06 (modèle de données), S-01 (SFD)

## Carte des contextes bornés

| Contexte borné | Responsabilité | Agrégat racine | Relation avec les autres contextes |
|---|---|---|---|
| **Ingestion** | Réception, normalisation, déduplication, complétude | `Ticket` | Amont de tout le pipeline ; conformiste vis-à-vis du canal externe |
| **Triage** | Classification, priorisation, routage, mal-dirigé | `DecisionTriage` | Client de Connaissance (recherche), client de Ingestion |
| **Résolution** | Application de runbook, plan d'action, réponse | `PlanAction` | Client de Connaissance (étage normatif seul), client de Supervision |
| **Investigation** | Corrélation, hypothèse, diagnostic | `Diagnostic` | Client de Connaissance (étage indicatif), lecture seule sur systèmes externes |
| **Escalade** | Constitution et dépôt du dossier | `DossierEscalade` | Aval de Investigation ; conformiste vis-à-vis du système N3 |
| **Connaissance** | Ingestion, indexation, recherche, promotion, validité | `Document`, `Runbook` | Fournisseur (upstream) de Triage, Résolution, Investigation |
| **Supervision** | 7 contrôles, verdicts, garde-fous | `Verdict` | **Partenaire obligatoire** de tout contexte à effet de bord — aucun ne le contourne |
| **Gouvernance** | Packs, fusion, niveaux d'autonomie, seuils | `Pack`, `NiveauAutonomie` | Fournisseur de contexte (configuration) à tous les autres |
| **Observabilité** | Trace d'audit, métriques, évaluation | `Trace`, `RunEvaluation` | **Aval conformiste** de tous les contextes — chacun publie vers lui, aucun n'en dépend fonctionnellement |

### Relations inter-contextes (patrons DDD)

- **Supervision ↔ tout contexte à effet de bord** : relation de type *Open Host Service* — Supervision expose un contrat unique (`verdict`), que Résolution, Triage etc. consomment sans connaître son fonctionnement interne.
- **Connaissance → Triage/Résolution/Investigation** : *Customer/Supplier* — Connaissance publie un contrat de recherche stable ; les contextes clients ne peuvent pas en modifier le comportement.
- **Gouvernance → tous** : *Shared Kernel* restreint aux seuls objets `Pack` et `NiveauAutonomie`, jamais étendu.
- **Ingestion → système externe (canal)** : *Anticorruption Layer* — l'adaptateur de canal traduit le format externe vers le langage ubiquitaire du domaine ; aucun terme externe ne fuite dans `Ticket`.
- **Escalade → système N3** : *Anticorruption Layer* symétrique en sortie.

## Agrégats, entités, objets de valeur — détail des contextes structurants

### Contexte Ingestion

```
Agrégat Ticket (racine)
├── identifiants_externes : Liste<IdentifiantExterne>   [objet de valeur]
├── contenu : ContenuTicket                              [objet de valeur, PII masquées]
├── score_completude : Decimal[0,1]                       [objet de valeur]
├── doublon : InformationDoublon | absent                [objet de valeur]
├── marqueurs : Ensemble<Marqueur>                        [objet de valeur]
└── etat : EtatTicket                                     [énumération, machine à états]

Invariants :
- identifiants_externes ne peut jamais être vide après normalisation
- un Ticket portant un marqueur "injection_suspectee" ne peut transiter que vers BLOQUE_SUPERVISEUR
- score_completude est immuable une fois le ticket sorti de l'état NORMALISE
```

### Contexte Triage

```
Agrégat DecisionTriage (racine)
├── classification : { type, categorie, impact, urgence, priorite }  [objet de valeur]
├── routage : { niveau_cible, groupe_support }                        [objet de valeur]
├── mal_dirige : Booleen
└── explicabilite : { confiance, justification, sources }             [objet de valeur, obligatoire]

Invariant : priorite est TOUJOURS dérivée de impact x urgence par la matrice du pack —
            jamais assignée directement par un agent.
```

### Contexte Supervision

```
Agrégat Verdict (racine)
├── resultat : { AUTORISE | BLOQUE | VALIDATION_HUMAINE | MODE_DEGRADE }
├── controles : Liste<ResultatControle>[7]                [objet de valeur, exactement 7]
├── motif : Texte
├── court_circuit : Marqueur | absent
└── contexte_versions : { socle, pack_client, pack_projet, prompts, modele }  [objet de valeur]

Invariant : un Verdict est immuable après création (append-only) ; toute réévaluation
            produit un NOUVEAU Verdict, jamais une modification du précédent.
```

### Contexte Connaissance

```
Agrégat Document (racine)
├── contenu : ContenuDocument                              [anonymisé, jamais brut]
├── etage : { indicatif | normatif }
├── version : VersionSemantique
├── proprietaire : IdentiteTechnicien | absent (obligatoire si etage=normatif)
├── date_expiration_revue : Date | absent (obligatoire si etage=normatif)
└── sources_generatrices : Liste<ReferenceTicket>          [si généré automatiquement]

Invariant : un Document ne peut porter etage=normatif que s'il porte proprietaire
            ET date_expiration_revue ET que chacune de ses étapes cite une source.
```

### Contexte Gouvernance

```
Agrégat Pack (racine)
├── type : { client | projet }
├── identifiant, version
├── contenu : ContenuPackClient | ContenuPackProjet         [selon type]
└── parent : Pack | absent                                  [pack projet référence son pack client]

Agrégat NiveauAutonomie (racine)
├── cle : (categorie, action)                                [identité composite]
├── niveau : Entier[0,4]
├── metriques : { decisions_observees, anciennete_jours, taux_accord }
└── historique : Liste<EvenementChangementNiveau>            [append-only]

Invariant : niveau ne peut augmenter que via un EvenementChangementNiveau portant
            une double validation ; il peut diminuer via un événement système seul.
```

## Langage ubiquitaire — extrait

Voir dictionnaire complet en F-08. Termes structurants du domaine, à utiliser identiquement dans le code, les spécifications et les échanges avec le métier : `Ticket`, `DecisionTriage`, `Verdict`, `PlanAction`, `Diagnostic`, `DossierEscalade`, `Pack`, `NiveauAutonomie`, `Trace`, `Runbook`, `étage (indicatif/normatif)`, `garde-fou absolu`, `court-circuit permanent`.
