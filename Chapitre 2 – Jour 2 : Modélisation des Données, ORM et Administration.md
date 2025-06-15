# **Chapitre 2 – Jour 2 : Modélisation des Données, ORM et Administration**

---

## **1. Objectifs pédagogiques**

À la fin de cette journée, l’apprenant sera capable de :

* Créer des modèles Django représentant des entités métier
* Relier des modèles entre eux via des relations OneToMany, ManyToMany, OneToOne
* Utiliser l’ORM pour insérer, lire et filtrer les données
* Créer et appliquer des migrations pour mettre à jour la base de données
* Utiliser l’interface d’administration pour gérer les données
* Personnaliser l’affichage des modèles dans l’interface admin

---

## **2. Modélisation des Données avec Django**

### **2.1. Les modèles Django (`models.Model`)**

Un modèle Django représente une **table de base de données**. Chaque attribut du modèle devient une **colonne**.

Exemple simple :

```python
from django.db import models

class Categorie(models.Model):
    nom = models.CharField(max_length=100)

    def __str__(self):
        return self.nom
```

### **2.2. Champs de base courants**

| Type de champ   | Utilisation                                 | Exemple                                   |
| --------------- | ------------------------------------------- | ----------------------------------------- |
| CharField       | Texte court                                 | nom, titre, etc.                          |
| TextField       | Texte long                                  | description                               |
| IntegerField    | Nombres entiers                             | prix, quantité                            |
| BooleanField    | Vrai / Faux                                 | actif, visible                            |
| DateTimeField   | Dates et heures                             | date de publication                       |
| ImageField      | Pour stocker un chemin d’image              | photo = models.ImageField(upload\_to=...) |
| ForeignKey      | Relation OneToMany                          | user = models.ForeignKey(User, ...)       |
| ManyToManyField | Relation multiple à multiple                | tags = models.ManyToManyField(Tag)        |
| OneToOneField   | Relation un à un (profil utilisateur, etc.) | profil = models.OneToOneField(...)        |

---

## **3. Création de modèles : Annonce, Categorie, Utilisateur**

### **3.1. Modèle Categorie**

```python
class Categorie(models.Model):
    nom = models.CharField(max_length=100)

    def __str__(self):
        return self.nom
```

### **3.2. Modèle Annonce**

```python
from django.contrib.auth import get_user_model

class Annonce(models.Model):
    titre = models.CharField(max_length=255)
    description = models.TextField()
    prix = models.DecimalField(max_digits=10, decimal_places=2)
    date_publication = models.DateTimeField(auto_now_add=True)
    categorie = models.ForeignKey(Categorie, on_delete=models.CASCADE)
    utilisateur = models.ForeignKey(get_user_model(), on_delete=models.CASCADE)
    image = models.ImageField(upload_to='annonces/', blank=True, null=True)

    def __str__(self):
        return self.titre
```

---

## **4. Migrations : création des tables**

Une migration est une "traduction" de votre modèle Python en une commande SQL exécutée par Django.

### Étapes :

```bash
python manage.py makemigrations
python manage.py migrate
```

* `makemigrations` : génère des fichiers de migration
* `migrate` : applique les fichiers et crée/modifie la base

---

## **5. L’ORM Django : lire et manipuler les données**

L’ORM (Object Relational Mapper) permet d’interagir avec la base **sans SQL**.

### Requêtes de base :

```python
# Tout récupérer
Annonce.objects.all()

# Récupérer par ID
Annonce.objects.get(id=3)

# Filtrer
Annonce.objects.filter(categorie__nom="Informatique")

# Exclure
Annonce.objects.exclude(prix__lt=10)

# Ordre
Annonce.objects.order_by('-date_publication')

# Créer
Annonce.objects.create(titre="Chaise", description="Neuve", prix=20.00, utilisateur=u, categorie=c)
```

---

## **6. Interface d’administration Django**

### **6.1. Enregistrement des modèles**

Dans `annonces/admin.py` :

```python
from django.contrib import admin
from .models import Annonce, Categorie

admin.site.register(Annonce)
admin.site.register(Categorie)
```

### **6.2. Personnalisation de l’affichage**

```python
@admin.register(Annonce)
class AnnonceAdmin(admin.ModelAdmin):
    list_display = ['id', 'titre', 'prix', 'categorie', 'utilisateur', 'date_publication']
    search_fields = ['titre', 'description']
    list_filter = ['categorie', 'date_publication']
```

---

## **7. Utilisateur personnalisé avec AbstractUser**

Créer une nouvelle app `comptes`, puis un modèle :

```python
from django.contrib.auth.models import AbstractUser
from django.db import models

class Utilisateur(AbstractUser):
    telephone = models.CharField(max_length=20, blank=True)
```

Dans `settings.py` :

```python
AUTH_USER_MODEL = 'comptes.Utilisateur'
```

Créer un formulaire de superuser et réinitialiser la base (car changement du modèle de User).

---

## **8. Travaux Pratiques (avec guidage)**

### **TP 1 : Créer les modèles**

* `Categorie` : nom
* `Annonce` : titre, description, prix, date\_publication, utilisateur, image, categorie
* `Utilisateur` : héritant de `AbstractUser` avec un champ `telephone`

### **TP 2 : Lier les modèles**

* Une `Annonce` appartient à une `Categorie`
* Une `Annonce` est postée par un `Utilisateur`

### **TP 3 : Personnaliser l’admin**

* Afficher les annonces avec les colonnes `titre`, `categorie`, `prix`, `utilisateur`, `date_publication`
* Ajouter un filtre par `categorie`
* Activer la recherche par `titre` ou `description`

---

## **9. Exercices de validation**

**Exercice 1**
Créer une catégorie “Véhicules” et une “Immobilier” dans l’admin

**Exercice 2**
Créer deux annonces depuis l’interface d’administration

**Exercice 3**
Afficher dans l’admin les annonces les plus récentes en haut (order\_by par défaut)

**Exercice 4**
Uploader une image pour une annonce et l’afficher dans le détail (prévoir MEDIA\_ROOT dans settings.py)

---

## **10. Configuration de gestion des fichiers médias**

Dans `settings.py` :

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

Dans `urls.py` :

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## **11. Pièges fréquents et conseils**

* Oublier `__str__()` : les objets dans l’admin s’affichent comme “Annonce object”
* Ne pas déclarer l’app dans `INSTALLED_APPS`
* Ne pas appliquer les migrations (`migrate`)
* Ne pas importer le bon modèle d’utilisateur (`get_user_model()` conseillé)
* Ne pas activer le support des médias en mode DEBUG

---


Modélisation, ORM et Admin**, avec :

1.  **Fichier `models.py` complet**
2.  **Fichier `admin.py` complet avec personnalisation**
3.  **Fichier `forms.py` pour la création d’une annonce (préparation Jour 3)**
4.  **Exercices corrigés complets en `.py`**

---

## **1. Fichier `models.py` complet**

```python
from django.db import models
from django.contrib.auth.models import AbstractUser

# Utilisateur personnalisé
class Utilisateur(AbstractUser):
    telephone = models.CharField(max_length=20, blank=True)

    def __str__(self):
        return f"{self.username} ({self.email})"


# Catégorie d'annonce
class Categorie(models.Model):
    nom = models.CharField(max_length=100)

    def __str__(self):
        return self.nom


# Annonce
class Annonce(models.Model):
    titre = models.CharField(max_length=255)
    description = models.TextField()
    prix = models.DecimalField(max_digits=10, decimal_places=2)
    date_publication = models.DateTimeField(auto_now_add=True)
    categorie = models.ForeignKey(Categorie, on_delete=models.CASCADE)
    utilisateur = models.ForeignKey(Utilisateur, on_delete=models.CASCADE)
    image = models.ImageField(upload_to='annonces/', blank=True, null=True)

    def __str__(self):
        return self.titre
```

---

## **2. Fichier `admin.py` complet avec personnalisation**

```python
from django.contrib import admin
from .models import Annonce, Categorie, Utilisateur
from django.contrib.auth.admin import UserAdmin

@admin.register(Utilisateur)
class UtilisateurAdmin(UserAdmin):
    model = Utilisateur
    list_display = ['username', 'email', 'telephone', 'is_staff', 'is_superuser']
    fieldsets = UserAdmin.fieldsets + (
        (None, {'fields': ('telephone',)}),
    )

@admin.register(Categorie)
class CategorieAdmin(admin.ModelAdmin):
    list_display = ['id', 'nom']
    search_fields = ['nom']

@admin.register(Annonce)
class AnnonceAdmin(admin.ModelAdmin):
    list_display = ['id', 'titre', 'prix', 'categorie', 'utilisateur', 'date_publication']
    list_filter = ['categorie', 'date_publication']
    search_fields = ['titre', 'description']
    ordering = ['-date_publication']
```

---

## **3. Fichier `forms.py` pour le modèle `Annonce` (préparation Jour 3)**

```python
from django import forms
from .models import Annonce

class AnnonceForm(forms.ModelForm):
    class Meta:
        model = Annonce
        fields = ['titre', 'description', 'prix', 'categorie', 'image']
```

---

## **4. Exercices corrigés complets**

### **Exercice 1 – Créer des catégories**

```python
from annonces.models import Categorie

Categorie.objects.create(nom="Véhicules")
Categorie.objects.create(nom="Immobilier")
```

### **Exercice 2 – Créer deux annonces via shell Django**

```python
from annonces.models import Annonce, Categorie, Utilisateur

# Supposons que l'utilisateur existe
user = Utilisateur.objects.get(username='admin')
cat1 = Categorie.objects.get(nom="Véhicules")

Annonce.objects.create(
    titre="Peugeot 208 d'occasion",
    description="Bonne voiture, 90 000 km.",
    prix=6200,
    categorie=cat1,
    utilisateur=user
)

Annonce.objects.create(
    titre="Garage à louer",
    description="Garage fermé en centre-ville.",
    prix=120,
    categorie=Categorie.objects.get(nom="Immobilier"),
    utilisateur=user
)
```

### **Exercice 3 – Affichage des annonces par date décroissante (dans admin)**

Déjà traité dans `AnnonceAdmin` :

```python
ordering = ['-date_publication']
```

### **Exercice 4 – Uploader une image (ajoutée à `Annonce`)**

* S'assurer que l'image est uploadée via l’admin
* Le champ `image` dans `models.py` :

```python
image = models.ImageField(upload_to='annonces/', blank=True, null=True)
```

* Configuration `settings.py` :

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

* Ajout dans `urls.py` :

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---
Voici un **jeu de données de test complet en JSON (`fixtures.json`)** pour Django, contenant :

* Deux utilisateurs (admin et user1)
* Deux catégories (`Véhicules`, `Immobilier`)
* Trois annonces associées aux utilisateurs et catégories

Ce fichier est prêt à être chargé avec la commande :

```bash
python manage.py loaddata fixtures.json
```

---

### `fixtures.json`

```json
[
  {
    "model": "comptes.utilisateur",
    "pk": 1,
    "fields": {
      "password": "pbkdf2_sha256$600000$...$hashval", 
      "last_login": null,
      "is_superuser": true,
      "username": "admin",
      "first_name": "Admin",
      "last_name": "User",
      "email": "admin@example.com",
      "is_staff": true,
      "is_active": true,
      "date_joined": "2024-01-01T00:00:00Z",
      "telephone": "0102030405",
      "groups": [],
      "user_permissions": []
    }
  },
  {
    "model": "comptes.utilisateur",
    "pk": 2,
    "fields": {
      "password": "pbkdf2_sha256$600000$...$hashval", 
      "last_login": null,
      "is_superuser": false,
      "username": "user1",
      "first_name": "Jean",
      "last_name": "Dupont",
      "email": "jean.dupont@example.com",
      "is_staff": false,
      "is_active": true,
      "date_joined": "2024-02-01T00:00:00Z",
      "telephone": "0601020304",
      "groups": [],
      "user_permissions": []
    }
  },
  {
    "model": "annonces.categorie",
    "pk": 1,
    "fields": {
      "nom": "Véhicules"
    }
  },
  {
    "model": "annonces.categorie",
    "pk": 2,
    "fields": {
      "nom": "Immobilier"
    }
  },
  {
    "model": "annonces.annonce",
    "pk": 1,
    "fields": {
      "titre": "Peugeot 208",
      "description": "Bonne occasion, faible kilométrage.",
      "prix": "6200.00",
      "date_publication": "2024-06-01T10:00:00Z",
      "categorie": 1,
      "utilisateur": 2,
      "image": ""
    }
  },
  {
    "model": "annonces.annonce",
    "pk": 2,
    "fields": {
      "titre": "Appartement T2 à louer",
      "description": "Appartement meublé proche du centre-ville.",
      "prix": "750.00",
      "date_publication": "2024-06-02T12:00:00Z",
      "categorie": 2,
      "utilisateur": 2,
      "image": ""
    }
  },
  {
    "model": "annonces.annonce",
    "pk": 3,
    "fields": {
      "titre": "Scooter électrique neuf",
      "description": "Autonomie 40 km, batterie amovible.",
      "prix": "800.00",
      "date_publication": "2024-06-03T14:00:00Z",
      "categorie": 1,
      "utilisateur": 2,
      "image": ""
    }
  }
]
```

---

### Remarques :

1. Les mots de passe ci-dessus sont tronqués pour des raisons de sécurité. Pour en générer un vrai :

```python
from django.contrib.auth.hashers import make_password
make_password("votre_mot_de_passe")
```

2. Place le fichier dans un dossier `fixtures/` à la racine de l'app ou du projet.

3. Commande de chargement :

```bash
python manage.py loaddata fixtures/fixtures.json
```

**script Python Django** pour générer dynamiquement des **fixtures (`fixtures.json`)** avec catégories, utilisateurs, et annonces — utile pour automatiser tes tests ou ton environnement de démo.

---

## Script : `generate_fixtures.py`

À placer dans la racine du projet ou dans un dossier `scripts/`. À exécuter avec `python manage.py shell < generate_fixtures.py`.

```python
import json
from django.utils.timezone import now
from django.contrib.auth.hashers import make_password

fixtures = []

# Utilisateurs
utilisateurs = [
    {
        "model": "comptes.utilisateur",
        "pk": 1,
        "fields": {
            "password": make_password("admin123"),
            "last_login": None,
            "is_superuser": True,
            "username": "admin",
            "first_name": "Admin",
            "last_name": "User",
            "email": "admin@example.com",
            "is_staff": True,
            "is_active": True,
            "date_joined": str(now()),
            "telephone": "0102030405",
            "groups": [],
            "user_permissions": []
        }
    },
    {
        "model": "comptes.utilisateur",
        "pk": 2,
        "fields": {
            "password": make_password("user123"),
            "last_login": None,
            "is_superuser": False,
            "username": "user1",
            "first_name": "Jean",
            "last_name": "Dupont",
            "email": "jean.dupont@example.com",
            "is_staff": False,
            "is_active": True,
            "date_joined": str(now()),
            "telephone": "0601020304",
            "groups": [],
            "user_permissions": []
        }
    }
]
fixtures.extend(utilisateurs)

# Catégories
categories = [
    {
        "model": "annonces.categorie",
        "pk": 1,
        "fields": {"nom": "Véhicules"}
    },
    {
        "model": "annonces.categorie",
        "pk": 2,
        "fields": {"nom": "Immobilier"}
    }
]
fixtures.extend(categories)

# Annonces
annonces = [
    {
        "model": "annonces.annonce",
        "pk": 1,
        "fields": {
            "titre": "Peugeot 208",
            "description": "Bonne occasion, faible kilométrage.",
            "prix": "6200.00",
            "date_publication": str(now()),
            "categorie": 1,
            "utilisateur": 2,
            "image": ""
        }
    },
    {
        "model": "annonces.annonce",
        "pk": 2,
        "fields": {
            "titre": "Appartement T2 à louer",
            "description": "Appartement meublé proche du centre-ville.",
            "prix": "750.00",
            "date_publication": str(now()),
            "categorie": 2,
            "utilisateur": 2,
            "image": ""
        }
    }
]
fixtures.extend(annonces)

# Export
with open("fixtures.json", "w", encoding="utf-8") as f:
    json.dump(fixtures, f, indent=2, ensure_ascii=False)

print("✔ Fichier fixtures.json généré.")
```

---

## Instructions

1. Place ce script dans le dossier de ton projet Django (ex: `scripts/generate_fixtures.py`)
2. Lance-le depuis un terminal :

```bash
python manage.py shell < scripts/generate_fixtures.py
```

3. Le fichier `fixtures.json` sera généré à la racine de ton projet.

4. Tu peux ensuite charger les données avec :

```bash
python manage.py loaddata fixtures.json
```

---
