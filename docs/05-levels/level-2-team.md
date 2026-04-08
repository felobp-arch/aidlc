# Nivel 2 — Team (Equipos de Desarrollo)

> Para proyectos con múltiples desarrolladores, usuarios reales externos,
> o proyectos personales que han crecido. Todos los controles de Nivel 1
> más los necesarios para trabajo colaborativo seguro.

---

## Contexto

El nivel Team está diseñado para proyectos donde:

- Hay 2 o más desarrolladores contribuyendo
- Hay usuarios externos reales
- Un bug en producción tiene consecuencias medibles
- El onboarding de nuevos devs debe ser reproducible

---

## Diferencias clave respecto a Nivel 1

| Control | Nivel 1 | Nivel 2 |
|---------|---------|---------|
| SRD | Opcional | Obligatorio |
| Execution Plan | Breve, informal | Completo, documentado |
| Definition of Done | Implícita | Explícita por feature |
| CI/CD | Recomendado | Obligatorio |
| Branch protection | Opcional | Obligatorio |
| Staging environment | Opcional | Obligatorio |
| Code review | Opcional | Obligatorio (al menos 1 aprobación) |
| Rate limiting | Opcional | Obligatorio en endpoints públicos |
| Audit logs | Opcional | Obligatorio para operaciones sensibles |
| Cobertura 70% | Opcional | Obligatorio |
| `CODEOWNERS` | No | Recomendado |

---

## Controles Obligatorios en Nivel 2

### 1. Branch Protection en GitHub

```yaml
# Configuración recomendada para main:
- Require pull request before merging: ON
- Required approving reviews: 1 (mínimo)
- Dismiss stale reviews when new commits are pushed: ON
- Require status checks to pass before merging: ON
  - ci/tests
  - ci/lint
  - ci/security-audit
- Require branches to be up to date before merging: ON
- Restrict who can push to matching branches: ON (solo maintainers)
```

### 2. CI/CD Completo

El pipeline debe tener al menos:

```
push/PR → lint → security-audit → tests (≥70% cobertura) → secret-scan
                                                                  ↓
push a main: → build → deploy-staging → smoke-tests
```

Ver template: `templates/.github/workflows/ci-security.yml`

### 3. Staging Environment

- Configuración idéntica a producción (mismas variables, mismo Docker image)
- Datos anonimizados — nunca datos reales de usuarios
- Deploy automático en cada merge a `main`
- URL documentada en el repositorio

### 4. Rate Limiting

Todos los endpoints públicos:
- Auth endpoints: máximo 5-10 req/min por IP
- Endpoints de datos: máximo 60-100 req/min por usuario autenticado
- Endpoints de búsqueda costosa: máximo 10-20 req/min

### 5. Audit Logs

Operaciones que generan audit log obligatorio:

- Login / logout exitoso y fallido
- Creación, modificación o eliminación de recursos de usuarios
- Cambios de permisos o roles
- Acceso a datos sensibles (si aplica)
- Errores de autorización (intentos de acceso no autorizado)

---

## Estructura de Proyecto Recomendada (Nivel 2)

```
project/
├── src/
│   ├── services/
│   ├── api/
│   └── db/
├── tests/
│   ├── unit/
│   ├── integration/
│   └── e2e/             # Flujos principales de usuario
├── docs/
│   ├── decisions/       # ADRs
│   └── api/             # Documentación de endpoints
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── release.yml
│   ├── CODEOWNERS
│   └── PULL_REQUEST_TEMPLATE.md
├── .env.example
├── .gitignore
├── .pre-commit-config.yaml
├── docker-compose.yml
├── docker-compose.staging.yml
└── README.md
```

---

## Onboarding Reproducible

El `README.md` de un proyecto Nivel 2 debe permitir que un nuevo dev esté
funcionando en menos de 30 minutos:

```markdown
## Setup en 5 pasos

1. `git clone <repo>`
2. `cp .env.example .env` — pedir valores al tech lead
3. `docker compose up -d`
4. `make migrate` (o comando equivalente)
5. `make test` — si pasan, está todo bien

## Arquitectura en 2 minutos
[Diagrama o descripción de los 3 componentes principales]

## Conventions
- Branches: `feat/`, `fix/`, `chore/`
- Commits: conventional commits
- PRs: requerir 1 review, template incluido
```

---

## Cuándo Subir a Nivel 3

Es momento de pasar a Nivel 3 (Enterprise) cuando:

- Hay regulación formal que cumplir (SFC, GDPR, SOC2)
- El equipo tiene más de 5 personas
- Hay múltiples equipos o squads trabajando en el mismo sistema
- Los datos manejados son financieros, médicos o de alto valor
- La disponibilidad del sistema está en un SLA formal con consecuencias

---

## Herramientas para Nivel 2

| Propósito | Opciones |
|-----------|---------|
| Hosting | Railway Pro, Render Team, AWS, GCP, Azure |
| DB | PlanetScale, Supabase, RDS |
| Secrets en deploy | Variables del proveedor + Doppler |
| Monitoring | Sentry (errores) + UptimeRobot o Better Uptime |
| Logs | Logtail, Papertrail, o CloudWatch básico |
| APM | Sentry Performance (free tier suficiente) |

---

## Recursos

- Nivel anterior: `docs/05-levels/level-1-starter.md`
- Siguiente nivel: `docs/05-levels/level-3-enterprise.md`
- Template CI: `templates/.github/workflows/ci-security.yml`
- Template PR: `templates/.github/PULL_REQUEST_TEMPLATE.md`
