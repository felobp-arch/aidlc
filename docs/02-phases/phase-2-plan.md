# Fase 2 — Plan (Execution Plan)

## Propósito

El Execution Plan traduce el SRD aprobado en un **plan técnico concreto**
que el humano revisa y aprueba antes de que el agente escriba código.

Es el contrato técnico entre el agente y el desarrollador.

---

## Entradas

- SRD aprobado (Fase 1 completada)
- Contexto del stack tecnológico del proyecto
- Estado actual del repositorio
- Lecciones aprendidas relevantes de `lessons-learned.md`

## Salida

- Execution Plan documentado y aprobado por el humano
- Definition of Done específica para la feature o módulo
- Lista de riesgos conocidos con mitigaciones

---

## Estructura del Execution Plan

```markdown
## Execution Plan — [Nombre de la Feature]

**SRD de referencia:** [RF-001, RF-002]
**Fecha:** YYYY-MM-DD
**Estimación:** [N horas / N sesiones]

### Objetivo
[Qué se implementa y por qué — una oración]

### Archivos a modificar
- [ ] `src/services/nombre.py` — agregar lógica X
- [ ] `src/api/routes/nombre.py` — exponer endpoint Y

### Archivos a crear
- [ ] `src/services/nuevo.py` — servicio para Z
- [ ] `tests/unit/test_nuevo.py` — tests unitarios

### Dependencias a instalar
- `paquete==1.2.3` — justificación de por qué este paquete

### Migraciones de base de datos
- [ ] Agregar tabla `nombre` con campos: id, campo1, campo2
- [ ] Índice en `campo1` para queries de búsqueda

### Análisis STRIDE (si aplica)
- S: [mitigación]
- T: [mitigación]
- I: [mitigación]

### Riesgos identificados
- **Riesgo 1**: descripción — Mitigación: acción concreta
- **Riesgo 2**: descripción — Mitigación: acción concreta

### Tests necesarios
- [ ] Unitario: `test_caso_exitoso` — verifica flujo principal
- [ ] Unitario: `test_caso_error` — verifica manejo de error
- [ ] Integración: `test_endpoint_X` — verifica respuesta HTTP

### Definition of Done
- [ ] Lógica en `services/` sin imports de framework
- [ ] Controller en `api/` solo orquesta, no tiene lógica
- [ ] Tests unitarios ≥ 70% cobertura del módulo nuevo
- [ ] Tests de integración para endpoints nuevos
- [ ] Sin secrets hardcodeados
- [ ] Pipeline en verde
- [ ] Documentación de endpoint actualizada (si aplica)
```

---

## Rol del Agente en esta Fase

1. **Analiza** el SRD y el estado actual del repo
2. **Consulta** `lessons-learned.md` — ¿hay patrones conocidos que apliquen?
3. **Redacta** el Execution Plan completo
4. **Presenta** el plan al humano para revisión
5. **Espera aprobación** — no toca código hasta recibirla
6. **Incorpora feedback** si el humano pide ajustes al plan

---

## Checklist del Agente antes de Presentar el Plan

- [ ] El plan cubre todos los RF del SRD referenciados
- [ ] No hay acciones en zona roja sin mención explícita
- [ ] Los riesgos están identificados, no ignorados
- [ ] La Definition of Done es verificable (no subjetiva)
- [ ] Los tests están especificados, no solo mencionados
- [ ] Se consultaron `lessons-learned.md` — nada nuevo que ya se resolvió antes

---

## Señales de que el Plan NO está listo para aprobación

- "Modificar lo necesario" — no es específico
- No menciona tests
- No identifica riesgos (todos los cambios tienen algún riesgo)
- La Definition of Done tiene ítems no verificables como "funciona bien"
- Cubre más del doble de lo que pide el SRD (gold plating)

---

## Transición a Fase 3

La Fase 2 está completa cuando:

1. El Execution Plan está documentado
2. El humano dice "aprobado" o equivalente
3. No hay ítems abiertos en el plan que requieran más clarificación

**El agente anuncia:** "Plan aprobado. Iniciando implementación (Fase 3)."

---

## Recursos

- Plantilla completa: `docs/03-templates/execution-plan-template.md`
- Plantilla STRIDE: `docs/03-templates/stride-review-template.md`
- Fase anterior: `docs/02-phases/phase-1-spec.md`
- Siguiente fase: `docs/02-phases/phase-3-implement.md`
