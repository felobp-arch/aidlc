# NFR — Non-Functional Requirements Template

> Los requisitos no funcionales definen las restricciones de calidad del sistema:
> rendimiento, seguridad, disponibilidad, escalabilidad, mantenibilidad.
> Incluir en el SRD antes de aprobar la Fase 1.

---

## NFR — [Nombre del Proyecto]

**Versión:** 1.0  
**Fecha:** YYYY-MM-DD  
**SRD de referencia:** [link]

---

## 1. Rendimiento (Performance)

| ID | Requisito | Valor objetivo | Cómo medirlo |
|----|-----------|---------------|--------------|
| NFR-P01 | Latencia p95 de endpoints principales | ≤ 300ms | APM / logs de request |
| NFR-P02 | Latencia p99 de endpoints principales | ≤ 1000ms | APM / logs de request |
| NFR-P03 | Throughput mínimo | [N req/s] | Load test con k6 / Locust |
| NFR-P04 | Tiempo de respuesta en cold start | ≤ [N]s | Medición en staging |

**Baseline de carga:**
- Usuarios concurrentes esperados: [N]
- Pico de tráfico: [N req/min]
- Distribución geográfica: [Colombia / LATAM / Global]

---

## 2. Disponibilidad (Availability)

| ID | Requisito | Valor objetivo | Notas |
|----|-----------|---------------|-------|
| NFR-A01 | Uptime mensual | ≥ 99.5% | ~3.6h downtime/mes permitido |
| NFR-A02 | Uptime para tier corporativo | ≥ 99.9% | ~43min downtime/mes |
| NFR-A03 | RTO (Recovery Time Objective) | ≤ [N] horas | Tiempo para restaurar servicio |
| NFR-A04 | RPO (Recovery Point Objective) | ≤ [N] horas | Pérdida máxima de datos aceptable |

**Mantenimientos programados:**
- Ventana permitida: [horario] — [días]
- Notificación previa mínima: [N] horas

---

## 3. Seguridad (Security)

| ID | Requisito | Obligatorio | Notas |
|----|-----------|------------|-------|
| NFR-S01 | TLS 1.2+ en todos los endpoints | Sí | Sin HTTP plano en staging/prod |
| NFR-S02 | Autenticación en todos los endpoints privados | Sí | JWT o equivalente |
| NFR-S03 | Rate limiting en endpoints públicos | Sí | [N req/min por IP] |
| NFR-S04 | Audit log para operaciones sensibles | Sí | Retención: [N] años |
| NFR-S05 | Sin secrets en código o repositorio | Sí | Pre-commit hook activo |
| NFR-S06 | Dependencias auditadas antes de deploy | Sí | pip-audit / npm audit |
| NFR-S07 | MFA habilitado | Por rol/env | Obligatorio para admins en prod |
| NFR-S08 | Cifrado en reposo para datos sensibles | Sí (si aplica) | AES-256 o equivalente |
| NFR-S09 | Revisión STRIDE por módulo de seguridad | Sí | Ver template STRIDE |
| NFR-S10 | SFC Circular 007 | Si datos financieros | Ver docs/01-principles/security-by-design.md |

---

## 4. Escalabilidad (Scalability)

| ID | Requisito | Valor | Notas |
|----|-----------|-------|-------|
| NFR-SC01 | Escala horizontal sin cambios de código | Sí/No | Stateless requerido |
| NFR-SC02 | Crecimiento de usuarios esperado a 12 meses | [N]x | Base para capacity planning |
| NFR-SC03 | Crecimiento de datos esperado a 12 meses | [N] GB/año | Base para DB sizing |
| NFR-SC04 | Auto-scaling configurado | Sí/No | Min: [N] / Max: [N] instancias |

---

## 5. Mantenibilidad (Maintainability)

| ID | Requisito | Valor objetivo | Cómo medirlo |
|----|-----------|---------------|--------------|
| NFR-M01 | Cobertura de tests | ≥ 70% | pytest-cov / jest --coverage |
| NFR-M02 | Tiempo para reproducir el entorno local | ≤ 30 min | Documentación + scripts |
| NFR-M03 | Tiempo para onboarding de nuevo dev | ≤ 1 día | README completo |
| NFR-M04 | Complejidad ciclomática máxima por función | ≤ 10 | ruff / ESLint |
| NFR-M05 | Documentación de APIs actualizada | Siempre | OpenAPI / Swagger auto-gen |

---

## 6. Observabilidad (Observability)

| ID | Requisito | Obligatorio | Herramienta sugerida |
|----|-----------|------------|---------------------|
| NFR-O01 | Logs estructurados en JSON | Sí | structlog / winston |
| NFR-O02 | Logs con nivel configurable por env | Sí | DEBUG local, INFO staging, WARNING prod |
| NFR-O03 | Health check endpoint | Sí | `GET /health` → 200 OK |
| NFR-O04 | Métricas de aplicación expuestas | Recomendado | Prometheus / Datadog |
| NFR-O05 | Tracing distribuido | Para nivel enterprise | OpenTelemetry |
| NFR-O06 | Alertas configuradas para errores críticos | Sí en prod | PagerDuty / Slack |

---

## 7. Cumplimiento (Compliance)

| ID | Requisito | Aplica cuando | Referencia |
|----|-----------|--------------|------------|
| NFR-C01 | SFC Circular 007 | Datos financieros de usuarios colombianos | SFC Colombia |
| NFR-C02 | Ley 1581 de 2012 (Habeas Data) | Datos personales de colombianos | SIC Colombia |
| NFR-C03 | Retención de logs de auditoría | NFR-S04 | Según regulación: mín. 5 años SFC |
| NFR-C04 | Procedimiento de respuesta a incidentes | Siempre en prod | Ver runbook de seguridad |

---

## 8. Compatibilidad (Compatibility)

| ID | Requisito | Valor |
|----|-----------|-------|
| NFR-CO01 | Browsers soportados (si hay frontend) | Chrome, Firefox, Edge — últimas 2 versiones |
| NFR-CO02 | Versión mínima de Python/Node/etc. | [versión] |
| NFR-CO03 | Versión mínima de SO para containers | Linux/amd64 |
| NFR-CO04 | Compatibilidad de API (versioning) | Semver — breaking changes en major version |

---

## 9. Restricciones de Deployment

| ID | Restricción | Detalle |
|----|-------------|---------|
| NFR-D01 | Environments obligatorios | local → staging → production |
| NFR-D02 | Deploy a producción | Aprobación manual requerida |
| NFR-D03 | Push directo a main | Bloqueado técnicamente |
| NFR-D04 | Containerización | Docker obligatorio para staging y prod |
| NFR-D05 | Infraestructura como código | Terraform / Pulumi (si aplica) |

---

## Aprobación

| Campo | Valor |
|-------|-------|
| Aprobado por | [nombre del stakeholder técnico] |
| Fecha | YYYY-MM-DD |
| Observaciones | [si algún NFR requiere negociación] |

---

> Los NFR aprobados aquí son restricciones, no aspiraciones.
> Si un requisito no puede cumplirse, se documenta como riesgo en el SRD
> antes de iniciar la implementación — no después.
