# STRIDE Review — Template

> Usar en el Execution Plan para features con implicaciones de seguridad.
> El agente completa este análisis. El humano lo revisa como parte del plan.

---

## STRIDE Review — [Nombre del Módulo o Feature]

**Fecha:** YYYY-MM-DD  
**Feature analizada:** [descripción]  
**SRD de referencia:** [RF-XXX]  
**Analista:** Agente ([modelo]) + Revisión: [nombre humano]

---

## Contexto del Sistema

```
[Diagrama simplificado de flujo de datos o descripción de actores]

Usuario → [API Gateway] → [Endpoint] → [Service] → [DB]
                               ↓
                         [Servicio externo]
```

**Actores identificados:**
- Actor 1: [descripción y nivel de confianza]
- Actor 2: [descripción y nivel de confianza]

**Datos sensibles en scope:**
- [tipo de dato] — clasificación: [público/interno/confidencial/restringido]

---

## Análisis por Amenaza

### S — Spoofing (Suplantación de identidad)

**¿Puede alguien hacerse pasar por un usuario legítimo o por el sistema?**

| Escenario | Aplica | Severidad | Control de mitigación |
|-----------|--------|-----------|----------------------|
| Token JWT robado | Sí/No | Alta/Media/Baja | Tokens de corta duración (exp 1h) + refresh rotation |
| Replay attack | Sí/No | Alta/Media/Baja | Nonce o timestamp en requests |
| API key comprometida | Sí/No | Alta/Media/Baja | Rotación automática + alertas |

**Controles implementados:**
- [ ] Autenticación en todos los endpoints protegidos
- [ ] Tokens con expiración corta
- [ ] Rate limiting por IP y por usuario

---

### T — Tampering (Manipulación de datos)

**¿Puede alguien modificar datos en tránsito o en reposo?**

| Escenario | Aplica | Severidad | Control de mitigación |
|-----------|--------|-----------|----------------------|
| MITM en tránsito | Sí/No | Alta/Media/Baja | TLS 1.2+ obligatorio |
| Modificación de payload | Sí/No | Alta/Media/Baja | Validación de schema en api/ |
| SQL injection | Sí/No | Alta/Media/Baja | ORM / queries parametrizadas |
| Modificación de DB directa | Sí/No | Alta/Media/Baja | Permisos mínimos en DB user |

**Controles implementados:**
- [ ] TLS en todos los environments (staging y prod)
- [ ] Validación de input con schema (Pydantic, Zod, etc.)
- [ ] Queries parametrizadas o ORM — sin concatenación de strings SQL
- [ ] Usuario de DB con permisos mínimos (no superuser)

---

### R — Repudiation (Repudio)

**¿Puede alguien negar haber realizado una acción?**

| Escenario | Aplica | Severidad | Control de mitigación |
|-----------|--------|-----------|----------------------|
| Usuario niega transacción | Sí/No | Alta/Media/Baja | Audit log con user_id, timestamp, IP |
| Agente niega cambio de config | Sí/No | Alta/Media/Baja | Git history + CI logs |
| Admin niega acción privilegiada | Sí/No | Alta/Media/Baja | Audit log inmutable |

**Controles implementados:**
- [ ] Audit log para operaciones que modifican datos
- [ ] Log incluye: user_id, action, resource_id, timestamp, IP, resultado
- [ ] Logs con retención adecuada (mínimo según regulación aplicable)

---

### I — Information Disclosure (Divulgación de información)

**¿Puede filtrarse información sensible?**

| Escenario | Aplica | Severidad | Control de mitigación |
|-----------|--------|-----------|----------------------|
| Stack trace en respuesta de error | Sí/No | Alta/Media/Baja | Error genérico al cliente, detalle en log |
| Datos de otro usuario en respuesta | Sí/No | Alta/Media/Baja | Filtro por user_id en queries |
| Secrets en logs | Sí/No | Alta/Media/Baja | Sanitización de logs |
| Enumeración de usuarios | Sí/No | Alta/Media/Baja | Respuestas genéricas en auth failures |
| Headers con versión de software | Sí/No | Baja | Remover headers Server, X-Powered-By |

**Controles implementados:**
- [ ] Respuestas de error no exponen stack trace ni rutas internas
- [ ] Logs sanitizados (sin passwords, tokens, números de tarjeta)
- [ ] Responses solo incluyen campos necesarios (no objeto completo de DB)
- [ ] Headers de seguridad configurados (HSTS, X-Content-Type-Options, etc.)

---

### D — Denial of Service (Denegación de servicio)

**¿Puede alguien degradar o bloquear el servicio?**

| Escenario | Aplica | Severidad | Control de mitigación |
|-----------|--------|-----------|----------------------|
| Flood de requests | Sí/No | Alta/Media/Baja | Rate limiting por IP |
| Queries costosas | Sí/No | Alta/Media/Baja | Timeout en queries + paginación |
| Upload de archivos grandes | Sí/No | Alta/Media/Baja | Límite de tamaño en multipart |
| Regex catastrophic backtracking | Sí/No | Media/Baja | Validar regexes con herramientas |

**Controles implementados:**
- [ ] Rate limiting en todos los endpoints públicos
- [ ] Timeout configurado en queries de DB
- [ ] Paginación obligatoria en endpoints de listado
- [ ] Límite de tamaño en uploads (si aplica)

---

### E — Elevation of Privilege (Escalada de privilegios)

**¿Puede alguien obtener más permisos de los que le corresponden?**

| Escenario | Aplica | Severidad | Control de mitigación |
|-----------|--------|-----------|----------------------|
| IDOR (acceso a recurso ajeno) | Sí/No | Alta/Media/Baja | Verificar owner en cada query |
| Bypass de autorización | Sí/No | Alta/Media/Baja | Middleware de authz en todas las rutas |
| Mass assignment | Sí/No | Alta/Media/Baja | Schema explícito de campos permitidos |
| JWT con rol manipulado | Sí/No | Alta/Media/Baja | Verificar firma del token siempre |

**Controles implementados:**
- [ ] Autorización verificada en cada endpoint (no solo autenticación)
- [ ] Verificación de ownership en recursos (no solo existencia)
- [ ] Schema de input explícito — no bind directo del body al modelo
- [ ] RBAC implementado si hay múltiples roles

---

## Resumen de Riesgos

| Amenaza | Severidad máxima | Controles | Estado |
|---------|-----------------|-----------|--------|
| Spoofing | Alta/Media/Baja | N controles | ✓ Mitigado / ⚠ Parcial / ✗ Pendiente |
| Tampering | Alta/Media/Baja | N controles | ✓ Mitigado / ⚠ Parcial / ✗ Pendiente |
| Repudiation | Alta/Media/Baja | N controles | ✓ Mitigado / ⚠ Parcial / ✗ Pendiente |
| Info Disclosure | Alta/Media/Baja | N controles | ✓ Mitigado / ⚠ Parcial / ✗ Pendiente |
| DoS | Alta/Media/Baja | N controles | ✓ Mitigado / ⚠ Parcial / ✗ Pendiente |
| Elevation | Alta/Media/Baja | N controles | ✓ Mitigado / ⚠ Parcial / ✗ Pendiente |

---

## Riesgos Residuales Aceptados

> Riesgos que se conocen pero se aceptan conscientemente, con justificación.

| Riesgo | Justificación para aceptar | Revisión en |
|--------|---------------------------|-------------|
| [descripción] | [razón: baja prob, control compensatorio, costo] | [fecha o sprint] |

---

## Aprobación del Review

| Campo | Valor |
|-------|-------|
| Aprobado por | [nombre] |
| Fecha | YYYY-MM-DD |
| Observaciones | [si hay riesgos que resolver antes de implementar] |
