# Exigences non fonctionnelles — Disponibilité, Performance, Scalabilité (NFR-01 à 03)

**Amont** : ENF-03, ENF-04, ENF-10, `PLAN-00-A1` §2, Q-10 (close), DAv1-06.3, DAv1-06.4
**Format** : chaque exigence porte identifiant, énoncé, valeur cible chiffrée, méthode de mesure, seuil d'alerte, conséquence en cas de dépassement. Une exigence sans valeur chiffrée est refusée à GATE-3.

---

## NFR-01 — Disponibilité

### Décision de fenêtre de service (ferme Q-10)

**Fenêtre de service humaine** : heures ouvrées, par défaut lundi-vendredi 8h-19h (paramètre du pack client, fuseau du pack client). Techniciens, console, validation humaine disponibles uniquement sur cette fenêtre.

**Décision architecturale associée, non explicite dans le DAv0** : que fait le pipeline hors fenêtre ?

| Fonction | Hors fenêtre de service | Justification |
|---|---|---|
| Ingestion, normalisation (Intake) | **Continue 24/7** | Sans effet de bord, aucune décision n'est prise, le ticket est simplement prêt au réveil de l'équipe |
| Triage, recherche RAG | **Continue 24/7** | Produit une proposition, ne l'exécute pas |
| Résolution N1 — niveaux 0 à 2 | **Continue 24/7** | Aucune exécution sans humain, pas de risque en l'absence de supervision humaine |
| Résolution N1 — niveaux 3 et 4 (exécution automatique) | **Suspendue hors fenêtre par défaut** | Personne n'est en mesure de réagir à une anomalie en production sans supervision humaine à proximité ; le risque d'un incident non détecté pendant la nuit dépasse le gain de réactivité |
| Investigation N2, préparation N3 | **Continue 24/7** | Lecture seule ou préparation, aucun effet de bord |

Le plafond de niveau 3-4 hors fenêtre est un paramètre du pack client (`fenetre_execution_automatique`), avec valeur par défaut égale à la fenêtre de service humaine. Un client pouvant justifier une astreinte réelle peut l'étendre — ce n'est pas une limite du socle, c'est une valeur par défaut prudente.

**Conséquence pour le superviseur** : un 8ᵉ cas de dégradation s'ajoute aux modes dégradés de DAv1-06.4 — *hors fenêtre d'exécution automatique* → verdict `VALIDATION_HUMAINE` forcé même si le niveau d'autonomie configuré autoriserait l'exécution directe. Ce cas est vérifié comme un contrôle contextuel du superviseur, au même titre que le niveau d'autonomie lui-même (contrôle 5).

### Cibles chiffrées

| Id | Énoncé | Cible | Méthode de mesure | Seuil d'alerte | Conséquence |
|---|---|---|---|---|---|
| NFR-01.1 | Disponibilité du service pendant la fenêtre de service | **99,5 %** mensuel | Uptime des composants critiques (API, Superviseur, passerelle modèles, index) pendant la fenêtre | < 99,5 % sur le mois glissant | Revue d'incident obligatoire |
| NFR-01.2 | Disponibilité hors fenêtre de service | **95 %** mensuel (best effort) | Idem, fenêtre élargie | < 90 % | Alerte, pas de revue formelle |
| NFR-01.3 | Fenêtre de maintenance planifiée | Hors fenêtre de service uniquement | Calendrier de déploiement | Déploiement pendant la fenêtre de service | Interdit sauf exception documentée |
| NFR-01.4 | RTO (temps de reprise) — composant applicatif | < 15 min | Test de bascule trimestriel | Dépassement en test | Révision du plan de reprise |
| NFR-01.5 | RTO — base de données transactionnelle | < 30 min | Test de restauration trimestriel | Dépassement en test | Révision du plan de sauvegarde |
| NFR-01.6 | RPO (perte de données maximale tolérée) | < 5 min sur la trace d'audit, < 1 h sur le reste | Fréquence de réplication/sauvegarde | — | La trace d'audit a un RPO plus strict que le reste : c'est le livrable contractuel (ENF-01) |

**Budget d'erreur** : 99,5 % mensuel sur la fenêtre de service laisse ~3h40 d'indisponibilité tolérée par mois. Ce budget est consommé en priorité par les fenêtres de déploiement contrôlées ; un dépassement du budget hors déploiement planifié déclenche un gel des changements jusqu'à la fin du mois.

---

## NFR-02 — Performance

Reprend et complète les budgets de latence de DAv1-06.3.

| Id | Énoncé | Cible | Méthode de mesure | Seuil d'alerte | Conséquence |
|---|---|---|---|---|---|
| NFR-02.1 | Latence bout en bout, chemin triage (ENF-03) | < 60 s, p95 | Trace horodatée par étape | > 50 s p95 sur 24h | Analyse de la répartition par étape |
| NFR-02.2 | Latence classe S | < 2 s, p95 | Trace par appel | > 1,8 s p95 | Vérification du fournisseur/région |
| NFR-02.3 | Latence classe M | < 10 s, p95 | Trace par appel | > 8 s p95 | Idem |
| NFR-02.4 | Latence classe L (asynchrone) | < 90 s, p95 | Trace par appel | > 75 s p95 | Idem, sans impact sur le chemin critique |
| NFR-02.5 | Temps de réponse de la console technicien (affichage d'une proposition) | < 1 s, p95 | Mesure applicative | > 2 s p95 | Optimisation requise |
| NFR-02.6 | Débit soutenu | 30 tickets/heure en pic, avec marge ×3 | Test de charge en préproduction | — | Dimensionnement revu si marge non tenue |
| NFR-02.7 | Coût moyen par ticket (ENF-04) | **Budget figé par le pack client**, valeur par défaut proposée : à chiffrer avec Q-13 | Somme des coûts d'appel / ticket traité | > 120 % du budget sur 7 jours glissants | Alerte, analyse de la part de classe L (R-09) |

**Note sur NFR-02.7** : sans réponse à Q-13, la valeur cible n'est pas chiffrée — seule la méthode de mesure et le mécanisme d'alerte le sont. C'est un chapitre partiellement en hypothèse, assumé comme tel.

---

## NFR-03 — Scalabilité

**Requalifiée par rapport au plan initial** : à 2 000 tickets/mois (< 0,01 req/s au pic, cf. `PLAN-00-A1` §2.1-2.2), le débit n'est pas l'axe dimensionnant. Cette section porte donc sur la capacité à absorber une croissance, pas sur un besoin actuel.

| Id | Énoncé | Cible | Méthode de mesure | Conséquence si dépassée |
|---|---|---|---|---|
| NFR-03.1 | Marge de croissance sans changement d'architecture | ×5 le volume actuel (10 000 tickets/mois) | Test de charge en préproduction avec volumétrie simulée | Réévaluation de la topologie (§DAv1-07.3) |
| NFR-03.2 | Nombre de clients servis sans passage au multi-tenant | 1 à 2 (mono-tenant, ADR-013) | Revue à chaque nouveau client | Déclenche la révision d'ADR-013 dès le 2ᵉ ou 3ᵉ client signé |
| NFR-03.3 | Volumétrie documentaire indexable sans changement de moteur | jusqu'à 10⁵ segments (hypothèse, à confirmer par Q-09) | Taille de l'index en production | Migration vers un moteur vectoriel dédié |
| NFR-03.4 | Élasticité horizontale des workers asynchrones (classe L, ingestion) | 2 à 6 répliques, déclenchement sur profondeur de file | Métrique de file d'attente | Ajustement du seuil de déclenchement |

**Ce que NFR-03 n'exige pas** : ni autoscaling agressif de l'API, ni index distribué, ni GPU dédié en v0 — écarté explicitement par le dimensionnement A1 §2.2, sauf branche « modèle auto-hébergé » conditionnée par Q-03.
