# Node.js Fastify — Dev Patterns

> **Companion to** `docs/roles/sa/stack-templates/STACK_CONTEXT_node_fastify.md`
>
> This file contains bootstrap steps, layer-by-layer code examples, and guardrails that prevent Claude Code from generating code that deviates from the agreed 5-layer Clean Architecture.
>
> **How to use:** Copy this file into the project as supplementary knowledge — attach to Claude Code project knowledge, or append below the project-specific section of `CLAUDE.md`. Add project-specific notes under `## Project Overrides` at the bottom. Do not modify the template sections.

---

## Stack

| Package | Version | Role |
|---------|---------|------|
| Fastify | v5 | HTTP framework |
| Zod | v4 | Schema validation + type inference |
| fastify-type-provider-zod | v7 | Zod ↔ Fastify type bridge |
| TypeScript | v5 | Language |
| Prisma | v7 | ORM — generated client in `src/generated/prisma/` |
| @prisma/adapter-pg + pg | v7 / v8 | Prisma driver adapter |
| Vitest | v4 | Testing |
| pnpm | v10 | Package manager |

---

## Bootstrap

**When setting up a new project from this template**, create all files in this section exactly as shown. These files are the foundation — do not modify them unless explicitly instructed.

### Step 1 — Initialize

```bash
mkdir <project-name> && cd <project-name>
git init
pnpm init -y
```

### Step 2 — Install dependencies

```bash
pnpm add fastify@^5 zod@^4 fastify-type-provider-zod@^7 @prisma/adapter-pg@^7 @prisma/client@^7 pg@^8

pnpm add -D typescript@^5 tsx@^4 vitest@^4 @vitest/coverage-v8@^4 \
  @types/node@^22 @types/pg@^8 prisma@^7 \
  eslint@^10 typescript-eslint@^8 eslint-plugin-prettier@^5 eslint-config-prettier@^10 \
  prettier@^3 husky@^9 lint-staged@^15
```

### Step 3 — Create config files

**`package.json`** — replace the generated one:

```json
{
  "name": "<project-name>",
  "version": "1.0.0",
  "packageManager": "pnpm@10",
  "scripts": {
    "dev": "tsx watch src/server.ts",
    "build": "tsc",
    "start": "node dist/server.js",
    "test": "vitest run",
    "test:unit": "vitest run tests/unit",
    "test:api": "vitest run tests/api",
    "test:integration": "vitest run tests/integration",
    "test:coverage": "vitest run --coverage",
    "lint": "eslint src",
    "lint:fix": "eslint src --fix",
    "format": "prettier --write src",
    "typecheck": "tsc --noEmit",
    "prepare": "husky"
  },
  "lint-staged": {
    "src/**/*.ts": ["eslint --fix", "prettier --write"]
  },
  "pnpm": {
    "onlyBuiltDependencies": ["esbuild", "@prisma/engines", "prisma"]
  }
}
```

**`tsconfig.json`**:

```json
{
  "compilerOptions": {
    "target": "ES2022",
    "module": "Node16",
    "moduleResolution": "node16",
    "rootDir": "./src",
    "outDir": "./dist",
    "strict": true,
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,
    "noImplicitOverride": true,
    "noFallthroughCasesInSwitch": true,
    "forceConsistentCasingInFileNames": true,
    "skipLibCheck": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true
  },
  "include": ["src"],
  "exclude": ["node_modules", "dist"]
}
```

**`eslint.config.js`**:

```js
// @ts-check
const tseslint = require("typescript-eslint")
const prettierRecommended = require("eslint-plugin-prettier/recommended")

/** @param {...string} patterns */
const noImportFrom = (...patterns) => ({
  "no-restricted-imports": ["error", { patterns }],
})

module.exports = tseslint.config(
  { ignores: ["dist/**", "src/generated/**", "node_modules/**"] },

  ...tseslint.configs.recommended,

  {
    rules: {
      "@typescript-eslint/no-explicit-any": "error",
      "@typescript-eslint/consistent-type-imports": [
        "error",
        { prefer: "type-imports", fixStyle: "inline-type-imports" },
      ],
      "@typescript-eslint/no-unused-vars": [
        "error",
        { argsIgnorePattern: "^_", varsIgnorePattern: "^_" },
      ],
    },
  },

  // domain: pure contracts
  {
    files: ["src/domain/**/*.ts"],
    rules: noImportFrom("**/application/**", "**/infrastructure/**", "**/presentation/**", "**/container/**"),
  },

  // application: depends only on domain
  {
    files: ["src/application/**/*.ts"],
    rules: noImportFrom("**/infrastructure/**", "**/presentation/**", "**/container/**"),
  },

  // infrastructure: depends only on domain + application
  {
    files: ["src/infrastructure/**/*.ts"],
    rules: noImportFrom("**/presentation/**", "**/container/**"),
  },

  // presentation: depends only on domain
  {
    files: ["src/presentation/**/*.ts"],
    rules: noImportFrom("**/infrastructure/**", "**/container/**"),
  },

  prettierRecommended,
)
```

**`.prettierrc`**:

```json
{
  "semi": false,
  "singleQuote": false,
  "trailingComma": "all",
  "printWidth": 100,
  "tabWidth": 2
}
```

**`vitest.config.ts`**:

```ts
import { defineConfig } from "vitest/config"

export default defineConfig({
  test: {
    include: ["tests/**/*.test.ts"],
  },
})
```

**`prisma/schema.prisma`**:

```prisma
generator client {
  provider = "prisma-client"
  output   = "../src/generated/prisma"
}

datasource db {
  provider = "postgresql"
}
```

**`prisma.config.ts`**:

```ts
import "dotenv/config"
import { defineConfig } from "prisma/config"

export default defineConfig({
  schema: "prisma/schema.prisma",
  migrations: {
    path: "prisma/migrations",
  },
  datasource: {
    url: process.env["DATABASE_URL"],
  },
})
```

### Step 4 — Create shared primitives

These files never change. Create them once and leave them.

**`src/shared/service.ts`**:

```ts
export type Service<TParams, TEntity> = (params: TParams) => Promise<TEntity>
```

**`src/shared/app-error.ts`**:

```ts
export class AppError extends Error {
  constructor(
    readonly code: string,
    readonly httpStatus: number,
    message: string,
  ) {
    super(message)
    this.name = "AppError"
  }
}
```

**`src/presentation/api/response.ts`**:

```ts
export type Response<T> = {
  readonly code: string
  readonly message: string
  readonly data: T
}

export const Response = {
  ok: <T>(data: T): Response<T> => ({ code: "0000", message: "success", data }),
  created: <T>(data: T): Response<T> => ({ code: "0001", message: "created", data }),
  fail: (code: string, message: string): Response<null> => ({ code, message, data: null }),
}
```

**`src/presentation/api/error-handler.plugin.ts`**:

```ts
import type { FastifyInstance } from "fastify"
import { AppError } from "../../shared/app-error"
import { Response } from "./response"

function isHttpError(error: unknown): error is Error & { statusCode: number } {
  return error instanceof Error && typeof (error as { statusCode?: unknown }).statusCode === "number"
}

export function registerErrorHandler(app: FastifyInstance) {
  app.setErrorHandler((error, _request, reply) => {
    if (error instanceof AppError) {
      return reply.status(error.httpStatus).send(Response.fail(error.code, error.message))
    }
    if (isHttpError(error) && error.statusCode === 400) {
      return reply.status(400).send(Response.fail("0400", error.message))
    }
    app.log.error(error)
    return reply.status(500).send(Response.fail("9000", "internal server error"))
  })
}
```

**`src/presentation/api/application.ts`**:

```ts
import Fastify from "fastify"
import { serializerCompiler, validatorCompiler } from "fastify-type-provider-zod"
import { registerRoutes } from "./routes"
import { registerErrorHandler } from "./error-handler.plugin"
import type { AppUseCases } from "./routes"

type BuildAppConfig = {
  readonly logger?: boolean
}

export function buildApp(options: AppUseCases, config: BuildAppConfig = {}) {
  const app = Fastify({ logger: config.logger ?? true })
  app.setValidatorCompiler(validatorCompiler)
  app.setSerializerCompiler(serializerCompiler)
  registerErrorHandler(app)
  registerRoutes(app, options)
  return app
}
```

**`src/presentation/api/routes.ts`** — update this file each time a new domain is added:

```ts
import type { FastifyInstance } from "fastify"
// import type { ICreateXUseCase } from "../../domain/x/create-x.use-case"
// import { xRoutes } from "./x/x.routes"

export type AppUseCases = {
  // readonly createX: ICreateXUseCase
}

export function registerRoutes(app: FastifyInstance, useCases: AppUseCases) {
  // app.register(xRoutes(useCases.createX))
}
```

**`src/server.ts`** — update this file each time a new domain is added:

```ts
import { Pool } from "pg"
import { PrismaPg } from "@prisma/adapter-pg"
import { PrismaClient } from "./generated/prisma/client"
import { buildApp } from "./presentation/api/application"
// import { buildCreateXUseCase } from "./container/x.container"

const pool = new Pool({ connectionString: process.env["DATABASE_URL"] })
const prisma = new PrismaClient({ adapter: new PrismaPg(pool) })

const app = buildApp({
  // createX: buildCreateXUseCase(prisma),
})

app.listen({ port: 3000, host: "0.0.0.0" }, (err) => {
  if (err) {
    app.log.error(err)
    process.exit(1)
  }
})
```

### Step 5 — Setup tooling

```bash
pnpm prisma generate
pnpm run prepare        # initialize husky
```

Create `.husky/pre-commit`:

```sh
pnpm lint-staged
```

### Step 6 — Verify

Run all four checks. All must pass before the project is ready.

**1. Tooling**

```bash
pnpm typecheck   # must exit 0 — TypeScript compiles
pnpm lint        # must exit 0 — ESLint rules load
pnpm build       # must exit 0 — dist/ builds
pnpm test        # must exit 0 — 0 tests is acceptable at bootstrap
```

**2. Architecture boundary enforcement** — ESLint config can be syntactically correct but rules can silently not apply. Verify with a probe:

```bash
# Create probe file
echo 'import { } from "../application/users/create-user.service"' > src/domain/_boundary-probe.ts

# Must produce a no-restricted-imports error
pnpm lint src/domain/_boundary-probe.ts

# Remove probe — do not commit
rm src/domain/_boundary-probe.ts
```

If lint does **not** error on the probe file, `eslint.config.js` is misconfigured — fix it before proceeding.

### Directory structure after bootstrap

```
<project-name>/
├── prisma/
│   ├── schema.prisma
│   └── migrations/
├── src/
│   ├── shared/
│   │   ├── service.ts
│   │   └── app-error.ts
│   ├── presentation/api/
│   │   ├── application.ts
│   │   ├── routes.ts
│   │   ├── response.ts
│   │   └── error-handler.plugin.ts
│   └── generated/prisma/     ← generated by prisma generate
├── tests/
│   ├── unit/
│   ├── api/
│   └── integration/
├── .husky/pre-commit
├── eslint.config.js
├── tsconfig.json
├── vitest.config.ts
├── prisma.config.ts
├── .prettierrc
└── package.json
```

---

## Domain Checklist

**After bootstrap passes (Step 6), add each domain by following this checklist exactly — one action at a time, in layer order.**

Do not skip steps. Do not implement layer N+1 before layer N compiles cleanly.

### Per-action checklist

Replace `{domain}` and `{action}` with the actual names (e.g. `users`, `create`).

```
□ Layer 1 — Domain
  □ src/domain/{domain}/{action}-{domain}.params.ts
  □ src/domain/{domain}/{action}-{domain}.entity.ts
  □ src/domain/{domain}/{action}-{domain}.use-case.ts
  □ src/domain/{domain}/{domain}.error.ts          ← only if this domain needs typed errors
  → pnpm typecheck

□ Layer 2 — Application
  □ src/application/{domain}/ports/persistent/{find or action}-{domain}.persistent.ts
  □ src/application/{domain}/{action}-{domain}.service.ts
  → pnpm typecheck

□ Layer 3 — Infrastructure
  □ src/infrastructure/postgresql/{domain}/model/{domain}.model.ts
  □ src/infrastructure/postgresql/{domain}/mapper/{domain}.mapper.ts
  □ src/infrastructure/postgresql/{domain}/command/{action}-{domain}.command.ts  ← writes
  □ src/infrastructure/postgresql/{domain}/query/{action}-{domain}.query.ts      ← reads
  □ src/infrastructure/postgresql/{domain}/{action}-{domain}.impl.ts
  → pnpm typecheck

□ Layer 4 — Container
  □ src/container/{domain}.container.ts
  → pnpm typecheck

□ Layer 5 — Presentation
  □ src/presentation/api/{domain}/request/{action}-{domain}.request.ts
  □ src/presentation/api/{domain}/response/{action}-{domain}.response.ts
  □ src/presentation/api/{domain}/handlers/{action}-{domain}.handler.ts
  □ src/presentation/api/{domain}/{domain}.routes.ts
  → pnpm typecheck

□ Wire up
  □ src/presentation/api/routes.ts   ← add use case type + register route
  □ src/server.ts                    ← import container + pass use case to buildApp
  → pnpm typecheck && pnpm test
```

### Rules

- `pnpm typecheck` after each layer — a layer that doesn't compile cannot be a foundation for the next
- `{domain}.error.ts` is optional — only create it when the domain needs errors beyond Zod validation
- Command files are for writes (INSERT/UPDATE/DELETE); query files are for reads (SELECT) — one file per operation
- `routes.ts` and `server.ts` are updated **last**, after all layers compile

---

## Layer 1 — Domain

**Rule:** type contracts only. No implementations, no imports from other layers.

### Folder structure

```
src/domain/{domain}/
├── {action}-{domain}.params.ts
├── {action}-{domain}.entity.ts
├── {action}-{domain}.use-case.ts
└── {domain}.error.ts
```

### Code

```ts
// src/shared/service.ts
export type Service<TParams, TEntity> = (params: TParams) => Promise<TEntity>
```

```ts
// create-user.params.ts
export type CreateUserParams = {
  readonly username: string
  readonly email: string
  readonly password: string
}
```

```ts
// create-user.entity.ts — never includes sensitive fields (password, tokens)
export type CreateUserEntity = {
  readonly id: string
  readonly username: string
  readonly email: string
  readonly createdAt: string
}
```

```ts
// create-user.use-case.ts
import type { Service } from "../../shared/service"
import type { CreateUserParams } from "./create-user.params"
import type { CreateUserEntity } from "./create-user.entity"

export type ICreateUserUseCase = Service<CreateUserParams, CreateUserEntity>
```

```ts
// create-task.params.ts — optional field: string | undefined (NOT string | null)
export type CreateTaskParams = {
  readonly title: string
  readonly description?: string | undefined   // convert to null at infra boundary
}
```

```ts
// create-task.entity.ts — nullable DB field and union status
export type CreateTaskEntity = {
  readonly id: string
  readonly title: string
  readonly description: string | null
  readonly status: "pending" | "in_progress" | "done"
  readonly createdAt: string
}
```

**`description` type across layers:**
- Params: `description?: string | undefined` — caller may omit
- Entity/Response: `description: string | null` — DB nullable, always present, may be null
- Command: `description: params.description ?? null` — convert at infra boundary

**`status` type flow:**
- DB stores as `String` (no constraint)
- Model: `status: string`
- Mapper casts: `status: model.status as CreateTaskEntity["status"]`
- Entity: `"pending" | "in_progress" | "done"` — constrained at domain boundary

**One entity type per action** — `CreateUserEntity` and `GetUserEntity` are separate even when fields are identical.

**Request type vs Params type** — separate but structurally compatible. Handler inputs `CreateUserRequest` (Zod-inferred); use case expects `CreateUserParams`. TypeScript accepts this structurally.

### Domain errors

```ts
// src/domain/users/user.error.ts — 1xxx range
import { AppError } from "../../shared/app-error"

export class UserAlreadyExistsError extends AppError {
  constructor(username: string) {
    super("1001", 409, `username ${username} already exists`)
  }
}

export class UserNotFoundError extends AppError {
  constructor(id: string) {
    super("1002", 404, `user ${id} not found`)
  }
}
```

---

## Layer 2 — Application

**Rule:** business logic only. Defines port interfaces. Never imports infra or presentation.

### Folder structure

```
src/application/{domain}/
├── {action}-{domain}.service.ts
└── ports/
    ├── persistent/
    │   ├── find-{domain}.persistent.ts    ← only if duplicate check needed
    │   └── {action}-{domain}.persistent.ts
    └── cache/
        └── {action}-{domain}.cache.ts     ← only if service caches
```

Port files are created **on demand** — only add files the service actually uses.

### Code

```ts
// ports/persistent/create-user.persistent.ts
import type { CreateUserParams } from "../../../../domain/users/create-user.params"
import type { CreateUserEntity } from "../../../../domain/users/create-user.entity"

export type ICreateUserPersistent = {
  save: (params: CreateUserParams) => Promise<CreateUserEntity>
}
```

```ts
// create-user.service.ts — multiple ports + cache
import type { ICreateUserUseCase } from "../../domain/users/create-user.use-case"
import { UserAlreadyExistsError } from "../../domain/users/user.error"
import type { IFindUserPersistent } from "./ports/persistent/find-user.persistent"
import type { ICreateUserPersistent } from "./ports/persistent/create-user.persistent"
import type { ICreateUserCache } from "./ports/cache/create-user.cache"

type Persistent = IFindUserPersistent & ICreateUserPersistent
type Cache = ICreateUserCache

export const CreateUserService = (
  persistent: Persistent,
  cache: Cache,
): ICreateUserUseCase =>
  async (params) => {
    const existing = await persistent.findByUsername(params.username)
    if (existing) throw new UserAlreadyExistsError(params.username)
    const user = await persistent.save(params)
    await cache.set(`user:${user.id}`, user)
    return user
  }
```

```ts
// create-task.service.ts — no validation: single port, passthrough
import type { ICreateTaskUseCase } from "../../domain/tasks/create-task.use-case"
import type { ICreateTaskPersistent } from "./ports/persistent/create-task.persistent"

export const CreateTaskService = (persistent: ICreateTaskPersistent): ICreateTaskUseCase =>
  async (params) => {
    return persistent.save(params)
  }
```

**When a service has no validation** — passthrough is valid. Do not add unnecessary cache, type aliases, or duplicate checks unless business logic requires them.

---

## Layer 3 — Infrastructure

**Rule:** implements port interfaces from application. Never imports presentation or container.

### Folder structure

```
src/infrastructure/
├── postgresql/{domain}/
│   ├── model/{domain}.model.ts
│   ├── mapper/{domain}.mapper.ts
│   ├── command/{action}-{domain}.command.ts
│   ├── query/{action}-{domain}.query.ts
│   └── {action}-{domain}.impl.ts
└── redis/
    ├── redis.client.ts
    └── {domain}/
        └── {action}-{domain}.cache.impl.ts
```

`command/` and `query/` files are created **on demand**.

### Code

```ts
// model/user.model.ts
export type UserModel = {
  readonly id: string
  readonly username: string
  readonly email: string
  readonly password: string
  readonly createdAt: Date
}
```

```ts
// mapper/user.mapper.ts — one mapper per domain, one function per entity type
import type { CreateUserEntity } from "../../../../domain/users/create-user.entity"
import type { UserModel } from "../model/user.model"

export function toUserEntity(model: UserModel): CreateUserEntity {
  return {
    id: model.id,
    username: model.username,
    email: model.email,
    createdAt: model.createdAt.toISOString(),
  }
}
```

```ts
// command/create-user.command.ts
import type { PrismaClient } from "../../../../generated/prisma/client"
import type { UserModel } from "../model/user.model"

export async function createUserCommand(
  prisma: PrismaClient,
  data: { username: string; email: string; password: string },
): Promise<UserModel> {
  return prisma.user.create({ data })
}
```

```ts
// command/create-task.command.ts — undefined → null for nullable DB field
export async function createTaskCommand(
  prisma: PrismaClient,
  data: { title: string; description?: string | undefined },
): Promise<TaskModel> {
  return prisma.task.create({
    data: {
      title: data.title,
      description: data.description ?? null,   // params undefined → Prisma null
    },
  })
}
```

```ts
// create-user.impl.ts
import type { PrismaClient } from "../../../generated/prisma/client"
import type { ICreateUserPersistent } from "../../../application/users/ports/persistent/create-user.persistent"
import { createUserCommand } from "./command/create-user.command"
import { toUserEntity } from "./mapper/user.mapper"

export function CreateUserPersistent(prisma: PrismaClient): ICreateUserPersistent {
  return {
    save: async (params) => {
      const model = await createUserCommand(prisma, params)
      return toUserEntity(model)
    },
  }
}
```

### Prisma v7

- Import from `src/generated/prisma/client` — NOT `@prisma/client`
- `schema.prisma` has no `url` in datasource — URL goes in `prisma.config.ts`
- `PrismaClient` requires a driver adapter:

```ts
const pool = new Pool({ connectionString: process.env["DATABASE_URL"] })
const prisma = new PrismaClient({ adapter: new PrismaPg(pool) })
```

---

## Layer 4 — Presentation

**Rule:** HTTP wiring only. No business logic. Depends on domain interfaces only.

### Folder structure

```
src/presentation/api/
├── response.ts
├── routes.ts
├── application.ts
├── error-handler.plugin.ts
└── {domain}/
    ├── {domain}.routes.ts
    ├── request/{action}-{domain}.request.ts
    ├── response/{action}-{domain}.response.ts
    └── handlers/{action}-{domain}.handler.ts
```

### Request schemas

```ts
// request/create-user.request.ts
import { z } from "zod"

export const CreateUserRequestSchema = z.object({
  username: z.string().min(1),
  email: z.string().email(),
  password: z.string().min(8),
})

export type CreateUserRequest = z.infer<typeof CreateUserRequestSchema>
```

**Zod method → TypeScript type:**

| Method | TypeScript type | When to use |
|--------|----------------|-------------|
| `.optional()` | `string \| undefined` | Request field that may be absent |
| `.nullable()` | `string \| null` | Entity/response field that is NULL in DB |
| `.nullish()` | `string \| null \| undefined` | Both absent and nullable |

### Handlers

**Always map entity → response field by field. Never spread `{ ...entity }` or return the entity directly.**

```ts
// handlers/create-user.handler.ts
export function CreateUserHandler(useCase: ICreateUserUseCase) {
  return async (request: CreateUserRequest): Promise<Response<CreateUserResponse>> => {
    const entity = await useCase(request)
    return Response.created({
      id: entity.id,
      username: entity.username,
      email: entity.email,
      createdAt: entity.createdAt,
    })
  }
}
```

### Routes

**Route factory returns a Fastify plugin — always pass to `app.register()`, never call directly.**

```ts
// users/user.routes.ts
export function userRoutes(createUser: ICreateUserUseCase, getUser: IGetUserUseCase) {
  return (app: FastifyInstance) => {
    app
      .withTypeProvider<ZodTypeProvider>()
      .post("/v1/users", { schema: { body: CreateUserRequestSchema } }, async (req, reply) => {
        const result = await CreateUserHandler(createUser)(req.body)
        return reply.status(201).send(result)
      })

    app
      .withTypeProvider<ZodTypeProvider>()
      .get("/v1/users/:id", { schema: { params: GetUserRequestSchema } }, async (req, reply) => {
        const result = await GetUserHandler(getUser)(req.params)
        return reply.status(200).send(result)
      })
  }
}
```

**Handler receives one sub-property of `req`:**

| HTTP method | Fastify property | Pattern |
|-------------|-----------------|---------|
| POST / PUT / PATCH | `req.body` | `Handler(useCase)(req.body)` |
| GET with path param | `req.params` | `Handler(useCase)(req.params)` |
| GET with filter | `req.query` | `Handler(useCase)(req.query)` |

---

## Layer 5 — Container

**Rule:** the only layer that imports across all other layers.

```ts
// container/users.container.ts
export function buildCreateUserUseCase(prisma: PrismaClient, redis: IRedisClient): ICreateUserUseCase {
  const persistent = {
    ...FindUserPersistent(prisma),    // { findByUsername }
    ...CreateUserPersistent(prisma),  // { save }
  }
  const cache = CreateUserCache(redis)
  return CreateUserService(persistent, cache)
}
```

---

## File Naming

`kebab-case` everywhere. Suffix declares the role:

| Role | Suffix | Example |
|------|--------|---------|
| Request shape | `.request.ts` | `create-user.request.ts` |
| Response shape | `.response.ts` | `create-user.response.ts` |
| Handler | `.handler.ts` | `create-user.handler.ts` |
| Route | `.routes.ts` | `user.routes.ts` |
| Fastify plugin | `.plugin.ts` | `error-handler.plugin.ts` |
| Domain params | `.params.ts` | `create-user.params.ts` |
| Domain entity | `.entity.ts` | `create-user.entity.ts` |
| Domain use case | `.use-case.ts` | `create-user.use-case.ts` |
| Domain error | `.error.ts` | `user.error.ts` |
| Application service | `.service.ts` | `create-user.service.ts` |
| Port (persistent) | `.persistent.ts` | `create-user.persistent.ts` |
| Port (cache) | `.cache.ts` | `create-user.cache.ts` |
| Infra implementation | `.impl.ts` | `create-user.impl.ts` |
| Infra DB model | `.model.ts` | `user.model.ts` |
| Infra mapper | `.mapper.ts` | `user.mapper.ts` |
| Infra command | `.command.ts` | `create-user.command.ts` |
| Infra query | `.query.ts` | `get-user.query.ts` |
| Container | `.container.ts` | `users.container.ts` |

Function and type names use PascalCase. Interface names prefixed with `I`.

---

## TypeScript Rules

- `strict: true` + `noUncheckedIndexedAccess: true` + `exactOptionalPropertyTypes: true`
- `module: "Node16"` + `moduleResolution: "node16"`
- `readonly` on all type/entity fields
- `type` for shapes; `interface` only for injectable contracts prefixed with `I`
- `import type` for type-only imports (enforced by ESLint)
- `as const` instead of `enum`
- No `any` — use `unknown` and narrow

**`noUncheckedIndexedAccess`** — array index access returns `T | undefined`:

```ts
// ❌ TypeScript error
const first = items[0].toUpperCase()

// ✅ Narrow first
const first = items[0]
if (first === undefined) throw new Error("empty")
console.log(first.toUpperCase())
```

**`exactOptionalPropertyTypes`** — optional fields must declare `| undefined` explicitly:

```ts
description?: string             // ❌ only accepts string, not undefined
description?: string | undefined // ✅ accepts absent or undefined
```

---

## Testing

### Unit test — Application service

```ts
// tests/unit/application/users/create-user.service.test.ts
const mockUser = { id: "1", username: "john", email: "john@example.com", createdAt: "2026-01-01T00:00:00.000Z" }
const mockPersistent = { findByUsername: vi.fn(), save: vi.fn() }
const mockCache = { set: vi.fn() }

describe("CreateUserService", () => {
  beforeEach(() => { vi.clearAllMocks() })

  it("creates user and caches result", async () => {
    mockPersistent.findByUsername.mockResolvedValue(null)
    mockPersistent.save.mockResolvedValue(mockUser)
    mockCache.set.mockResolvedValue(undefined)

    const result = await CreateUserService(mockPersistent, mockCache)(
      { username: "john", email: "john@example.com", password: "secret123" }
    )

    expect(result).toEqual(mockUser)
    expect(mockCache.set).toHaveBeenCalledWith(`user:${mockUser.id}`, mockUser)
  })
})
```

**`status: "pending" as const` in mock objects** — union type fields require `as const` to prevent TypeScript widening `"pending"` to `string`.

### API test — full HTTP cycle

No real server — Fastify's `inject()` handles routing and serialization.

```ts
// tests/api/users/create-user.api.test.ts
function buildTestApp() {
  const createUser = vi.fn().mockResolvedValue(mockUser)
  const app = buildApp({ createUser, getUser: stubGetUser, createTask: stubCreateTask }, { logger: false })
  return { app, createUser }
}

describe("POST /v1/users", () => {
  it("returns 201 on success", async () => {
    const { app } = buildTestApp()
    const res = await app.inject({ method: "POST", url: "/v1/users", payload: validPayload })
    expect(res.statusCode).toBe(201)
    expect(res.json()).toEqual({ code: "0001", message: "created", data: mockUser })
  })
})
```

When a new use case is added to `AppUseCases`, update **every** `buildApp({...})` call in existing API tests — run `pnpm typecheck` to catch missed files.

### Integration test

Guard suite with `TEST_DATABASE_URL`:

```ts
const TEST_DB_URL = process.env["TEST_DATABASE_URL"]
const describeIntegration = TEST_DB_URL ? describe : describe.skip
```

---

## Guardrails

**Read before generating any code.** These rules prevent going outside this architecture.

### 1. Never add try-catch outside `error-handler.plugin.ts`

`src/presentation/api/error-handler.plugin.ts` is the **only** place that catches errors. A try-catch anywhere else silently swallows typed domain errors.

### 2. Never use packages not already imported in source files

Do not import into any source file until the integration pattern is defined in this document:

| Package | Reason blocked |
|---------|---------------|
| `@fastify/cors` | No integration pattern defined |
| `@fastify/helmet` | No integration pattern defined |
| `@fastify/jwt` | Auth pattern not yet defined |
| `dotenv` | CLI-only (`prisma.config.ts`); runtime reads `process.env["KEY"]` directly |

Do not add new packages to `package.json` without explicit instruction.

### 3. Never implement patterns not yet in this codebase

Do not implement until explicitly asked and the pattern is defined in this document:

- List endpoints (`GET /v1/resource` returning array)
- Pagination (offset or cursor)
- Update endpoints (`PUT` / `PATCH`)
- Delete endpoints (`DELETE`)
- Soft delete
- Database transactions (`prisma.$transaction`)
- JWT authentication / route guards

---

## Anti-Patterns

| Anti-pattern | Do instead |
|-------------|-----------|
| `try-catch` in service, handler, or route | Let errors propagate to `error-handler.plugin.ts` |
| Importing `@fastify/cors`, `@fastify/jwt`, etc. in source files | Only use packages already integrated |
| Implementing list/update/delete/pagination before pattern is defined | Stop and ask |
| Logic inside route handler | Move to handler function |
| Handler without use case injection | Factory pattern: `Handler(useCase)(request)` |
| Domain layer with `const` implementations | Domain defines type contracts only |
| Use case type without `Service<>` generic | `type ICreateXUseCase = Service<XParams, XEntity>` |
| Service as a class | Factory function: `const XService = (ports): IUseCase => async (params) => ...` |
| Infra impl with inline `toEntity` | Extract to `mapper/{domain}.mapper.ts` |
| `request/` or `response/` at `api/` root | Always inside `{domain}/` subfolder |
| Importing `@prisma/client` directly | Import from `src/generated/prisma/client` |
| `throw new Error()` in a service | Throw a typed domain error extending `AppError` |
| Plain `type` for request/response shape | Zod schema + `z.infer<>` |
| `app.get<{ Params: { id: string } }>` | `withTypeProvider<ZodTypeProvider>()` with `schema: { params: ParamsSchema }` |
| Entity with sensitive fields (password, token) | Entity exposes only what the caller needs |
| New endpoint wired in `application.ts` | Add to `AppUseCases` in `routes.ts` first |
| `description?: string` with `exactOptionalPropertyTypes` | `description?: string \| undefined` |
| `schema: { response: { 200: SomeSchema } }` on route | Never — response is enforced by handler return type |
| `z.string().optional()` for DB nullable response field | `z.string().nullable()` |
| `params.description` passed directly to Prisma nullable field | `description: params.description ?? null` at command boundary |
| `return Response.ok({ ...entity })` | Map field by field — spread leaks future entity fields |
| `generator client { provider = "prisma-client-js" }` | Prisma v7: `provider = "prisma-client"` |
| Omitting `output` from generator client | Must set `output = "../src/generated/prisma"` |
| `items[0].field` without narrowing | Array index returns `T \| undefined` — check before use |
| Handler input typed as domain Params | Use Zod-inferred Request type |
| `userRoutes(createUser, getUser)(app)` | `app.register(userRoutes(createUser, getUser))` |
| `import "dotenv/config"` in server.ts | Only in `prisma.config.ts` |
| `beforeEach(() => vi.clearAllMocks())` for every test | Only when module-level mocks need different return values per test |
| Handler test missing `expect(useCase).toHaveBeenCalledWith(...)` | Always assert use case args |
| Union type field in mock without `as const` | `status: "pending" as const` |

---

## Project Overrides

_Add project-specific notes here. Do not modify sections above._
