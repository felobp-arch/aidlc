# Security Checklist — Python + FastAPI

> Verificar antes de cada PR que involucre endpoints, autenticación,
> manejo de datos o cambios de configuración.

---

## Pre-PR Checklist (el agente verifica antes de abrir PR)

### Secrets y Configuración

- [ ] Sin API keys, passwords o tokens hardcodeados en ningún archivo del diff
- [ ] `git log --all --full-history -- "*.env"` — ningún `.env` commiteado
- [ ] Variables de entorno nuevas documentadas en `.env.example` (sin valores reales)
- [ ] `.gitignore` incluye `.env`, `*.pem`, `*.key`, `secrets/`
- [ ] `pip-audit` ejecutado — sin vulnerabilidades críticas o altas sin resolver

### Endpoints y Autenticación

- [ ] Todos los endpoints protegidos tienen verificación de token
- [ ] El endpoint `/health` no expone información interna del sistema
- [ ] Respuestas de error usan mensajes genéricos (no stack traces, no rutas internas)
- [ ] Rate limiting configurado en endpoints públicos
- [ ] CORS configurado explícitamente — sin `allow_origins=["*"]` en producción

### Validación de Input

- [ ] Todos los bodies de request tienen schema Pydantic con validación explícita
- [ ] Path parameters y query params validados con tipos estrictos
- [ ] Tamaño máximo de upload configurado si el endpoint acepta archivos
- [ ] Sin concatenación de strings en queries SQL (usar ORM o queries parametrizadas)

### Autorización

- [ ] Endpoints verifican ownership del recurso (no solo existencia)
- [ ] Usuarios no pueden acceder a recursos de otros usuarios (IDOR)
- [ ] Acciones de admin verifican rol, no solo autenticación
- [ ] Sin mass assignment — schema de input explícito con campos permitidos

### Logs y Auditoría

- [ ] Operaciones sensibles generan audit log (crear, modificar, eliminar, acceder a datos financieros)
- [ ] Logs no incluyen passwords, tokens ni números de tarjeta
- [ ] Logs incluyen: user_id, action, resource_id, timestamp, IP, resultado
- [ ] Level de log apropiado por tipo de evento

---

## Implementación de Controles

### Rate Limiting con slowapi

```python
# src/api/main.py
from fastapi import FastAPI, Request
from slowapi import Limiter, _rate_limit_exceeded_handler
from slowapi.util import get_remote_address
from slowapi.errors import RateLimitExceeded

limiter = Limiter(key_func=get_remote_address)
app = FastAPI()
app.state.limiter = limiter
app.add_exception_handler(RateLimitExceeded, _rate_limit_exceeded_handler)

# src/api/routes/users.py
from slowapi import Limiter
from slowapi.util import get_remote_address

limiter = Limiter(key_func=get_remote_address)

@router.post("/login")
@limiter.limit("5/minute")  # límite estricto en auth
async def login(request: Request, body: LoginRequest):
    ...
```

### Autenticación JWT

```python
# src/api/dependencies.py
from fastapi import Depends, HTTPException, status
from fastapi.security import HTTPBearer, HTTPAuthorizationCredentials
from jose import JWTError, jwt
from src.api.config import settings

security = HTTPBearer()

async def get_current_user(
    credentials: HTTPAuthorizationCredentials = Depends(security),
) -> str:
    token = credentials.credentials
    try:
        payload = jwt.decode(token, settings.SECRET_KEY, algorithms=["HS256"])
        user_id: str = payload.get("sub")
        if user_id is None:
            raise HTTPException(status_code=status.HTTP_401_UNAUTHORIZED)
        return user_id
    except JWTError:
        raise HTTPException(
            status_code=status.HTTP_401_UNAUTHORIZED,
            detail="Invalid or expired token",
        )
```

### Audit Logging

```python
# src/services/audit_log_service.py
import logging
from dataclasses import dataclass
from datetime import datetime, timezone

logger = logging.getLogger("audit")

@dataclass
class AuditEvent:
    user_id: str
    action: str
    resource_type: str
    resource_id: str
    ip_address: str
    result: str  # "success" | "failure"
    details: dict | None = None

def log_audit_event(event: AuditEvent) -> None:
    logger.info(
        "audit_event",
        extra={
            "user_id": event.user_id,
            "action": event.action,
            "resource_type": event.resource_type,
            "resource_id": event.resource_id,
            "ip": event.ip_address,
            "result": event.result,
            "timestamp": datetime.now(timezone.utc).isoformat(),
            **(event.details or {}),
        },
    )
```

### Headers de Seguridad

```python
# src/api/middleware.py
from fastapi import Request
from starlette.middleware.base import BaseHTTPMiddleware

class SecurityHeadersMiddleware(BaseHTTPMiddleware):
    async def dispatch(self, request: Request, call_next):
        response = await call_next(request)
        response.headers["X-Content-Type-Options"] = "nosniff"
        response.headers["X-Frame-Options"] = "DENY"
        response.headers["X-XSS-Protection"] = "1; mode=block"
        response.headers["Strict-Transport-Security"] = "max-age=31536000; includeSubDomains"
        response.headers["Referrer-Policy"] = "strict-origin-when-cross-origin"
        # Remover headers que revelan versión del software
        response.headers.pop("Server", None)
        return response
```

### Manejo de Errores Seguro

```python
# src/api/exception_handlers.py
from fastapi import Request
from fastapi.responses import JSONResponse
import logging

logger = logging.getLogger(__name__)

async def generic_exception_handler(request: Request, exc: Exception) -> JSONResponse:
    # Log el error completo internamente
    logger.error(
        "Unhandled exception",
        exc_info=exc,
        extra={"path": request.url.path, "method": request.method},
    )
    # Respuesta genérica al cliente — sin stack trace
    return JSONResponse(
        status_code=500,
        content={"detail": "Internal server error"},
    )
```

---

## Checklist de Dependencias

```bash
# Ejecutar antes de cada release
pip-audit --desc

# Si hay vulnerabilidades:
# 1. Verificar si hay versión con fix disponible
# 2. Si sí: actualizar
# 3. Si no: documentar como riesgo aceptado en el PR
```

---

## Verificación Pre-Deploy a Producción

Adicionalmente al checklist de PR:

- [ ] `pip-audit` en verde o riesgos documentados
- [ ] Variables de entorno de producción verificadas en el secrets manager
- [ ] Logs de staging revisados — sin errores inesperados
- [ ] Rate limits configurados para carga de producción esperada
- [ ] MFA habilitado para roles de admin (si NFR-S07 aplica)
- [ ] Revisión STRIDE completada para features de seguridad nuevas

---

## Recursos

- Principio Security by Design: `docs/01-principles/security-by-design.md`
- Template STRIDE: `docs/03-templates/stride-review-template.md`
- Pipeline con security scan: `docs/04-playbooks/python-fastapi/pipeline-template.yml`
