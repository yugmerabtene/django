
## Définition simple

| Terme             | Définition                                                                         |
| ----------------- | ---------------------------------------------------------------------------------- |
| **Orchestration** | Un **composant central** (orchestrateur) pilote et coordonne les services.         |
| **Chorégraphie**  | Les services **communiquent de manière autonome**, en réagissant à des événements. |

---

## Analogie musicale

| Style             | Analogie                                                                                 |
| ----------------- | ---------------------------------------------------------------------------------------- |
| **Orchestration** | Comme un **chef d’orchestre** qui dit à chaque musicien quand et quoi jouer.             |
| **Chorégraphie**  | Comme une **danse** : chaque danseur connaît son rôle et se synchronise avec les autres. |

---

## Exemple concret

### Scénario : Traitement d'une commande

---

### A. **Orchestration** (pilotée)

```
          [ Service Orchestrateur ]
                    |
       +------------+-------------+
       |            |             |
 [Service Paiement] [Service Stock] [Service Livraison]
```

* L’orchestrateur appelle **Paiement**, puis **Stock**, puis **Livraison**.
* Il gère le **workflow, les erreurs, les séquences**.

Avantage :

* Centralisation logique, facile à suivre
  Inconvénient :
* Fort couplage, point de défaillance unique

---

### B. **Chorégraphie** (événementielle)

```
[ Service Commande ]
        |
        | → Émet un événement Kafka : "CommandeCréée"
        ↓
     [ Kafka Topic ]
        |
  +-----+----------------------------+
  |     |                            |
[Service Paiement]          [Service Stock]
     |                             |
     → Événement "PaiementOk"     → Événement "StockRéservé"
                                     ↓
                               [Service Livraison]
```

* Chaque service **s’abonne** à des événements et réagit.
* Le flux se construit **sans pilote central**.

Avantage :

* Faible couplage, scalabilité
  Inconvénient :
* Difficile à tracer, debugging plus complexe

---

## Comparaison rapide

| Critère               | Orchestration                          | Chorégraphie                             |
| --------------------- | -------------------------------------- | ---------------------------------------- |
| Contrôle              | Centralisé                             | Distribué                                |
| Coordination          | Pilotée (souvent via un orchestrateur) | Basée sur des événements                 |
| Visibilité            | Bonne (centralisation des flux)        | Complexe (écoute d'événements)           |
| Résilience            | Moins bonne (point central)            | Meilleure (pas de point unique de panne) |
| Complexité de gestion | Moins complexe                         | Plus complexe à déboguer                 |
| Flexibilité           | Moindre                                | Très élevée                              |
| Outils courants       | Camunda, Temporal, Apache Airflow      | Kafka, RabbitMQ, EventBridge             |

---

## Quand utiliser quoi ?

| Cas d’usage                               | Recommandation |
| ----------------------------------------- | -------------- |
| Flux métier strict, séquentiel            | Orchestration  |
| Système orienté événements (event-driven) | Chorégraphie   |
| Petits workflows simples                  | Chorégraphie   |
| Processus métier complexes à tracer       | Orchestration  |

---

## Outils associés

| Style             | Outils recommandés                                      |
| ----------------- | ------------------------------------------------------- |
| **Orchestration** | Camunda, Temporal.io, Apache Airflow, Netflix Conductor |
| **Chorégraphie**  | Kafka, RabbitMQ, AWS EventBridge, NATS                  |

---
