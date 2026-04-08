# AIDLC — AI-Driven Development Lifecycle

> Framework v0.1 Beta para desarrollo profesional de software con IA,  
> basado en Spec Driven Development (SDD).

## ¿Qué es AIDLC?

AIDLC es un framework de metodología y principios que define **cómo colaborar
con agentes de IA** para construir software de calidad profesional, desde
proyectos personales hasta entornos corporativos (probado en BTG Pactual Colombia).

No es una librería ni un CLI. Es un conjunto de **reglas, flujos y contratos**
que hacen que el desarrollo con IA sea predecible, seguro y auditable.

## Inspiración

- [Gentleman-Programming](https://github.com/gentleman-programming) —
  agent-teams-lite, Gentleman-Skills, Engram
- Spec Driven Development (SDD)
- Arquitectura Hexagonal

## Principios del Framework

| # | Regla | Descripción |
|---|-------|-------------|
| 1 | **Pipeline verde** | Nunca commitear sin CI/CD en verde |
| 2 | **No hardcode secrets** | Variables de entorno o vault siempre |
| 3 | **SRD primero** | Ningún proyecto empieza sin Software Requirements Document aprobado |
| 4 | **Execution Plan** | El agente propone plan antes de escribir código |
| 5 | **Definition of Done** | Definida antes de cada feature |
| 6 | **Seguridad corporativa** | Rate limiting, audit logs, MFA configurable, STRIDE, SFC Circular 007 |
| 7 | **Arquitectura Hexagonal** | `services/` lógica · `api/` controllers · `db/` adapters |
| 8 | **Testing mínimo 70%** | Cobertura obligatoria antes de merge |
| 9 | **Tres environments** | `local` → `staging` → `production` |
| 10 | **Aprendizaje continuo** | El agente documenta errores y actualiza MEMORY.md tras cada sesión |

## Flujo de Trabajo

```
HUMANO:  Aprueba SRD
             └── AGENTE:  Propone Execution Plan
                 └── HUMANO:  Aprueba o rechaza el plan
                     └── AGENTE:  Implementa código + tests
                         └── AGENTE:  Verifica pipeline verde
                             └── HUMANO:  Revisa PR y aprueba merge
                                 └── AGENTE:  Deploy a staging automático
                                     └── HUMANO:  Aprueba deploy a producción
```

## Agnóstico al Stack

Los principios aplican a cualquier lenguaje o framework.  
El caso de estudio de referencia es **Financial Health Companion (FHC)**.

## Environments

| Environment | Propósito |
|-------------|-----------|
| `local` | Desarrollo y experimentación |
| `staging` | QA, integración, demos |
| `production` | Usuarios reales, auditoría activa |

## Estado

`v0.1 Beta` — En desarrollo activo.

## Licencia

MIT
