## Qu’est-ce qu’un **load balancer** ?

Un **load balancer** (équilibreur de charge) est un composant réseau qui :

* **répartit intelligemment** le trafic entre plusieurs **instances** d’un même service
* améliore la **disponibilité**, la **scalabilité**, et la **résilience**

Il fonctionne soit au niveau **HTTP (L7)**, soit au niveau **TCP (L4)**.

---

## Pourquoi c’est important en microservices ?

Chaque microservice peut être :

* répliqué (plusieurs instances)
* conteneurisé (ex: 3 conteneurs `user_service`)
* distribué (sur plusieurs nœuds)

Le load balancer est celui qui décide **vers quelle instance** envoyer chaque requête.

---

## Où se place le Load Balancer dans l'architecture ?

### 1. **Entre le client et l'API Gateway**

* Si tu as plusieurs API Gateway (ex: `gateway-1`, `gateway-2`), un load balancer frontal peut répartir le trafic HTTP global.

```
[Client]
   ↓
[Load Balancer L7] (HTTP)
   ↓
[API Gateway] (1 ou plusieurs instances)
```

### 2. **Entre l’API Gateway et les microservices**

* L’API Gateway appelle un **service logique** (ex: `/api/users`), mais ce service peut avoir **plusieurs instances**, par exemple :

  * `user_service_1:8000`
  * `user_service_2:8000`

Le load balancer répartit le trafic entre ces instances.

```
[API Gateway]
   ↓
[Load Balancer interne]
   ↓        ↓
[user_service_1] [user_service_2]
```

### 3. **En interne Kubernetes** (optionnel)

* Kubernetes fournit un **load balancing DNS-based** automatiquement (via `ClusterIP` + `kube-proxy`).
* Ex : `http://user-service.default.svc.cluster.local` balance entre pods.

---

## Types de load balancing

| Type                   | Description                                                             |
| ---------------------- | ----------------------------------------------------------------------- |
| **Round-robin**        | Chaque requête va à la prochaine instance en rotation                   |
| **Least connections**  | Envoie la requête à l'instance la moins occupée                         |
| **Random**             | Sélection aléatoire                                                     |
| **IP hash**            | La requête va toujours à la même instance en fonction de l’IP           |
| **Weighted**           | Certains services reçoivent plus ou moins de trafic selon leur capacité |
| **Health-check aware** | Ignore les instances qui sont hors ligne ou en erreur                   |

---

## Outils de Load Balancing

| Outil                   | Cas d’usage principal                      |
| ----------------------- | ------------------------------------------ |
| **NGINX**               | Simple, HTTP load balancing                |
| **HAProxy**             | TCP/HTTP, haute performance                |
| **Traefik**             | Load balancing dynamique + auto-découverte |
| **Kubernetes**          | Load balancer DNS entre pods/services      |
| **Cloud Load Balancer** | AWS ELB, GCP LB, Azure FrontDoor           |

---

## Exemple avec NGINX

### Configuration pour équilibrer `user_service` :

```nginx
upstream user_backend {
    server user_service_1:8000;
    server user_service_2:8000;
}

server {
    listen 80;

    location /api/users/ {
        proxy_pass http://user_backend;
    }
}
```

---

## Utilité Résumée

| Bénéfice                    | Explication                                               |
| --------------------------- | --------------------------------------------------------- |
| **Haute disponibilité**     | Si une instance tombe, le trafic continue vers les autres |
| **Scalabilité horizontale** | Tu peux ajouter des instances à chaud                     |
| **Performance**             | Répartition du trafic = meilleures réponses               |
| **Résilience**              | Empêche la surcharge d’une seule instance                 |

---

## Quand tu en as **impérativement besoin**

* Dès que tu as **plusieurs instances d’un service**
* Dès que tu veux gérer du **failover automatique**
* Si tu veux faire du **rolling update** sans interruption de service

  ----


---

##  Le modèle OSI : résumé des 7 couches

| Couche | Nom                    | Fonction principale                                   | Exemple           |
| ------ | ---------------------- | ----------------------------------------------------- | ----------------- |
| 7      | **Application**        | Interface avec l’utilisateur ou une appli (HTTP, FTP) | HTTP, DNS, SMTP   |
| 6      | **Présentation**       | Encodage, chiffrement, compression                    | TLS/SSL, JPEG     |
| 5      | **Session**            | Ouverture, gestion, fermeture de sessions             | API WebSocket     |
| 4      | **Transport**          | Transmission fiable, segmentation                     | TCP, UDP          |
| 3      | **Réseau**             | Routage entre réseaux, adressage IP                   | IP, ICMP          |
| 2      | **Liaison de données** | Transmission sur un lien physique                     | Ethernet, Wi-Fi   |
| 1      | **Physique**           | Transmission brute (électrique, optique…)             | Câble RJ45, fibre |

---

##  Focus sur **L4** vs **L7** pour le Load Balancing et le Proxy

| Niveau | Couche | Nom OSI     | Load Balancing basé sur...         | Exemples d'outils                           | Cas d’usage typique                                              |
| ------ | ------ | ----------- | ---------------------------------- | ------------------------------------------- | ---------------------------------------------------------------- |
| L4     | 4      | Transport   | Adresse IP + Port (TCP/UDP)        | HAProxy (TCP mode), NGINX (stream), AWS NLB | Équilibrer des connexions TCP (ex: base de données, Kafka, SMTP) |
| L7     | 7      | Application | Contenu HTTP : URL, Header, Cookie | NGINX (HTTP mode), Traefik, Kong, AWS ALB   | API REST, routage par chemin, authentification, compression      |

---

##  Comparaison L4 vs L7 (résumé simplifié)

| Critère                      | L4 (Transport)                | L7 (Application)                              |
| ---------------------------- | ----------------------------- | --------------------------------------------- |
| Analyse du trafic            | Basé sur IP et port           | Basé sur contenu HTTP (URL, headers, cookies) |
| Performances                 | Plus rapide (moins d’analyse) | Légèrement plus lent (inspection complète)    |
| Routage avancé (path, host…) | ❌ Non                         | ✅ Oui                                         |
| TLS termination (HTTPS)      | ❌ Non                         | ✅ Oui                                         |
| Authentification             | ❌ Non                         | ✅ Possible                                    |
| Compression / cache          | ❌ Non                         | ✅ Possible                                    |
| Exemple de protocole         | TCP, UDP                      | HTTP, HTTPS, gRPC                             |
| Utilisation typique          | DB, Kafka, Redis, SMTP        | API Gateway, Web frontends, REST/GraphQL      |

---

##  Exemple concret

### L4 Load Balancer :

* Tu veux équilibrer plusieurs bases de données PostgreSQL
* Le LB reçoit du **TCP**, regarde l’**IP/port**, et transfère sans inspecter le contenu.

### L7 Load Balancer :

* Tu veux qu’un reverse proxy envoie :

  * `/api/users` vers `user_service`
  * `/api/orders` vers `order_service`
* Le proxy **lit l’URL** HTTP, peut vérifier un **token JWT**, compresser la réponse, etc.

---

##  En résumé

| Si tu veux…                                  | Choisis… |
| -------------------------------------------- | -------- |
| Gérer du trafic HTTP, REST, API versionnée   | L7       |
| Load-balancer du Kafka, PostgreSQL, Redis    | L4       |
| Ajouter de l’auth, du cache, du CORS, du SSL | L7       |
| Faire du routage rapide et bas niveau        | L4       |

