# Financial Health Companion (FHC)

> Caso de estudio de referencia para AIDLC v0.1 Beta.
> FHC es una aplicación de salud financiera personal que implementa
> todos los principios del framework en un contexto real.

---

## ¿Qué es FHC?

Financial Health Companion es una herramienta que ayuda a los usuarios a:

- Registrar y categorizar ingresos y gastos
- Calcular un "health score" financiero personalizado
- Identificar patrones de gasto y oportunidades de ahorro
- Establecer y hacer seguimiento de metas financieras
- Generar reportes para toma de decisiones

**Público objetivo:** Personas naturales en Colombia que quieren tomar
control de sus finanzas personales.

**Relevancia corporativa:** Los principios aplicados son los mismos que
aplican en entornos como BTG Pactual Colombia, incluyendo cumplimiento
con SFC Circular 007 para manejo de datos financieros.

---

## Estado del Proyecto

| Módulo | Estado | Nivel AIDLC |
|--------|--------|-------------|
| Autenticación básica | Planificado | Nivel 1 |
| Gestión de transacciones | Planificado | Nivel 1 |
| Categorización | Planificado | Nivel 1 |
| Health Score engine | Planificado | Nivel 2 |
| Dashboard y reportes | Planificado | Nivel 2 |
| Metas financieras | Planificado | Nivel 2 |
| Alertas y notificaciones | Planificado | Nivel 2 |
| Multi-usuario (familiar) | Planificado | Nivel 3 |

---

## Stack Tecnológico

| Componente | Tecnología | Justificación |
|------------|-----------|---------------|
| Backend | Python + FastAPI | Ver ADR-0001 |
| Base de datos | PostgreSQL 15 | Ver ADR-0002 |
| ORM | SQLAlchemy 2.0 | Maduro, async support |
| Validación | Pydantic v2 | Integración nativa con FastAPI |
| Autenticación | JWT (jose) | Stateless, escalable |
| Tests | pytest + pytest-cov | Cobertura ≥ 70% |
| Contenedor | Docker + Docker Compose | Reproducibilidad |
| CI/CD | GitHub Actions | Ver templates AIDLC |

---

## Arquitectura

```
fhc/
├── src/
│   ├── services/                    # Lógica de negocio pura
│   │   ├── entities/
│   │   │   ├── transaction.py       # Entidad Transaction
│   │   │   ├── user.py              # Entidad User
│   │   │   └── health_score.py      # Value Object HealthScore
│   │   ├── interfaces/
│   │   │   ├── transaction_repo.py  # Protocolo para adapter
│   │   │   └── user_repo.py
│   │   ├── transaction_service.py   # CRUD de transacciones
│   │   ├── health_score_service.py  # Algoritmo de scoring
│   │   └── report_service.py        # Generación de reportes
│   ├── api/
│   │   ├── main.py                  # FastAPI app + lifespan
│   │   ├── dependencies.py          # DI container
│   │   ├── middleware/
│   │   │   ├── auth.py
│   │   │   ├── rate_limit.py
│   │   │   └── audit_log.py
│   │   └── routes/
│   │       ├── auth.py
│   │       ├── transactions.py
│   │       └── health_score.py
│   └── db/
│       ├── models.py                # SQLAlchemy models
│       ├── session.py               # Engine factory
│       └── adapters/
│           ├── sql_transaction_repo.py
│           └── sql_user_repo.py
├── tests/
│   ├── unit/
│   │   ├── test_health_score_service.py
│   │   └── test_transaction_service.py
│   └── integration/
│       ├── test_transactions_api.py
│       └── test_auth_api.py
├── migrations/
└── docs/
    ├── srd.md                       # Software Requirements Document
    └── decisions/                   # ADRs específicos de FHC
```

---

## Seguridad en FHC

Por manejar datos financieros de usuarios colombianos, FHC aplica Nivel 2
con elementos de Nivel 3:

- **Cifrado**: Datos financieros cifrados en reposo
- **Audit logs**: Toda operación sobre datos financieros queda registrada
- **Rate limiting**: Estricto en endpoints de auth y consulta de datos
- **MFA**: Configurable, obligatorio para acceso desde dispositivos nuevos
- **SFC Circular 007**: Guía las decisiones de retención de datos y acceso

Ver: `docs/01-principles/security-by-design.md`

---

## Cómo Usar FHC como Referencia

Cuando una regla de FOUNDATION.md parezca abstracta, FHC es el ejemplo concreto:

| Regla | Ejemplo en FHC |
|-------|---------------|
| Arquitectura Hexagonal | `services/health_score_service.py` sin imports de FastAPI |
| Sin secrets hardcodeados | `.env.example` con DATABASE_URL y SECRET_KEY |
| Audit logs | Middleware que registra cada acceso a datos financieros |
| STRIDE | Ver `decisions/ADR-0003-auth-design.md` |
| Tests ≥ 70% | `test_health_score_service.py` — algoritmo core con 100% cobertura |

---

## Recursos FHC

- Decisiones de arquitectura: `examples/fhc/decisions.md`
- Lecciones aprendidas: `examples/fhc/lessons-learned.md`
- Playbook técnico: `docs/04-playbooks/python-fastapi/README.md`
- Checklist de seguridad: `docs/04-playbooks/python-fastapi/security-checklist.md`
