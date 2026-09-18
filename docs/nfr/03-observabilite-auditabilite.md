# Exigences non fonctionnelles — Observabilité, Auditabilité (NFR-05 à 06)

**Amont** : ENF-01, ENF-02, DAv1-13, Q-06

---

## NFR-05 — Observabilité

| Id | Énoncé | Cible | Méthode de mesure | Seuil d'alerte |
|---|---|---|---|---|
| NFR-05.1 | Corrélation d'une exécution de bout en bout | 100 % des étapes reliées par un identifiant de corrélation unique | Vérification automatique en test d'intégration | Toute rupture de corrélation est un défaut bloquant |
| NFR-05.2 | Latence de disponibilité des métriques | < 1 min entre l'événement et sa visibilité dans le dashboard | Mesure du pipeline d'observabilité | > 5 min |
| NFR-05.3 | Couverture des dashboards | 4 dashboards métier + 1 IA + 1 exploitation, tous les KPI du catalogue affichés | Revue de conformité au catalogue KPI | KPI manquant à un dashboard |
| NFR-05.4 | Rétention des métriques agrégées | 24 mois minimum | Configuration du système de métriques | — |
| NFR-05.5 | Rétention des traces techniques (hors trace d'audit) | 90 jours | Configuration | — |

## NFR-06 — Auditabilité

| Id | Énoncé | Cible | Méthode de mesure | Seuil d'alerte |
|---|---|---|---|---|
| NFR-06.1 | Traçabilité de toute décision (ENF-01) | 100 % | Contrôle 6 du superviseur (garde-fou absolu) | Toute décision sans trace : incident bloquant |
| NFR-06.2 | Déterminisme du routage (ENF-02) | 100 %, même entrée → même chemin | Test de rejeu sur le banc d'évaluation | Tout écart : incident de traçabilité |
| NFR-06.3 | Rejeu à l'identique (EF-09) | 100 % des cas du jeu d'évaluation | `POST /audit/traces/{id}/rejouer`, diff vide | Tout diff non vide : blocage du lot en cours |
| NFR-06.4 | Rétention de la trace d'audit | **Portée par le pack client** | Configuration | **Bloqué par Q-06** — valeur par défaut proposée : 3 ans, à confirmer |
| NFR-06.5 | Intégrité de la trace | Vérifiable par chaînage (append-only, empreinte) | Vérification périodique d'intégrité | Toute rupture de chaîne : incident de sécurité (M-08) |
| NFR-06.6 | Complétude du contenu de la trace | 100 % des champs obligatoires de DAv1-13.2 présents | Validation de schéma à l'écriture | Écriture refusée si champ manquant |
| NFR-06.7 | Accès en lecture à la trace | Rôle Auditeur, lecture seule, sans accès au contenu opérationnel courant | Test de contrôle d'accès | Tout accès en écriture par ce rôle : incident |

**Proposition sur NFR-06.4** : en l'absence de réponse à Q-06, retenir une valeur par défaut de 3 ans (alignée sur une pratique courante de rétention d'audit IT), configurable à la baisse ou à la hausse par pack client dès que la base légale est confirmée. Cette valeur n'engage rien juridiquement — c'est un paramètre technique de repli, pas une position légale.
