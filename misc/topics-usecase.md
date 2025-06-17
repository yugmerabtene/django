## Résumé du fonctionnement

1. **Un microservice A** (ex: `order_service`) **publie** un message dans un **topic Kafka** (`order.created`).
2. **Kafka stocke et gère** ce message dans le topic correspondant.
3. **Un microservice B** (ex: `payment_service`) **est abonné** à ce topic (`order.created`), il **récupère** l’événement.
4. `payment_service` **traite** cet événement, puis **publie un nouvel événement** (par exemple `payment.successful`) dans **un autre topic Kafka**.
5. D’autres services (ex: `shipping_service`, `email_service`) **écoutent** ce nouveau topic, réagissent, et ainsi de suite...

---

## Schéma complet du cycle

```
Étape 1 : Création commande
[Client] --> [order_service]
                  |
                  |--> Kafka.publish("order.created", {order_id: 12})
                         |
                         V
             [Kafka] Topic: order.created

Étape 2 : Paiement
[payment_service] -- Kafka.subscribe("order.created")
        |
        |--> traite commande (ex: Stripe)
        |
        |--> Kafka.publish("payment.successful", {order_id: 12})

Étape 3 : Livraison
[shipping_service] -- Kafka.subscribe("payment.successful")
        |
        |--> organise livraison
        |
        |--> Kafka.publish("delivery.shipped", {order_id: 12})

Étape 4 : Notification
[email_service] -- Kafka.subscribe("order.created", "payment.successful", "delivery.shipped")
        |
        |--> envoie mails aux bonnes étapes
```

---

## Ce qu’il faut retenir

* **Kafka** joue le rôle de **messagerie centrale**. Il n’intervient pas dans le contenu, il se contente de :

  * Stocker les messages dans des **topics**
  * Permettre aux services **producteurs** d’écrire
  * Permettre aux **consommateurs** de lire

* Les **microservices** sont **responsables de ce qu’ils publient et consomment**. Kafka ne déclenche rien, c’est le service abonné qui **réagit**.

* **La chaîne d'événements se propage naturellement** par la publication/abonnement de topic en topic.

---

## Exemple concret avec un seul événement

### Situation : commande créée

1. `order_service` envoie :

   ```json
   Kafka.publish("order.created", {
     "order_id": 42,
     "user_id": 5,
     "total": 89.99
   })
   ```

2. `payment_service` reçoit ce message via son abonnement :

   ```python
   @kafka_consumer("order.created")
   def handle_order_created(event):
       # déclenche le paiement
       process_payment(event["order_id"])
       Kafka.publish("payment.successful", {"order_id": event["order_id"]})
   ```

3. `shipping_service` écoute `payment.successful` et livre.

