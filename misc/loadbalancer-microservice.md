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
