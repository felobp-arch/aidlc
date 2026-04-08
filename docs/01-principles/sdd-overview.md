# Spec Driven Development (SDD) — Overview

## ¿Qué es SDD?

Spec Driven Development es una metodología donde **la especificación precede
siempre a la implementación**. El código es consecuencia de un contrato
previamente aprobado, no al revés.

En AIDLC, SDD es el principio fundacional que hace que la colaboración
humano-agente sea predecible y auditable.

---

## El Problema que Resuelve

Sin especificación previa, los agentes de IA tienden a:

- Implementar lo que *creen* que el usuario quiere
- Tomar decisiones de arquitectura sin consultar
- Generar código que no refleja los requisitos reales
- Crear trabajo que luego hay que deshacer

SDD elimina este problema al establecer un contrato claro **antes** de que
el agente escriba la primera línea de código.

---

## El Software Requirements Document (SRD)

El SRD es el artefacto central de SDD. Define:

### Secciones obligatorias

```markdown
# SRD — [Nombre del Proyecto o Feature]

## 1. Contexto
¿Por qué existe este proyecto? ¿Qué problema resuelve?

## 2. Stakeholders
¿Quién lo usa? ¿Quién lo aprueba? ¿Quién lo mantiene?

## 3. Alcance
### En scope:
- Feature 1
- Feature 2

### Fuera de scope (explícito):
- Lo que no se va a hacer

## 4. Requisitos Funcionales
RF-001: El sistema debe...
RF-002: El usuario debe poder...

## 5. Requisitos No Funcionales
Ver docs/03-templates/nfr-requirements-template.md

## 6. Casos de Uso
### CU-001: Nombre del caso de uso
- Actor: ...
- Precondición: ...
- Flujo principal: ...
- Flujos alternativos: ...

## 7. Definition of Done global
- [ ] criterio 1
- [ ] criterio 2

## 8. Aprobaciones
| Stakeholder | Firma | Fecha |
|-------------|-------|-------|
| [nombre] | [ ] | — |
```

---

## Flujo SDD en AIDLC

```
1. HUMANO redacta borrador del SRD
       ↓
2. AGENTE revisa y señala ambigüedades o riesgos
       ↓
3. HUMANO actualiza y aprueba el SRD
       ↓
4. AGENTE propone Execution Plan basado en el SRD
       ↓
5. HUMANO aprueba el plan
       ↓
6. AGENTE implementa — solo lo que dice el SRD
       ↓
7. AGENTE verifica contra Definition of Done del SRD
```

---

## Reglas SDD en AIDLC

1. **Sin SRD no hay código** — el agente se niega a implementar sin SRD aprobado
2. **Cambios de alcance = actualización del SRD** — no workarounds en el código
3. **El agente implementa el SRD, no lo interpreta creativamente**
4. **Ambigüedades se resuelven antes de implementar, no durante**

---

## Beneficios Comprobados

- El humano siempre sabe qué está construyendo el agente
- Los PR son revisables: el código debe corresponder al SRD
- Onboarding más rápido: el SRD explica el *por qué* del código
- Auditoría corporativa: existe trazabilidad requisito → código → test

---

## Recursos

- Plantillas: `docs/03-templates/`
- Caso de estudio: `examples/fhc/`
- Fases de implementación: `docs/02-phases/`
