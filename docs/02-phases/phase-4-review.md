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

- PR mergeado por el agente (si CI verde y sin comentarios pendientes)
- Branch de feature eliminado
- `develop` actualizado

---

## Protocolo Completo de PR — Comandos Exactos

El agente gestiona el ciclo completo sin intervención humana, salvo la
aprobación explícita inicial del Execution Plan.

```bash
# 1. Push del branch al remoto
git push -u origin feat/nombre-feature

# 2. Abrir PR (respetar flujo: feat/* → develop → staging → main)
gh pr create \
  --base develop \
  --head feat/nombre-feature \
  --title "tipo(scope): descripción" \
  --body "$(cat <<'BODY'
## Resumen
[bullets del qué y el por qué]

## Execution Plan ejecutado
[link o referencia]

## Definition of Done
- [x] item 1
- [x] item 2

## Instrucción de merge
Esperar CI verde. No hacer squash.

🤖 Generated with Claude Code
BODY
)"

# 3. Verificar checks — esperar hasta que terminen
gh pr checks <PR_NUMBER> --watch

# 4a. CI verde → mergear y limpiar
gh pr merge <PR_NUMBER> --merge --delete-branch

# 4b. CI rojo → diagnosticar, corregir, re-pushear (no abrir PR nuevo)
git add <archivos-corregidos>
git commit -m "fix: corregir fallo de CI — <descripción>"
git push
# El CI corre de nuevo automáticamente; volver al paso 3

# 5. Volver a la rama base y actualizar
git checkout develop
git pull
```

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
- Comentarios del humano sin resolver

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

## Flujo Completo

```
feat/branch
    │
    ▼
gh pr create --base develop   ← nunca saltar al main directamente
    │
    ▼
CI corre automáticamente
    │
    ├── ROJO → diagnosticar → fix → push → CI corre de nuevo
    │
    └── VERDE
          │
          ▼
    gh pr checks --watch   ← esperar si aún está corriendo
          │
          ▼
    gh pr merge --merge --delete-branch
          │
          ▼
    git checkout develop && git pull
          │
          ▼
    Reportar al humano: "PR #N mergeado. Branch eliminado. En develop."
```

---

## Manejo de Comentarios de Revisión

Cuando el humano deja un comentario en el PR:

1. El agente **lee** el comentario completo antes de responder
2. Si es un cambio pedido: propone el cambio específico para aprobación
3. Si es una pregunta: responde con contexto del Execution Plan o SRD
4. Si el agente no está de acuerdo: argumenta con razones técnicas, no solo acepta
5. El humano tiene la decisión final
6. **Nunca mergear con comentarios sin resolver**

---

## Transición a Fase 5

La Fase 4 está completa cuando:

1. CI verde ✓
2. Merge completado ✓
3. Branch de feature eliminado ✓
4. `develop` actualizado localmente ✓

**El agente reporta:** "PR #N mergeado a develop. Branch feat/X eliminado. Próxima tarea: [siguiente del backlog]."

---

## Recursos

- Template PR: `templates/.github/PULL_REQUEST_TEMPLATE.md`
- Fase anterior: `docs/02-phases/phase-3-implement.md`
- Siguiente fase: `docs/02-phases/phase-5-deploy.md`
