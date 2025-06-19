* Microservice Auth
* Microservice User
* API Gateway
* Kafka (bus de messages)

---

## Objectif :

Un utilisateur se connecte, puis accède à son profil (`GET /user/profile`).

---

## Architecture simplifiée

```
[Client]
   |
   v
[API Gateway]
   |
   +---> [Auth MS] <------> [Kafka] <------> [User MS]
   |
   +---> [User MS]   ←─── (pour /user/profile)
```

---

## Étapes détaillées avec les microservices concernés

### ÉTAPE 1 — Le client envoie une requête de connexion

**Microservices concernés : Client → API Gateway**

Le client envoie une requête HTTP :

```
POST /auth/login
Body: { "email": "user@example.com", "password": "123456" }
```

---

### ÉTAPE 2 — L'API Gateway route la requête vers Auth MS

**Microservices concernés : API Gateway → Auth MS**

La Gateway détecte que la route `/auth/**` appartient au microservice Auth.
Elle redirige la requête vers l'endpoint approprié dans Auth MS.

---

### ÉTAPE 3 — Auth MS vérifie les identifiants via Kafka

**Microservice concerné : Auth MS**

* Le mot de passe est hashé
* Auth MS publie un message sur Kafka dans le topic `auth.user.verify` :

```json
{
  "type": "CHECK_CREDENTIALS",
  "data": {
    "email": "user@example.com",
    "passwordHash": "hash..."
  }
}
```

---

### ÉTAPE 4 — Kafka transmet le message à User MS

**Microservices concernés : Kafka → User MS**

Kafka distribue le message au microservice User, abonné au topic `auth.user.verify`.

---

### ÉTAPE 5 — User MS traite la vérification

**Microservice concerné : User MS**

* Recherche l'utilisateur par email
* Compare le hash
* Si les identifiants sont valides, il publie une réponse sur Kafka (`auth.user.response`) :

```json
{
  "type": "USER_VERIFIED",
  "data": {
    "userId": "u123",
    "roles": ["USER"],
    "status": "success"
  }
}
```

---

### ÉTAPE 6 — Auth MS reçoit la réponse et génère le JWT

**Microservices concernés : Kafka → Auth MS**

Auth MS consomme la réponse de Kafka, crée un JWT (contenant userId, roles, expiration), et le retourne à l’API Gateway :

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6..."
}
```

---

### ÉTAPE 7 — L’API Gateway renvoie le token au client

**Microservices concernés : Auth MS → API Gateway → Client**

La Gateway renvoie la réponse à l’utilisateur, qui stocke le token JWT.

---

### ÉTAPE 8 — Le client appelle `/user/profile` avec le token

**Microservices concernés : Client → API Gateway**

Le client envoie :

```
GET /user/profile
Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6...
```

---

### ÉTAPE 9 — API Gateway valide le token et route vers User MS

**Microservices concernés : API Gateway → User MS**

* La Gateway valide la signature, expiration et claims du token JWT
* Elle transmet la requête au microservice User

---

### ÉTAPE 10 — User MS renvoie les infos du profil

**Microservices concernés : User MS → API Gateway → Client**

* Utilise le userId pour accéder aux données
* Retourne le profil de l'utilisateur :

```json
{
  "userId": "u123",
  "email": "user@example.com",
  "name": "John Doe"
}
```

---

## Tableau récapitulatif

| Étape | Action                   | Source → Destination                 | Technologie      |
| ----- | ------------------------ | ------------------------------------ | ---------------- |
| 1     | Requête login            | Client → API Gateway                 | HTTP             |
| 2     | Routage login            | API Gateway → Auth MS                | HTTP             |
| 3     | Vérification via Kafka   | Auth MS → Kafka (auth.user.verify)   | Kafka            |
| 4     | Consommation             | Kafka → User MS                      | Kafka            |
| 5     | Réponse                  | User MS → Kafka (auth.user.response) | Kafka            |
| 6     | JWT généré               | Kafka → Auth MS                      | Traitement local |
| 7     | Réponse login            | Auth MS → API Gateway → Client       | HTTP             |
| 8     | Accès profil             | Client → API Gateway                 | HTTP             |
| 9     | Validation JWT + routage | API Gateway → User MS                | HTTP             |
| 10    | Réponse profil           | User MS → API Gateway → Client       | HTTP             |

