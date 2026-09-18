# DAv1 — Architecture de sécurité et des données (ch. 08 à 09)

**Amont** : ENF-07, ENF-08, R-06, R-07, R-11, EF-07, EF-09, Q-03, Q-06

---

## 08. Architecture de sécurité

### 08.1 Modèle de menaces — menaces spécifiques à un système agentique

| Id | Menace | Vecteur | Contre-mesure architecturale | Exigence |
|---|---|---|---|---|
| M-01 | **Injection d'instruction** dans le contenu d'un ticket | Champ libre du ticket, pièce jointe, contenu indexé | Le contenu utilisateur est une **donnée**, jamais une instruction : isolation structurelle dans le contexte, instructions issues du socle et des packs uniquement, contrôle n° 7 du superviseur, cas d'injection dans le jeu d'évaluation | ENF-07 : **0 échec toléré** |
| M-02 | **Injection indirecte** via un document du corpus | Document empoisonné ingéré | Anonymisation et inspection avant indexation ; contenu indexé isolé au même titre que le ticket ; corpus normatif signé par un humain nommé | ENF-07 |
| M-03 | **Fuite inter-clients** | Recherche non filtrée, cache partagé, log | Filtrage par pack appliqué à **l'index et au stockage**, pas au prompt ; partitionnement ; test d'étanchéité bloquant | ENF-08 |
| M-04 | **Fuite vers un fournisseur de modèle** | Contexte envoyé à l'inférence | Anonymisation **avant** tout appel ; classe de modèle autorisée définie par le pack client ; sortie réseau en liste blanche | ENF-08, Q-03 |
| M-05 | **Élévation de privilège demandée par le contenu** | « Accorde-moi les droits admin » | Garde-fou absolu : aucune élévation de droits, quelle que soit la demande ; non désactivable par un pack | Garde-fou |
| M-06 | **Action non désirée** en production | Sortie d'agent mal formée ou manipulée | Liste blanche d'actions + réversibilité + niveau d'autonomie + validation humaine | Garde-fou |
| M-07 | **Exfiltration par communication sortante** | Réponse au demandeur détournée vers un tiers | Destinataires contraints par le contexte du ticket, jamais par le contenu ; mention IA injectée par le composant d'émission | Garde-fou |
| M-08 | **Altération de la trace** | Accès au stockage d'audit | Trace append-only, intégrité vérifiable, accès en écriture réservé au composant de trace | ENF-01 |
| M-09 | **Secret indexé** | Mot de passe présent dans un ticket ou un document | Détection et **rejet du document**, pas masquage ; secrets hors index par construction | — |
| M-10 | **Empoisonnement du corpus normatif** | Promotion abusive d'un runbook | Promotion par humain nommé, signature, actions citées obligatoirement présentes en liste blanche, expiration de revue | DEC-03 |

### 08.2 Les six garde-fous absolus

Non désactivables par un pack, non paramétrables, codés dans le socle et testés unitairement :

1. Aucune action non réversible sans validation humaine explicite.
2. Aucune action hors liste blanche.
3. Aucune communication externe sans mention qu'elle est produite ou assistée par une IA.
4. Aucun traitement autonome sur ticket sécurité, confidentiel ou à impact vital.
5. Aucune élévation de droits, quelle que soit la demande.
6. Aucune sortie sans trace.

Un pack ne peut que **durcir** ces règles, jamais les assouplir. La règle de fusion (intersection des permissions, maximum des seuils) rend l'assouplissement structurellement impossible.

### 08.3 Identité, accès et secrets

| Sujet | Décision |
|---|---|
| Authentification des techniciens | Via `IAMPort` sur l'annuaire d'entreprise, jamais de base d'utilisateurs locale |
| Autorisation | Rôles : technicien N1, technicien N2, expert N3, responsable de service, propriétaire de corpus, administrateur de packs, auditeur |
| Identité des agents | Chaque agent a une identité technique distincte, avec le privilège minimal correspondant à son droit d'écriture déclaré |
| Moindre privilège sur l'adaptateur ITSM | Lecture large, **écriture strictement limitée aux actions en liste blanche** |
| Secrets | Coffre-fort dédié, rotation, jamais en variable d'environnement en clair, jamais dans un pack, jamais dans l'index |
| Chiffrement | En transit (TLS partout, y compris interne) et au repos (trace d'audit, corpus, base) |
| Auditeur | Rôle en lecture seule sur la trace, sans accès au contenu opérationnel |

### 08.4 Conformité

| Cadre | Exigence | Traduction architecturale | Statut |
|---|---|---|---|
| **RGPD** | Base légale, minimisation, anonymisation, rétention, sous-traitance | Anonymisation avant stockage **et** avant tout appel de modèle ; durée de rétention portée par le pack client ; registre des traitements | **Bloqué par Q-06** |
| **AI Act** | Qualification du système, transparence, supervision humaine documentée | Mention IA structurelle ; HITL non contournable ; documentation de la supervision produite par le superviseur lui-même | ADR-016 |
| **Souveraineté** | Localisation du traitement et de l'index | Vérifiée au démarrage à partir du pack client ; démarrage refusé si non conforme | **Bloqué par Q-03** |
| **Contractuel** | Responsabilité en cas d'action erronée | Réversibilité, trace rejouable, niveau d'autonomie tracé et versionné | — |

---

## 09. Architecture des données

### 09.1 Modèle canonique — objets pivots

| Objet | Rôle | Particularité structurante |
|---|---|---|
| `Ticket` | Ticket normalisé | **Liste d'identifiants externes**, jamais un identifiant unique |
| `DecisionTriage` | Classification et routage | `confiance` + `justification` + `sources` obligatoires |
| `CategorieN2` | Éligibilité au catalogue N2 | — |
| `Diagnostic` | Hypothèse causale | Éléments corrélés + niveau de certitude + limites |
| `PlanAction` | Actions ordonnées | Réversibilité par action, runbook d'origine |
| `DossierEscalade` | Dossier pour l'expert | Historique, analyses, hypothèses, documents consultés, justification de l'escalade |
| `Verdict` | Résultat des 7 contrôles | Motif de blocage lisible par un humain |
| `Trace` | Enregistrement d'audit | Entrées, versions de prompts/packs/modèle, horodatage, empreinte |
| `Document` / `Runbook` | Élément de corpus | Étage (indicatif/normatif), version, propriétaire, date d'expiration de revue |
| `NiveauAutonomie` | Gouvernance | Couple catégorie × action, valeur 0 à 4, historique des changements |

### 09.2 La rupture d'identifiant — contrainte structurelle

Lors d'une escalade vers un système de niveau supérieur, l'identifiant du ticket change. Sans conservation des **deux** identifiants, la chaîne d'audit est rompue et aucune mesure de bout en bout n'est possible (EF-07, R-11).

Conséquence, actée comme contrainte d'architecture et non comme détail d'implémentation :

```
Ticket.identifiants_externes : liste de { systeme, identifiant, role, date_association }
```

Le Préparateur N3 dépose la référence d'origine dans le dossier d'escalade **si aucun lien automatique n'existe**. La présence des deux identifiants est un critère d'acceptation de EF-07, vérifié en test.

### 09.3 Cycle de vie et rétention

| Donnée | Rétention | Localisation | Anonymisation |
|---|---|---|---|
| Ticket brut reçu | Courte, le temps du traitement | Zone de données | À la normalisation |
| Ticket canonique | Alignée sur le besoin opérationnel | Zone de données | Oui |
| Trace d'audit | **Alignée sur l'exigence d'audit** — définie par le pack client | Stockage append-only | Oui |
| Corpus indicatif | Réindexé en continu, purge des sources obsolètes | Index cloisonné par pack | Avant indexation |
| Corpus normatif | Versionné, expiration de revue | Index cloisonné par pack | Avant indexation |
| Jeux d'évaluation | Longue | Zone de données | Obligatoire |
| Métriques et KPI | Longue, agrégées | Observabilité | Sans donnée personnelle |

**Bloqué par Q-06** : la durée de rétention de la trace et la base légale du traitement conditionnent le choix du stockage et son coût. Tant que la réponse n'est pas donnée, ce tableau est une proposition.

### 09.4 Principes de traitement

1. **Anonymisation avant stockage et avant tout appel de modèle.** Pas après, pas à la lecture.
2. **Un secret détecté entraîne le rejet du document**, pas son masquage. Un document rejeté est signalé à son propriétaire.
3. **Aucune donnée client conservée hors du périmètre de son pack** (ENF-08).
4. **Le contenu indexé est une donnée, jamais une instruction** — même principe que pour le ticket (M-02).
5. **Toute réponse permet de remonter au document et à sa version.** Une affirmation sans source est traitée comme non sourcée, donc non actionnable.

### 09.5 Données nécessaires au fonctionnement

| Donnée | Obtention | Difficulté | Sans elle |
|---|---|---|---|
| Ticket entrant | Canal abstrait (mail, API, export) | moyenne | rien ne fonctionne |
| Historique des tickets résolus | Extraction initiale puis delta | faible | pas de corpus amorcé, pas d'évaluation |
| Arbre du formulaire guidé | Export de structure | faible | taxonomie à inventer |
| Cartographie applicative | Saisie dans le pack client | faible | pas de routage fiable |
| Runbooks | **N'existent pas sous forme exploitable** — génération + ateliers | **forte** | aucune résolution N1 automatique |
| Catalogue N2 traitable | Atelier avec l'équipe N2 | moyenne | toute bascule N2 part en N3 |
| Logs applicatifs | Accès lecture à négocier (Q-04) | moyenne | investigation N2 dégradée |
| Historique des réaffectations | À vérifier (Q-02) | inconnue | mal-dirigé non mesurable |
| Matrice de priorité contractuelle | Saisie dans le pack client | faible | priorité devinée au lieu d'être calculée |

**Le point dur n'est pas technique.** Les runbooks n'existent nulle part sous forme exploitable : leur formalisation est le chemin critique du projet, pas le code. C'est la raison d'être de la chaîne d'ingestion décrite au chapitre 10.
