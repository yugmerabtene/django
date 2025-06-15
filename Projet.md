# Création du projet Django `lebonclone` avec PyCharm

---

## Étape 1 : Préparer ton environnement

1. Installer **Python 3.10+** si ce n’est pas déjà fait
   (vérifie avec `python --version`)

2. Installer **PyCharm Community ou Professional**

3. Créer un **nouveau projet Django** dans PyCharm

   * Ouvre PyCharm > New Project
   * Choisis **Django** comme type de projet
   * Coche "Create a new environment using venv"
   * Nom du projet : `lebonclone`
   * Décoche "Enable Django admin" si tu veux tout faire manuellement (facultatif)
   * Valide

---

## Étape 2 : Créer le projet Django (si tu ne l’as pas fait via l’assistant)

Si tu es dans un projet vide :

```bash
django-admin startproject lebonclone .
```

Cela va créer les fichiers :

* manage.py
* lebonclone/settings.py
* lebonclone/urls.py

---

## Étape 3 : Vérifier que le projet fonctionne

Dans le terminal de PyCharm :

```bash
python manage.py runserver
```

Puis ouvre le navigateur sur [http://127.0.0.1:8000](http://127.0.0.1:8000)
Tu dois voir "The install worked successfully!"

---

## Étape 4 : Créer l’application principale `annonces`

```bash
python manage.py startapp annonces
```

Puis ajoute `"annonces"` dans `INSTALLED_APPS` de `lebonclone/settings.py` :

```python
INSTALLED_APPS = [
    ...
    'annonces',
]
```

---

## Étape 5 : Configurer la base de données

Par défaut, Django utilise **SQLite**. Tu n’as rien à faire si tu veux garder cette configuration simple :

```python
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.sqlite3',
        'NAME': BASE_DIR / "db.sqlite3",
    }
}
```

## Étape 6 : Créer les modèles

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

## Étape 7 : Indiquer à Django d’utiliser ton modèle utilisateur personnalisé

Dans `lebonclone/settings.py` :

```python
AUTH_USER_MODEL = 'annonces.Utilisateur'
```

---

## Étape 8 : Appliquer les migrations

```bash
python manage.py makemigrations
python manage.py migrate
```

---

## Étape 9 : Créer un superutilisateur pour l’administration

```bash
python manage.py createsuperuser
```

---

## Étape 10 : Enregistrer les modèles dans l’admin

Dans `annonces/admin.py` :

```python
from django.contrib import admin
from django.contrib.auth.admin import UserAdmin
from .models import Utilisateur, Annonce, Categorie

@admin.register(Utilisateur)
class UtilisateurAdmin(UserAdmin):
    model = Utilisateur
    list_display = ['username', 'email', 'telephone']

admin.site.register(Annonce)
admin.site.register(Categorie)
```

---

## Étape 11 : Créer les templates de base

Créer un dossier `templates/annonces/`
Créer `base.html`, `liste_annonces.html`, `detail_annonce.html`, `creer_annonce.html`

Exemple de `base.html` :

```html
<!DOCTYPE html>
<html>
<head>
    <title>Lebonclone</title>
</head>
<body>
    <h1>Lebonclone</h1>
    {% block content %}{% endblock %}
</body>
</html>
```

---

## Étape 12 : Vues et URLs

Dans `annonces/views.py` :

```python
from django.shortcuts import render, get_object_or_404, redirect
from .models import Annonce
from .forms import AnnonceForm
from django.contrib.auth.decorators import login_required

def liste_annonces(request):
    annonces = Annonce.objects.all()
    return render(request, 'annonces/liste_annonces.html', {'annonces': annonces})

def detail_annonce(request, id):
    annonce = get_object_or_404(Annonce, pk=id)
    return render(request, 'annonces/detail_annonce.html', {'annonce': annonce})

@login_required
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

Dans `annonces/urls.py` :

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.liste_annonces, name='liste_annonces'),
    path('annonce/<int:id>/', views.detail_annonce, name='detail_annonce'),
    path('creer/', views.creer_annonce, name='creer_annonce'),
]
```

Et dans `lebonclone/urls.py` :

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('annonces.urls')),
]
```

---

## Étape 13 : Formulaire

Dans `annonces/forms.py` :

```python
from django import forms
from .models import Annonce

class AnnonceForm(forms.ModelForm):
    class Meta:
        model = Annonce
        fields = ['titre', 'description', 'prix', 'categorie', 'image']
```

---

## Étape 14 : Configuration des fichiers médias

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

## Étape 15 : Test final

* Lance le serveur avec `python manage.py runserver`
* Va sur `http://127.0.0.1:8000/`
* Crée une annonce, consulte les détails
* Teste l’admin sur `/admin/`
