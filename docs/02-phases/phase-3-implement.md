# Fase 3 — Implementación

## Propósito

El agente implementa exactamente lo que dice el Execution Plan aprobado.
Ni más, ni menos.

---

## Entradas

- Execution Plan aprobado (Fase 2 completada)
- SRD de referencia
- Repositorio en estado conocido (sin cambios no commiteados ajenos a la tarea)

## Salida

- Código implementado siguiendo Arquitectura Hexagonal
- Tests escritos (≥ 70% cobertura en código nuevo)
- Pipeline en verde
- PR listo para revisión

---

## Orden de Implementación

El agente siempre implementa en este orden:

```
1. Tests que fallan (TDD opcional pero recomendado)
   └── 2. Lógica en services/ (núcleo del negocio)
       └── 3. Adapter en db/ (si hay persistencia)
           └── 4. Controller en api/ (exposición)
               └── 5. Tests de integración
                   └── 6. Verificación de pipeline
```

Empezar por `services/` garantiza que la lógica sea testeable de forma
aislada antes de conectar con el framework o la base de datos.

---

## Arquitectura Hexagonal — Reglas de Implementación

### `services/` — Lógica de negocio

```python
# BIEN: services/ no importa nada del framework ni de la DB
class HealthScoreService:
    def __init__(self, user_repository: UserRepository):
        self.user_repo = user_repository  # interfaz, no implementación

    def calculate_score(self, user_id: str) -> HealthScore:
        user = self.user_repo.find_by_id(user_id)
        # lógica pura aquí
        return HealthScore(...)

# MAL: services/ importando FastAPI o SQLAlchemy directamente
from fastapi import HTTPException  # NO en services/
from app.db.models import UserModel  # NO en services/
```

### `api/` — Controllers

```python
# BIEN: api/ solo orquesta, no tiene lógica de negocio
@router.post("/health-score/{user_id}")
async def get_health_score(user_id: str, service: HealthScoreService = Depends()):
    score = service.calculate_score(user_id)  # delega a services/
    return {"score": score.value}

# MAL: lógica de negocio en el controller
@router.post("/health-score/{user_id}")
async def get_health_score(user_id: str, db: Session = Depends()):
    user = db.query(User).filter(User.id == user_id).first()  # NO
    score = user.income * 0.3 + ...  # lógica de negocio NO va aquí
```

### `db/` — Adapters

```python
# BIEN: db/ implementa una interfaz definida en services/
class SqlUserRepository(UserRepository):
    def find_by_id(self, user_id: str) -> User:
        record = self.session.query(UserModel).filter_by(id=user_id).first()
        return User.from_orm(record)  # transforma a entidad de dominio
```

---

## Checklist Durante la Implementación

El agente verifica en cada archivo antes de pasar al siguiente:

- [ ] ¿Los imports son correctos para la capa? (services/ no importa framework)
- [ ] ¿Hay algún secret hardcodeado en este archivo?
- [ ] ¿El manejo de errores no expone información interna?
- [ ] ¿Hay un test que cubre este código?

---

## Manejo de Desviaciones

Si durante la implementación el agente descubre que el plan necesita cambiar:

1. **Para** la implementación
2. **Informa** al humano: "Encontré [situación]. El plan necesita ajustarse en [punto]."
3. **Propone** el ajuste específico al plan
4. **Espera aprobación** antes de continuar
5. **Documenta** el ajuste en el Execution Plan

El agente **no toma decisiones de arquitectura o alcance de forma autónoma**
durante la implementación.

---

## Commits Durante la Implementación

- Commits atómicos: un propósito por commit
- Frecuencia: al completar cada ítem del plan, no al final de todo
- Mensaje en formato: `tipo(scope): descripción en presente`

```
feat(services): add health score calculation logic
test(services): add unit tests for health score
feat(api): expose health score endpoint
feat(db): add SQL adapter for user repository
```

---

## Verificación de Pipeline

Antes de declarar la implementación completa:

```bash
# El agente verifica localmente antes de push:
# 1. Tests pasan
pytest --cov=src tests/

# 2. Cobertura ≥ 70%
pytest --cov=src --cov-fail-under=70 tests/

# 3. Linter sin errores
ruff check src/

# 4. Sin secrets detectados
git diff HEAD | grep -i "password\|secret\|api_key\|token"
```

---

## Transición a Fase 4

La Fase 3 está completa cuando:

1. Todos los ítems del Execution Plan están implementados
2. La Definition of Done está verificada punto a punto
3. El pipeline local está en verde
4. El PR está abierto con descripción completa

**El agente anuncia:** "Implementación completa. PR abierto para revisión (Fase 4)."

---

## Recursos

- Fase anterior: `docs/02-phases/phase-2-plan.md`
- Siguiente fase: `docs/02-phases/phase-4-review.md`
- Playbook FastAPI: `docs/04-playbooks/python-fastapi/README.md`
