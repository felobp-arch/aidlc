# Fase 1 — Especificación (Spec)

## Propósito

La fase de especificación produce el **Software Requirements Document (SRD)**
aprobado por el stakeholder. Es el único artefacto que autoriza al agente a
continuar a la Fase 2.

Sin SRD aprobado, el agente no avanza.

---

## Entradas

- Idea o problema del stakeholder (conversación, documento, ticket)
- Contexto de negocio
- Restricciones conocidas (tecnología, presupuesto, regulación)

## Salida

- SRD firmado / aprobado en el PR o en el documento
- Criterios de aceptación claros para cada requisito funcional
- Definition of Done global del proyecto

---

## Rol del Agente en esta Fase

El agente **no decide**, **facilita**:

1. **Escucha** el problema del stakeholder
2. **Hace preguntas** para resolver ambigüedades antes de que se conviertan en bugs
3. **Propone estructura** del SRD con secciones pre-llenadas
4. **Señala riesgos y dependencias** que el stakeholder puede no haber considerado
5. **Redacta el SRD** con el input del humano
6. **Espera aprobación** — no continúa hasta recibirla

---

## Checklist de Fase 1

### El agente verifica que el SRD incluye:

- [ ] Contexto de negocio claro
- [ ] Stakeholders identificados (usuario, aprobador, mantenedor)
- [ ] Alcance definido — incluyendo **lo que NO está en scope** (explícito)
- [ ] Requisitos funcionales numerados (RF-001, RF-002...)
- [ ] Requisitos no funcionales (performance, seguridad, disponibilidad)
- [ ] Casos de uso para cada flujo principal
- [ ] Casos de borde identificados
- [ ] Definition of Done global
- [ ] Aprobación del stakeholder registrada

### El agente también verifica:

- [ ] ¿Hay dependencias externas (APIs, servicios de terceros)?
- [ ] ¿Hay requisitos de compliance (SFC Circular 007, GDPR, etc.)?
- [ ] ¿El alcance es implementable en un sprint razonable o hay que partir?

---

## Preguntas Clave que el Agente Debe Hacer

Cuando el stakeholder presenta un requisito ambiguo:

```
- "¿Qué pasa si [caso de borde]?"
- "¿Este requisito aplica para todos los roles o solo para [rol específico]?"
- "¿Cuál es el comportamiento esperado cuando [condición de error]?"
- "¿Este feature reemplaza algo existente o es adicional?"
- "¿Hay un deadline o restricción de tiempo que afecte el alcance?"
```

---

## Señales de que el SRD NO está listo para aprobar

- Contiene frases como "el sistema debería poder" o "quizás" — son ambigüedades
- Los casos de borde no están definidos
- No hay criterios de aceptación medibles
- El alcance no tiene límite explícito ("y lo que haga falta")
- No hay stakeholder identificado que pueda aprobar

---

## Tiempo Esperado

| Complejidad del proyecto | Tiempo típico de Fase 1 |
|--------------------------|------------------------|
| Feature pequeña | 30 min — 1 sesión |
| Módulo mediano | 1-2 sesiones |
| Proyecto completo | 2-5 sesiones |

---

## Transición a Fase 2

La Fase 1 está completa cuando:

1. El SRD existe como documento en el repositorio
2. El stakeholder ha aprobado explícitamente ("aprobado", "listo", "adelante")
3. No quedan ambigüedades abiertas en los requisitos

**El agente anuncia:** "SRD aprobado. Listo para proponer Execution Plan (Fase 2)."

---

## Recursos

- Principio SDD: `docs/01-principles/sdd-overview.md`
- Plantilla NFR: `docs/03-templates/nfr-requirements-template.md`
- Siguiente fase: `docs/02-phases/phase-2-plan.md`
