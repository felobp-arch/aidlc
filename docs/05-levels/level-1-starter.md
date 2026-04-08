# Nivel 1 — Starter (Uso Personal)

> Para desarrolladores individuales, proyectos personales, freelance y
> prototipos. El nivel base de AIDLC: estructura mínima que vale la pena
> tener desde el día uno.

---

## Contexto

El nivel Starter está diseñado para proyectos donde:

- Hay un solo desarrollador (tú + el agente)
- No hay equipo que onboardear
- El riesgo de un bug en producción es recuperable
- No hay regulación formal que cumplir

Aún así, algunos principios son no-negociables incluso en este nivel.

---

## Lo que es obligatorio en Nivel 1

### No-negociables (aplican en todos los niveles)

- **Sin secrets hardcodeados** — `.env` desde el día uno
- **Git desde el inicio** — no hay "lo agrego después"
- **El agente propone plan antes de código** — aunque sea breve
- **Tests para la lógica crítica** — al menos los casos de error
- **Push directo a main solo con cuidado** — preferible usar branches incluso solo

### Estructura mínima

```
project/
├── src/
│   ├── services/     # lógica aquí — aunque sea pequeña
│   └── api/          # endpoints separados de la lógica
├── tests/
├── .env.example      # documentar qué variables necesita el proyecto
├── .gitignore        # incluir .env y archivos de credenciales
└── README.md         # cómo correr el proyecto localmente
```

---

## Lo que es opcional en Nivel 1

| Componente | Opcional | Recomendado cuando |
|------------|---------|-------------------|
| CI/CD completo | Sí | El proyecto crece o hay deploys frecuentes |
| Staging environment | Sí | Hay usuarios externos |
| Coverage 70% | Sí | Código que no cambia seguido |
| SRD formal | Sí | La idea está clara en tu cabeza |
| Execution Plan formal | Sí | Pero el agente lo hace igual, aunque sea corto |
| Audit logs | Sí | Si hay más de un usuario |
| Rate limiting | Sí | Si el endpoint es público |
| Docker | Sí | Útil desde el inicio para consistency |

---

## Flujo Simplificado para Nivel 1

```
HUMANO: "Quiero hacer X"
    └── AGENTE: propone qué archivos toca y qué tests escribe (5 min)
        └── HUMANO: "adelante" o ajusta el plan
            └── AGENTE: implementa
                └── AGENTE: verifica que los tests pasan
                    └── HUMANO: revisa y mergea
```

No hay aprobación formal de staging. No hay environment protection rules.
Pero el agente igual presenta el plan y espera el "adelante".

---

## Checklist de Inicio de Proyecto (Nivel 1)

- [ ] `git init` y primer commit vacío con `.gitignore`
- [ ] `.env.example` con todas las variables necesarias (sin valores reales)
- [ ] `README.md` con: qué hace el proyecto, cómo correrlo
- [ ] Estructura `src/services/` y `src/api/` desde el inicio
- [ ] Al menos un test de humo que verifica que el servidor arranca

---

## Cuándo Subir a Nivel 2

Es momento de pasar a Nivel 2 cuando:

- Hay otra persona usando el código (aunque sea un solo usuario externo)
- Alguien más va a contribuir al repositorio
- Hay datos reales de usuarios (aunque sean de un amigo)
- El downtime empieza a tener consecuencias

No esperes a que sea un problema — sube de nivel antes.

---

## Herramientas Recomendadas para Nivel 1

| Propósito | Opción simple |
|-----------|--------------|
| Hosting | Railway, Render, Fly.io |
| DB | Supabase (free tier) |
| Secrets en deploy | Variables del proveedor (Railway, Render) |
| CI básico | GitHub Actions (template incluido en AIDLC) |
| Monitoreo | UptimeRobot (free) |

---

## Recursos

- Siguiente nivel: `docs/05-levels/level-2-team.md`
- Playbook Python/FastAPI: `docs/04-playbooks/python-fastapi/README.md`
- Playbook Node.js: `docs/04-playbooks/nodejs/README.md`
