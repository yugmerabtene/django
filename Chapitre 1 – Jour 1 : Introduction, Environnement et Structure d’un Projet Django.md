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
Très bien. Voici une **présentation complète de Django** enrichie avec les **noms précis des outils/fonctionnalités** fournis par le framework (ORM, Admin, Template Engine, Middleware, etc.). Ce contenu est idéal pour un support de formation clair et précis.

---

# **Présentation complète de Django avec ses composants**

---

## **1. Qu’est-ce que Django ?**

Django est un **framework web Python haut niveau et open-source**, conçu pour le développement rapide, sécurisé et structuré d’applications web.

Il est fondé sur trois principes clés :

* **DRY** : *Don’t Repeat Yourself* (éviter la duplication de code)
* **“Batteries included”** : tout est inclus pour démarrer rapidement
* **Sécurité par défaut** : prévention intégrée contre les attaques courantes

---

## **2. Principales fonctionnalités intégrées de Django**

Django inclut nativement une **boîte à outils complète**, sans nécessiter de bibliothèques tierces :

| Fonctionnalité               | Nom de l’outil dans Django                     |
| ---------------------------- | ---------------------------------------------- |
| Mappage objets/relations     | **Django ORM** (`django.db.models`)            |
| Génération de back-office    | **Django Admin** (`django.contrib.admin`)      |
| Moteur de templates HTML     | **Django Template Engine** (`django.template`) |
| Routage des requêtes HTTP    | **URL Dispatcher** (`django.urls`)             |
| Gestion des formulaires HTML | **Forms / ModelForms** (`django.forms`)        |
| Authentification / sessions  | **Auth system** (`django.contrib.auth`)        |
| Gestion de la sécurité web   | **Middleware & CSRF Protection**               |
| API REST (optionnel)         | **Django REST Framework (DRF)**                |
| Internationalisation         | **i18n / l10n** (`django.utils.translation`)   |
| Tests automatisés            | **Test framework intégré** (`django.test`)     |

---

## **3. Django ORM (Object Relational Mapper)**

L’**ORM de Django** permet de manipuler la base de données via des objets Python, sans écrire de SQL.
Définition d’un modèle (représente une table) :

```python
class Annonce(models.Model):
    titre = models.CharField(max_length=255)
    prix = models.DecimalField(max_digits=10, decimal_places=2)
```

Requêtes courantes avec `QuerySet` :

```python
Annonce.objects.all()
Annonce.objects.filter(prix__lt=1000)
Annonce.objects.create(titre="Chaise", prix=20.00)
```

**Avantages** :

* Compatible avec PostgreSQL, MySQL, SQLite, Oracle
* Migrable automatiquement (`makemigrations`, `migrate`)
* Sécurité renforcée (requêtes préparées)

---

## **4. Django Admin (interface d’administration)**

L’**interface admin** est un back-office généré automatiquement pour gérer les modèles enregistrés.

Activation :

```python
from django.contrib import admin
from .models import Annonce

admin.site.register(Annonce)
```

Fonctionnalités :

* CRUD complet sans ligne de HTML
* Recherche, tri, filtres
* Personnalisation via `ModelAdmin`

L’interface se trouve par défaut à `/admin/`.

---

## **5. Django Template Engine (moteur de templates)**

Django intègre un moteur de templates performant et simple à prendre en main.
Les fichiers `.html` utilisent une syntaxe similaire à Jinja :

```html
{% for annonce in annonces %}
  <h2>{{ annonce.titre }}</h2>
  <p>{{ annonce.prix }} €</p>
{% endfor %}
```

Fonctionnalités :

* Héritage de templates avec `{% extends %}`
* Blocs définis avec `{% block %}`
* Filtres (`{{ prix|floatformat:2 }}`) et tags (`{% if %}`, `{% for %}`, etc.)

---

## **6. Django Forms & ModelForms**

Django fournit une API robuste pour créer et valider des formulaires HTML.

* `forms.Form` : pour des formulaires libres
* `forms.ModelForm` : pour créer des formulaires directement liés à un modèle

Exemple :

```python
class AnnonceForm(forms.ModelForm):
    class Meta:
        model = Annonce
        fields = ['titre', 'prix', 'description']
```

Fonctionnalités :

* Génération automatique des champs HTML
* Validation sécurisée (serveur)
* Gestion des erreurs avec messages utilisateurs

---

## **7. Authentification et gestion des utilisateurs**

Django inclut une application `django.contrib.auth` pour gérer :

* L’authentification (`login`, `logout`, `authenticate`)
* Les sessions
* Les groupes et permissions
* Les superutilisateurs
* Les utilisateurs personnalisés via `AbstractUser`

L’authentification s’intègre parfaitement avec l’admin et les permissions API/DRF.

---

## **8. Routage avec URL Dispatcher**

Le système de routing Django permet d’associer des URLs à des vues.

Exemple :

```python
urlpatterns = [
    path('annonces/', views.liste_annonces, name='liste_annonces'),
    path('annonces/<int:id>/', views.detail_annonce, name='detail_annonce'),
]
```

Fonctionnalités :

* Définition claire des routes
* Routes nommées avec `name=`
* Inclusion d’URLs d’applications (`include()`)

---

## **9. Middleware et sécurité intégrée**

Django embarque des **middlewares** chargés de gérer :

* La protection **CSRF** (`CsrfViewMiddleware`)
* Le contrôle des sessions (`SessionMiddleware`)
* La sécurité des headers HTTP
* La prévention contre le **XSS**, **clickjacking**, **SQL Injection**

Ajout automatique du token CSRF dans les formulaires :

```html
<form method="post">
  {% csrf_token %}
  ...
</form>
```

---

## **10. Outils complémentaires**

Django propose aussi :

* **Django Signals** : observer des événements (création d’objet, etc.)
* **Django Messages Framework** : messages flash dans les vues
* **Django Static Files** : gestion des fichiers CSS/JS/images en développement et production
* **Django Management Commands** : ajout de commandes personnalisées (`python manage.py ...`)

---

## **Conclusion**

Django est un framework **complet, robuste et sécurisé**, prêt pour la production dès le départ.

Il est particulièrement adapté pour :

* Des applications web rapides à déployer
* Des projets à moyen et long terme avec des équipes structurées
* Des API REST (avec DRF)
* Des portails administrables (grâce à l’admin auto-généré)

---
Un **tableau synthétique des composants Django et de leur rôle** ?


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
