Un **QuerySet** en Django est **un ensemble de requêtes vers la base de données**, représentant **une collection d’objets d’un modèle**. C’est l’élément central du **Django ORM** (Object-Relational Mapping) qui permet de manipuler les données de la base **en Python**, sans écrire de SQL.

---

## **Définition simple**

> Un **QuerySet** est une **liste paresseuse d’objets Django** issue d’une requête vers la base de données. Elle peut être **filtrée, ordonnée, combinée, ou exécutée**.

---

## **Caractéristiques principales**

### 1. **Paresseux (lazy loading)**

Le QuerySet **ne déclenche pas immédiatement la requête SQL**. Elle ne s’exécute **que quand tu accèdes aux données** :

```python
annonces = Annonce.objects.all()  # pas encore de requête SQL
for a in annonces:                # ici la requête s'exécute
    print(a.titre)
```

### 2. **Chaînable**

Tu peux chaîner plusieurs filtres ou opérations :

```python
Annonce.objects.filter(categorie="Immobilier").order_by('-date_publication')
```

### 3. **Filtrable avec `.filter()`**

Permet de filtrer les objets :

```python
Annonce.objects.filter(prix__lt=1000)  # prix inférieur à 1000 €
```

### 4. **Indexable et itérable**

```python
annonces[0]         # premier objet
for a in annonces:  # boucle sur tous les objets
    ...
```

---

## **Méthodes courantes sur un QuerySet**

| Méthode                | Description                                       |
| ---------------------- | ------------------------------------------------- |
| `.all()`               | Renvoie tous les objets du modèle                 |
| `.filter(**kwargs)`    | Filtre les objets selon les critères              |
| `.exclude(**kwargs)`   | Exclut les objets selon les critères              |
| `.get(**kwargs)`       | Renvoie un seul objet (lève une exception si >1)  |
| `.order_by('champ')`   | Trie les résultats                                |
| `.count()`             | Compte le nombre d’objets                         |
| `.first()` / `.last()` | Renvoie le premier / dernier objet                |
| `.exists()`            | Renvoie `True` si au moins un résultat            |
| `.values()`            | Renvoie un dictionnaire au lieu d’un objet Python |
| `.distinct()`          | Évite les doublons (utile avec des jointures)     |

---

## **Exemples concrets**

### Récupérer toutes les annonces :

```python
Annonce.objects.all()
```

### Récupérer toutes les annonces de la catégorie "Véhicules" :

```python
Annonce.objects.filter(categorie__nom="Véhicules")
```

### Compter toutes les annonces publiées par un utilisateur :

```python
Annonce.objects.filter(utilisateur__username="admin").count()
```

---

## **QuerySet vs. liste Python**

Un `QuerySet` ressemble à une liste, mais **c’est un objet spécial** :

* Il représente une **requête préparée**, mais non exécutée
* Il permet de faire des **requêtes SQL performantes**
* Il consomme moins de mémoire (streaming partiel possible)

---

## **1. Créer des QuerySet personnalisés avec `QuerySet.as_manager()`**

### Objectif

Tu veux créer **des méthodes de requêtage personnalisées** directement utilisables comme :

```python
Annonce.objects.publiques()
Annonce.objects.recentes()
```

### Étapes

### a. Crée une classe personnalisée héritant de `models.QuerySet`

```python
from django.db import models
from django.utils.timezone import now, timedelta

class AnnonceQuerySet(models.QuerySet):
    def publiques(self):
        return self.filter(est_publique=True)

    def recentes(self):
        return self.filter(date_publication__gte=now() - timedelta(days=7))
```

---

### b. Dans ton modèle, lie ce QuerySet à l’attribut `objects` via `as_manager()`

```python
class Annonce(models.Model):
    titre = models.CharField(max_length=255)
    est_publique = models.BooleanField(default=False)
    date_publication = models.DateTimeField(auto_now_add=True)

    objects = AnnonceQuerySet.as_manager()
```

---

### c. Utilisation dans ton code

```python
Annonce.objects.publiques()
Annonce.objects.recentes()
Annonce.objects.publiques().recentes()
```

Tu peux chaîner les méthodes comme si c’était des `filter()`.

---

## **2. Différence entre `QuerySet`, `Manager` et `Model Manager`**

| Élément                          | Rôle principal                                                      | Exemple                               |
| -------------------------------- | ------------------------------------------------------------------- | ------------------------------------- |
| `QuerySet`                       | Représente une requête SQL — itérable, chaînable, paresseux         | `Annonce.objects.filter(...).count()` |
| `Manager`                        | Point d’entrée pour accéder aux requêtes (souvent appelé `objects`) | `Annonce.objects.all()`               |
| `Model Manager` (custom Manager) | Un `Manager` personnalisé avec des méthodes propres                 | `Annonce.objects.validees()`          |

---

### Explication claire :

### a. **QuerySet**

Contient **la logique de filtrage**, pagination, ordering, etc.
Il est **retourné par le Manager**.
On peut en créer un personnalisé pour y centraliser les règles de filtrage métier.

Exemple :

```python
class AnnonceQuerySet(models.QuerySet):
    def actives(self):
        return self.filter(est_active=True)
```

---

### b. **Manager**

C’est **le point d’entrée principal** de l’ORM :
`objects = models.Manager()` (par défaut)
Tu peux définir des méthodes de haut niveau comme :

```python
class AnnonceManager(models.Manager):
    def valides(self):
        return self.get_queryset().filter(est_valide=True)
```

---

### c. **Combiner Manager + QuerySet personnalisé**

C’est le plus puissant :

```python
class AnnonceQuerySet(models.QuerySet):
    def actives(self):
        return self.filter(est_active=True)

class AnnonceManager(models.Manager):
    def get_queryset(self):
        return AnnonceQuerySet(self.model, using=self._db)

    def actives(self):
        return self.get_queryset().actives()
```

Dans le modèle :

```python
class Annonce(models.Model):
    ...
    objects = AnnonceManager()
```

Avantage : tu peux utiliser :

```python
Annonce.objects.actives()
Annonce.objects.actives().filter(prix__lt=100)
```

---

## Résumé visuel

```
Annonce (Model)
│
├── objects (Manager) ────────► AnnonceQuerySet (QuerySet personnalisé)
│                               ├── filter()
│                               ├── order_by()
│                               └── actives(), recentes(), etc.
```

---
