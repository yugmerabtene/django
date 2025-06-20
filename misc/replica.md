## Qu’est-ce qu’un réplica dans Kafka ?

Un **réplica** est une **copie exacte d’une partition**, stockée sur un **broker différent** de celui qui détient le **leader** de cette partition.

Chaque partition Kafka peut avoir :

* Un **leader** : la seule instance qui accepte les lectures et écritures
* Un ou plusieurs **followers** : ce sont les **réplicas** passifs du leader

---

## Objectifs des réplicas (utilité)

### 1. Haute disponibilité

Si le broker qui contient une partition leader tombe, Kafka élit un **nouveau leader** parmi les réplicas à jour (In-Sync Replica, abrégé ISR).

Cela permet au cluster de rester fonctionnel même en cas de panne.

---

### 2. Tolérance aux pannes

Si un broker ou son disque tombe, les données sont toujours disponibles sur les autres brokers.

Aucune perte de données, même en cas de crash.

---

### 3. Continuité de service

Les producteurs et les consommateurs ne sont pas bloqués : Kafka redirige automatiquement vers un autre broker qui devient leader.

Kafka est donc très résilient.

---

### 4. Sécurité des données

Kafka peut être configuré pour que les messages ne soient considérés comme "confirmés" qu’après avoir été écrits sur plusieurs réplicas.

Cela réduit les risques de perte de données avant l'accusé de réception.

---

## Fonctionnement technique des réplicas

### 1. Liste des ISR (In-Sync Replica)

Kafka maintient une liste appelée **In-Sync Replica** (ISR) qui contient tous les réplicas synchronisés avec le leader.

Le leader envoie les messages aux followers, qui accusent réception.

Si un follower prend trop de retard ou tombe, Kafka le retire de la liste ISR.

---

### 2. Réplication asynchrone

La réplication est **asynchrone**, ce qui signifie qu’elle n’est pas bloquante pour les producteurs.

Mais elle est très rapide : généralement, le décalage entre le leader et les followers est inférieur à 10 millisecondes.

---

### 3. Élection de nouveau leader

Lorsqu’un broker leader tombe, Kafka :

* Consulte la liste des réplicas à jour (ISR)
* Élit un follower à jour comme nouveau leader
* Redirige automatiquement les producteurs et consommateurs vers le nouveau leader

---

### 4. Fiabilité configurable avec acks (Acknowledgments)

Le producteur Kafka peut configurer le niveau de fiabilité de livraison via un paramètre nommé `acks` (pour "acknowledgments").

| acks | Description                                                                      |
| ---- | -------------------------------------------------------------------------------- |
| 0    | Le producteur n’attend aucune confirmation (rapide, mais risqué)                 |
| 1    | Le producteur attend que le **leader** ait reçu le message                       |
| all  | Le producteur attend que **tous les réplicas dans l’ISR** aient écrit le message |

La configuration `acks=all` combinée à `min.insync.replicas=2` garantit une très bonne durabilité et sécurité.

---

## Exemple concret

Imaginons un topic `orders` avec :

* 3 partitions
* Un facteur de réplication de 3

Partition 0 est :

* Leader sur Broker 1
* Répliquée sur Broker 2 et Broker 3

Si Broker 1 tombe, Kafka élit automatiquement Broker 2 comme nouveau leader.

La production et la consommation de messages continuent sans interruption.

Quand Broker 1 revient, il est resynchronisé avec les autres brokers et réintègre la liste des ISR.

---

## Résumé

| Élément                | Rôle                                                         |
| ---------------------- | ------------------------------------------------------------ |
| Leader                 | Partition active qui accepte les lectures et écritures       |
| Follower (réplica)     | Copie passive utilisée pour la tolérance aux pannes          |
| ISR (In-Sync Replica)  | Réplicas à jour éligibles pour devenir leader                |
| acks (Acknowledgments) | Niveau de confirmation de livraison choisi par le producteur |

