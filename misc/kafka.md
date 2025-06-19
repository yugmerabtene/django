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
