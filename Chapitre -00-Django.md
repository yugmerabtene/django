**1. Définition rapide**

| Aspect   | MVC (Model - View - Controller)                                                           | MTV (Model - Template - View)                    |
| -------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------ |
| Origine  | Architecture générale utilisée dans de nombreux frameworks (comme Laravel, Spring, Rails) | Variante spécifique du MVC utilisée par Django   |
| Objectif | Séparer la logique métier, l'interface utilisateur et le contrôle des actions             | Même but, mais adapté à la philosophie de Django |

---

**2. Correspondances entre composants**

| Composant MVC | Équivalent MTV     | Rôle                                                                                                                           |
| ------------- | ------------------ | ------------------------------------------------------------------------------------------------------------------------------ |
| Model         | Model              | Gère la logique métier, la base de données et les règles métier                                                                |
| View          | Template           | Affiche les données à l'utilisateur final, interface HTML                                                                      |
| Controller    | View (dans Django) | Gère la logique de traitement : reçoit les requêtes, interagit avec les modèles, envoie une réponse (souvent un template HTML) |

Remarque :
Dans Django, la "View" correspond au contrôleur du flux (fonction ou classe Python), tandis que ce qui est affiché est appelé "Template".

---

**3. Fonctionnement résumé**

MVC classique (ex : Laravel, Spring)
L’utilisateur fait une requête.
Le Controller la traite, utilise le Model pour accéder aux données, puis choisit une View à afficher.

MTV (Django)
L’utilisateur fait une requête.
La View (Django) la traite (fonction ou classe Python), interagit avec le Model, puis utilise un Template pour afficher les données.

---

**4. Exemple concret**

Supposons une application qui affiche une liste de produits.

| Élément                  | MVC                              | MTV (Django)                                  |
| ------------------------ | -------------------------------- | --------------------------------------------- |
| Modèle                   | Product.java (Spring)            | models.py avec classe Product                 |
| Contrôleur / Vue logique | ProductController.java (Spring)  | views.py avec def product\_list(request)      |
| Vue / Interface          | product.html (Thymeleaf, Blade…) | product\_list.html dans le dossier templates/ |

---

**5. Avantages et limites**

| Critère                       | MVC                                     | MTV                                                         |
| ----------------------------- | --------------------------------------- | ----------------------------------------------------------- |
| Clarté des rôles              | Séparation explicite                    | Moins clair pour les débutants à cause du nom "View"        |
| Adoption                      | Utilisé dans de nombreux langages       | Spécifique à Django                                         |
| Flexibilité                   | Plus adaptable à différents cas d’usage | Fortement structuré par Django                              |
| Simplicité pour les débutants | Peut être plus complexe à comprendre    | Plus simple avec l’approche de Django et son automatisation |

---

**En résumé**

MVC est un paradigme universel, adopté par de nombreux frameworks.
MTV est une adaptation du MVC pour Django, où :

* la View = contrôleur
* le Template = interface utilisateur

MTV n’est pas fondamentalement différent, c’est une réinterprétation du MVC avec des noms adaptés à la structure Django.

----

## **1. `manage.py`**

**Rôle :**
C’est le **point d’entrée principal** pour interagir avec ton projet Django via la ligne de commande.

**Fonctions principales :**

* Lancer le serveur de développement (`python manage.py runserver`)
* Appliquer des migrations (`python manage.py migrate`)
* Créer une application (`python manage.py startapp nom_app`)
* Accéder à la console interactive Django (`python manage.py shell`)

**Contenu typique :**

```python
from django.core.management import execute_from_command_line
```

Il importe et lance les commandes selon les paramètres que tu passes.

---

## **2. `settings.py`**

**Rôle :**
Contient **toute la configuration** du projet Django.

**Principaux éléments :**

* `INSTALLED_APPS` : liste des applications activées (ex. `django.contrib.admin`, `ton_app`)
* `DATABASES` : configuration de la base de données (SQLite, PostgreSQL, etc.)
* `TEMPLATES` : paramètres pour le moteur de template (HTML)
* `MIDDLEWARE` : liste des middlewares (fonctions qui s’exécutent avant/après chaque requête)
* `STATIC_URL`, `MEDIA_URL` : gestion des fichiers statiques et médias
* `DEBUG` : active ou désactive le mode debug
* `ALLOWED_HOSTS` : liste des hôtes autorisés (en prod, obligatoire)

---

## **3. `urls.py`**

**Rôle :**
C’est le **routeur principal** du projet. Il fait le lien entre les URL et les vues.

**Contenu typique :**

```python
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('produits/', include('produits.urls')),
]
```

**Fonctions :**

* Associe une URL à une vue (fonction ou classe)
* Peut déléguer vers les `urls.py` des applications internes

---

## **4. `wsgi.py`**

**Rôle :**
Fichier d’entrée pour les serveurs **WSGI** en production (comme Gunicorn, uWSGI, mod\_wsgi).

**Fonctions :**

* Prépare l'application Django à être exécutée par un serveur web
* Sert d’interface entre le serveur et Django

**Contenu typique :**

```python
from django.core.wsgi import get_wsgi_application

application = get_wsgi_application()
```

C’est ce fichier qui sera utilisé quand tu héberges ton site sur un serveur web (ex : Apache, Nginx + Gunicorn).

---

## **Résumé**

| Fichier       | Rôle                                                        |
| ------------- | ----------------------------------------------------------- |
| `manage.py`   | Lancer des commandes Django (dev, migration, test…)         |
| `settings.py` | Configuration globale du projet                             |
| `urls.py`     | Table de routage principale (URL vers vues)                 |
| `wsgi.py`     | Point d’entrée WSGI pour les déploiements web en production |


