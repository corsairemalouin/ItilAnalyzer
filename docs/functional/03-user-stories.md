# F-03 — User stories

**Amont** : F-02 (use cases) · **Aval** : S-01 (spécifications fonctionnelles détaillées), S-13 (plan de tests)

Format : `US-xxx`, rattachement UC, critères d'acceptation en Gherkin. Groupées en épopées.

---

## Épopée EPIC-01 — Ingestion et normalisation (UC-01)

### US-101 — Normaliser un ticket complet
**En tant qu'** Intake, **je veux** produire un ticket canonique à partir d'un message brut, **afin de** l'engager dans le pipeline.

```gherkin
Scenario: Ticket complet normalisé avec succès
  Given un message brut reçu sur un canal configuré
  And le contenu ne porte aucun marqueur de court-circuit
  When l'Intake traite le message
  Then un contrat ticket-canonique valide est produit
  And le score de complétude est calculé
  And le ticket est tracé avant toute transition
```

### US-102 — Retourner des questions sur ticket incomplet
**En tant que** demandeur, **je veux** recevoir au plus 3 questions ciblées, **afin de** compléter mon ticket sans échange inutile.

```gherkin
Scenario: Ticket incomplet
  Given un ticket dont le score de complétude est inférieur à 0.5
  When l'Intake le traite
  Then au plus 3 questions sont générées
  And la réponse porte la mention de production par IA
  And le ticket passe à l'état EN_ATTENTE_DEMANDEUR
```

### US-103 — Détecter un doublon
**En tant que** technicien N1, **je veux** être averti d'un doublon, **afin de** ne pas traiter deux fois le même incident.

```gherkin
Scenario: Doublon détecté
  Given un ticket dont le contenu est fortement similaire à un ticket déjà ouvert
  When l'Intake le traite
  Then le champ doublon.est_doublon est vrai
  And un commentaire de rattachement réversible est proposé, jamais exécuté seul
```

### US-104 — Rejeter un document contenant un secret
**En tant que** RSSI, **je veux** qu'aucun secret ne soit jamais indexé, **afin de** garantir la sécurité du corpus.

```gherkin
Scenario: Secret détecté dans un message entrant
  Given un message contenant une chaîne reconnue comme secret ou identifiant sensible
  When l'anonymisation s'exécute
  Then le document est rejeté du traitement automatique
  And son propriétaire est notifié
  And aucun secret n'apparaît dans la trace ni dans l'index
```

---

## Épopée EPIC-02 — Triage et routage (UC-02)

### US-201 — Trier un ticket avec confiance suffisante
```gherkin
Scenario: Triage automatique
  Given un ticket canonique complet
  And la recherche de contexte retourne des sources pertinentes
  When le Triage produit sa décision
  Then confiance, justification et sources sont renseignées
  And la priorité est calculée par la matrice impact x urgence du pack client, jamais devinée
```

### US-202 — Afficher une proposition de triage incertaine
```gherkin
Scenario: Confiance sous le seuil
  Given une décision de triage dont la confiance est inférieure au seuil du pack
  When le Superviseur évalue le contrôle 4
  Then le verdict est VALIDATION_HUMAINE
  And la proposition est affichée au technicien, jamais appliquée seule
```

### US-203 — Escalader une seule fois vers la classe de raisonnement
```gherkin
Scenario: Cas limite entre deux catégories
  Given une décision de triage en confiance basse sur un cas limite
  When une escalade vers la classe L est déclenchée
  Then une seconde escalade n'est jamais déclenchée pour la même décision
  And la décision finale utilise le résultat de la classe L
```

### US-204 — Détecter un ticket mal dirigé
```gherkin
Scenario: Écart entre chemin déclaré et contenu réel
  Given un ticket dont le chemin déclaré au formulaire ne correspond pas au contenu analysé
  When le Triage produit sa décision
  Then mal_dirige est vrai
  And le groupe_support recalculé diffère du chemin déclaré
```

---

## Épopée EPIC-03 — Résolution N1 (UC-03)

### US-301 — Exécuter automatiquement une action réversible autorisée
```gherkin
Scenario: Résolution automatique au niveau 3
  Given un runbook normatif applicable
  And toutes les actions du plan figurent en liste blanche et sont réversibles
  And le niveau d'autonomie du couple catégorie x action est 3 ou plus
  And le verdict du Superviseur est AUTORISE
  When le plan d'action est soumis
  Then l'action s'exécute sans intervention humaine
  And elle reste annulable
  And la réponse au demandeur porte la mention IA
```

### US-302 — Exiger une validation en un clic
```gherkin
Scenario: Niveau d'autonomie insuffisant
  Given un plan d'action valide
  And le niveau d'autonomie du couple catégorie x action est 2
  When le verdict est produit
  Then le verdict est VALIDATION_HUMAINE
  And le technicien peut valider l'action en un clic sans la ressaisir
```

### US-303 — Refuser une action hors liste blanche
```gherkin
Scenario: Action non autorisée
  Given un plan d'action contenant une action absente de la liste blanche du pack projet
  When le Superviseur évalue le contrôle 3
  Then le verdict est BLOQUE
  And le motif nomme précisément l'action refusée
```

---

## Épopée EPIC-04 — Investigation et escalade (UC-04, UC-05)

### US-401 — Produire un diagnostic sourcé en lecture seule
```gherkin
Scenario: Investigation N2
  Given un ticket dans le catalogue N2
  When l'Investigateur produit son diagnostic
  Then chaque hypothèse porte au moins une source
  And aucune action d'écriture n'est tentée par cet agent
```

### US-402 — Constituer un dossier d'escalade complet
```gherkin
Scenario: Dépôt du dossier N3
  Given un diagnostic non concluant ou un sujet hors catalogue N2
  When le Préparateur N3 constitue le dossier
  Then le dossier contient l'identifiant d'origine et l'identifiant du système cible
  And le dossier est refusé au dépôt si l'un des deux manque
```

---

## Épopée EPIC-05 — Supervision (UC-06)

### US-501 — Bloquer en cas de doute
```gherkin
Scenario: Contrôle en échec
  Given une sortie d'agent dont au moins un des 7 contrôles échoue
  When le Superviseur produit son verdict
  Then le verdict est BLOQUE
  And le motif est lisible par un humain, sans jargon technique
```

### US-502 — Refuser toute action sans trace
```gherkin
Scenario: Absence de trace
  Given une tentative d'effet de bord sans trace préalablement écrite
  When l'adaptateur reçoit l'appel
  Then l'appel est refusé
  And aucune action n'est exécutée
```

### US-503 — Basculer en mode dégradé
```gherkin
Scenario: Composant critique indisponible
  Given l'indisponibilité du superviseur, de la passerelle modèles ou de l'index
  When un ticket doit être traité
  Then le ticket est mis en file
  And aucune décision approximative n'est produite
```

---

## Épopée EPIC-06 — Gouvernance de l'autonomie (UC-07)

### US-601 — Proposer une montée de niveau sur preuve
```gherkin
Scenario: Conditions de montée réunies
  Given au moins 200 décisions observées sur la catégorie
  And au moins 30 jours d'observation
  And un taux d'accord d'au moins 95 pour cent
  And zéro incident grave imputé
  When la proposition de montée est soumise
  Then elle est présentée pour validation conjointe service et client
  And le niveau ne change pas sans cette double validation
```

### US-602 — Redescendre automatiquement sans validation
```gherkin
Scenario: Dérive détectée
  Given un taux d'accord inférieur à 90 pour cent sur 7 jours glissants
  When le système évalue les seuils
  Then le niveau d'autonomie de la catégorie concernée redescend d'un cran
  And aucune validation humaine n'est requise pour cette descente
  And l'événement est tracé avec la valeur précédente
```

---

## Épopée EPIC-07 — Connaissance et promotion (UC-08)

### US-701 — Générer un brouillon de runbook sourcé
```gherkin
Scenario: Cluster suffisant
  Given au moins 3 tickets résolus concordants sur une même cause
  When la synthèse s'exécute
  Then un brouillon de runbook est produit
  And chaque étape du brouillon cite sa source
```

### US-702 — Publier un runbook approuvé
```gherkin
Scenario: Promotion validée
  Given un brouillon approuvé par un technicien nommé
  And toutes les actions citées existent en liste blanche
  When la publication s'exécute
  Then le runbook est versionné, signé, daté
  And une date d'expiration de revue est fixée
```

### US-703 — Rétrograder un runbook expiré
```gherkin
Scenario: Revue non effectuée
  Given un runbook normatif dont la date d'expiration de revue est dépassée
  When le système vérifie la validité documentaire
  Then le document retourne automatiquement à l'étage 1
  And il ne peut plus justifier d'action tant qu'il n'est pas revu
```

---

## Épopée EPIC-08 — Console technicien (UC-09)

### US-801 — Accepter une proposition en un clic
```gherkin
Scenario: Acceptation simple
  Given une proposition affichée avec ses sources et sa confiance
  When le technicien clique sur accepter
  Then l'action associée s'exécute selon le niveau d'autonomie
  And l'identité du technicien est inscrite dans la trace
```

### US-802 — Corriger une proposition
```gherkin
Scenario: Correction du technicien
  Given une proposition de triage jugée incorrecte
  When le technicien la corrige et enregistre sa correction
  Then la correction est conservée comme annotation de référence
  And elle contribue au calcul du taux d'accord
```

---

## Épopée EPIC-09 — Auditabilité (UC-10)

### US-901 — Rejouer une décision à l'identique
```gherkin
Scenario: Rejeu conforme
  Given une exécution passée entièrement tracée
  When un auditeur demande le rejeu
  Then la décision reconstituée est strictement identique à la décision d'origine
  And le diff retourné est vide
```

---

## Couverture UC → US

| UC | US couvrantes |
|---|---|
| UC-01 | US-101 à US-104 |
| UC-02 | US-201 à US-204 |
| UC-03 | US-301 à US-303 |
| UC-04, UC-05 | US-401, US-402 |
| UC-06 | US-501 à US-503 |
| UC-07 | US-601, US-602 |
| UC-08 | US-701 à US-703 |
| UC-09 | US-801, US-802 |
| UC-10 | US-901 |
