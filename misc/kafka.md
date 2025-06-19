## 1. Qu’est-ce qu’Apache Kafka ?

Apache Kafka est une plateforme de **streaming distribuée** conçue pour :

* Transmettre de gros volumes de données en temps réel
* Découpler les producteurs (ceux qui émettent les données) des consommateurs (ceux qui les lisent)
* Offrir une haute tolérance aux pannes
* Assurer la persistance et la scalabilité horizontale

Kafka fonctionne comme une **file de messages persistante**, rapide, partitionnée et répliquée.

---

## 2. Architecture globale de Kafka

### Composants principaux :

* **Producer** : application qui envoie des messages à Kafka
* **Consumer** : application qui lit les messages depuis Kafka
* **Broker** : serveur Kafka qui stocke et distribue les messages
* **Topic** : canal nommé dans lequel les producteurs écrivent et les consommateurs lisent
* **Partition** : découpage d’un topic pour paralléliser la lecture et l’écriture
* **Zookeeper** (optionnel depuis Kafka 2.8) : utilisé auparavant pour la coordination des brokers
* **Kafka Controller** : dans le mode KRaft (Kafka sans Zookeeper), il gère les brokers, partitions et élections de leader

---

## 3. Fonctionnement détaillé

### 3.1. Création d’un topic

Un topic est une abstraction logique représentant un flux de messages (exemples : "commande", "paiement", "email").

Un topic peut être découpé en plusieurs partitions, ce qui permet de paralléliser le traitement.

### 3.2. Envoi d’un message par un producer

Le producer envoie un message à un topic Kafka.

* S’il fournit une clé, Kafka applique un hachage sur la clé pour déterminer la partition cible.
* S’il n’y a pas de clé, Kafka répartit les messages en round-robin entre les partitions.

Chaque message reçoit un identifiant unique appelé **offset** dans la partition.

### 3.3. Stockage dans le broker Kafka

Les brokers Kafka sont responsables du stockage physique des partitions sur disque. Kafka utilise un système de log append-only.

Les messages sont conservés pendant une durée configurable (par défaut 7 jours), même s’ils ont été lus.

### 3.4. Lecture par un consumer

Un consumer lit les messages dans une ou plusieurs partitions d’un topic.

Chaque consumer peut gérer manuellement ou automatiquement son offset (position de lecture), ce qui permet de relire d’anciens messages si nécessaire.

### 3.5. Groupes de consumers

Les consumers peuvent être regroupés en **consumer groups**.

* Un consumer group permet de paralléliser la consommation des messages : chaque partition est consommée par un seul consumer dans le groupe.
* Plusieurs groupes peuvent consommer le même topic indépendamment (les messages sont donc "broadcastés" par topic, mais "load balancés" par partition à l’intérieur d’un groupe).

### 3.6. Réplication et tolérance aux pannes

Chaque partition peut être répliquée sur plusieurs brokers.

Un leader est élu pour chaque partition. Les autres brokers conservent des copies appelées réplicas.

En cas de panne du broker leader, un réplica est promu leader automatiquement.

---

## 4. Exemple de flux complet

1. Le service Paiement envoie un événement `payment_success` au topic `paiement`
2. Kafka stocke ce message dans une des partitions du topic `paiement`
3. Le service Facturation, qui est un consumer, lit depuis cette partition
4. Si le service Facturation échoue, Kafka garde le message disponible pour qu’un autre consumer puisse le traiter
5. Le consumer peut décider quand il "commit" l’offset pour signaler que le message a été traité

---

## 5. Résumé des composants par couche

* Client producteur : envoie des messages Kafka
* Kafka Broker : stocke et répartit les messages
* Partition : segment du topic, gérée par un broker
* Consumer Group : ensemble de consommateurs pour paralléliser la lecture
* Offset : position du message dans une partition
* Controller (ou Zookeeper) : gère les métadonnées et l’équilibrage

---

## 6. Avantages clés de Kafka

* Très haute performance (des millions de messages/seconde)
* Faible latence (temps réel ou proche)
* Persistant (les messages sont conservés plusieurs jours)
* Scalable horizontalement (on peut ajouter des brokers facilement)
* Séparé en lecture/écriture (les consumers ne bloquent pas les producers)
* Permet la relecture des messages (gestion manuelle ou automatique des offsets)

---

## 7. Cas d’usage classiques

* Architecture microservices (communication asynchrone entre services)
* Journalisation centralisée des logs
* Pipelines de données (ETL, ingestion temps réel)
* Systèmes de recommandation ou de détection de fraudes
* Communication entre systèmes hétérogènes

  ----



# 1. Définition de Kafka

Kafka est un système de messagerie distribué de type **publish/subscribe** conçu pour des applications à très fort débit, nécessitant :

* la **persistance des messages**
* la **tolérance aux pannes**
* la **scalabilité horizontale**
* le **rejeu possible** des événements (reprocessing, débogage)

Contrairement à d'autres systèmes comme RabbitMQ, Kafka ne supprime pas les messages dès qu’ils sont lus. Il stocke un **log immuable** réparti sur plusieurs partitions.

---

# 2. Architecture logique de Kafka

Kafka repose sur les éléments suivants :

**Producer** : envoie des messages à un topic Kafka.

**Consumer** : lit les messages depuis Kafka.

**Broker** : serveur Kafka qui héberge des partitions.

**Topic** : canal nommé auquel les producers écrivent et dont les consumers lisent.

**Partition** : division physique d’un topic pour permettre la parallélisation.

**Offset** : numéro croissant attribué à chaque message dans une partition.

**Consumer Group** : ensemble de consumers travaillant ensemble pour consommer un topic sans doublon.

**Zookeeper** ou **KRaft** : système de gestion des métadonnées (le mode KRaft remplace Zookeeper depuis Kafka 2.8+).

---

# 3. Écriture d’un message par un Producer

## Étapes détaillées :

1. Le producer crée un **Kafka Record** contenant :

   * un `topic`
   * une `key` (optionnelle)
   * une `value`
   * éventuellement des `headers`

2. Kafka choisit la **partition cible** :

   * Si une `key` est fournie : Kafka calcule un hash pour déterminer la partition
   * Si aucune `key` : Kafka applique une stratégie de round-robin

3. Kafka bufferise localement les messages dans une mémoire temporaire (RecordAccumulator)

4. Le batch est compressé (Snappy, Gzip ou LZ4) et sérialisé (JSON, Avro, Protobuf)

5. Kafka envoie le batch à un broker, selon la configuration de fiabilité (`acks`)

   * `acks=0` : aucun accusé de réception attendu
   * `acks=1` : le leader de la partition confirme l’écriture
   * `acks=all` : tous les réplicas doivent confirmer

---

# 4. Traitement d’un message par le Broker

Une fois reçu, le broker exécute les étapes suivantes :

1. Stocke le message dans un **fichier de log append-only**

   * Chaque partition correspond à un fichier sur disque
   * Le message est ajouté à la fin du fichier, avec un offset unique

2. Met à jour les fichiers d’index (.index, .timeindex)

3. Si la partition est répliquée :

   * Le leader envoie le message aux **ISR** (In-Sync Replicas)
   * Les ISR confirment l’écriture
   * Kafka attend les ACKs selon la config (`acks=all`)

4. Le broker retourne un ACK au Producer, incluant l’offset du message

---

# 5. Lecture d’un message par un Consumer

## Étapes détaillées :

1. Le consumer est assigné à une ou plusieurs **partitions**

2. Il lit séquentiellement les messages en appelant `poll()`

   * Peut lire par lots (`max.poll.records`)
   * Peut gérer un `timeout`

3. Il désérialise et traite les messages

4. Il effectue un **commit** :

   * Automatique : Kafka enregistre l’offset à intervalle régulier
   * Manuel : l’application contrôle précisément quand un offset est validé

5. Le consumer peut relire un message depuis un offset spécifique :

   * Pour reprocessing
   * Pour corriger une erreur
   * Pour reconstruire un état

Kafka ne supprime pas les messages une fois lus. Ils restent disponibles jusqu’à expiration (`retention.ms`) ou dépassement de taille (`retention.bytes`).

---

# 6. Groupes de Consumers et parallélisme

Un **Consumer Group** est un ensemble de consumers qui coopèrent pour consommer un topic.

Règles de fonctionnement :

* Une **partition donnée** est consommée par **au plus un consumer** dans le groupe
* Si un groupe a plus de consumers que de partitions, certains consumers restent inactifs
* Plusieurs groupes peuvent consommer indépendamment le même topic

Cela permet :

* La **scalabilité** horizontale en lecture
* Le **rééquilibrage automatique** quand un consumer meurt ou arrive (`rebalance`)

---

# 7. Réplication, disponibilité et tolérance aux pannes

Kafka assure la haute disponibilité via :

* **Réplication de chaque partition**

  * Un leader (élu automatiquement) écrit les données
  * Les followers (ISR) répliquent les données

* **Failover automatique**

  * Si un leader tombe, un ISR prend le relais
  * Le controller Kafka orchestre ce changement

* **Zookeeper** (ancien) ou **KRaft mode** (nouveau)

  * Gère la configuration des brokers
  * Gère l’élection du controller

---

# 8. Fichiers internes de Kafka

Chaque partition est stockée sous forme de plusieurs fichiers sur disque :

* `.log` : les messages (append-only)
* `.index` : offset → position byte dans le log
* `.timeindex` : offset → timestamp

Quand un fichier atteint `segment.bytes` (ex : 1Go), un **nouveau segment** est créé.

Les segments plus anciens sont supprimés selon :

* `log.retention.ms` : durée maximale
* `log.retention.bytes` : taille totale maximale

---

# 9. Fonctionnement dans une architecture microservices

Kafka est souvent utilisé comme **message broker central** dans une architecture microservices :

1. Microservice `Commande` émet des événements dans le topic `commande`
2. Kafka les stocke et les ordonne par partition
3. Microservices `Paiement`, `Stock`, `Facturation` sont des consumers indépendants
4. Chacun peut consommer à son rythme, ou rejouer l’historique en cas d’erreur
5. Kafka assure la **résilience** si un service tombe, aucun message n’est perdu

---

# 10. Avantages techniques

* **Débit élevé** : Kafka peut traiter des millions de messages par seconde
* **Persistant** : les messages sont stockés sur disque
* **Rejouabilité** : les consumers contrôlent leur offset
* **Indépendance** entre producteurs et consommateurs
* **Partitionnement** : permet la scalabilité horizontale
* **Tolérance aux pannes** : via réplication et failover automatique

---
