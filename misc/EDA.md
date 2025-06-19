**Définition :**
L’EDA est un modèle dans lequel les composants réagissent à des événements émis par d’autres composants, généralement de manière asynchrone.

---

**Comment cela s’intègre dans les microservices :**
Chaque microservice est indépendant et communique indirectement via un message broker comme Kafka, RabbitMQ ou NATS.
Lorsqu’un événement se produit (par exemple : `commande_passée`, `utilisateur_inscrit`, `paiement_validé`), un microservice publie cet événement sur un **topic**.
D’autres microservices s’abonnent à ce topic et réagissent à cet événement.

---

**Architecture typique (EDA avec message broker) :**

```
Service A (Producer) ---> Message Broker (Kafka, RabbitMQ) ---> Service B (Consumer)
                                                  |
                                                  +--> Service C (Consumer)
```

---

**Avantages de l’EDA dans les microservices :**

* Faible couplage entre les services.
* Scalabilité améliorée.
* Résilience accrue (le broker peut stocker temporairement les messages).
* Auditabilité (possibilité de rejouer les événements).

---

**Exemple concret :**

1. `OrderService` publie un événement `OrderPlaced`.
2. `BillingService` s’abonne à cet événement pour effectuer le paiement.
3. `InventoryService` le consomme également pour décrémenter le stock.
4. `NotificationService` envoie un email de confirmation.

---

**Précautions à prendre :**

* L’EDA n’est pas toujours adaptée : pour des appels critiques ou instantanés, une API REST peut rester plus appropriée.
* Il faut gérer l’idempotence (éviter les traitements en double), les erreurs (système de retry), et éventuellement utiliser des files d’attente secondaires (dead-letter queues).

---

**Conclusion :**
L’utilisation d’un message broker dans une architecture microservices est très souvent associée à une Event-Driven Architecture. C’est un style d’architecture bien adapté aux systèmes distribués modernes.
