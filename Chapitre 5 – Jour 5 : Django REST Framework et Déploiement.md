# **Chapitre 5 – Jour 5 : API REST avec Django REST Framework et Déploiement**

---

## **1. Objectifs pédagogiques**

À l’issue de cette journée, l’apprenant sera capable de :

* Créer une API REST avec Django REST Framework (DRF)
* Sérialiser les données des modèles
* Exposer des endpoints GET, POST, PUT, DELETE
* Protéger l’API avec des permissions et de l’authentification
* Préparer et déployer une application Django sur un serveur distant (Railway, Render, VPS)

---

## **2. Introduction à Django REST Framework**

Django REST Framework (DRF) est une extension de Django qui permet de créer facilement des APIs RESTful.

### Pourquoi DRF ?

* Sérialisation automatique des modèles
* Vues génériques (ListAPIView, RetrieveAPIView, etc.)
* Browsable API (interface web pour tester)
* Gestion native des permissions, authentification, pagination, throttling

---

## **3. Installation de DRF**

```bash
pip install djangorestframework
```

Ajouter à `settings.py` :

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
]
```

---

## **4. Création d’un Serializer**

```python
# annonces/serializers.py
from rest_framework import serializers
from .models import Annonce

class AnnonceSerializer(serializers.ModelSerializer):
    class Meta:
        model = Annonce
        fields = '__all__'
```

---

## **5. Vues API avec ViewSets**

```python
# annonces/api_views.py
from rest_framework import viewsets
from .models import Annonce
from .serializers import AnnonceSerializer
from rest_framework.permissions import IsAuthenticatedOrReadOnly

class AnnonceViewSet(viewsets.ModelViewSet):
    queryset = Annonce.objects.all().order_by('-date_publication')
    serializer_class = AnnonceSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]
```

---

## **6. Router pour les URL automatiques**

```python
# annonces/api_urls.py
from rest_framework.routers import DefaultRouter
from .api_views import AnnonceViewSet

router = DefaultRouter()
router.register('annonces', AnnonceViewSet)

urlpatterns = router.urls
```

À inclure dans `lebonclone/urls.py` :

```python
path('api/', include('annonces.api_urls')),
```

---

## **7. Authentification avec Token**

```bash
pip install djangorestframework-simplejwt
```

Ajouter à `settings.py` :

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ),
}
```

---

## **8. Endpoints API**

Avec `ModelViewSet`, tu obtiens automatiquement :

* `GET /api/annonces/` : liste
* `GET /api/annonces/1/` : détail
* `POST /api/annonces/` : créer
* `PUT /api/annonces/1/` : mettre à jour
* `DELETE /api/annonces/1/` : supprimer

---

## **9. Test de l’API dans le navigateur**

Visite `/api/annonces/` pour explorer l’API
Utilise l’interface web ou Postman pour tester les requêtes avec headers et tokens

---

## **10. Préparer le projet pour la production**

### Fichiers indispensables

* `requirements.txt`

```bash
pip freeze > requirements.txt
```

* `Procfile` (pour Railway/Render/Heroku)

```
web: gunicorn lebonclone.wsgi
```

* `.env` (variables de prod : secret key, DB...)

### Paramètres `settings.py`

```python
DEBUG = False
ALLOWED_HOSTS = ['tonapp.render.com']
SECRET_KEY = os.getenv('SECRET_KEY')
```

---

## **11. Déploiement sur Railway (exemple)**

1. Crée un compte sur [https://railway.app](https://railway.app)

2. Clique sur “New Project” > “Deploy from GitHub”

3. Ajoute les variables d’environnement nécessaires :

   * `DEBUG = false`
   * `SECRET_KEY = ...`
   * `DATABASE_URL = ...`

4. Railway détecte automatiquement Django, installe les dépendances, et lance `gunicorn`

---

## **12. Exercices corrigés**

### Exercice 1 – Créer une API CRUD

Créer un `AnnonceViewSet`, un `AnnonceSerializer`, et exposer toutes les routes.

### Exercice 2 – Protéger l’API

Limiter la création/modification/suppression aux utilisateurs authentifiés (`IsAuthenticatedOrReadOnly`).

### Exercice 3 – Exporter le projet pour déploiement

Générer les fichiers :

* `requirements.txt`
* `Procfile`
* `.env.example`

Faire un test de build local avec :

```bash
gunicorn lebonclone.wsgi
```
