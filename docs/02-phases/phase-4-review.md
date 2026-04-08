# Fase 4 — Revisión (Code Review)

## Propósito

La revisión garantiza que el código implementado corresponde al SRD y al
Execution Plan aprobados, cumple los estándares de calidad y seguridad,
y está listo para llegar a staging.

---

## Entradas

- PR abierto con implementación completa (Fase 3 completada)
- Execution Plan aprobado (para verificar correspondencia)
- Pipeline en verde

## Salida

- PR aprobado por el humano
- Merge a `main` completado
- Deploy automático a staging activado

---

## Quién Revisa Qué

### El agente hace auto-review antes de abrir el PR:

- [ ] Cada archivo modificado corresponde a un ítem del Execution Plan
- [ ] No hay código que no esté en el plan (sin gold plating)
- [ ] Arquitectura Hexagonal respetada en todos los archivos
- [ ] Sin secrets hardcodeados (revisión de todo el diff)
- [ ] Tests existen para el código nuevo
- [ ] Cobertura ≥ 70% en el módulo nuevo
- [ ] El PR no incluye cambios de archivos no relacionados con la tarea
- [ ] La descripción del PR referencia el SRD y el Execution Plan

### El humano revisa:

- Lógica de negocio: ¿el código hace lo que dice el SRD?
- Decisiones de diseño: ¿hay algo que no le convenza?
- Seguridad: ¿algo parece riesgoso?
- Casos de borde: ¿el código maneja las situaciones edge del SRD?

---

## Template del PR

```markdown
## ¿Qué hace este PR?
[Una oración. Si necesita más, el PR es probablemente demasiado grande.]

## SRD de referencia
RF-001, RF-002 — [link al SRD]

## Execution Plan aprobado
[link al plan o mención de la sesión]

## Cambios principales
- `src/services/X.py` — [descripción]
- `src/api/routes/X.py` — [descripción]
- `tests/unit/test_X.py` — [descripción]

## Definition of Done
- [x] Lógica en services/
- [x] Controller en api/ sin lógica de negocio
- [x] Tests unitarios ≥ 70%
- [x] Tests de integración
- [x] Sin secrets hardcodeados
- [x] Pipeline en verde

## Cómo probar
1. Paso concreto para reproducir el flujo
2. Qué endpoint llamar o qué acción ejecutar
3. Qué respuesta esperar

## Screenshots / evidencia (si aplica)
```

---

## Señales de que el PR NO está listo para merge

- Pipeline en rojo
- Descripción vacía o genérica
- Archivos modificados que no están en el Execution Plan
- Tests que no existen o están ignorados (`@pytest.mark.skip`)
- Secrets detectados en el diff
- Cambios en `main` directamente (sin PR)
- El PR cubre más de un Execution Plan (demasiado grande)

---

## Tamaño de PR Recomendado

| Tipo de cambio | Tamaño máximo recomendado |
|----------------|--------------------------|
| Bug fix | 1 archivo de código + tests |
| Feature pequeña | < 300 líneas modificadas |
| Feature mediana | < 500 líneas — considerar partir |
| Refactor grande | PR separado, sin nueva funcionalidad |

PRs grandes son más difíciles de revisar y más probables de introducir bugs.

---

## Proceso de Revisión

```
1. Agente abre PR con descripción completa
       ↓
2. Pipeline corre automáticamente (CI)
       ↓
3. Humano revisa el diff
       ↓
4a. Sin comentarios → aprueba merge
       ↓
4b. Con comentarios → agente responde o corrige
       ↓
5. Humano aprueba merge
       ↓
6. Merge a main → deploy automático a staging
```

---

## Manejo de Comentarios de Revisión

Cuando el humano deja un comentario en el PR:

1. El agente **lee** el comentario completo antes de responder
2. Si es un cambio pedido: propone el cambio específico para aprobación
3. Si es una pregunta: responde con contexto del Execution Plan o SRD
4. Si el agente no está de acuerdo: argumenta con razones técnicas, no solo acepta
5. El humano tiene la decisión final

---

## Transición a Fase 5

La Fase 4 está completa cuando:

1. El PR tiene aprobación del humano
2. Pipeline está en verde
3. Merge completado a `main`
4. Deploy a staging iniciado automáticamente

**El agente anuncia:** "Merge completado. Deploy a staging en curso (Fase 5)."

---

## Recursos

- Template PR: `templates/.github/PULL_REQUEST_TEMPLATE.md`
- Fase anterior: `docs/02-phases/phase-3-implement.md`
- Siguiente fase: `docs/02-phases/phase-5-deploy.md`
