# **Cahier des charges – Application LebonClone**

## **1. Contexte du projet**

L’application **LebonClone** est une plateforme de dépôt, de consultation et de gestion d’annonces en ligne. Elle doit proposer :

* Une interface web utilisateur (front HTML Django)
* Une interface administrateur (Django admin)
* Une API REST sécurisée (Django REST Framework)

---

## **2. Objectifs fonctionnels**

### 2.1. Gestion des comptes utilisateurs

* Création de compte avec :

  * Nom d’utilisateur
  * Email
  * Numéro de téléphone
  * Mot de passe sécurisé
* Connexion / déconnexion
* Authentification requise pour déposer une annonce
* Interface de gestion via le back-office admin

### 2.2. Gestion des catégories

* Création d’une ou plusieurs catégories d’annonces (ex. : Immobilier, Véhicules, Informatique…)
* Chaque annonce est associée à une catégorie

### 2.3. Dépôt et consultation des annonces

* Création d’une annonce avec :

  * Titre
  * Description
  * Prix
  * Catégorie
  * Image (upload)
  * Date de publication
* Un utilisateur connecté peut :

  * Créer une annonce
  * Modifier ou supprimer ses propres annonces
* Un visiteur peut :

  * Consulter la liste des annonces
  * Voir le détail d’une annonce

### 2.4. Affichage

* Page d’accueil listant toutes les annonces
* Page de détail de chaque annonce
* Formulaire de création/modification
* Affichage conditionnel si l’utilisateur est connecté ou non

---

## **3. API REST à exposer**

L’API doit permettre l’accès aux ressources suivantes au format JSON :

### 3.1. Annonces

* `GET /api/annonces/` : liste des annonces
* `GET /api/annonces/<id>/` : détail d’une annonce
* `POST /api/annonces/` : création (authentification requise)
* `PUT /api/annonces/<id>/` : modification (authentification requise, propriétaire uniquement)
* `DELETE /api/annonces/<id>/` : suppression (authentification requise, propriétaire uniquement)

### 3.2. Catégories

* `GET /api/categories/` : liste des catégories
* `GET /api/categories/<id>/` : détail d’une catégorie

### 3.3. Authentification

* `POST /api/token/` : obtention du token JWT
* `POST /api/token/refresh/` : renouvellement du token

### 3.4. Permissions API

* Lecture publique (GET autorisé à tous)
* Création/modification/suppression restreintes aux utilisateurs connectés
* Accès autorisé uniquement aux propriétaires pour modifier/supprimer leurs annonces

---

## **4. Contraintes techniques**

* Framework backend : **Django 4.x+**
* API REST : **Django REST Framework (DRF)**
* Base de données : **SQLite** (développement), extensible à PostgreSQL
* Authentification API : **JWT** via `djangorestframework-simplejwt`
* Frontend : HTML/CSS avec Django Template Engine
* Hébergement : local, puis préparation possible pour Railway ou VPS
* Fichiers média : upload dans `/media/`, avec configuration `MEDIA_ROOT`

---

## **5. Architecture applicative**

### Apps Django

* `annonces` : gestion des modèles Annonce, Categorie
* `comptes` : gestion des utilisateurs personnalisés
* `api` : vues DRF (optionnel ou intégré dans `annonces`)

### Modèles principaux

```python
class Utilisateur(AbstractUser):
    telephone = models.CharField(max_length=20)

class Categorie(models.Model):
    nom = models.CharField(max_length=100)

class Annonce(models.Model):
    titre = models.CharField(max_length=255)
    description = models.TextField()
    prix = models.DecimalField(...)
    categorie = models.ForeignKey(Categorie, ...)
    utilisateur = models.ForeignKey(Utilisateur, ...)
    image = models.ImageField(...)
    date_publication = models.DateTimeField(auto_now_add=True)
```

---

## **6. Sécurité attendue**

* Protection CSRF sur les formulaires web
* Protection des routes avec `@login_required`
* Authentification des endpoints API via JWT
* Permissions DRF par utilisateur
* Accès admin réservé au personnel (`is_staff=True`)

---

## **7. Tests à réaliser**

* Inscription / connexion utilisateur
* Ajout d’une annonce
* Modification par le propriétaire
* Interdiction d’accès pour un autre utilisateur
* Navigation complète via l’interface HTML
* Requêtes API via Postman ou Swagger

---

## **8. Livrables attendus**

* Code source complet du projet Django
* Fichiers `requirements.txt`, `.env.example`, `Procfile` (si déploiement)
* Documentation technique :

  * Installation locale
  * Routes disponibles (web + API)
  * Screenshots ou capture d’écran de test
* Fixtures : catégories + quelques annonces
* README.md complet
