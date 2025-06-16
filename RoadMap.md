
### **01 – Projet Django (Architecture MTV)**

Référence : [Projet.md sur GitHub](https://github.com/yugmerabtene/django/blob/main/Projet.md)

* Mettre en place un projet Django en suivant le pattern MTV (Model – Template – View).
* Comprendre la structure du projet, la gestion des vues, modèles, templates et de l’interface d’administration.
* Utiliser des fichiers fixtures.json pour précharger des données.
* Créer un système de création et de gestion d’annonces (type Leboncoin).

---

### **02 – API REST et Frontend React**

Objectifs :

* Ajouter une couche API REST au projet Django à l’aide de Django REST Framework.
* Concevoir un frontend React qui consomme cette API via fetch ou axios.
* Séparer les responsabilités : backend (Django) et frontend (React) doivent fonctionner de façon autonome.
* Mettre en place un système de création, consultation et suppression d’annonces depuis l’interface React.

---

### **03 – Architecture Microservices et Kafka**

Refactorisation du projet en microservices :

* Découper le projet monolithique Django en plusieurs microservices indépendants.
* Mettre en place un message broker avec Apache Kafka pour la communication inter-services.
* Ajouter une API Gateway (ex : Django avec DRF Gateway ou solution proxy) pour centraliser les requêtes externes.
* Définir plusieurs endpoints exposés par les microservices : annonces, utilisateurs, authentification, etc.

Frontend :

* Utiliser React Native pour les applications mobiles (iOS / Android).
* Utiliser ReactJS pour la version desktop du site.
