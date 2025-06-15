# **Chapitre 3 – Jour 3 : Vues, Templates, Formulaires et Routage**

---

## **1. Objectifs pédagogiques**

À l’issue de cette journée, l’apprenant sera capable de :

* Comprendre l’organisation logique d’une application Django côté affichage
* Créer des vues simples (FBV – Function-Based Views)
* Lier les vues aux URLs
* Utiliser les templates Django avec héritage, conditions, boucles
* Gérer des formulaires avec `Form` et `ModelForm`
* Valider les champs de manière dynamique et afficher les erreurs à l’utilisateur

---

## **2. Organisation d’une application Django**

### 2.1. Fichiers concernés

* `views.py` : contient les fonctions ou classes appelées pour chaque route
* `urls.py` : associe chaque URL à une vue
* `templates/` : contient les fichiers HTML liés aux vues
* `forms.py` : contient la logique des formulaires (on y revient plus loin)

### 2.2. Exemple de structure

```
annonces/
    views.py
    urls.py
    templates/
        annonces/
            base.html
            liste_annonces.html
            detail_annonce.html
            creer_annonce.html
```

---

## **3. Vues Django (Function-Based Views)**

### 3.1. Vue simple

```python
# annonces/views.py
from django.http import HttpResponse

def home(request):
    return HttpResponse("Bienvenue sur LeBonClone !")
```

### 3.2. Vue avec render() et template

```python
from django.shortcuts import render

def accueil(request):
    return render(request, 'annonces/accueil.html')
```

---

## **4. Syntaxe des templates Django**

### 4.1. Structure avec héritage

#### base.html

```html
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}LeBonClone{% endblock %}</title>
</head>
<body>
    <h1>LeBonClone</h1>
    {% block content %}{% endblock %}
</body>
</html>
```

#### liste\_annonces.html

```html
{% extends 'annonces/base.html' %}

{% block title %}Annonces{% endblock %}

{% block content %}
  <h2>Liste des annonces</h2>
  <ul>
    {% for annonce in annonces %}
      <li>
        <a href="{% url 'detail_annonce' annonce.id %}">{{ annonce.titre }}</a> - {{ annonce.prix }} €
      </li>
    {% empty %}
      <li>Aucune annonce trouvée.</li>
    {% endfor %}
  </ul>
{% endblock %}
```

---

## **5. Routage avec `urls.py`**

### 5.1. Inclusion dans le projet principal

`lebonclone/urls.py`

```python
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', include('annonces.urls')),
]
```

### 5.2. URLs de l’app `annonces`

`annonces/urls.py`

```python
from django.urls import path
from . import views

urlpatterns = [
    path('', views.liste_annonces, name='liste_annonces'),
    path('annonce/<int:id>/', views.detail_annonce, name='detail_annonce'),
    path('creer/', views.creer_annonce, name='creer_annonce'),
]
```

---

## **6. Affichage de la liste des annonces**

```python
# annonces/views.py
from django.shortcuts import render
from .models import Annonce

def liste_annonces(request):
    annonces = Annonce.objects.all().order_by('-date_publication')
    return render(request, 'annonces/liste_annonces.html', {'annonces': annonces})
```

---

## **7. Affichage du détail d’une annonce**

```python
# annonces/views.py
from django.shortcuts import get_object_or_404

def detail_annonce(request, id):
    annonce = get_object_or_404(Annonce, pk=id)
    return render(request, 'annonces/detail_annonce.html', {'annonce': annonce})
```

`detail_annonce.html`

```html
{% extends 'annonces/base.html' %}

{% block content %}
  <h2>{{ annonce.titre }}</h2>
  <p>{{ annonce.description }}</p>
  <p>Prix : {{ annonce.prix }} €</p>
  {% if annonce.image %}
    <img src="{{ annonce.image.url }}" alt="Image de l'annonce" />
  {% endif %}
{% endblock %}
```

---

## **8. Formulaires avec `ModelForm`**

### 8.1. Formulaire dans `forms.py`

```python
# annonces/forms.py
from django import forms
from .models import Annonce

class AnnonceForm(forms.ModelForm):
    class Meta:
        model = Annonce
        fields = ['titre', 'description', 'prix', 'categorie', 'image']
```

---

### 8.2. Vue `creer_annonce`

```python
from django.contrib.auth.decorators import login_required
from django.shortcuts import redirect
from .forms import AnnonceForm

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

---

### 8.3. Template `creer_annonce.html`

```html
{% extends 'annonces/base.html' %}

{% block content %}
  <h2>Déposer une annonce</h2>
  <form method="POST" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Publier</button>
  </form>
{% endblock %}
```

---

## **9. Validation dynamique et erreurs**

Django affiche automatiquement les erreurs de validation dans `form.as_p`. Pour un contrôle personnalisé :

```html
{% for field in form %}
  <div>
    {{ field.label_tag }} {{ field }}
    {% if field.errors %}
      <div style="color: red">{{ field.errors }}</div>
    {% endif %}
  </div>
{% endfor %}
```

---

## **10. Exercices corrigés**

### Exercice 1 – Afficher toutes les annonces

Déjà couvert avec la vue `liste_annonces` et le template `liste_annonces.html`

### Exercice 2 – Créer une page de création d’annonce

Fait avec la vue `creer_annonce` + formulaire `AnnonceForm` + template `creer_annonce.html`

### Exercice 3 – Validation dynamique

* Supprimer un champ requis dans le formulaire
* Vérifier que l’erreur s’affiche dans le navigateur
* Afficher manuellement les erreurs par champ avec `field.errors`
