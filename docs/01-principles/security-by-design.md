# Security by Design — Principio AIDLC

## Definición

Security by Design significa que la seguridad no es una capa que se agrega
al final — es una restricción que informa cada decisión de diseño desde el
inicio del proyecto.

En AIDLC, el agente aplica consideraciones de seguridad en tres momentos:
durante la especificación (SRD), durante el diseño (Execution Plan), y durante
la revisión de código (Definition of Done).

---

## Marco de Amenazas: STRIDE

STRIDE es el modelo de amenazas que usa AIDLC para analizar seguridad en
cada feature. El agente lo aplica en el Execution Plan cuando la feature
tiene implicaciones de seguridad.

| Letra | Amenaza | Pregunta clave |
|-------|---------|----------------|
| **S** | Spoofing | ¿Puede alguien hacerse pasar por otro usuario o sistema? |
| **T** | Tampering | ¿Puede alguien modificar datos en tránsito o en reposo? |
| **R** | Repudiation | ¿Puede alguien negar haber realizado una acción? |
| **I** | Information Disclosure | ¿Puede filtrarse información sensible? |
| **D** | Denial of Service | ¿Puede alguien degradar o bloquear el servicio? |
| **E** | Elevation of Privilege | ¿Puede alguien obtener más permisos de los que le corresponden? |

Para cada feature, el agente documenta qué amenazas STRIDE aplican y qué
controles mitigan cada una. Ver plantilla: `docs/03-templates/stride-review-template.md`

---

## Controles Obligatorios por Nivel

### Todo proyecto (nivel 1 — Starter)

- Variables de entorno para todos los secrets (nunca hardcode)
- `.gitignore` actualizado para excluir `.env` y archivos de credenciales
- HTTPS en todos los environments (staging y production)
- Dependencias auditadas antes del primer deploy

### Proyectos en equipo (nivel 2 — Team)

Todo lo anterior, más:

- **Rate limiting** en todos los endpoints públicos
- **Autenticación** con tokens de corta duración (JWT exp ≤ 1h)
- **CORS** configurado explícitamente, no `*`
- **Audit logs** para operaciones que modifican datos
- **Validación de input** en la capa `api/` antes de llegar a `services/`
- **Manejo de errores** que no expone stack traces al cliente

### Proyectos corporativos (nivel 3 — Enterprise)

Todo lo anterior, más:

- **MFA** configurable por rol y environment
- **Secrets manager** (AWS Secrets Manager, Vault, GCP Secret Manager)
- **Audit logs** con retención mínima según regulación aplicable
- **Cifrado en reposo** para datos sensibles
- **Penetration testing** antes de producción
- **SFC Circular 007** — cumplimiento en manejo de datos financieros (Colombia)
- **Revisión STRIDE** documentada para cada módulo de seguridad

---

## Secrets: Gestión Correcta

### Nunca hacer esto:
```python
# MAL — hardcode
DATABASE_URL = "postgresql://user:password@host:5432/db"
API_KEY = "sk-1234abcd..."
```

### Siempre hacer esto:
```python
# BIEN — variable de entorno
import os
DATABASE_URL = os.environ["DATABASE_URL"]
API_KEY = os.environ["API_KEY"]
```

### Por environment:

| Environment | Método |
|-------------|--------|
| local | `.env` file (en `.gitignore`) |
| staging | Variables del proveedor (Railway, Render, AWS) |
| production | Secrets Manager o Vault |

---

## Checklist de Seguridad Pre-Deploy

El agente verifica este checklist antes de proponer cualquier merge a `main`:

- [ ] Sin secrets en el código o en el historial de git
- [ ] Rate limiting configurado en endpoints públicos
- [ ] Validación de input en todos los endpoints
- [ ] Errores no exponen información interna al cliente
- [ ] STRIDE revisado para features con implicaciones de seguridad
- [ ] Dependencias auditadas (`npm audit` / `pip-audit` / `cargo audit`)
- [ ] Variables de entorno documentadas en `.env.example`
- [ ] Logs de auditoría activos para operaciones sensibles

---

## SFC Circular 007 — Contexto Colombia

Para proyectos que manejen datos financieros de usuarios colombianos:

- Datos personales y financieros deben cifrarse en tránsito (TLS 1.2+) y en reposo
- Logs de acceso a datos sensibles con retención mínima de 5 años
- Controles de acceso basados en roles (RBAC) documentados
- Procedimiento de respuesta a incidentes definido antes de producción
- Notificación a la SFC ante brechas de seguridad dentro de los plazos regulatorios

---

## Recursos

- Plantilla STRIDE: `docs/03-templates/stride-review-template.md`
- Checklist FastAPI: `docs/04-playbooks/python-fastapi/security-checklist.md`
- NFR template: `docs/03-templates/nfr-requirements-template.md`
