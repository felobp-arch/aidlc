# Execution Plan — Template

> Copiar este template al inicio de cada sesión de implementación.
> El agente completa todas las secciones. El humano aprueba antes de que
> se escriba una línea de código.

---

## Execution Plan — [Nombre de la Feature]

**SRD de referencia:** [RF-XXX, RF-YYY]  
**Fecha:** YYYY-MM-DD  
**Agente:** [Claude Sonnet / GPT-4 / etc.]  
**Estimación:** [N horas / N sesiones de trabajo]  
**Branch de feature:** `feat/[nombre-descriptivo]`

---

### 1. Objetivo

[Una oración clara de qué se va a implementar y para qué.
Si necesita más de dos oraciones, el scope es probablemente demasiado grande.]

---

### 2. Archivos a Modificar

| Archivo | Tipo de cambio | Descripción |
|---------|---------------|-------------|
| `src/services/X.py` | Modificar | Agregar método Y |
| `src/api/routes/X.py` | Modificar | Exponer endpoint Z |
| `tests/unit/test_X.py` | Modificar | Tests para método Y |

---

### 3. Archivos a Crear

| Archivo | Propósito |
|---------|-----------|
| `src/services/nuevo.py` | Lógica de negocio para [feature] |
| `src/db/adapters/nuevo.py` | Adapter de DB para [entidad] |
| `tests/unit/test_nuevo.py` | Tests unitarios |
| `tests/integration/test_endpoint.py` | Tests de integración |

---

### 4. Dependencias a Instalar

| Paquete | Versión | Justificación |
|---------|---------|---------------|
| `paquete` | `^1.2.3` | [por qué este paquete y no otro] |

> Si no se instalan dependencias nuevas: "Ninguna"

---

### 5. Cambios de Base de Datos

```sql
-- Migración: [descripción]
CREATE TABLE nombre (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    campo1 VARCHAR(255) NOT NULL,
    created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_nombre_campo1 ON nombre(campo1);
```

> Si no hay cambios de DB: "Ninguno"

---

### 6. Análisis de Seguridad (STRIDE)

> Completar solo si la feature tiene implicaciones de seguridad.
> Para features puramente internas sin acceso externo, indicar "N/A con justificación".

| Amenaza | ¿Aplica? | Mitigación |
|---------|----------|------------|
| **S** Spoofing | Sí/No | [acción concreta o N/A] |
| **T** Tampering | Sí/No | [acción concreta o N/A] |
| **R** Repudiation | Sí/No | [acción concreta o N/A] |
| **I** Info Disclosure | Sí/No | [acción concreta o N/A] |
| **D** Denial of Service | Sí/No | [acción concreta o N/A] |
| **E** Elevation of Privilege | Sí/No | [acción concreta o N/A] |

---

### 7. Riesgos Identificados

| # | Riesgo | Probabilidad | Impacto | Mitigación |
|---|--------|-------------|---------|------------|
| 1 | [descripción] | Alta/Media/Baja | Alto/Medio/Bajo | [acción] |
| 2 | [descripción] | Alta/Media/Baja | Alto/Medio/Bajo | [acción] |

> Si no hay riesgos identificados: justificar brevemente por qué.

---

### 8. Tests Necesarios

#### Tests Unitarios (`tests/unit/`)

- [ ] `test_[caso_exitoso]` — verifica el flujo principal del happy path
- [ ] `test_[caso_error_X]` — verifica manejo de [tipo de error]
- [ ] `test_[caso_borde_Y]` — verifica comportamiento en [condición límite]

#### Tests de Integración (`tests/integration/`)

- [ ] `test_[endpoint]_[metodo]_success` — respuesta HTTP 200 con payload correcto
- [ ] `test_[endpoint]_[metodo]_unauthorized` — respuesta HTTP 401 sin token
- [ ] `test_[endpoint]_[metodo]_validation` — respuesta HTTP 422 con input inválido

---

### 9. Definition of Done

> Esta sección se verifica punto a punto antes de abrir el PR.

- [ ] Lógica de negocio implementada en `services/` sin imports de framework
- [ ] Controller en `api/` sin lógica de negocio (solo orquesta)
- [ ] Adapter en `db/` si hay persistencia nueva
- [ ] Tests unitarios escritos y pasando
- [ ] Cobertura de código ≥ 70% en archivos nuevos/modificados
- [ ] Tests de integración escritos y pasando
- [ ] Sin secrets hardcodeados en ningún archivo del diff
- [ ] Variables de entorno nuevas documentadas en `templates/.env.example`
- [ ] Pipeline local en verde (tests + lint + cobertura)
- [ ] PR description completa con referencia al SRD
- [ ] `lessons-learned.md` actualizado si hubo errores relevantes

---

### 10. Aprobación

| Campo | Valor |
|-------|-------|
| Aprobado por | [nombre del humano] |
| Fecha de aprobación | YYYY-MM-DD |
| Observaciones | [ajustes pedidos, si los hay] |

> **Sin esta aprobación, el agente no inicia la implementación.**
