# F-07 — Event Storming

**Amont** : F-06 (domain model), DAv1-04 (agents) · **Aval** : S-04 (contrats d'événements AsyncAPI), ADR-014

Notation : `Événement` (orange, passé composé) · `Commande` (bleu) · `Politique` (mauve, règle automatique déclenchant une commande) · `Vue de lecture` (vert, ce qu'un acteur consulte) · `Point chaud` (rouge, question ouverte ou tension).

## Flux principal — de la réception à la clôture

```
[Commande: RecevoirMessage]
        ↓
Événement: TicketRecu
        ↓ (Politique: SiCourtCircuitDetecte)
        ↓                                    ↘
        ↓                          [Commande: BloquerPourSupervision]
        ↓                                    ↓
        ↓                          Événement: TicketBloqueSuperviseur
[Commande: NormaliserTicket]
        ↓
Événement: TicketNormalise
        ↓
   ⚠ POINT CHAUD : "score_completude est-il stable si le message
      est complété par un second envoi du demandeur avant traitement ?"
        ↓ (Politique: SiCompletudeInsuffisante)
        ↓                                    ↘
        ↓                       [Commande: GenererQuestionsDemandeur]
        ↓                                    ↓
        ↓                       Événement: QuestionsEnvoyeesAuDemandeur
        ↓ (sinon)
[Commande: TrierTicket]
        ↓
Événement: TicketTrie (DecisionTriage produite)
        ↓ (Politique: SiConfianceBasseCasLimite)
        ↓                                    ↘
        ↓                        [Commande: EscaladerVersClasseL] (une fois)
        ↓                                    ↓
        ↓                        Événement: DecisionRaffineeParClasseL
        ↓
[Commande: SoumettreAuSuperviseur]
        ↓
Événement: VerdictProduit
        ↓
   ⚠ POINT CHAUD : "que se passe-t-il si le verdict expire entre sa
      production et l'action de l'humain qui le valide en un clic ?"
        ↓ (Politique: SiVerdictAutoriseEtRunbookApplicable)
        ↓                                    ↘
        ↓                     [Commande: AppliquerRunbook]
        ↓                                    ↓
        ↓                     Événement: PlanActionExecute
        ↓ (sinon, si non concluant)
[Commande: InvestiguerN2]
        ↓
Événement: DiagnosticProduit
        ↓ (Politique: SiDiagnosticNonConcluant)
        ↓
[Commande: PreparerDossierEscalade]
        ↓
Événement: DossierEscaladeDepose
        ↓
Vue de lecture: DossierExpertN3 (consultée par l'Expert N3)
```

## Flux secondaire — promotion de connaissance

```
Événement: TicketResolu (multiple)
        ↓ (Politique: SiClusterDeTroisTicketsConcordants)
[Commande: SynthetiserBrouillonRunbook]
        ↓
Événement: BrouillonRunbookGenere
        ↓
Vue de lecture: FileDeRevue (consultée par le Propriétaire du corpus)
        ↓
[Commande: ApprouverBrouillon | CorrigerBrouillon | RejeterBrouillon]
        ↓
Événement: RunbookApprouve
        ↓ (Politique: SiActionsCiteesEnListeBlanche)
[Commande: PublierRunbook]
        ↓
Événement: RunbookPublieEtage2
        ↓
   ⚠ POINT CHAUD : "qui est notifié si aucune revue n'a lieu avant
      la date d'expiration ? Aujourd'hui : personne d'identifié — Q-05."
        ↓ (Politique: SiDateExpirationAtteinteSansRevue)
[Commande: RetrograderRunbook]
        ↓
Événement: RunbookRetrogradeEtage1
```

## Flux secondaire — gouvernance de l'autonomie

```
Événement: DecisionEvaluee (agrégée en continu par catégorie x action)
        ↓ (Politique: SiSeuilDescenteAtteint — 3 déclencheurs possibles)
[Commande: DescendreNiveauAutonomie]
        ↓
Événement: NiveauAutonomieDescendu

Événement: DecisionEvaluee
        ↓ (Politique: SiConditionsMonteeReunies — 5 conditions cumulatives)
[Commande: ProposerMonteeNiveau]
        ↓
Événement: MonteeProposee
        ↓
Vue de lecture: PropositionMontee (consultée par Responsable de service + Client)
        ↓
[Commande: ValiderMontee] (double validation requise)
        ↓
Événement: NiveauAutonomieMonte
```

## Politiques transverses (s'appliquent à plusieurs flux)

| Politique | Déclenche | Sur quel événement |
|---|---|---|
| `SiSecretDetecte` | `RejeterDocument` | Tout événement de collecte ou de normalisation |
| `SiChangementModeleOuPromptOuPack` | `InvaliderEvaluationsEtDescendreNiveau` | `SocleRedeploye`, `PackModifie`, `RegistrePromptsModifie` |
| `SiCompensantIndisponible` | `MettreEnFileModeDegrade` | Tout événement nécessitant Superviseur, passerelle modèles, ou index |

## Points chauds identifiés — à trancher en S-01/S-02

| Point chaud | Impact si non tranché | Portée vers |
|---|---|---|
| Stabilité du score de complétude en cas de complément asynchrone | Risque de double traitement ou d'état incohérent | S-01 (règle de fusion de messages sur un même ticket) |
| Expiration d'un verdict entre production et clic humain | Risque d'action exécutée sur verdict expiré, ou de blocage injustifié | S-10 (spécification du superviseur — durée de validité du verdict) |
| Notification d'absence de revue de runbook | Retour silencieux à l'étage 1, dégradation de couverture non détectée | Q-05 (propriétaire nommé) + S-11 (RAG — alerte de revue) |

## Vers les contrats d'événements

Chaque `Événement` de ce document devient une entrée du catalogue AsyncAPI (S-04). Convention de nommage retenue : `<Contexte>.<Agrégat><Participe passé>.v<version>`, par exemple `triage.TicketTrie.v1`, `supervision.VerdictProduit.v1`.
