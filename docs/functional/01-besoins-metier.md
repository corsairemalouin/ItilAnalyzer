# F-01 — Besoins métier

**Amont** : DAv0 §Besoin métier, DAv1-01, DAv1-02 · **Aval** : F-02 (use cases)

Format : `BM-xx` — énoncé, valeur, acteur porteur, mesure de succès, priorité MoSCoW.

| Id | Énoncé | Acteur | Mesure de succès | Priorité |
|---|---|---|---|---|
| BM-01 | Réduire le temps consacré à la qualification d'un ticket | Technicien N1 | Temps moyen de qualification, avant/après | Must |
| BM-02 | Réduire le temps de recherche documentaire | Technicien N1/N2 | Temps moyen de traitement | Must |
| BM-03 | Homogénéiser les réponses apportées à des cas similaires | Responsable de service | Taux de similarité inter-réponses sur cas identiques | Should |
| BM-04 | Capitaliser la connaissance sans dépendre d'un individu | Responsable de service | Nombre de runbooks normatifs actifs | Must |
| BM-05 | Continuer à assister les équipes en cas d'absence d'un expert | Technicien N2 | Taux de résolution N2 sans expert disponible | Should |
| BM-06 | Disposer de recommandations explicables et sourcées | Technicien, Auditeur | Taux de sorties avec sources complètes | Must |
| BM-07 | Affecter le bon dossier à la bonne équipe dès l'ouverture | Responsable de service | Taux de mal-dirigé résiduel | Must |
| BM-08 | Transmettre un dossier structuré et complet à l'expert N3 | Expert N3 | Taux de dossiers complets à l'ouverture N3 | Must |
| BM-09 | Garder la décision et la responsabilité côté humain | Responsable de service, RSSI | Taux d'actions exécutées sans validation en dessous du seuil contractuel | Must |
| BM-10 | Pouvoir expliquer une décision passée en audit | RSSI, Auditeur | Taux de décisions rejouées à l'identique | Must |
| BM-11 | Maîtriser le coût d'utilisation de l'IA | Responsable de service | Coût moyen par ticket vs budget | Must |
| BM-12 | Réutiliser le socle sur un autre compte sans réécriture | Éditeur du socle | Part de code non réécrite au 2ᵉ projet | Should |
| BM-13 | Garantir l'étanchéité des données entre clients | RSSI | Résultat du test d'étanchéité inter-packs | Must |
| BM-14 | Informer systématiquement le demandeur qu'une IA est intervenue | Demandeur, RSSI | Taux de communications sans mention IA | Must (0 toléré) |
| BM-15 | Augmenter progressivement la délégation à l'IA, sur preuve | Responsable de service | Courbe de montée du curseur d'autonomie par catégorie | Should |
| BM-16 | Détecter les incidents récurrents avant qu'ils ne se reproduisent | Responsable de service | Nombre de signaux de récurrence remontés | Could |
| BM-17 | Conserver la possibilité de changer de fournisseur (modèle, ITSM, index) | RSSI, Éditeur du socle | Résultat du test de réversibilité par axe | Should |

## Positionnement

L'IA est un **copilote** du processus de support : elle propose, justifie, prépare ; l'organisation décide du niveau de délégation autorisé. Aucun besoin métier de ce catalogue ne doit être satisfait par un mécanisme qui retire à l'humain la capacité de reprendre le contrôle — c'est un invariant vérifié en revue de chaque UC dérivé.
