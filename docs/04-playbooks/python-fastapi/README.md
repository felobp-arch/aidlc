# Playbook — Python + FastAPI

> Guía de referencia para proyectos Python con FastAPI bajo AIDLC.
> Aplica Arquitectura Hexagonal, SDD y los 10 contratos de FOUNDATION.md.

---

## Stack de Referencia

| Componente | Tecnología | Versión mínima |
|------------|-----------|---------------|
| Runtime | Python | 3.11+ |
| Framework | FastAPI | 0.110+ |
| ORM | SQLAlchemy | 2.0+ |
| Validación | Pydantic | 2.0+ |
| Tests | pytest | 7.0+ |
| Cobertura | pytest-cov | 4.0+ |
| Linter | Ruff | 0.3+ |
| Migraciones | Alembic | 1.13+ |
| DB | PostgreSQL | 15+ |
| Contenedor | Docker + Docker Compose | — |

---

## Estructura de Proyecto

```
project/
├── src/
│   ├── services/           # Lógica de negocio — sin framework, sin DB
│   │   ├── __init__.py
│   │   ├── user_service.py
│   │   └── interfaces/     # Protocolos/ABCs para adapters
│   │       └── user_repository.py
│   ├── api/                # FastAPI routers y controllers
│   │   ├── __init__.py
│   │   ├── main.py         # App FastAPI + lifespan
│   │   ├── dependencies.py # Inyección de dependencias
│   │   └── routes/
│   │       └── users.py
│   └── db/                 # Adapters de persistencia
│       ├── __init__.py
│       ├── models.py       # SQLAlchemy models
│       ├── session.py      # Engine y session factory
│       └── adapters/
│           └── sql_user_repository.py
├── tests/
│   ├── unit/               # Sin DB, sin HTTP
│   ├── integration/        # Con DB y HTTP reales
│   └── conftest.py
├── migrations/             # Alembic
│   └── versions/
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── pyproject.toml
└── README.md
```

---

## Configuración Inicial

### `pyproject.toml`

```toml
[project]
name = "project-name"
version = "0.1.0"
requires-python = ">=3.11"
dependencies = [
    "fastapi>=0.110.0",
    "uvicorn[standard]>=0.27.0",
    "sqlalchemy>=2.0.0",
    "pydantic>=2.0.0",
    "pydantic-settings>=2.0.0",
    "alembic>=1.13.0",
    "psycopg2-binary>=2.9.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-cov>=4.0.0",
    "pytest-asyncio>=0.23.0",
    "httpx>=0.27.0",
    "ruff>=0.3.0",
    "pip-audit>=2.7.0",
]

[tool.ruff]
line-length = 88
select = ["E", "F", "I", "N", "S", "UP"]

[tool.pytest.ini_options]
asyncio_mode = "auto"
addopts = "--cov=src --cov-report=term-missing --cov-fail-under=70"
```

---

## Patrones de Arquitectura Hexagonal

### Interface en `services/interfaces/`

```python
# src/services/interfaces/user_repository.py
from typing import Protocol
from src.services.entities import User

class UserRepository(Protocol):
    def find_by_id(self, user_id: str) -> User | None: ...
    def save(self, user: User) -> User: ...
    def exists_by_email(self, email: str) -> bool: ...
```

### Servicio en `services/`

```python
# src/services/user_service.py
from src.services.interfaces.user_repository import UserRepository
from src.services.entities import User, CreateUserRequest
from src.services.exceptions import UserAlreadyExistsError

class UserService:
    def __init__(self, repository: UserRepository) -> None:
        self._repo = repository

    def create_user(self, request: CreateUserRequest) -> User:
        if self._repo.exists_by_email(request.email):
            raise UserAlreadyExistsError(request.email)
        user = User.create(request)
        return self._repo.save(user)
```

### Adapter en `db/adapters/`

```python
# src/db/adapters/sql_user_repository.py
from sqlalchemy.orm import Session
from src.services.interfaces.user_repository import UserRepository
from src.services.entities import User
from src.db.models import UserModel

class SqlUserRepository:
    def __init__(self, session: Session) -> None:
        self._session = session

    def find_by_id(self, user_id: str) -> User | None:
        record = self._session.get(UserModel, user_id)
        return User.from_orm(record) if record else None

    def save(self, user: User) -> User:
        record = UserModel.from_entity(user)
        self._session.add(record)
        self._session.flush()
        return user
```

### Controller en `api/routes/`

```python
# src/api/routes/users.py
from fastapi import APIRouter, Depends, HTTPException, status
from src.services.user_service import UserService
from src.services.exceptions import UserAlreadyExistsError
from src.api.dependencies import get_user_service
from src.api.schemas import CreateUserRequest, UserResponse

router = APIRouter(prefix="/users", tags=["users"])

@router.post("/", response_model=UserResponse, status_code=status.HTTP_201_CREATED)
async def create_user(
    body: CreateUserRequest,
    service: UserService = Depends(get_user_service),
) -> UserResponse:
    try:
        user = service.create_user(body.to_domain())
        return UserResponse.from_entity(user)
    except UserAlreadyExistsError:
        raise HTTPException(
            status_code=status.HTTP_409_CONFLICT,
            detail="User already exists",
        )
```

---

## Configuración por Environment

```python
# src/api/config.py
from pydantic_settings import BaseSettings, SettingsConfigDict

class Settings(BaseSettings):
    model_config = SettingsConfigDict(env_file=".env", env_file_encoding="utf-8")

    DATABASE_URL: str
    SECRET_KEY: str
    ENVIRONMENT: str = "local"
    LOG_LEVEL: str = "INFO"
    RATE_LIMIT_PER_MINUTE: int = 60
    MFA_ENABLED: bool = False

settings = Settings()
```

---

## Comandos de Desarrollo

```bash
# Setup inicial
python -m venv .venv
source .venv/bin/activate  # Windows: .venv\Scripts\activate
pip install -e ".[dev]"
cp .env.example .env

# Desarrollo
uvicorn src.api.main:app --reload

# Tests
pytest
pytest --cov=src --cov-report=html  # HTML report

# Linter
ruff check src/ tests/
ruff format src/ tests/

# Auditoría de seguridad
pip-audit

# Migraciones
alembic revision --autogenerate -m "descripción"
alembic upgrade head
```

---

## Docker

```dockerfile
# Dockerfile
FROM python:3.11-slim

WORKDIR /app

COPY pyproject.toml .
RUN pip install --no-cache-dir -e .

COPY src/ ./src/

EXPOSE 8000
CMD ["uvicorn", "src.api.main:app", "--host", "0.0.0.0", "--port", "8000"]
```

```yaml
# docker-compose.yml
services:
  api:
    build: .
    ports:
      - "8000:8000"
    env_file: .env
    depends_on:
      db:
        condition: service_healthy

  db:
    image: postgres:15-alpine
    environment:
      POSTGRES_DB: ${DB_NAME}
      POSTGRES_USER: ${DB_USER}
      POSTGRES_PASSWORD: ${DB_PASSWORD}
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U ${DB_USER}"]
      interval: 5s
      retries: 5
    volumes:
      - postgres_data:/var/lib/postgresql/data

volumes:
  postgres_data:
```

---

## Recursos

- Checklist de seguridad: `docs/04-playbooks/python-fastapi/security-checklist.md`
- Pipeline CI: `docs/04-playbooks/python-fastapi/pipeline-template.yml`
- Caso de estudio: `examples/fhc/`
