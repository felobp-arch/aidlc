# Decisiones de Arquitectura — Financial Health Companion

> Este documento registra las decisiones de arquitectura tomadas en FHC
> y el razonamiento detrás de cada una. Es la referencia para entender
> por qué el sistema está construido como está.
>
> Para el formato completo de ADR, ver: `docs/03-templates/adr-template.md`

---

## ADR-0001 — Python + FastAPI como stack backend

**Fecha:** 2026-04-07  
**Estado:** Aceptada

**Contexto:**
FHC necesita un backend con endpoints REST para una aplicación de finanzas
personales. El agente y el desarrollador principal tienen experiencia en
Python. El proyecto puede crecer a nivel corporativo.

**Decisión:**
Usar Python 3.11+ con FastAPI como framework.

**Razones:**
- FastAPI tiene soporte nativo para async, validación con Pydantic y documentación OpenAPI automática
- Python tiene el ecosistema más maduro para análisis de datos financieros
- Type hints nativos mejoran la mantenibilidad del código
- Bien soportado en todos los proveedores de cloud

**Trade-offs aceptados:**
- Python es más lento que Go o Rust para alta concurrencia — aceptable para el volumen esperado en FHC
- El GIL limita el paralelismo real — mitigable con async/await y workers múltiples

---

## ADR-0002 — PostgreSQL como base de datos principal

**Fecha:** 2026-04-07  
**Estado:** Aceptada

**Contexto:**
FHC maneja datos financieros que requieren consistencia, transacciones ACID,
y capacidad de queries complejos para reportes.

**Decisión:**
Usar PostgreSQL 15+ como única base de datos.

**Razones:**
- ACID completo — crítico para datos financieros
- JSONB para metadatos flexibles sin perder estructura
- Full-text search para búsqueda de transacciones
- Excelente soporte en todos los proveedores (Supabase, RDS, Cloud SQL)
- UUID nativo para IDs — sin colisiones en sistemas distribuidos

**Alternativas rechazadas:**
- MongoDB: consistencia eventual no apropiada para datos financieros
- MySQL: menor soporte para tipos avanzados y JSONB
- SQLite: no escala para múltiples usuarios concurrentes

---

## ADR-0003 — JWT sin estado para autenticación

**Fecha:** 2026-04-07  
**Estado:** Aceptada

**Contexto:**
FHC necesita autenticación que escale horizontalmente sin coordinación
entre instancias del servicio.

**Decisión:**
Usar JWT (JSON Web Tokens) con HS256, access token de 60 min y refresh
token de 7 días almacenado en DB.

**Razones:**
- Sin estado: cualquier instancia puede validar el token
- El refresh token en DB permite revocación (logout, compromiso)
- Access token corto minimiza el impacto de tokens comprometidos

**Controles de seguridad:**
- Access token: expiración 60 minutos, en memoria del cliente
- Refresh token: expiración 7 días, en DB (revocable), en cookie HttpOnly
- Rotación de refresh token en cada uso (rotation)
- SECRET_KEY diferente por environment, rotación cada 30 días

**STRIDE aplicado:**
- S (Spoofing): mitigado con firma del token
- T (Tampering): mitigado con verificación de firma en cada request
- I (Disclosure): mitigado con tokens de corta duración y HttpOnly cookies

---

## ADR-0004 — Arquitectura Hexagonal con separación estricta de capas

**Fecha:** 2026-04-07  
**Estado:** Aceptada

**Contexto:**
El código financiero del health score engine debe ser testeable de forma
completamente aislada, sin depender de FastAPI ni de la base de datos.

**Decisión:**
Separación estricta en tres capas: `services/` (dominio), `api/` (entrada),
`db/` (persistencia). `services/` no importa nada de `api/` ni `db/`.

**Razones:**
- El algoritmo de health score puede testearse sin DB ni HTTP
- Si cambia el framework (FastAPI → otro), solo cambia `api/`
- Si cambia la DB (PostgreSQL → otro), solo cambia `db/`
- Onboarding más rápido: la lógica de negocio está aislada

**Consecuencias:**
- Más archivos que en una arquitectura monolítica
- Requiere disciplina del agente para no mezclar capas
- Interfaces (Protocols) necesarias en `services/interfaces/`

---

## ADR-0005 — Health Score como Value Object inmutable

**Fecha:** 2026-04-07  
**Estado:** Aceptada

**Contexto:**
El score financiero es el valor central de FHC. Necesita ser calculado de
forma reproducible y testeable.

**Decisión:**
`HealthScore` es un Value Object inmutable: se calcula, no se modifica.
Cada cálculo produce un nuevo objeto. Se persiste como snapshot histórico.

**Razones:**
- Reproducibilidad: el mismo input siempre produce el mismo score
- Auditoría: el historial de scores refleja la evolución financiera
- Testing: verificar el score es comparar dos objetos, sin efectos laterales

**Estructura:**
```python
@dataclass(frozen=True)
class HealthScore:
    value: int           # 0-100
    category: str        # "crítico" | "en riesgo" | "estable" | "saludable"
    calculated_at: date
    components: dict     # desglose por dimensión
```

---

## Decisiones Pendientes

| ID | Tema | Estado | Fecha límite |
|----|------|--------|-------------|
| ADR-0006 | Estrategia de categorización de transacciones (ML vs reglas) | Por decidir | — |
| ADR-0007 | Proveedor de notificaciones (email/push) | Por decidir | — |
| ADR-0008 | Estrategia de backup y retención de datos | Por decidir | — |
