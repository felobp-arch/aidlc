# Playbook — Node.js

> Guía de referencia para proyectos Node.js bajo AIDLC.
> Aplica Arquitectura Hexagonal, SDD y los 10 contratos de FOUNDATION.md.

---

## Stack de Referencia

| Componente | Tecnología | Versión mínima |
|------------|-----------|---------------|
| Runtime | Node.js | 20 LTS+ |
| Framework | Express / Fastify / NestJS | — |
| Validación | Zod | 3.0+ |
| ORM | Prisma / Drizzle | — |
| Tests | Jest / Vitest | — |
| Linter | ESLint + Prettier | — |
| TypeScript | typescript | 5.0+ |
| DB | PostgreSQL | 15+ |
| Contenedor | Docker + Docker Compose | — |

> Este playbook usa TypeScript. JavaScript sin tipos no es recomendado para
> proyectos bajo AIDLC en entornos profesionales.

---

## Estructura de Proyecto

```
project/
├── src/
│   ├── services/           # Lógica de negocio — sin framework, sin DB
│   │   ├── user.service.ts
│   │   ├── entities/
│   │   │   └── user.entity.ts
│   │   ├── interfaces/     # Contratos para adapters
│   │   │   └── user.repository.ts
│   │   └── exceptions/
│   │       └── user.exceptions.ts
│   ├── api/                # Controllers y rutas
│   │   ├── app.ts          # Express/Fastify app setup
│   │   ├── middleware/
│   │   │   ├── auth.middleware.ts
│   │   │   ├── rate-limit.middleware.ts
│   │   │   └── error-handler.middleware.ts
│   │   └── routes/
│   │       └── users.routes.ts
│   └── db/                 # Adapters de persistencia
│       ├── prisma/
│       │   └── schema.prisma
│       └── adapters/
│           └── prisma-user.repository.ts
├── tests/
│   ├── unit/
│   ├── integration/
│   └── setup.ts
├── .env.example
├── Dockerfile
├── docker-compose.yml
├── package.json
└── tsconfig.json
```

---

## Arquitectura Hexagonal en Node.js

### Interface en `services/interfaces/`

```typescript
// src/services/interfaces/user.repository.ts
export interface UserRepository {
  findById(userId: string): Promise<User | null>;
  save(user: User): Promise<User>;
  existsByEmail(email: string): Promise<boolean>;
}
```

### Servicio en `services/`

```typescript
// src/services/user.service.ts
import { UserRepository } from './interfaces/user.repository';
import { User, CreateUserRequest } from './entities/user.entity';
import { UserAlreadyExistsError } from './exceptions/user.exceptions';

export class UserService {
  constructor(private readonly repository: UserRepository) {}

  async createUser(request: CreateUserRequest): Promise<User> {
    const exists = await this.repository.existsByEmail(request.email);
    if (exists) {
      throw new UserAlreadyExistsError(request.email);
    }
    const user = User.create(request);
    return this.repository.save(user);
  }
}
```

### Adapter en `db/adapters/`

```typescript
// src/db/adapters/prisma-user.repository.ts
import { PrismaClient } from '@prisma/client';
import { UserRepository } from '../../services/interfaces/user.repository';
import { User } from '../../services/entities/user.entity';

export class PrismaUserRepository implements UserRepository {
  constructor(private readonly prisma: PrismaClient) {}

  async findById(userId: string): Promise<User | null> {
    const record = await this.prisma.user.findUnique({ where: { id: userId } });
    return record ? User.fromPrisma(record) : null;
  }

  async save(user: User): Promise<User> {
    const record = await this.prisma.user.create({ data: user.toPrisma() });
    return User.fromPrisma(record);
  }

  async existsByEmail(email: string): Promise<boolean> {
    const count = await this.prisma.user.count({ where: { email } });
    return count > 0;
  }
}
```

### Controller en `api/routes/`

```typescript
// src/api/routes/users.routes.ts
import { Router, Request, Response, NextFunction } from 'express';
import { z } from 'zod';
import { UserService } from '../../services/user.service';
import { UserAlreadyExistsError } from '../../services/exceptions/user.exceptions';

const CreateUserSchema = z.object({
  email: z.string().email(),
  name: z.string().min(2).max(100),
});

export function createUserRoutes(service: UserService): Router {
  const router = Router();

  router.post('/', async (req: Request, res: Response, next: NextFunction) => {
    const parsed = CreateUserSchema.safeParse(req.body);
    if (!parsed.success) {
      return res.status(422).json({ detail: parsed.error.issues });
    }

    try {
      const user = await service.createUser(parsed.data);
      return res.status(201).json({ id: user.id, email: user.email });
    } catch (error) {
      if (error instanceof UserAlreadyExistsError) {
        return res.status(409).json({ detail: 'User already exists' });
      }
      next(error);
    }
  });

  return router;
}
```

---

## Configuración y Variables de Entorno

```typescript
// src/config.ts
import { z } from 'zod';

const ConfigSchema = z.object({
  DATABASE_URL: z.string().url(),
  SECRET_KEY: z.string().min(32),
  NODE_ENV: z.enum(['local', 'staging', 'production']).default('local'),
  PORT: z.coerce.number().default(3000),
  RATE_LIMIT_PER_MINUTE: z.coerce.number().default(60),
  MFA_ENABLED: z.coerce.boolean().default(false),
});

export const config = ConfigSchema.parse(process.env);
```

---

## Comandos de Desarrollo

```bash
# Setup inicial
npm install
cp .env.example .env

# Desarrollo
npm run dev

# Tests
npm test
npm run test:coverage   # requiere ≥ 70%
npm run test:watch

# Lint y formato
npm run lint
npm run format

# Auditoría de seguridad
npm audit

# Prisma
npx prisma migrate dev --name descripcion
npx prisma generate
npx prisma studio  # UI de DB local
```

---

## `package.json` — Scripts Recomendados

```json
{
  "scripts": {
    "dev": "tsx watch src/api/app.ts",
    "build": "tsc",
    "start": "node dist/api/app.js",
    "test": "jest",
    "test:coverage": "jest --coverage --coverageThreshold='{\"global\":{\"lines\":70}}'",
    "test:watch": "jest --watch",
    "lint": "eslint src/ tests/ --ext .ts",
    "format": "prettier --write src/ tests/",
    "audit": "npm audit --audit-level=high",
    "db:migrate": "prisma migrate dev",
    "db:generate": "prisma generate"
  }
}
```

---

## Seguridad — Controles Obligatorios

```typescript
// src/api/middleware/rate-limit.middleware.ts
import rateLimit from 'express-rate-limit';
import { config } from '../../config';

export const rateLimiter = rateLimit({
  windowMs: 60 * 1000,
  max: config.RATE_LIMIT_PER_MINUTE,
  standardHeaders: true,
  legacyHeaders: false,
  message: { detail: 'Too many requests' },
});

// src/api/middleware/error-handler.middleware.ts
import { Request, Response, NextFunction } from 'express';
import { logger } from '../logger';

export function errorHandler(
  err: Error,
  req: Request,
  res: Response,
  _next: NextFunction,
): void {
  logger.error('Unhandled error', { error: err.message, path: req.path });
  // Nunca exponer el stack trace al cliente
  res.status(500).json({ detail: 'Internal server error' });
}
```

---

## Pendientes del Playbook

- [ ] Agregar ejemplos con NestJS (para proyectos enterprise)
- [ ] Agregar ejemplos con Fastify
- [ ] Security checklist específica para Node.js
- [ ] Pipeline template para Node.js (adaptación del de FastAPI)

---

## Recursos

- Playbook Python/FastAPI (más completo): `docs/04-playbooks/python-fastapi/README.md`
- Principios de arquitectura: `docs/01-principles/`
- Caso de estudio: `examples/fhc/`
