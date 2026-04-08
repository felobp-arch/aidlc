# Human-in-the-Loop — Principio AIDLC

## Definición

Human-in-the-Loop (HITL) es el principio que define **qué decisiones toma el
agente de forma autónoma y cuáles requieren aprobación humana explícita**.

En AIDLC, el agente nunca opera en modo "autopilot" en decisiones de alto impacto.
La autonomía del agente está calibrada según la reversibilidad y el blast radius
de cada acción.

---

## Mapa de Decisiones

### Zona Verde — Agente autónomo

Acciones que el agente puede tomar sin consultar:

| Acción | Justificación |
|--------|---------------|
| Leer cualquier archivo del repo | Solo lectura, sin efecto |
| Proponer código en respuesta | No ejecuta nada |
| Crear archivos locales | Reversible con git |
| Instalar dependencias de dev | Reversible |
| Ejecutar tests locales | Sin efectos externos |
| Buscar en documentación | Informativo |

### Zona Amarilla — Agente propone, humano aprueba

Acciones que requieren aprobación antes de ejecutarse:

| Acción | Por qué se requiere aprobación |
|--------|-------------------------------|
| Execution Plan completo | Define el rumbo de la implementación |
| Crear branch de feature | Impacta el flujo de git |
| Instalar dependencias de producción | Afecta el build final |
| Modificar configuración de CI/CD | Afecta todo el equipo |
| Hacer commit | Punto de no retorno fácil |
| Push a rama remota | Visible para el equipo |

### Zona Roja — Humano siempre

Acciones donde el agente **nunca actúa**, solo propone o ejecuta si hay
confirmación explícita:

| Acción | Consecuencias si se hace mal |
|--------|------------------------------|
| Merge a `main` | Afecta a todos los usuarios |
| Deploy a producción | Usuarios reales, rollback costoso |
| Borrar branches o archivos | Pérdida de trabajo |
| Modificar secrets o credenciales | Riesgo de seguridad |
| Cambios a base de datos en producción | Pérdida de datos potencial |
| Modificar permisos de IAM/roles | Brecha de seguridad |

---

## El Flujo HITL en la Práctica

```
AGENTE lee contexto
    └── AGENTE propone Execution Plan
        └── HUMANO revisa y aprueba / rechaza / modifica
            └── AGENTE implementa (zona verde)
                └── AGENTE presenta resultado para revisión
                    └── HUMANO aprueba PR
                        └── Pipeline automático → staging
                            └── HUMANO aprueba deploy → producción
```

---

## Por qué HITL en IA

Los agentes de IA, incluso los más capaces, pueden:

- **Malinterpretar requisitos ambiguos** — y construir algo correcto pero incorrecto
- **Optimizar la métrica equivocada** — código que pasa tests pero no resuelve el problema
- **Introducir sesgos de su entrenamiento** — patrones que funcionan en general pero no aquí
- **Perder contexto de negocio** — lo técnicamente correcto puede ser comercialmente incorrecto

El humano en el loop no es una limitación — es el mecanismo que convierte
al agente en un multiplicador de productividad en lugar de un punto de falla.

---

## HITL en Contexto Corporativo (BTG Pactual)

En entornos financieros regulados, HITL tiene implicaciones de compliance:

- **Audit trail**: cada aprobación humana debe quedar registrada
- **Segregación de funciones**: quien propone no puede aprobar sin revisión
- **SFC Circular 007**: controles de cambios requieren aprobación formal
- **No-repudiación**: las aprobaciones deben ser trazables a personas reales

---

## Señales de Alerta

Si el agente está haciendo alguna de estas cosas, algo está mal:

- Actúa en zona roja sin haber recibido aprobación
- No presenta Execution Plan y va directo al código
- Hace múltiples cambios sin checkpoints intermedios
- No documenta lo que hizo al cerrar la sesión

En ese caso: detener la sesión, revisar CLAUDE.md, reiniciar el protocolo.
