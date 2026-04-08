## ¿Qué hace este PR?

<!-- Una oración. Si necesitas más de dos oraciones, el PR es probablemente demasiado grande. -->

## Tipo de cambio

- [ ] Bug fix
- [ ] Nueva feature
- [ ] Refactor (sin cambio de comportamiento)
- [ ] Docs / configuración
- [ ] Chore (dependencias, CI/CD)

## Referencias

**SRD:** <!-- RF-XXX, RF-YYY o link al documento -->
**Execution Plan aprobado:** <!-- Link a la sesión / documento o "incluido en descripción" -->
**Issue relacionado:** <!-- Closes #123 -->

## Cambios principales

<!-- Lista concisa de los archivos clave modificados y por qué -->

- `src/services/X.py` —
- `src/api/routes/X.py` —
- `tests/unit/test_X.py` —

## Definition of Done

- [ ] Lógica implementada en `services/` (sin imports de framework)
- [ ] Controller en `api/` sin lógica de negocio
- [ ] Adapter en `db/` si hay persistencia nueva
- [ ] Tests unitarios escritos y pasando
- [ ] Cobertura ≥ 70% en código nuevo/modificado
- [ ] Tests de integración para endpoints nuevos (si aplica)
- [ ] Sin secrets hardcodeados en el diff
- [ ] Variables de entorno nuevas documentadas en `.env.example`
- [ ] Pipeline en verde (todos los checks pasan)
- [ ] `lessons-learned.md` actualizado si hubo errores relevantes

## Seguridad

- [ ] Sin secrets en el código o historial de git
- [ ] Rate limiting presente en endpoints públicos nuevos
- [ ] Validación de input en todos los endpoints nuevos
- [ ] Respuestas de error no exponen información interna
- [ ] STRIDE revisado (si la feature tiene implicaciones de seguridad)

## Cómo probar

<!-- Pasos concretos para que el reviewer valide el comportamiento -->

1.
2.
3.

**Resultado esperado:**

## Notas para el reviewer

<!-- Contexto adicional, decisiones de diseño, trade-offs, o áreas donde quieres feedback específico -->

## Screenshots / evidencia

<!-- Si hay cambios visuales o de comportamiento observable -->
