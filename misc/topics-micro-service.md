## Qu’est-ce qu’un **Topic** ?

Un **topic** est un **canal de communication** **nommé** dans lequel des producteurs publient des messages, et que des consommateurs écoutent pour recevoir ces messages.

On peut le comparer à :

* une **chaîne de télévision** : celui qui "publie" est le diffuseur, et tous les "abonnés" qui regardent sont les consommateurs.
* un **fil de discussion** sur un forum : chaque message est une publication, et ceux qui suivent ce fil reçoivent les infos.

---

## Comment ça marche concrètement ?

### Exemple dans un système Kafka :

1. Un **producteur** (producer) envoie un message dans le topic `commande_creee` :

   ```json
   {"order_id": 123, "user_id": 45, "total": 79.99}
   ```

2. Plusieurs **consommateurs** (consumers) peuvent être abonnés à ce topic :

   * `email_service` pour envoyer un mail de confirmation
   * `payment_service` pour lancer le paiement
   * `log_service` pour enregistrer la commande

Chaque service lit les messages et réagit **indépendamment**.

---

## Pourquoi utiliser des topics ?

### 1. **Découplage**

Le producteur **ne connaît pas** les consommateurs. Il envoie un message sur un topic, et c’est tout. Cela rend les services **indépendants** et plus faciles à maintenir.

### 2. **Scalabilité**

Plusieurs consommateurs peuvent lire en parallèle. Tu peux aussi avoir plusieurs instances d’un même service pour traiter plus vite les messages.

### 3. **Tolérance aux pannes**

Les messages peuvent être **persistés** dans Kafka pendant une durée configurable. Si un service tombe en panne, il peut reprendre la lecture plus tard.

---

## Détail technique d’un topic dans Kafka

* Un **topic** est divisé en **partitions**

  * Cela permet un traitement parallèle par plusieurs consommateurs
  * Chaque message dans une partition a un **offset** (un numéro unique)

* Exemple de topic `paiement_effectue` avec 3 partitions :

  * partition 0 → messages 0, 1, 2
  * partition 1 → messages 0, 1
  * partition 2 → messages 0, 1, 2, 3

Kafka garde ces messages pour un certain **temps configurable**, même s'ils sont déjà lus.

---

## Différence avec une file classique (queue)

| Fonctionnalité | Queue (ex: RabbitMQ)            | Topic (Kafka)                       |
| -------------- | ------------------------------- | ----------------------------------- |
| Mode           | Push                            | Pull                                |
| Destinataire   | Un seul consommateur            | Plusieurs consommateurs possibles   |
| Stockage       | Court terme                     | Long terme possible (par défaut 7j) |
| Performance    | Moins bonne pour gros volumes   | Très bon pour très haut débit       |
| Cas typique    | Traitement séquentiel de tâches | Event-driven, logs, data streaming  |

---

## Exemple d’usage dans un système e-commerce

| Topic               | Message publié                                                          | Services abonnés                 |
| ------------------- | ----------------------------------------------------------------------- | -------------------------------- |
| `order_created`     | {"order\_id": 101, "user\_id": 5}                                       | email\_service, payment\_service |
| `payment_completed` | {"order\_id": 101, "status": "success"}                                 | shipping\_service, analytics     |
| `user_registered`   | {"user\_id": 5, "email": "[user@example.com](mailto:user@example.com)"} | email\_service, crm\_service     |

---

## En résumé

* Un **topic** est un **canal de publication/abonnement**
* Les **producteurs** y publient des messages
* Les **consommateurs** s’y abonnent pour traiter ces messages
* Cela permet un **découplage fort**, une **scalabilité élevée** et une **architecture événementielle**

  ----



## 1. Pourquoi certains services ne doivent **pas** être abonnés à certains topics ?

### Raisons principales :

* **Principe de moindre privilège** : un service ne doit traiter que ce dont il a besoin.
* **Éviter le bruit** : recevoir des messages inutiles alourdit le traitement et augmente le risque d'erreur.
* **Séparation des responsabilités** : chaque service a une responsabilité unique (Single Responsibility Principle).
* **Sécurité des données** : certains topics contiennent des données sensibles (ex. infos bancaires).

---

## 2. Exemples Concrets

### Contexte : Application e-commerce avec les topics suivants

* `order_created`
* `payment_successful`
* `user_registered`
* `product_added`
* `delivery_dispatched`

### Cas corrects

| Service            | Doit s'abonner à                   | Raison                                    |
| ------------------ | ---------------------------------- | ----------------------------------------- |
| `email_service`    | `order_created`, `user_registered` | Envoyer confirmation et mail de bienvenue |
| `payment_service`  | `order_created`                    | Déclenche le paiement après une commande  |
| `shipping_service` | `payment_successful`               | Organise la livraison après paiement      |
| `crm_service`      | `user_registered`                  | Intègre l’utilisateur au CRM              |

### Cas incorrects

| Service            | Ne doit **pas** s'abonner à           | Pourquoi ?                                          |
| ------------------ | ------------------------------------- | --------------------------------------------------- |
| `email_service`    | `payment_successful`                  | Il n’a rien à faire avec les paiements              |
| `payment_service`  | `delivery_dispatched`                 | La livraison ne le concerne pas                     |
| `crm_service`      | `order_created`, `payment_successful` | Ce sont des événements métiers, pas CRM             |
| `shipping_service` | `user_registered`                     | La création de compte n’est pas liée à la livraison |

---

## 3. Cas de figure mal conçus

### a. Tous les services écoutent tous les topics

* Problème : grosse **perte de performance**, confusion dans la logique, sécurité affaiblie.
* Risque : un bug dans un service peut affecter des flux qui ne le concernent pas.

### b. Un service écoute un topic "au cas où"

* Mauvaise pratique. Cela conduit à du code spaghetti.
* Il faut au contraire avoir une architecture propre, où **chaque service sait pourquoi** il lit tel ou tel événement.

---

## 4. Meilleures pratiques

* **Documenter clairement** quels services publient et consomment chaque topic.
* **Limiter les droits** dans Kafka avec des ACLs (Kafka Access Control Lists).
* **Valider le schéma des messages** avec des outils comme Avro ou JSON Schema.
* **Utiliser des noms de topics explicites** pour éviter les erreurs (ex: `user.registered.v1`, `order.paid.v1`).

---

## 5. Schéma synthétique

Voici un exemple correct d'abonnement par topic :

| Topic                 | Producteur         | Consommateurs                      |
| --------------------- | ------------------ | ---------------------------------- |
| `user_registered`     | `user_service`     | `email_service`, `crm_service`     |
| `order_created`       | `order_service`    | `payment_service`, `email_service` |
| `payment_successful`  | `payment_service`  | `shipping_service`                 |
| `delivery_dispatched` | `shipping_service` | `notification_service`             |

---

