# Projet Django : LebonClone - Application d'annonces avec API REST

## Étape 1 : Initialisation du projet

### 1.1 Créer le dossier de projet et l'environnement virtuel
```bash
mkdir leBonClone
cd leBonClone
python -m venv venv
# Windows :
venv\Scripts\activate
# Linux/macOS :
source venv/bin/activate
```

### 1.2 Installer les dépendances
```bash
pip install django djangorestframework djangorestframework-simplejwt pillow
```

### 1.3 Créer le projet Django et l'application
```bash
django-admin startproject leBonClone .
python manage.py startapp annonces
```

## Étape 2 : Configuration de base

### 2.1 `leBonClone/settings.py`
```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'rest_framework',
    'annonces',
]

MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'

TEMPLATES[0]['DIRS'] = [BASE_DIR / 'templates']

AUTH_USER_MODEL = 'annonces.Utilisateur'

REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}
```

## Étape 3 : Modèles de données

### 3.1 `annonces/models.py`
```python
from django.db import models
from django.contrib.auth.models import AbstractUser

class Utilisateur(AbstractUser):
    telephone = models.CharField(max_length=20, blank=True)

class Categorie(models.Model):
    nom = models.CharField(max_length=100)

    def __str__(self):
        return self.nom

class Annonce(models.Model):
    titre = models.CharField(max_length=200)
    description = models.TextField()
    prix = models.DecimalField(max_digits=10, decimal_places=2)
    date_publication = models.DateTimeField(auto_now_add=True)
    image = models.ImageField(upload_to='annonces/', blank=True, null=True)
    categorie = models.ForeignKey(Categorie, on_delete=models.CASCADE)
    utilisateur = models.ForeignKey(Utilisateur, on_delete=models.CASCADE)

    def __str__(self):
        return self.titre
```

### 3.2 Migrer la base de données
```bash
python manage.py makemigrations
python manage.py migrate
```

## Étape 4 : Interface d'administration

### 4.1 `annonces/admin.py`
```python
from django.contrib import admin
from .models import Utilisateur, Categorie, Annonce
from django.contrib.auth.admin import UserAdmin

admin.site.register(Utilisateur, UserAdmin)
admin.site.register(Categorie)
admin.site.register(Annonce)
```

## Étape 5 : Création des templates

### 5.1 Arborescence
```
templates/
  base.html
  annonces/
    liste_annonces.html
    detail_annonce.html
    creer_annonce.html
```

### 5.2 Exemple `base.html`
```html
<!DOCTYPE html>
<html>
<head>
  <title>LebonClone</title>
</head>
<body>
  <h1>LebonClone</h1>
  {% block content %}{% endblock %}
</body>
</html>
```

### 5.3 Exemple `liste_annonces.html`
```html
{% extends 'base.html' %}
{% block content %}
<h2>Liste des annonces</h2>
{% for annonce in annonces %}
  <div>
    <h3><a href="{% url 'detail_annonce' annonce.id %}">{{ annonce.titre }}</a></h3>
    <p>{{ annonce.description|truncatewords:20 }}</p>
    <p>{{ annonce.prix }} €</p>
  </div>
{% empty %}<p>Aucune annonce.</p>{% endfor %}
<a href="{% url 'creer_annonce' %}">Créer une annonce</a>
{% endblock %}
```

## Étape 6 : Les vues

### 6.1 `annonces/views.py`
```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Annonce
from .forms import AnnonceForm

def liste_annonces(request):
    annonces = Annonce.objects.all()
    return render(request, 'annonces/liste_annonces.html', {'annonces': annonces})

def detail_annonce(request, id):
    annonce = get_object_or_404(Annonce, id=id)
    return render(request, 'annonces/detail_annonce.html', {'annonce': annonce})

def creer_annonce(request):
    if request.method == 'POST':
        form = AnnonceForm(request.POST, request.FILES)
        if form.is_valid():
            annonce = form.save(commit=False)
            annonce.utilisateur = request.user
            annonce.save()
            return redirect('liste_annonces')
    else:
        form = AnnonceForm()
    return render(request, 'annonces/creer_annonce.html', {'form': form})
```

## Étape 7 : Formulaires

### 7.1 `annonces/forms.py`
```python
from django import forms
from .models import Annonce

class AnnonceForm(forms.ModelForm):
    class Meta:
        model = Annonce
        fields = ['titre', 'description', 'prix', 'image', 'categorie']
```

## Étape 8 : Routage

### 8.1 `annonces/urls.py`
```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.liste_annonces, name='liste_annonces'),
    path('annonce/<int:id>/', views.detail_annonce, name='detail_annonce'),
    path('creer/', views.creer_annonce, name='creer_annonce'),
]
```

### 8.2 `leBonClone/urls.py`
```python
from django.contrib import admin
from django.urls import path, include
from django.conf import settings
from django.conf.urls.static import static
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('annonces.urls')),
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
    path('api/', include('annonces.api_urls')),
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

## Étape 9 : API REST avec DRF

### 9.1 `annonces/serializers.py`
```python
from rest_framework import serializers
from .models import Annonce, Categorie

class AnnonceSerializer(serializers.ModelSerializer):
    class Meta:
        model = Annonce
        fields = '__all__'

class CategorieSerializer(serializers.ModelSerializer):
    class Meta:
        model = Categorie
        fields = '__all__'
```

### 9.2 `annonces/api_views.py`
```python
from rest_framework import viewsets
from .models import Annonce, Categorie
from .serializers import AnnonceSerializer, CategorieSerializer
from rest_framework.permissions import IsAuthenticatedOrReadOnly

class AnnonceViewSet(viewsets.ModelViewSet):
    queryset = Annonce.objects.all()
    serializer_class = AnnonceSerializer
    permission_classes = [IsAuthenticatedOrReadOnly]

class CategorieViewSet(viewsets.ReadOnlyModelViewSet):
    queryset = Categorie.objects.all()
    serializer_class = CategorieSerializer
```

### 9.3 `annonces/api_urls.py`
```python
from rest_framework.routers import DefaultRouter
from .api_views import AnnonceViewSet, CategorieViewSet

router = DefaultRouter()
router.register('annonces', AnnonceViewSet)
router.register('categories', CategorieViewSet)

urlpatterns = router.urls
```

## Étape 10 : Générer des données fictives (fixtures)

### 10.1 Exemple `fixtures.json`
Créer le fichier `annonces/fixtures/annonces.json` :
```json
[
  {
    "model": "annonces.categorie",
    "pk": 1,
    "fields": {
      "nom": "Électronique"
    }
  },
  {
    "model": "annonces.utilisateur",
    "pk": 1,
    "fields": {
      "username": "admin",
      "password": "pbkdf2_sha256$...",  # Générer via createsuperuser
      "is_superuser": true,
      "is_staff": true
    }
  },
  {
    "model": "annonces.annonce",
    "pk": 1,
    "fields": {
      "titre": "iPhone 13",
      "description": "Neuf sous blister",
      "prix": "799.00",
      "categorie": 1,
      "utilisateur": 1,
      "date_publication": "2025-06-15T10:00:00Z"
    }
  }
]
```

### 10.2 Charger les données
```bash
python manage.py loaddata annonces/fixtures/annonces.json
```

## Étape 11 : Lancer l'application
```bash
python manage.py runserver
```


## Étape 15 : Test final

* Lance le serveur avec `python manage.py runserver`
* Va sur `http://127.0.0.1:8000/`
* Crée une annonce, consulte les détails
* Teste l’admin sur `/admin/`
