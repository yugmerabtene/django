# Guide Complet – Projet Django : LebonClone

## **Étape 1 – Création du projet Django**

### 1.1 Créer le dossier du projet et l’environnement virtuel

Dans ton terminal (ou PowerShell sous Windows) :

```bash
mkdir leBonClone
cd leBonClone
python -m venv venv
```

### 1.2 Activer l’environnement virtuel

* Sous **Windows** :

  ```bash
  venv\Scripts\activate
  ```

* Sous **Linux/macOS** :

  ```bash
  source venv/bin/activate
  ```

### 1.3 Installer les dépendances

```bash
pip install django djangorestframework djangorestframework-simplejwt pillow
```

---

## **Étape 2 – Initialiser le projet Django**

```bash
django-admin startproject leBonClone .
python manage.py startapp annonces
```

Cela crée deux dossiers :

* `leBonClone/` : le cœur du projet
* `annonces/` : l'application qui gère les annonces

---

## **Étape 3 – Configurer `settings.py`**

Dans `leBonClone/settings.py`, modifie :

### 3.1 Ajoute les apps :

```python
INSTALLED_APPS = [
    ...
    'rest_framework',
    'annonces',
]
```

### 3.2 Configure les templates :

```python
TEMPLATES[0]['DIRS'] = [BASE_DIR / 'templates']
```

### 3.3 Configure les médias :

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

### 3.4 Utilisateur personnalisé :

```python
AUTH_USER_MODEL = 'annonces.Utilisateur'
```

### 3.5 REST Framework + JWT :

```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    ]
}
```

---

## **Étape 4 – Modèles de base : `models.py`**

Dans `annonces/models.py` :

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

---

## **Étape 5 – Migration de la base**

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## **Étape 6 – Interface admin : `admin.py`**

```python
from django.contrib import admin
from .models import Utilisateur, Categorie, Annonce
from django.contrib.auth.admin import UserAdmin

admin.site.register(Utilisateur, UserAdmin)
admin.site.register(Categorie)
admin.site.register(Annonce)
```

---

## **Étape 7 – Création manuelle de l'utilisateur admin**

```bash
python manage.py createsuperuser
```

Saisis : **username**, **email**, **mot de passe**.

---

## **Étape 8 – Templates HTML**

### Structure :

```
templates/
  base.html
  annonces/
    liste_annonces.html
    detail_annonce.html
    creer_annonce.html
```

### `templates/base.html`

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

### `templates/annonces/liste_annonces.html`

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

---

## **Étape 9 – Les vues : `views.py`**

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

---

## **Étape 10 – Les formulaires : `forms.py`**

```python
from django import forms
from .models import Annonce

class AnnonceForm(forms.ModelForm):
    class Meta:
        model = Annonce
        fields = ['titre', 'description', 'prix', 'image', 'categorie']
```

---

## **Étape 11 – Routage**

### `annonces/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.liste_annonces, name='liste_annonces'),
    path('annonce/<int:id>/', views.detail_annonce, name='detail_annonce'),
    path('creer/', views.creer_annonce, name='creer_annonce'),
]
```

### `leBonClone/urls.py`

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
] + static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## **Étape 12 – API REST**

### `annonces/serializers.py`

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

### `annonces/api_views.py`

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

### `annonces/api_urls.py`

```python
from rest_framework.routers import DefaultRouter
from .api_views import AnnonceViewSet, CategorieViewSet

router = DefaultRouter()
router.register('annonces', AnnonceViewSet)
router.register('categories', CategorieViewSet)

urlpatterns = router.urls
```

---

## **Étape 13 – Fixtures de test**

### `annonces/fixtures/annonces.json`

```json
[
  {
    "model": "annonces.categorie",
    "pk": 1,
    "fields": { "nom": "Informatique" }
  },
  {
    "model": "annonces.annonce",
    "pk": 1,
    "fields": {
      "titre": "PC Gamer Ryzen",
      "description": "Tour neuve, très puissante",
      "prix": "1200.00",
      "categorie": 1,
      "utilisateur": 1,
      "date_publication": "2025-06-15T14:00:00Z"
    }
  }
]
```

### Commande :

```bash
python manage.py loaddata annonces/fixtures/annonces.json
```

---

## **Étape 14 – Lancer le serveur**

```bash
python manage.py runserver
```

---

## **Étape 15 – Test final**

* Va sur : `http://127.0.0.1:8000/`
* Crée une annonce
* Clique dessus pour consulter les détails
* Va sur `/admin` pour gérer les utilisateurs, annonces, catégories

---

**Aller plus loin :**

* ajouter la **pagination, la recherche, le filtrage par catégorie** 
* créer des **tests unitaires** 
* Créer un endpoint crossplateform avec ReactNative 
