# MEMORY.md — Memoria Persistente del Proyecto AIDLC

Este archivo es mantenido por el agente. Se actualiza al cerrar cada sesión.
Es la fuente de verdad sobre el estado actual del proyecto.

---

## Estado General

| Campo | Valor |
|-------|-------|
| Versión | v0.1 Beta |
| Fase actual | Estructura inicial creada |
| Última sesión | 2026-04-07 |
| Próxima acción | Definir SRD para primer caso de uso real |

---

## Decisiones Tomadas

| Fecha | Decisión | Razón |
|-------|----------|-------|
| 2026-04-07 | Framework agnóstico al lenguaje | Aplicar a cualquier stack |
| 2026-04-07 | Arquitectura Hexagonal como estándar | Separación de concerns, testabilidad |
| 2026-04-07 | Staging obligatorio antes de producción | Evitar regresiones en prod |
| 2026-04-07 | Aprobación manual en deploy a producción | Control humano en momentos críticos |

---

## Contexto Activo

- **Caso de estudio**: Financial Health Companion (FHC) — `examples/fhc/`
- **Audiencia objetivo**: Personal → Equipos → Corporativo (BTG Pactual Colombia)
- **Playbooks disponibles**: Python/FastAPI, Node.js
- **Compliance relevante**: SFC Circular 007 (datos financieros Colombia)

---

## Patrones Conocidos

> Se puebla con el uso. Ver `docs/06-learning/error-patterns.md` para el catálogo.

---

## Skills Disponibles

> Se puebla cuando se generan skills reutilizables. Ver `docs/06-learning/skill-creation.md`.

---

## Pendientes del Framework

- [ ] Agregar playbook para React/Next.js
- [ ] Agregar playbook para infraestructura (Terraform/Pulumi)
- [ ] Completar SRD del caso de estudio FHC
- [ ] Definir plantilla de onboarding para nivel corporativo
- [ ] Agregar ejemplos de STRIDE completados

---

## Historial de Sesiones

| Fecha | Resumen | Agente |
|-------|---------|--------|
| 2026-04-07 | Estructura inicial del framework creada. README, FOUNDATION, docs completos. | Claude Sonnet 4.6 |
