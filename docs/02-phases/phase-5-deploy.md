# Fase 5 — Deploy

## Propósito

Llevar el código revisado y aprobado desde `main` hasta los usuarios finales,
con controles técnicos que garantizan que ningún deploy a producción ocurre
sin aprobación humana explícita.

---

## Entradas

- Merge completado a `main` (Fase 4 completada)
- Pipeline de CI en verde
- Staging environment disponible

## Salida

- Código desplegado en staging (automático)
- Validación en staging completada por el humano
- Código desplegado en producción (con aprobación manual)

---

## Flujo de Deploy

```
merge a main
    └── Pipeline CI corre (automático)
        └── Tests pasan ✓
            └── Build exitoso ✓
                └── Deploy a STAGING (automático)
                    └── HUMANO valida en staging
                        └── HUMANO aprueba en GitHub Actions
                            └── Deploy a PRODUCTION (manual gate)
                                └── Smoke tests en producción
                                    └── Monitoreo activo (primera hora)
```

---

## Controles Técnicos Obligatorios

### Branch Protection en `main`
```yaml
# Configuración en GitHub
protection_rules:
  required_status_checks:
    strict: true
    contexts:
      - ci/tests
      - ci/security-scan
      - ci/lint
  required_pull_request_reviews:
    required_approving_review_count: 1
  restrict_pushes: true  # push directo rechazado
```

### Environment Protection en GitHub Actions
```yaml
# .github/workflows/release.yml
jobs:
  deploy-production:
    environment:
      name: production
      # GitHub solicita aprobación manual aquí
      # ningún agente puede bypassear esto
    needs: [deploy-staging, staging-tests]
```

---

## Staging — Validación Humana

Antes de aprobar el deploy a producción, el humano verifica:

- [ ] El feature funciona como describe el SRD
- [ ] Los casos de borde del SRD se comportan correctamente
- [ ] No hay regresiones en features existentes
- [ ] Performance aceptable bajo carga representativa
- [ ] Logs y métricas se ven normales
- [ ] Smoke tests pasan

---

## Deploy a Producción

El agente **no puede** iniciar ni aprobar el deploy a producción.

Solo puede:
- Preparar el release (changelog, tag de versión)
- Verificar que staging está en el estado correcto
- Informar al humano que staging está listo para aprobación
- Monitorear el deploy una vez que el humano lo inicia

El humano:
- Revisa el estado de staging
- Aprueba en la UI de GitHub Actions (environment protection rule)
- Monitorea la primera hora post-deploy

---

## Rollback

Si algo falla en producción después del deploy:

### Rollback automático (si está configurado):
- El pipeline detecta fallo en smoke tests post-deploy
- Revierte automáticamente al release anterior
- Notifica al equipo

### Rollback manual:
```bash
# El humano ejecuta o aprueba:
git revert HEAD  # revertir el commit
# o
# GitHub Actions: re-deploy del release anterior
```

**El agente puede proponer el rollback, pero el humano lo ejecuta.**

---

## Post-Deploy Checklist

Dentro de la primera hora post-deploy a producción:

- [ ] Métricas de error rate normales (< baseline)
- [ ] Latencia p95 dentro del SLA
- [ ] Logs de audit sin anomalías
- [ ] Features nuevas funcionan para usuarios reales (smoke test)
- [ ] Alertas silenciadas correctamente (ninguna inesperada)

---

## Versionado

AIDLC recomienda Semantic Versioning (semver):

| Tipo de cambio | Versión |
|----------------|---------|
| Bug fix | PATCH (1.0.1) |
| Feature nueva compatible | MINOR (1.1.0) |
| Breaking change | MAJOR (2.0.0) |

El agente propone la versión correcta; el humano la aprueba antes del tag.

---

## Ambientes y Secretos por Environment

| Var | local | staging | production |
|-----|-------|---------|------------|
| DATABASE_URL | `.env` | Var del proveedor | Secrets Manager |
| API_KEY | `.env` | Var del proveedor | Secrets Manager |
| LOG_LEVEL | `DEBUG` | `INFO` | `WARNING` |
| MFA_ENABLED | `false` | `true` | `true` |

---

## Documentación del Deploy

El agente documenta en `MEMORY.md` después de cada deploy exitoso:

- Versión deployada
- Features incluidas
- Incidentes durante el deploy (si los hubo)
- Tiempo de deploy (staging → producción)

---

## Recursos

- Workflow CI: `templates/.github/workflows/ci-security.yml`
- Workflow Release: `templates/.github/workflows/release.yml`
- Fase anterior: `docs/02-phases/phase-4-review.md`
- Fase 1 (reinicio del ciclo): `docs/02-phases/phase-1-spec.md`
