## 1. **Qu’est-ce qu’une API ?**

API signifie **Application Programming Interface**.
Dans le contexte du web, une **API web** est une **interface qui permet à des applications (frontends, apps mobiles, autres serveurs…) de communiquer avec ton serveur via HTTP**, sans passer par des pages HTML.

### Exemple concret :

Une API peut retourner les annonces d’un site comme LeBonCoin sous forme de données brutes JSON :

```json
[
  {
    "id": 1,
    "titre": "Peugeot 208",
    "prix": 6200
  },
  ...
]
```

---

## 2. **Qu’est-ce que Django REST Framework (DRF) ?**

**Django REST Framework**, souvent abrégé **DRF**, est une **extension de Django** spécialement conçue pour **créer des APIs RESTful** de manière propre, puissante et rapide.

### Pourquoi l’utiliser ?

* Django est parfait pour faire des sites HTML (MVC/MTV)
* Mais quand on veut **exposer une API JSON**, il faut :

  * Sérialiser les modèles (`Model → JSON`)
  * Gérer les permissions, l'authentification, la pagination, les erreurs
  * Répondre proprement aux requêtes GET, POST, PUT, DELETE

DRF s’occupe de tout cela **avec des outils puissants et des conventions bien pensées**.

---

## 3. **Composants principaux de DRF**

| Élément DRF       | Rôle                                                      |
| ----------------- | --------------------------------------------------------- |
| **Serializer**    | Transforme un objet Python ou un QuerySet en JSON         |
| **ViewSet**       | Fournit automatiquement les routes GET, POST, PUT, DELETE |
| **Router**        | Génère automatiquement les URL à partir du ViewSet        |
| **Permissions**   | Contrôle qui peut faire quoi (authentifié, staff, etc.)   |
| **Browsable API** | Interface web automatique pour tester ton API             |

---

## 4. **Exemple concret avec DRF**

### Modèle :

```python
class Annonce(models.Model):
    titre = models.CharField(max_length=100)
    prix = models.DecimalField(max_digits=10, decimal_places=2)
```

### Serializer :

```python
from rest_framework import serializers

class AnnonceSerializer(serializers.ModelSerializer):
    class Meta:
        model = Annonce
        fields = '__all__'
```

### ViewSet :

```python
from rest_framework import viewsets
from .models import Annonce

class AnnonceViewSet(viewsets.ModelViewSet):
    queryset = Annonce.objects.all()
    serializer_class = AnnonceSerializer
```

### URLs (avec un Router) :

```python
from rest_framework.routers import DefaultRouter
from .views import AnnonceViewSet

router = DefaultRouter()
router.register('annonces', AnnonceViewSet)

urlpatterns = router.urls
```

Résultat : tu obtiens automatiquement les routes suivantes :

* `GET /api/annonces/` → liste JSON
* `POST /api/annonces/` → ajouter une annonce
* `GET /api/annonces/1/` → détail
* `PUT /api/annonces/1/` → modifier
* `DELETE /api/annonces/1/` → supprimer

---

## 5. **Pourquoi c’est utile ?**

Une API (avec DRF) est essentielle pour :

* Une application mobile (Flutter, React Native, etc.)
* Un frontend JS (React, Vue) qui consomme des données via AJAX/Fetch
* Des automatisations avec des systèmes tiers
* Des services publics exposés (comme l’API de transport, météo, etc.)

---

## 6. **API REST : définition rapide**

DRF est basé sur le **style REST** :

| Action utilisateur | Méthode HTTP | Exemple route      |
| ------------------ | ------------ | ------------------ |
| Lire une liste     | GET          | `/api/annonces/`   |
| Lire un objet      | GET          | `/api/annonces/1/` |
| Créer un objet     | POST         | `/api/annonces/`   |
| Modifier un objet  | PUT/PATCH    | `/api/annonces/1/` |
| Supprimer un objet | DELETE       | `/api/annonces/1/` |

---

## Conclusion

* Une **API** permet à d'autres logiciels d’accéder à ton application Django sans HTML
* **DRF** est la **boîte à outils officielle** pour construire facilement une API REST avec Django
* Elle est puissante, sécurisée, personnalisable et très utilisée en production
