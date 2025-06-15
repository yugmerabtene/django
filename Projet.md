## **Nom du projet : `lebonclone`**

### **Structure du dépôt :**

```
lebonclone/
├── annonces/                # App principale (Annonces)
│   ├── migrations/
│   ├── templates/annonces/
│   │   ├── base.html
│   │   ├── liste_annonces.html
│   │   ├── detail_annonce.html
│   │   └── creer_annonce.html
│   ├── forms.py
│   ├── models.py
│   ├── serializers.py
│   ├── urls.py
│   ├── views.py
│   ├── api_views.py
│   └── api_urls.py
│
├── comptes/                # App utilisateurs personnalisée
│   ├── forms.py
│   ├── models.py
│   ├── views.py
│   ├── urls.py
│   └── templates/comptes/
│       ├── inscription.html
│       └── connexion.html
│
├── lebonclone/             # Configuration principale
│   ├── settings.py
│   ├── urls.py
│   ├── wsgi.py
│   ├── asgi.py
│   └── __init__.py
│
├── media/                  # Fichiers uploadés
├── static/                 # Fichiers statiques (facultatif pour ce cours)
├── manage.py
├── requirements.txt
├── Procfile
├── .env.example
└── fixtures/
    ├── fixtures.json
    └── fixtures.yaml
```

---

## **Fichiers clés prêts à intégrer sur GitHub**

### `requirements.txt`

```
Django>=4.2
djangorestframework
djangorestframework-simplejwt
gunicorn
pillow
PyYAML
```

### `.env.example`

```
DEBUG=False
SECRET_KEY=changeme
ALLOWED_HOSTS=127.0.0.1,localhost
```

### `Procfile`

```
web: gunicorn lebonclone.wsgi
```

---

## **Étapes pour publier sur GitHub**

1. Crée un dossier `lebonclone` en local.
2. Colle tous les fichiers et dossiers ci-dessus.
3. Initialise un dépôt :

```bash
git init
git add .
git commit -m "Initial commit – Projet Django complet LebonClone"
git remote add origin https://github.com/toncompte/lebonclone.git
git push -u origin main
```

---

## **Étapes pour démarrer en local**

```bash
git clone https://github.com/toncompte/lebonclone.git
cd lebonclone
python -m venv venv
source venv/bin/activate  # ou venv\Scripts\activate sur Windows
pip install -r requirements.txt
cp .env.example .env
python manage.py makemigrations
python manage.py migrate
python manage.py loaddata fixtures/fixtures.json
python manage.py runserver
```
