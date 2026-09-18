# F-10 — Matrice RACI

**Amont** : F-04 (processus), DAv1-02 (parties prenantes), Q-05 (propriétaire du corpus) · **Aval** : documentation d'exploitation, procédures d'astreinte

R = Responsible (réalise) · A = Accountable (rend compte, un seul par ligne) · C = Consulted (consulté avant décision) · I = Informed (informé après action)

| Processus / Activité | Technicien N1 | Technicien N2 | Expert N3 | Responsable de service | Propriétaire du corpus | RSSI / Auditeur | Éditeur du socle | Système (agents) |
|---|---|---|---|---|---|---|---|---|
| Traitement N1 automatique (niveau ≥ 3) | I | — | — | A | — | — | — | R |
| Validation d'une proposition N1 (niveau 2) | R/A | — | — | I | — | — | — | R (prépare) |
| Investigation N2 | C | R/A | — | I | — | — | — | R (diagnostic) |
| Escalade et dossier N3 | — | R | A | I | — | — | — | R (constitue) |
| Traitement expert N3 | — | I | R/A | I | — | — | — | — |
| Contestation d'une proposition | R | R | — | I | — | — | — | I |
| Décision de blocage du superviseur | I | I | I | I | — | I | C (règles) | R/A |
| Promotion d'un runbook (étage 1→2) | — | C | C | I | R/A | — | C (liste blanche) | R (brouillon) |
| Revue périodique du corpus normatif | — | — | — | I | R/A | — | — | I (alerte expiration) |
| Proposition de montée du curseur d'autonomie | I | I | — | R | — | C | — | R (calcule, propose) |
| Validation de montée du curseur (double validation) | — | — | — | A/R | — | C | — | I |
| Descente automatique du curseur | I | I | — | I | — | I | — | R/A |
| Définition du pack client | I | I | — | C | — | A/R | C | — |
| Définition du pack projet | C | C | C | A/R | C | I | C | — |
| Contrôle de non-contamination du socle (R-08) | — | — | — | I | — | C | A/R | R (CI) |
| Consultation de la trace d'audit | — | — | — | C | — | R/A | I | I |
| Rejeu d'une décision passée | — | — | — | I | — | R/A | C | R (exécute) |
| Analyse d'un incident imputé au pipeline | I | I | — | A | — | R | C | I |
| Définition des seuils de sécurité et de confiance | — | C | — | C | — | A/R | C | — |
| Gestion des accès et des rôles (IAM) | I | I | I | C | — | A/R | — | — |
| Décision de mise en pilote (GATE lots DAv0) | I | I | I | A | C | C | R | — |
| Mesure des KPI et dashboards | I | I | I | R/A | I | C | C | R (calcule) |

## Notes de lecture

- **Propriétaire du corpus** est le seul rôle nommé introduit par ce document sans équivalent direct dans les acteurs génériques du DAv1-02 : il répond directement à Q-05 (« qui est le propriétaire nommé du corpus normatif »). Sans ce rôle assigné, la ligne « Promotion d'un runbook » n'a pas d'Accountable, et R-01 (les runbooks ne se formalisent pas) se matérialise mécaniquement.
- **Système (agents)** apparaît en colonne à dessein : plusieurs activités du pipeline sont *réalisées* par le système, mais **aucune ligne de ce tableau ne place le système en Accountable d'une décision à effet de bord** — l'Accountable reste toujours un rôle humain (Responsable de service, Propriétaire du corpus, RSSI). Seules les activités mécaniques et réversibles (blocage superviseur, descente automatique) ont le système en A/R, et ce sont précisément les deux seuls cas où le comportement est entièrement déterministe et sans marge d'interprétation.
- **Éditeur du socle** est consulté sur tout ce qui touche la frontière socle/pack (R-08), jamais Accountable d'une activité opérationnelle du client.
