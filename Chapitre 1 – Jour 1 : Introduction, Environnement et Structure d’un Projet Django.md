# **Chapitre 1 – Jour 1 : Introduction, Environnement et Structure d’un Projet Django**

---

## **1. Objectifs pédagogiques**

À l’issue de cette première journée, l’apprenant sera capable de :

* Comprendre ce qu’est Django, sa philosophie, son architecture MTV
* Installer un environnement de travail propre et professionnel
* Créer un projet Django et une première application
* Comprendre la structure d’un projet Django
* Lancer un serveur de développement
* Créer un superutilisateur pour accéder à l’interface d’administration

---

## **2. Introduction à Django et à l’architecture MTV**

### **2.1. Qu’est-ce qu’un framework web ?**

Un framework web est un ensemble d’outils et de bibliothèques qui facilitent le développement d'applications web. Il gère :

* Les requêtes HTTP/HTTPS
* La gestion des routes et des URLs
* La connexion à la base de données
* Le rendu des pages HTML
* La sécurité, les sessions, les cookies, etc.

### **2.2. Présentation de Django**

Django est un framework Python **haut niveau**, **open-source**, qui permet de construire des sites web rapidement, avec :

* Un ORM intégré
* Une interface d’administration automatique
* Des vues et templates
* Des outils de sécurité intégrés

Philosophie Django : **"Batteries included"**, **"DRY" (Don't Repeat Yourself)** et sécurité dès le départ.

### **2.3. Architecture MTV (Model - Template - View)**

**MTV ≠ MVC**, mais concepts similaires :

* **Model** : structure des données (comme les entités dans une base de données)
* **Template** : affichage HTML, Jinja-like
* **View** : logique métier (appel du modèle, choix du template)

---

## **3. Mise en place de l’environnement de développement**

### **3.1. Prérequis**

* Python 3.10+
* pip
* Un terminal (Bash, PowerShell ou Terminal sous macOS/Linux)
* Un éditeur de code : VS Code ou PyCharm

### **3.2. Création d’un environnement virtuel**

Sous Linux/macOS :

```bash
python3 -m venv venv
source venv/bin/activate
```

Sous Windows :

```powershell
python -m venv venv
venv\Scripts\activate
```

### **3.3. Installation de Django**

```bash
pip install django
```

Vérification de l’installation :

```bash
django-admin --version
```

---

## **4. Création d’un projet Django**

```bash
django-admin startproject lebonclone
cd lebonclone
```

Contenu généré :

* **manage.py** : point d’entrée pour les commandes Django
* **lebonclone/** : dossier de configuration contenant :

  * **init**.py : marque un paquet Python
  * settings.py : configuration globale du projet
  * urls.py : définition des routes principales
  * wsgi.py : point d’entrée pour déploiement WSGI
  * asgi.py : idem pour ASGI (WebSocket)

---

## **5. Lancement du serveur de développement**

```bash
python manage.py runserver
```

Par défaut : [http://127.0.0.1:8000/](http://127.0.0.1:8000/)
Si tout est bon : “L’installation s’est déroulée avec succès…”

---

## **6. Création d’une première application**

```bash
python manage.py startapp annonces
```

Dans `settings.py`, enregistrer l’app dans `INSTALLED_APPS` :

```python
INSTALLED_APPS = [
    ...
    'annonces',
]
```

Structure d’une app :

* models.py : définition des modèles de données
* views.py : logique métier
* urls.py (à créer)
* admin.py : enregistrement des modèles dans l’admin
* apps.py : configuration de l’app
* migrations/ : fichiers de migration générés

---

## **7. Premiers pas avec les vues et les URLs**

### **7.1. Créer une vue simple dans `annonces/views.py`**

```python
from django.http import HttpResponse

def home(request):
    return HttpResponse("Bienvenue sur LeBonClone !")
```

### **7.2. Définir un fichier `annonces/urls.py`**

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.home, name='home'),
]
```

### **7.3. Inclure les routes de l'app dans `lebonclone/urls.py`**

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('annonces.urls')),
]
```

Lancer le serveur et vérifier que la page [http://127.0.0.1:8000/](http://127.0.0.1:8000/) affiche “Bienvenue sur LeBonClone !”

---

## **8. Interface d’administration Django**

### **8.1. Création d’un superutilisateur**

```bash
python manage.py createsuperuser
```

Entrer un nom, email, mot de passe. Accéder à l’admin :
[http://127.0.0.1:8000/admin/](http://127.0.0.1:8000/admin/)

### **8.2. Activer les modèles dans l’admin (préparation pour le jour 2)**

Dans `admin.py` :

```python
from django.contrib import admin
from .models import MonModel

admin.site.register(MonModel)
```

---

## **9. Exercices de fin de journée**

**Exercice 1 : Créer un nouveau projet Django nommé `plateforme_annonces`**

* Créer une app `annonces`
* Afficher un message personnalisé dans la vue d’accueil

**Exercice 2 : Créer un environnement virtuel pour ce projet**

* Installer Django
* Documenter toutes les étapes dans un fichier `README.md`

**Exercice 3 : Créer un superutilisateur et accéder à l’interface d’administration**

---

## **10. Points d’attention**

* Insister sur la distinction entre projet (global) et app (fonctionnelle)
* Bien expliquer l’intérêt d’un environnement virtuel
* Montrer les erreurs courantes : app non ajoutée dans `INSTALLED_APPS`, mauvaise indentation Python, oubli du `include()` dans `urls.py`
