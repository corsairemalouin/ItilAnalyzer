# Exigences non fonctionnelles — Qualité IA (NFR-12)

**Amont** : ENF-05, ENF-06, ENF-07, DAv1-13.5, DAv0 §Pilotage

| Id | Énoncé | Cible | Méthode de mesure | Seuil d'alerte | Conséquence |
|---|---|---|---|---|---|
| NFR-12.1 | Accord avec l'annotation humaine (ENF-05) | ≥ 85 % sur `categorie` et `niveau_cible` | Matrice de confusion sur le banc d'évaluation | < 85 % au lancement d'un lot | Blocage du passage en pilote |
| NFR-12.2 | Calibration (ENF-06) | Écart < 10 points entre confiance annoncée et justesse observée | Courbe de calibration | Écart ≥ 10 points | Blocage de toute montée de curseur sur la catégorie concernée |
| NFR-12.3 | Cas d'injection refusé (ENF-07) | 100 % | Cas dédié du banc, rejoué à chaque changement | Tout échec | Blocage immédiat de la livraison (gating CI) |
| NFR-12.4 | Taux d'hallucination détectée | < 2 % des sorties avec source | Détection automatique : affirmation non rattachable à une source citée | > 5 % | Analyse prioritaire, gel de la classe de modèle concernée |
| NFR-12.5 | Taux de blocage superviseur | Suivi, pas de cible fixe | Dashboard exploitation | > 25 % | Analyse des motifs (DAv1-11.4) |
| NFR-12.6 | Taux de mal-dirigé résiduel | Amélioration mesurable vs baseline historique | Comparaison trimestrielle | Dégradation vs baseline | Revue de la taxonomie et du pack projet |
| NFR-12.7 | Composition du jeu d'évaluation | Minimum 20 cas : 8 pauvres/ambigus, 6 bien rédigés (1/catégorie), 3 cas limites, 2 doublons, 1 injection | Revue du jeu avant chaque lot | Jeu non conforme à cette composition | Jeu refusé, pas d'évaluation valide possible |
| NFR-12.8 | Taille minimale par catégorie dans le jeu d'évaluation | Aucune catégorie < 3 % du volume total évalué (R-13) | Revue de composition | Catégorie sous le seuil | La catégorie est exclue du calcul de montée de curseur individuel, agrégée avec une catégorie voisine |
| NFR-12.9 | Fraîcheur du jeu d'évaluation | Révisé à chaque changement de modèle, de prompt ou de pack majeur | Traçabilité des révisions | Jeu non révisé après un changement majeur | Descente automatique du curseur (BR-23) |
| NFR-12.10 | Taux d'acceptation des recommandations par les techniciens | Suivi, cible indicative ≥ 60 % en fin de pilote | Journal des décisions | < 50 % (signal R-04) | Revue du rejet par les techniciens, pas d'assouplissement des contrôles |
| NFR-12.11 | Taux de correction humaine | Suivi | Journal des décisions | Tendance à la hausse sur 4 semaines | Analyse de dérive de qualité |

**Principe transverse à ce chapitre** : aucune de ces cibles n'est atteinte en assouplissant un contrôle du superviseur ou un garde-fou absolu. Un écart se corrige par la qualité du pack, du corpus ou du modèle — jamais par le relâchement d'une règle de sécurité ou d'auditabilité. C'est la même logique que « le contexte manquant dit quel pack enrichir, pas quel prompt retoucher » (DAv1-13.5).
