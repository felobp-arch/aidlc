# How Agents Learn — AIDLC

Este documento describe cómo un agente de IA acumula y aplica conocimiento
dentro del framework AIDLC.

## Principio Base

El agente no empieza cada sesión desde cero. Consulta memoria persistente,
lecciones documentadas y patrones conocidos antes de proponer cualquier solución.

## Ciclo de Aprendizaje

```
Sesión de trabajo
    └── Error o decisión relevante identificada
        └── Documentación en lessons-learned.md
            └── Si el patrón se repite → se convierte en skill
                └── El skill se agrega a docs/06-learning/
                    └── Próxima sesión: el agente lo consulta primero
```

## Qué Documenta el Agente

- Errores técnicos resueltos (con causa raíz, no solo el fix)
- Decisiones de arquitectura tomadas y por qué
- Patrones que funcionaron bien en este proyecto
- Patrones que fallaron y no deben repetirse

## Qué NO Documenta el Agente

- Código que ya está en el repositorio (git es la fuente de verdad)
- Información temporal o de una sola sesión
- Duplicados de lo ya registrado en MEMORY.md

## Actualización de MEMORY.md

Al cierre de cada sesión el agente:

1. Revisa qué fue relevante en la sesión
2. Agrega entradas nuevas a `MEMORY.md` si aplica
3. Actualiza entradas existentes que hayan cambiado
4. Elimina entradas obsoletas

## Consulta Obligatoria Pre-Solución

Antes de proponer una solución técnica, el agente consulta en este orden:

1. `lessons-learned.md` — ¿ya se resolvió algo similar?
2. `error-patterns.md` — ¿este es un patrón conocido?
3. `MEMORY.md` — ¿hay contexto de proyecto relevante?
