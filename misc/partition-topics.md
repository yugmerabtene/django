## 📦 1. **Organisation logique des topics**

Un **topic Kafka** est découpé en plusieurs **partitions**.

```
Topic : commande_validée
  ├── Partition 0 : [msg1, msg2, msg3, ...]
  ├── Partition 1 : [msg4, msg5, msg6, ...]
  └── Partition 2 : [msg7, msg8, msg9, ...]
```

* Chaque partition est un **journal (log) ordonné, append-only**
* Chaque message dans une partition a un **offset unique** (`msg0`, `msg1`, …)
* Kafka **écrit séquentiellement sur disque** (très performant)

---

## 💾 2. **Stockage physique sur disque**

* Les messages sont stockés **durablement sur disque**, dans des **fichiers segmentés**.
* Chaque partition = **dossier** dans Kafka, contenant des **fichiers `.log`** :

```bash
/kafka-data/
├── commande_validée-0/
│   ├── 00000000000000000000.log
│   ├── 00000000000000000000.index
│   └── 00000000000000000000.timeindex
├── commande_validée-1/
└── ...
```

* Kafka **n'efface pas** les messages après consommation (sauf si configuration de rétention l’exige).

---

## ⏳ 3. **Durée de stockage (rétention)**

Kafka stocke les messages selon la configuration du topic :

| Mode de rétention      | Description                                     |
| ---------------------- | ----------------------------------------------- |
| `retention.ms`         | Garde les messages pendant X millisecondes      |
| `retention.bytes`      | Garde les messages jusqu’à X octets             |
| `log.compaction`       | Garde **uniquement le dernier message** par clé |
| `delete.policy=delete` | Par défaut, les anciens messages sont supprimés |

**Exemple :**

```bash
retention.ms=604800000   # garde les messages 7 jours
```

---

## 🔄 4. **Lecture par les consommateurs**

* Chaque consommateur lit les messages **à son rythme**, en suivant un **offset**.
* Kafka **ne supprime pas un message quand un consommateur l’a lu**.
* Cela permet à :

  * plusieurs consommateurs de lire le même topic
  * redémarrer une lecture à un ancien offset (reprocessing)

---

## 🔐 5. **Durabilité et performance**

| Propriété            | Kafka                                              |
| -------------------- | -------------------------------------------------- |
| **Durabilité**       | Messages écrits sur disque, confirmés à l’émetteur |
| **Performance**      | Très élevée (I/O séquentiel, cache, compression)   |
| **Tolérance pannes** | Réplication des partitions sur plusieurs brokers   |

---

## ✅ Résumé

| Élément       | Fonction                                                   |
| ------------- | ---------------------------------------------------------- |
| **Topic**     | Canal logique de messages                                  |
| **Partition** | Journal séquentiel ordonné, contenant les messages         |
| **Offset**    | Position unique d’un message dans une partition            |
| **Stockage**  | Fichiers `.log` segmentés sur disque, un par partition     |
| **Rétention** | Messages conservés selon durée, taille ou politique de clé |
| **Lecture**   | Les consommateurs lisent indépendamment, selon leur offset |

