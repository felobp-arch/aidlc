# Skill Creation — AIDLC

Protocolo para convertir patrones de error recurrentes en skills reutilizables.

## ¿Cuándo Crear un Skill?

Un patrón se convierte en skill cuando:

- El mismo tipo de error aparece **más de una vez** en `error-patterns.md`
- La solución es **generalizable** a otros proyectos o contextos
- El agente puede aplicarla **de forma autónoma** sin consultar al humano

## Estructura de un Skill

```markdown
# Skill: [nombre-del-skill]

## Problema que resuelve
Descripción concisa del patrón de error o situación que activa este skill.

## Cuándo aplicar
Condiciones específicas que indican que este skill es relevante.

## Pasos
1. Paso concreto
2. Paso concreto
3. Verificación

## Ejemplo
Caso real donde se aplicó por primera vez.

## Origen
Fecha y contexto donde se identificó el patrón.
```

## Proceso de Creación

1. Identificar el patrón en `error-patterns.md`
2. Verificar que ocurrió más de una vez
3. Redactar el skill siguiendo la estructura anterior
4. Guardar en `docs/06-learning/skills/[nombre-del-skill].md`
5. Agregar referencia en `MEMORY.md`

## Ciclo de Vida de un Skill

- **Activo**: se consulta y aplica regularmente
- **Obsoleto**: el contexto cambió, se marca como deprecated
- **Eliminado**: si contradice reglas actuales del framework

Un skill obsoleto nunca se elimina sin antes documentar por qué dejó de aplicar.
