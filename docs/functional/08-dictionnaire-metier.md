# F-08 — Dictionnaire métier

**Amont** : DAv0, DAv1, F-06 · **Aval** : tous les documents ultérieurs — ce dictionnaire fait autorité sur le vocabulaire

Règle : un terme de ce dictionnaire est utilisé **identiquement** dans le code, les spécifications, les échanges avec le métier. Un synonyme terrain existe et est toléré en conversation, jamais dans un artefact versionné.

| Terme canonique | Définition | Synonyme(s) terrain | Correspondance ITIL | Porté par | Piège à éviter |
|---|---|---|---|---|---|
| **Ticket** | Unité de travail entrant dans le pipeline, quel que soit le canal d'origine | « demande », « incident », « cas » (avant qualification) | Objet générique, se spécialise en `type` | Socle | Ne pas confondre avec l'objet natif de l'ITSM client — le Ticket est le modèle canonique |
| **Ticket canonique** | Ticket normalisé, produit par l'agent Intake | — | — | Socle | N'existe qu'après passage par l'Intake ; un message brut n'en est pas un |
| **Identifiant externe** | Référence d'un ticket dans un système tiers (origine ou cible d'escalade) | — | — | Socle | Toujours une **liste**, jamais un identifiant unique (EF-07) |
| **Type ITIL** | Incident, demande de service, ou problème | — | Noyau ITIL retenu (DEC-02) | Socle | Déterminé **avant** tout diagnostic — confondre incident et demande est la principale source de faux N2 |
| **Complétude** | Score entre 0 et 1 mesurant si le ticket contient assez d'information pour être traité | — | — | Socle | N'est pas une mesure de qualité rédactionnelle, seulement de suffisance informationnelle |
| **Mal-dirigé** | Écart détecté entre le chemin déclaré au formulaire et le contenu réel du ticket | — | — | Socle | Un signal, pas une accusation — le ticket reste traité normalement |
| **Priorité** | P1 à P4, **calculée** par la matrice impact × urgence du pack client | — | Matrice ITIL de priorisation | Pack client (matrice), Socle (calcul) | Ne jamais assigner directement — toujours dérivée |
| **Groupe support** | Équipe cible du routage, nommée dans le vocabulaire terrain | « équipe N1 », « TMA-XXX » | `groupe_support` interne | Pack projet | Le nom terrain n'apparaît jamais dans le socle |
| **Niveau cible** | N1, N2 ou N3 : niveau de support auquel le ticket doit être traité | — | — | Socle (calcul), Pack (mapping vers systèmes réels) | Ne pas confondre avec le niveau d'autonomie (curseur) |
| **Runbook** | Procédure de résolution documentée, avec actions et conditions | — | Procédure d'exploitation | Pack projet (contenu), Socle (mécanisme de promotion) | Un runbook en étage 1 n'est qu'une proposition, il n'autorise rien |
| **Corpus indicatif (étage 1)** | Ensemble de connaissances générées automatiquement, non validées | — | — | Socle (mécanisme), Pack (contenu) | Ne peut jamais déclencher une action |
| **Corpus normatif (étage 2)** | Ensemble de connaissances validées par un humain nommé | — | — | Socle (mécanisme), Pack (contenu) | Seul autorisé à justifier une action à effet de bord |
| **Liste blanche** | Ensemble exhaustif des actions qu'un agent est autorisé à exécuter | — | — | Pack projet | Une action absente de la liste blanche est refusée, sans exception ni contournement |
| **Réversibilité** | Propriété d'une action : peut-elle être annulée sans effet résiduel ? | — | — | Pack projet (déclaration), Socle (contrôle) | Une action non réversible exige toujours une validation humaine explicite |
| **Confiance** | Score 0-1 exprimant la certitude d'un agent sur sa sortie | — | — | Socle | N'est **pas** une preuve de justesse — se combine à sources, criticité, réversibilité |
| **Calibration** | Propriété mesurant si la confiance annoncée prédit effectivement la justesse | — | — | Socle (mesure) | Une confiance haute sur une décision fausse est le défaut prioritaire à détecter |
| **Verdict** | Sortie du Superviseur : AUTORISE, BLOQUE, VALIDATION_HUMAINE, ou MODE_DEGRADE | — | — | Socle | Immuable une fois produit — toute réévaluation crée un nouveau verdict |
| **Garde-fou absolu** | Une des 6 règles non désactivables, quel que soit le pack | — | — | Socle | Un pack peut durcir, jamais assouplir |
| **Court-circuit permanent** | Sortie immédiate du chemin automatique sur marqueur sécurité/confidentiel/impact vital/injection | — | — | Socle | Prioritaire sur le niveau d'autonomie configuré, quel qu'il soit |
| **Curseur d'autonomie** | Paramètre 0-4 gouvernant le degré de délégation à l'IA, par couple catégorie × action | « niveau d'automatisation » | — | Pack (valeur), Socle (mécanisme) | N'est pas une propriété du projet mais de chaque couple catégorie × action |
| **Pack client** | Contextualisation déclarative propre à un client : vocabulaire, seuils, souveraineté | — | — | Client | Ne contient jamais de code |
| **Pack projet** | Contextualisation déclarative propre à un projet : taxonomie, runbooks, liste blanche | — | — | Équipe projet | Hérite et peut durcir le pack client, jamais l'inverse |
| **Trace d'audit** | Enregistrement complet permettant de rejouer une décision à l'identique | « log », « historique » (impropres) | — | Socle | Distincte des logs applicatifs — c'est un livrable contractuel, pas un artefact d'exploitation |
| **Adaptateur** | Implémentation d'un port pour un système externe nommé | « connecteur », « intégration » | — | Selon contrat | Le socle ne dépend jamais d'un adaptateur, seulement d'un port |
| **Port** | Interface abstraite définissant un contrat d'accès à une famille de systèmes externes | — | — | Socle | Ne jamais laisser fuir un détail d'implémentation d'adaptateur dans un port |
| **Classe de modèle (S/M/L)** | Catégorie de complexité de raisonnement affectée à un appel de modèle | — | — | Socle (règles), Pack (fournisseur autorisé) | La classe est un paramètre de charge cognitive, pas un fournisseur |
| **Étanchéité** | Garantie qu'aucune donnée d'un client n'est visible depuis le contexte d'un autre | « cloisonnement », « isolation » | RGPD, ENF-08 | Socle | Garantie « par construction » signifie appliquée à l'index et au stockage, jamais seulement au prompt |
| **Mention IA** | Indication systématique qu'une communication a été produite ou assistée par une IA | — | AI Act (transparence) | Socle (injection structurelle) | N'est jamais une option du prompt — portée par le composant d'émission |

## Termes explicitement bannis du socle (`src/core/`)

Ces termes n'apparaissent que dans les packs, les adaptateurs, ou l'annexe pilote du DAv1 : tout nom d'ITSM nommé, tout nom de client, tout terme de vocabulaire terrain non générique. La liste exhaustive est dérivée automatiquement des packs chargés et alimente le contrôle CI de non-contamination (R-08).
