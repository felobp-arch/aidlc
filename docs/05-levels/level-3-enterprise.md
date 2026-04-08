# Nivel 3 — Enterprise (Corporativo)

> Para organizaciones con regulación formal, múltiples equipos, SLAs con
> consecuencias económicas o legales, y datos sensibles de alto valor.
> Referencia directa: BTG Pactual Colombia bajo SFC Circular 007.

---

## Contexto

El nivel Enterprise está diseñado para entornos donde:

- Hay regulación externa que audita el proceso (SFC, GDPR, SOC2, ISO 27001)
- Los datos manejados son financieros, médicos o de identidad
- El downtime tiene consecuencias legales o económicas directas
- Hay múltiples equipos y squads con diferentes niveles de acceso
- Los cambios en producción requieren aprobación formal documentada

---

## Diferencias clave respecto a Nivel 2

| Control | Nivel 2 | Nivel 3 |
|---------|---------|---------|
| SRD | Obligatorio | Obligatorio + aprobación formal registrada |
| Execution Plan | Completo | Completo + revisión de seguridad obligatoria |
| STRIDE | Para features de seguridad | Para todos los módulos nuevos |
| Aprobaciones | 1 reviewer | 2+ reviewers, roles diferenciados |
| Secrets | Variables del proveedor | Secrets Manager dedicado (Vault, AWS SM) |
| MFA | Recomendado | Obligatorio para todos los accesos |
| Audit logs | Operaciones sensibles | Todas las operaciones + retención regulatoria |
| Pen testing | No | Antes de cada release mayor |
| Incident response | Informal | Runbook formal documentado |
| DR/BCP | No | Plan de Continuidad de Negocio documentado |
| Compliance review | No | Antes de cada release |
| Change management | Informal | Proceso formal con aprobaciones |

---

## Gestión de Identidad y Acceso (IAM)

### Principio de mínimo privilegio

Cada desarrollador, agente y sistema tienen solo los permisos necesarios:

```
Roles en el repositorio:
├── Maintainer   → puede aprobar y mergear PRs a main
├── Developer    → puede crear PRs, no puede mergear a main directamente
├── Reviewer     → puede aprobar PRs, no puede pushear
└── Read-only    → acceso de solo lectura (auditores, stakeholders)

Roles en producción:
├── SRE / Ops    → puede aprobar deploy a producción
├── Developer    → puede aprobar deploy a staging
└── Agente IA    → sin acceso directo a producción (nunca)
```

### MFA Obligatorio

- Obligatorio para acceso a GitHub en la organización
- Obligatorio para acceso a consola de cloud (AWS, GCP, Azure)
- Obligatorio para acceso a base de datos de producción
- Obligatorio para acceso al secrets manager

---

## Gestión de Secrets (Enterprise)

```
Environments:
├── local      → .env (en .gitignore)
├── staging    → Vault namespace: staging/ o AWS SM con prefijo /app/staging/
└── production → Vault namespace: prod/   o AWS SM con prefijo /app/prod/

Rotación:
├── API keys de terceros    → rotar cada 90 días
├── JWT signing keys        → rotar cada 30 días
├── DB credentials          → rotar cada 60 días
└── Secrets comprometidos   → rotar inmediatamente + audit

Acceso:
├── Secreto de producción   → solo rol SRE/Ops, nunca developers
├── Secreto de staging      → developers autorizados
└── Agente IA               → nunca acceso directo a secrets de producción
```

---

## Change Management

Proceso formal para cambios en producción:

```
1. SRD aprobado por Product Owner
       ↓
2. Execution Plan aprobado por Tech Lead
       ↓
3. Revisión STRIDE completada (si aplica)
       ↓
4. Implementación en feature branch
       ↓
5. Code review (mínimo 2 aprobaciones: 1 dev senior + 1 security/compliance)
       ↓
6. Deploy a staging con validation checklist
       ↓
7. Change Request formal para producción (ticket en sistema de gestión)
       ↓
8. Ventana de mantenimiento o aprobación del Change Advisory Board (CAB)
       ↓
9. Deploy a producción con aprobación en GitHub Actions
       ↓
10. Monitoreo post-deploy (mínimo 24h para cambios críticos)
        ↓
11. Cierre del Change Request con evidencia
```

---

## Audit Logging (Nivel Enterprise)

### Qué se registra

Todo. Sin excepciones.

| Categoría | Eventos |
|-----------|---------|
| Autenticación | Login, logout, fallo de auth, MFA, cambio de password |
| Datos de usuario | Creación, lectura, modificación, eliminación |
| Datos financieros | Cualquier acceso, modificación o exportación |
| Administración | Cambios de permisos, creación de roles, configuración |
| Sistema | Deploy, rollback, cambios de configuración, inicio/parada |
| Seguridad | Intentos fallidos de autorización, anomalías detectadas |

### Estructura del audit log

```json
{
  "event_id": "uuid-v4",
  "timestamp": "2026-04-07T14:30:00.000Z",
  "event_type": "user.data.read",
  "severity": "INFO",
  "actor": {
    "user_id": "usr_123",
    "role": "customer",
    "ip_address": "190.x.x.x",
    "user_agent": "Mozilla/5.0...",
    "session_id": "sess_456"
  },
  "resource": {
    "type": "financial_account",
    "id": "acc_789",
    "owner_id": "usr_123"
  },
  "result": "success",
  "metadata": {
    "request_id": "req_abc",
    "service": "account-service",
    "version": "1.2.3"
  }
}
```

### Retención

| Tipo de log | Retención mínima | Base regulatoria |
|-------------|-----------------|-----------------|
| Audit logs generales | 5 años | SFC Circular 007 |
| Logs de acceso a datos financieros | 10 años | SFC Colombia |
| Logs de autenticación | 2 años | Buena práctica |
| Logs de sistema | 1 año | Buena práctica |

---

## SFC Circular 007 — Requisitos Técnicos

Para entidades bajo supervisión de la SFC Colombia:

### Seguridad de la información

- [ ] Política de seguridad de la información documentada y aprobada
- [ ] Clasificación de datos (público, interno, confidencial, restringido)
- [ ] Cifrado en tránsito: TLS 1.2+ obligatorio
- [ ] Cifrado en reposo: AES-256 para datos clasificados como confidencial o restringido
- [ ] Control de acceso basado en roles (RBAC) documentado

### Gestión de incidentes

- [ ] Procedimiento de respuesta a incidentes documentado
- [ ] Contactos de escalamiento definidos
- [ ] Notificación a la SFC dentro de los plazos reglamentarios
- [ ] Post-mortem documentado para incidentes mayores

### Continuidad del negocio

- [ ] Plan de Continuidad de Negocio (BCP) documentado
- [ ] RTO y RPO definidos y probados
- [ ] Backups automatizados con prueba de restauración periódica
- [ ] Ambiente alterno documentado

### Gestión de terceros

- [ ] Inventario de proveedores de tecnología críticos
- [ ] Contratos con cláusulas de seguridad y SLA
- [ ] Due diligence de seguridad para proveedores cloud

---

## Monitoreo y Observabilidad (Enterprise)

```
Métricas de negocio    → Dashboard ejecutivo
Métricas técnicas      → Dashboard de ingeniería (Grafana / Datadog)
Logs estructurados     → Centralización (ELK, Splunk, CloudWatch)
Alertas               → PagerDuty con escalamiento definido
APM / Tracing         → OpenTelemetry + backend (Jaeger, Datadog APM)
Anomaly detection     → Para logs de seguridad y transacciones
```

### SLAs típicos para nivel enterprise

| Métrica | Objetivo |
|---------|---------|
| Disponibilidad | ≥ 99.9% mensual |
| Latencia p95 | ≤ 300ms |
| Latencia p99 | ≤ 1s |
| MTTR (Mean Time to Recovery) | ≤ 4 horas |
| RTO | ≤ 4 horas |
| RPO | ≤ 1 hora |

---

## El Agente en Contexto Enterprise

En nivel Enterprise, el agente opera con restricciones adicionales:

- **No tiene acceso** a credenciales de producción
- **No puede** iniciar deploys a producción
- **Debe** presentar revisión STRIDE en todo Execution Plan
- **Debe** documentar compliance con SFC Circular 007 en features que manejen datos financieros
- **Debe** esperar dos aprobaciones humanas (no una) antes de implementar features críticas
- **Genera** evidencia de revisión para auditorías (audit trail de las sesiones de trabajo)

---

## Recursos

- Nivel anterior: `docs/05-levels/level-2-team.md`
- Security by Design: `docs/01-principles/security-by-design.md`
- Template STRIDE: `docs/03-templates/stride-review-template.md`
- Template NFR: `docs/03-templates/nfr-requirements-template.md`
- Pipeline CI: `templates/.github/workflows/ci-security.yml`
- Release workflow: `templates/.github/workflows/release.yml`
