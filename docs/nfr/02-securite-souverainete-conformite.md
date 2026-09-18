# Exigences non fonctionnelles — Sécurité, Souveraineté, Conformité (NFR-04, 11, 13)

**Amont** : ENF-07, ENF-08, DAv1-08, Q-03, Q-06

---

## NFR-04 — Sécurité

| Id | Énoncé | Cible | Méthode de mesure | Conséquence si dépassée |
|---|---|---|---|---|
| NFR-04.1 | Résistance à l'injection d'instruction (ENF-07) | **0 échec toléré** | Cas d'injection du banc d'évaluation, rejoué à chaque changement | Blocage de la livraison (gating CI) |
| NFR-04.2 | Étanchéité inter-clients (ENF-08) | **0 fuite tolérée**, garantie par construction | Test d'étanchéité automatisé (tentative d'accès cross-pack) | Arrêt immédiat, incident de sécurité (R-06) |
| NFR-04.3 | Chiffrement en transit | TLS 1.2 minimum, 1.3 préféré, partout y compris interne | Scan de configuration | Blocage du déploiement |
| NFR-04.4 | Chiffrement au repos | AES-256 ou équivalent, sur trace d'audit, corpus normatif, secrets | Audit de configuration du stockage | Blocage du déploiement |
| NFR-04.5 | Rotation des secrets | 90 jours maximum, automatisée | Journal du coffre-fort | Alerte à 80 jours |
| NFR-04.6 | Détection de secret dans un document entrant | 100 % des formats de secrets connus (clés API, mots de passe, tokens) | Cas de test dédiés dans le banc | Défaut de revue si un format connu n'est pas couvert |
| NFR-04.7 | Moindre privilège sur l'adaptateur ITSM | Lecture large, écriture strictement limitée aux codes en liste blanche | Revue de configuration de l'adaptateur | Blocage du déploiement |
| NFR-04.8 | Délai de révocation d'un accès technicien | < 1 h après révocation côté IAM | Test de révocation | Alerte si dépassé |
| NFR-04.9 | Scan de vulnérabilités des dépendances (SCA) | 0 vulnérabilité critique ou haute non traitée | Pipeline CI | Blocage de la livraison |
| NFR-04.10 | Signature et provenance des artefacts | 100 % des images de production signées | Vérification au déploiement | Déploiement refusé si non signé |

---

## NFR-11 — Souveraineté

**Statut : partiellement bloquée par Q-03.** Les cibles ci-dessous sont posées comme mécanisme ; les valeurs dépendantes de Q-03 sont marquées.

| Id | Énoncé | Cible | Méthode de mesure | Statut |
|---|---|---|---|---|
| NFR-11.1 | Localisation du traitement | Région déclarée par le pack client, vérifiée au démarrage | Contrôle de démarrage, refus si non conforme | Mécanisme prêt |
| NFR-11.2 | Localisation de l'index et de la trace | Idem NFR-11.1 | Idem | Mécanisme prêt |
| NFR-11.3 | Classes de modèles autorisées (hébergement, fournisseur) | Liste déclarée par le pack client | Contrôle de démarrage | **Bloqué par Q-03** — sans réponse, aucune classe n'est autorisée par défaut (fail-closed) |
| NFR-11.4 | Sortie réseau | Liste blanche stricte de destinations | Test de pénétration réseau | Mécanisme prêt |
| NFR-11.5 | Sous-traitance des modèles | Documentée par fournisseur, dans le registre de traitement | Registre de traitement | **Bloqué par Q-03/Q-06** |

**Conséquence architecturale du fail-closed appliqué à la souveraineté** : en l'absence de réponse à Q-03, le pack client ne peut déclarer aucune classe de modèle autorisée, et le système ne peut donc traiter aucun ticket au-delà de la normalisation. C'est un choix délibéré : mieux vaut un système bloqué à l'installation qu'un système qui suppose une autorisation non confirmée.

---

## NFR-13 — Conformité

| Id | Cadre | Exigence | Traduction | Statut |
|---|---|---|---|---|
| NFR-13.1 | RGPD | Anonymisation avant tout traitement | Contrôlée en amont de chaque appel de modèle et de tout stockage | Mécanisme prêt |
| NFR-13.2 | RGPD | Durée de rétention et base légale | Portée par le pack client | **Bloqué par Q-06** |
| NFR-13.3 | RGPD | Droit d'accès et d'effacement | Procédure documentée, applicable sur ticket canonique et trace anonymisée | À spécifier en S-01 |
| NFR-13.4 | AI Act | Qualification du système | Documentée en ADR-016, à confirmer selon la classification finale du système (risque limité présumé, à valider juridiquement) | À confirmer hors du périmètre technique |
| NFR-13.5 | AI Act | Transparence vis-à-vis du demandeur | Mention IA structurelle, garde-fou absolu 3 | Mécanisme prêt |
| NFR-13.6 | AI Act | Supervision humaine documentée | Le superviseur produit lui-même sa documentation de fonctionnement (verdicts, motifs, trace) | Mécanisme prêt |
| NFR-13.7 | Accessibilité | RGAA niveau AA sur le frontend Angular | Audit d'accessibilité avant mise en production | À vérifier en phase 5 |
| NFR-13.8 | Contractuel | Responsabilité en cas d'action erronée | Réversibilité, trace rejouable, niveau d'autonomie tracé | Mécanisme prêt, clause contractuelle hors périmètre technique |
