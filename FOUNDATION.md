# FOUNDATION — Contratos Absolutos de AIDLC

Este documento es la fuente de verdad del framework. Todo agente, desarrollador
o equipo que trabaje bajo AIDLC debe leer y aceptar estas reglas antes de
escribir una sola línea de código.

---

## 1. Spec Driven Development (SDD)

Todo trabajo comienza con un **Software Requirements Document (SRD)**.

- El SRD debe estar aprobado por el stakeholder antes de iniciar
- El agente no escribe código sin SRD activo
- Cambios de alcance requieren actualización del SRD, no workarounds

## 2. Execution Plan Obligatorio

Antes de escribir código, el agente debe presentar un **Execution Plan** que incluya:

- Archivos a crear o modificar
- Dependencias a instalar
- Migraciones de base de datos afectadas
- Riesgos identificados
- Estimación de tests necesarios

El plan requiere aprobación explícita del desarrollador.

## 3. Definition of Done (DoD)

Cada feature tiene su DoD definida **antes** de comenzar la implementación.

DoD mínima estándar:
- [ ] Lógica implementada en `services/`
- [ ] Controller en `api/` sin lógica de negocio
- [ ] Adapter en `db/` si hay persistencia
- [ ] Tests unitarios ≥ 70% cobertura
- [ ] Tests de integración para rutas críticas
- [ ] Sin secrets hardcodeados
- [ ] Pipeline en verde
- [ ] Revisión de seguridad (STRIDE aplicado)

## 4. Arquitectura Hexagonal

```
src/
├── services/     # Lógica de negocio pura (sin framework, sin DB directa)
├── api/          # Controllers, rutas, validación de entrada
└── db/           # Adapters, repositorios, ORM/queries
```

**Regla:** `api/` y `db/` dependen de `services/`. Nunca al revés.

## 5. Seguridad Corporativa

Aplicable especialmente en entornos como BTG Pactual Colombia.

### Obligatorio en todo proyecto:
- **Rate limiting** en todos los endpoints públicos
- **Audit logs** para operaciones sensibles (quién, qué, cuándo, desde dónde)
- **MFA configurable** — activable por entorno o rol
- **STRIDE** aplicado en diseño de cada módulo de seguridad
- **SFC Circular 007** — cumplimiento en manejo de datos financieros

### Variables de entorno:
- Nunca hardcodear API keys, passwords, tokens o connection strings
- Usar `.env` local, vault o secrets manager en staging/production
- `.env` siempre en `.gitignore`

## 6. Gestión de Secrets

| Ambiente | Método |
|----------|--------|
| `local` | Archivo `.env` (nunca commiteado) |
| `staging` | Variables de entorno del proveedor / vault |
| `production` | Secrets manager (AWS Secrets Manager, Vault, etc.) |

## 7. Pipeline y Commits

- **Nunca** hacer commit si el pipeline no está en verde
- Commits atómicos: un propósito por commit
- Mensaje de commit describe el *por qué*, no el *qué*
- Branches de feature, nunca trabajar directo en `main`

## 8. Testing

| Tipo | Cobertura mínima | Ubicación |
|------|-----------------|-----------|
| Unitarios | 70% | `tests/unit/` |
| Integración | Rutas críticas | `tests/integration/` |
| E2E | Flujos principales | `tests/e2e/` |

Tests deben correr en el pipeline antes de cualquier merge.

## 9. Environments

```
local → staging → production
```

- **local**: libre para experimentar, nunca con datos reales
- **staging**: espejo de producción, datos anonimizados, CI/CD activo
- **production**: acceso restringido, audit logs encendidos, alertas configuradas

### Controles técnicos obligatorios en el pipeline:

- **Push directo a `main` rechazado** — branch protection rules activas;
  todo cambio entra por PR
- **Staging es obligatorio** — ningún PR puede mergearse a `main` sin haber
  pasado por staging primero; el pipeline lo verifica y bloquea si no se cumple
- **Deploy a producción requiere aprobación manual explícita** — configurado
  mediante GitHub Actions environment protection rules; ningún agente ni
  automatización puede deployar a producción sin un humano que apruebe

## 10. Aprendizaje Continuo

El framework mejora con cada sesión. El agente tiene la responsabilidad
activa de documentar lo que aprende.

### Protocolo post-sesión:

- **MEMORY.md** — el agente actualiza el índice de memoria al cerrar
  cada sesión de trabajo
- **lessons-learned.md** — cada error resuelto se documenta con:
  - Contexto del error
  - Causa raíz identificada
  - Solución aplicada
  - Cómo detectarlo antes la próxima vez
- **Errores recurrentes → skills** — si un mismo patrón de error aparece
  más de una vez, se convierte en un skill reutilizable en `docs/06-learning/`
- **Consulta obligatoria** — antes de proponer una solución, el agente
  consulta `lessons-learned.md` para no repetir errores ya documentados

### Estructura de aprendizaje:

```
docs/06-learning/
├── how-agents-learn.md   # Cómo el agente acumula y aplica conocimiento
├── skill-creation.md     # Protocolo para convertir patrones en skills
└── error-patterns.md     # Catálogo de errores recurrentes y sus fixes
```

---

> AIDLC v0.1 Beta · Agnóstico al lenguaje · Del uso personal al corporativo
