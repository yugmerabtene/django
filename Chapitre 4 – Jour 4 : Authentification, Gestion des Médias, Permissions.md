# **Chapitre 4 – Jour 4 : Authentification, Gestion des Médias, Permissions**

---

## **1. Objectifs pédagogiques**

À la fin de cette journée, l’apprenant sera capable de :

* Gérer l’inscription, la connexion et la déconnexion des utilisateurs
* Restreindre certaines pages aux utilisateurs connectés
* Ajouter un champ personnalisé à l’utilisateur
* Permettre l’upload d’images pour les annonces
* Servir et sécuriser les fichiers médias en mode développement

---

## **2. Système d’authentification Django**

Django intègre un système complet de gestion des utilisateurs :

* Modèle `User` ou modèle personnalisé (`AbstractUser`)
* Système de sessions avec login/logout
* Décorateur `@login_required` pour sécuriser l’accès

---

## **3. Création d’un formulaire d’inscription**

### 3.1. Formulaire d’inscription

```python
# comptes/forms.py
from django import forms
from django.contrib.auth.forms import UserCreationForm
from comptes.models import Utilisateur

class InscriptionForm(UserCreationForm):
    class Meta:
        model = Utilisateur
        fields = ['username', 'email', 'telephone', 'password1', 'password2']
```

---

### 3.2. Vue d’inscription

```python
# comptes/views.py
from django.shortcuts import render, redirect
from .forms import InscriptionForm

def inscription(request):
    if request.method == 'POST':
        form = InscriptionForm(request.POST)
        if form.is_valid():
            form.save()
            return redirect('connexion')
    else:
        form = InscriptionForm()
    return render(request, 'comptes/inscription.html', {'form': form})
```

---

### 3.3. Template d’inscription

```html
{% extends 'annonces/base.html' %}

{% block content %}
<h2>Créer un compte</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">S’inscrire</button>
</form>
{% endblock %}
```

---

## **4. Connexion et déconnexion**

### 4.1. Vue de connexion

```python
from django.contrib.auth.views import LoginView

class ConnexionView(LoginView):
    template_name = 'comptes/connexion.html'
```

---

### 4.2. Template de connexion

```html
{% extends 'annonces/base.html' %}

{% block content %}
<h2>Connexion</h2>
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Se connecter</button>
</form>
{% endblock %}
```

---

### 4.3. Vue de déconnexion

```python
from django.contrib.auth.views import LogoutView

class DeconnexionView(LogoutView):
    next_page = '/'
```

---

### 4.4. URLs d’authentification

```python
# comptes/urls.py
from django.urls import path
from .views import inscription, ConnexionView, DeconnexionView

urlpatterns = [
    path('inscription/', inscription, name='inscription'),
    path('connexion/', ConnexionView.as_view(), name='connexion'),
    path('deconnexion/', DeconnexionView.as_view(), name='deconnexion'),
]
```

À inclure dans `lebonclone/urls.py` :

```python
path('comptes/', include('comptes.urls')),
```

---

## **5. Restriction d’accès avec `@login_required`**

```python
from django.contrib.auth.decorators import login_required

@login_required
def creer_annonce(request):
    ...
```

Dans le template, tu peux tester :

```html
{% if request.user.is_authenticated %}
  Bonjour {{ request.user.username }} | <a href="{% url 'deconnexion' %}">Déconnexion</a>
{% else %}
  <a href="{% url 'connexion' %}">Connexion</a> | <a href="{% url 'inscription' %}">Inscription</a>
{% endif %}
```

---

## **6. Gestion des fichiers médias (image des annonces)**

### 6.1. Dans `models.py` (rappel)

```python
image = models.ImageField(upload_to='annonces/', blank=True, null=True)
```

### 6.2. Dans `settings.py`

```python
MEDIA_URL = '/media/'
MEDIA_ROOT = BASE_DIR / 'media'
```

### 6.3. Dans `urls.py`

```python
from django.conf import settings
from django.conf.urls.static import static

urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

---

## **7. Affichage de l’image dans le template**

```html
{% if annonce.image %}
  <img src="{{ annonce.image.url }}" alt="{{ annonce.titre }}" width="300">
{% endif %}
```

---

## **8. Travaux pratiques**

Créer une app `comptes` si ce n’est pas fait
Créer les vues et templates pour :

* Inscription
* Connexion
* Déconnexion

Protéger l'accès à `creer_annonce` avec `@login_required`
Afficher l’état de connexion dans `base.html`
Uploader une image via le formulaire et l’afficher dans la fiche annonce

---

## **9. Exercices corrigés**

### Exercice 1 – Forcer la connexion

* Tester l’accès direct à `/creer/` sans être connecté → redirection
* Ajouter un bouton “Ajouter une annonce” visible uniquement si connecté

### Exercice 2 – Ajouter un champ téléphone à l’utilisateur

Déjà fait via `AbstractUser`, s’assurer qu’il s’affiche dans le formulaire et dans l’admin

### Exercice 3 – Uploader une image

* Tester le champ `image` dans `AnnonceForm`
* S’assurer que le fichier est bien stocké dans `/media/annonces/`

