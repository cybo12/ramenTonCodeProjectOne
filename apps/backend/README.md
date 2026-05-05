# Backend - Ramen API

API REST moderne avec FastAPI et SQLModel (Python 3.14).

## 🚀 Stack Technologique

### Core
- **Python** : 3.14+
- **Framework** : [FastAPI](https://fastapi.tiangolo.com/) - Web framework async haute-performance
- **Package Manager** : [uv](https://astral.sh/uv) - Remplaçant ultra-rapide de pip/poetry
- **Build System** : Hatchling (PEP 517/518)

### ORM & Database
- **SQLModel** 0.0.14+ - Fusion SQLAlchemy + Pydantic (un seul modèle pour tout)
- **Alembic** - Migrations de base de données
- **PostgreSQL** 16 - Base de données relationnelle

### Validation & Config
- **SQLModel Pydantic** - Validation de données native dans les modèles
- **Pydantic Settings** - Gestion des configuration

### Outils & Utilitaires
- **Uvicorn** - Serveur ASGI
- **HTTPx** - Client HTTP async
- **APScheduler** - Planification de tâches
- **python-dotenv** - Variables d'environnement

### Développement
- **pytest** - Framework de test
- **pytest-asyncio** - Support async pour pytest
- **black** - Formatage de code
- **ruff** - Linter ultra-rapide
- **mypy** - Type checking statique

---

## 📦 Installation

### Prérequis
- Python 3.14 installé
- uv installé : `curl -LsSf https://astral.sh/uv/install.sh | sh`

### Setup local

```bash
# Cloner et accéder au backend
cd apps/backend

# Créer l'environnement Python 3.14 et installer les dépendances
uv sync

# Copier les variables d'environnement
cp .env.example .env

# Activer le shell avec l'env
uv run bash
# Ou directement exécuter les commandes avec `uv run`
```

---

## 🐳 Docker & Docker Compose

### Démarrer les services (DB, Redis, Backend)

```bash
# À la racine du projet
docker compose up -d

# Vérifier les logs
docker compose logs -f backend

# Arrêter tout
docker compose down
```

### Accès après démarrage
- **API** : http://localhost:8000
- **Docs Swagger** : http://localhost:8000/docs
- **Docs ReDoc** : http://localhost:8000/redoc
- **Adminer (DB UI)** : http://localhost:8080

### Services disponibles
- `backend` - API FastAPI
- `db` - PostgreSQL 16
- `redis` - Cache et sessions
- `adminer` - Interface pour gérer PostgreSQL

---

## 🔧 Commandes uv

```bash
# Synchroniser l'environnement (installer/mettre à jour les dépendances)
uv sync

# Ajouter une dépendance
uv add package-name

# Ajouter une dépendance de développement
uv add --dev package-name

# Exécuter une commande dans l'env
uv run python app/main.py
uv run uvicorn app.main:app --reload

# Outils de développement (avec uvx - aucune installation nécessaire)
uvx pytest
uvx ruff check .
uvx black .
uvx mypy app/
```
---

### Mode développement (avec hot-reload)

```bash
# Avec Docker Compose
docker compose up backend

# Ou localement
uv run uvicorn app.main:app --reload --host 0.0.0.0 --port 8000
```

### Mode production

```bash
# Via Docker
docker build -t backend:latest .
docker run -p 8000:8000 backend:latest

# Ou directement
uv run uvicorn app.main:app --host 0.0.0.0 --port 8000 --workers 4
```


---

## 🧪 Tests

```bash
# Exécuter tous les tests
uvx pytest

# Avec couverture
uvx pytest --cov=app

# En mode verbose
uvx pytest -v

# Exécuter un fichier test spécifique
uvx pytest tests/test_items.py
```

---

## 🔍 Linting & Formatting

```bash
# Vérifier le style de code
uvx ruff check .

# Corriger automatiquement les problèmes
uvx ruff check . --fix

# Formater avec Black
uvx black .

# Type checking
uvx mypy app/
```

---

## 🗄️ Migrations de Base de Données

```bash
# Créer une nouvelle migration
uvx alembic revision --autogenerate -m "Description du changement"

# Appliquer les migrations
uvx alembic upgrade head

# Revenir en arrière
uvx alembic downgrade -1
```

---

## ⚙️ Configuration

Les variables d'environnement se trouvent dans `.env` (copier depuis `.env.example`).

Voir [core/config.py](core/config.py) pour la liste complète des configurations disponibles.

---

## 📖 Documentation

- **FastAPI** : https://fastapi.tiangolo.com/
- **SQLModel** : https://sqlmodel.tiangolo.com/
- **uv** : https://astral.sh/uv/

---

## 📝 Notes

- Python 3.14 est requis (voir `pyproject.toml` pour `requires-python`)
- Les type hints sont utilisés systématiquement pour la maintenabilité
- FastAPI génère automatiquement la documentation OpenAPI (`/docs`)
