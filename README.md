# **Chapitre 1 – Jour 1 : Introduction, Environnement et Structure d’un Projet Django**

**Objectifs pédagogiques**
Comprendre le fonctionnement de Django (MTV)
Maîtriser la création d’un environnement de développement
Démarrer un projet Django structuré

**Contenu théorique**

* Qu’est-ce qu’un framework web
* Pourquoi Django (sécurité, rapidité, DRY, ORM intégré)
* Différences entre MVC et MTV
* Présentation des fichiers de projet : `manage.py`, `settings.py`, `urls.py`, `wsgi.py`
* Lancement du serveur et modes DEBUG/PROD

**Travaux pratiques**

* Installer Python, pip, virtualenv
* Créer un environnement virtuel
* Installer Django dans l’environnement virtuel
* Lancer un projet avec `django-admin startproject lebonclone`
* Créer une app `annonces`
* Lancer le serveur local avec `runserver`
* Créer un superutilisateur avec `createsuperuser`

**Exercices**

1. Démarrer un nouveau projet Django
2. Créer une app et la connecter au projet
3. Afficher une page "Hello Django" via une vue simple

---

# **Chapitre 2 – Jour 2 : Modélisation des Données, ORM et Administration**

**Objectifs pédagogiques**
Maîtriser la création de modèles relationnels
Comprendre les migrations et l’ORM
Gérer les données via l’interface d’administration

**Contenu théorique**

* Définition des modèles avec `models.Model`
* Champs de base : `CharField`, `TextField`, `DateTime`, `Boolean`, `ForeignKey`
* Relations : OneToMany, ManyToMany, OneToOne
* Commandes de migration : `makemigrations`, `migrate`
* Requêtes ORM : `objects.all()`, `filter()`, `get()`, `exclude()`
* Personnalisation de l’interface admin : `list_display`, `search_fields`, `list_filter`

**Travaux pratiques**

* Créer les modèles `Categorie`, `Annonce`, `Utilisateur` personnalisé avec `AbstractUser`
* Ajouter les modèles dans `admin.py`
* Enrichir l’administration avec filtres et colonnes personnalisées

**Exercices**

1. Créer les modèles avec relations
2. Ajouter des objets via l’admin
3. Ajouter une image à un modèle avec `ImageField`

---

# **Chapitre 3 – Jour 3 : Vues, Templates, Formulaires et Routage**

**Objectifs pédagogiques**
Comprendre le fonctionnement des vues (function-based views)
Créer des templates HTML avec héritage
Gérer des formulaires HTML et Django

**Contenu théorique**

* Organisation entre `views.py`, `templates/`, `urls.py`
* Vue simple vs vue avec formulaire
* Syntaxe des templates Django : `{% for %}`, `{% if %}`, `{% block %}`, `{% extends %}`
* Création et utilisation des `Form` et `ModelForm`
* Gestion des URLs : `path()`, `include()`, `name=`

**Travaux pratiques**

* Afficher la liste des annonces
* Créer une page de détail d’annonce
* Créer une vue de formulaire de création
* Afficher les erreurs de validation

**Exercices**

1. Afficher toutes les annonces dans un template
2. Créer une page de création d’annonce
3. Valider dynamiquement un champ obligatoire

---

# **Chapitre 4 – Jour 4 : Authentification, Gestion des Médias, Permissions**

**Objectifs pédagogiques**
Maîtriser l’authentification et la gestion des sessions
Gérer l’accès avec les décorateurs
Uploader et servir des images

**Contenu théorique**

* Système d’authentification Django : `User`, `login`, `logout`, `authenticate`
* Personnalisation du modèle `User` via `AbstractUser`
* Templates conditionnels avec `request.user`
* Configuration des médias : `MEDIA_URL`, `MEDIA_ROOT`
* Utilisation des décorateurs `login_required` et tests avec `user.is_authenticated`

**Travaux pratiques**

* Créer un formulaire d’inscription
* Mettre en place un système de connexion et de déconnexion
* Restreindre l’accès à la création d’annonce aux utilisateurs connectés
* Ajouter une image à chaque annonce

**Exercices**

1. Créer la page d’inscription utilisateur
2. Uploader une image pour chaque annonce
3. Protéger les pages avec le décorateur `login_required`

---

# **Chapitre 5 – Jour 5 : Django REST Framework et Déploiement**

**Objectifs pédagogiques**
Créer une API RESTful avec Django REST Framework
Sécuriser l’API
Comprendre le déploiement en production

**Contenu théorique**

* Présentation de Django REST Framework
* Création de `Serializers` avec `ModelSerializer`
* Création de vues avec `ViewSets` et `Router`
* Authentification avec `TokenAuthentication`, `permissions.IsAuthenticated`
* Préparation du projet pour la production : `DEBUG=False`, variables d’environnement
* Bases du déploiement : `requirements.txt`, `gunicorn`, `Procfile`

**Travaux pratiques**

* Créer un endpoint API pour les annonces
* Protéger les routes API avec permissions
* Exporter le projet avec un fichier `.env`
* Simuler un déploiement avec Docker ou un hébergeur comme Railway

**Exercices**

1. Créer l’API GET/POST/PUT/DELETE pour le modèle `Annonce`
2. Protéger la création via l’API avec un token
3. Générer un script de déploiement basique

---

# **Ressources complémentaires**

Documentation Django : [https://docs.djangoproject.com](https://docs.djangoproject.com)
Django REST Framework : [https://www.django-rest-framework.org](https://www.django-rest-framework.org)
Cheat Sheets : template tags Django, requêtes ORM
Bonnes pratiques : validation, sécurité, tests unitaires, organisation du code
