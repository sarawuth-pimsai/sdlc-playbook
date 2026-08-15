# Last updated: 2026-08-15 | Version: 1 | Status: Template

# STACK_CONTEXT — Node.js Fastify Clean Architecture

> **This is a stack template** — copy this file as the project's `STACK_CONTEXT.md` and customize it.
> See usage instructions at [README.md](README.md)

---

## Purpose

Template for Node.js backend projects using Fastify v5 + Prisma v7 + 5-layer Clean Architecture + PostgreSQL.

SA copies this template, fills in SA Decision Fields, and removes sections the project does not use.

---

## Project Settings

| Field | Value | Note |
|-------|-------|------|
| Output language | `en` | `en` = English (default) \| `th` = Thai — change to `th` for Thai-language teams |

---

## Versioning Principle

> **Always verify versions with WebSearch before finalizing** — Fastify, Prisma, and Zod have breaking changes between major versions. Pin every package to a specific major version.

---

## 1. Backend — Node.js (Fastify)

### Stack

| Package | Version | Role |
|---------|---------|------|
| Node.js | 22+ | Runtime |
| TypeScript | v5 | Language |
| Fastify | v5 | HTTP framework |
| Zod | v4 | Schema validation + type inference |
| fastify-type-provider-zod | v7 | Zod ↔ Fastify type bridge |
| Prisma | v7 | ORM — generated client in `src/generated/prisma/` |
| @prisma/adapter-pg + pg | v7 / v8 | Prisma driver adapter |
| Vitest | v4 | Testing |
| pnpm | v10 | Package manager |

### Architecture — 5-Layer Clean Architecture

Dependency direction flows **inward only**. ESLint enforces this at file level via `no-restricted-imports`.

```
HTTP Request
    ↓
src/presentation/     HTTP wiring — routes, request/response shapes, handlers
    ↓
src/domain/           Contracts only — use case types, params, entities
    ↓
src/application/      Business logic — service factories + port interfaces
    ↓
src/infrastructure/   Implementations — PostgreSQL (Prisma), Redis
    ↑
src/container/        Composition root — imports all layers, assembles use cases
```

`src/shared/` holds cross-layer primitives (`AppError`, `Service<>` generic).

### Architecture Boundary Enforcement

Boundaries are enforced by ESLint `no-restricted-imports` rules — a misconfigured `eslint.config.js` lets violations compile silently. After bootstrap, verify with a probe file before proceeding (see companion Dev template).

| Layer | Cannot import from |
|-------|--------------------|
| `src/domain/**` | `application`, `infrastructure`, `presentation`, `container` |
| `src/application/**` | `infrastructure`, `presentation`, `container` |
| `src/infrastructure/**` | `presentation`, `container` |
| `src/presentation/**` | `infrastructure`, `container` |
| `src/container/**` | — (composition root, imports all) |

### TypeScript Strict Configuration

```json
"strict": true,
"noUncheckedIndexedAccess": true,
"exactOptionalPropertyTypes": true,
"noImplicitOverride": true,
"module": "Node16",
"moduleResolution": "node16"
```

These flags are **non-negotiable** — they prevent entire classes of runtime errors. Do not relax without an ADR.

---

## 2. Database — PostgreSQL (Prisma v7)

### Prisma v7 Critical Notes

> Prisma v7 has breaking changes from v6 that cause silent failures if missed.

| Rule | Correct | Wrong |
|------|---------|-------|
| Generator provider | `"prisma-client"` | ~~`"prisma-client-js"`~~ |
| Client import path | `src/generated/prisma/client` | ~~`@prisma/client`~~ |
| `output` in schema | Must set `output = "../src/generated/prisma"` | Omitting breaks all imports |
| DB URL in schema | Not present — goes in `prisma.config.ts` only | ~~`url = env("DATABASE_URL")`~~ in schema |
| Runtime client | Requires driver adapter: `new PrismaClient({ adapter: new PrismaPg(pool) })` | ~~`new PrismaClient()`~~ without adapter |

### SA Decision Fields — Database

| Field | Value |
|-------|-------|
| Database host | _[fill in: Aurora / Cloud SQL / self-hosted / other]_ |
| Database version | _[fill in: PostgreSQL 16 / 15 / other]_ |
| Migration strategy | _[fill in: Prisma Migrate / manual / other]_ |
| Connection pool size | _[fill in or leave default]_ |

---

## 3. Testing

### Test Layer Strategy

| Layer | Test type | Reason |
|-------|-----------|--------|
| `domain/` | None | Type contracts only — no logic to test |
| `application/` | Unit (Vitest) | All business logic lives here |
| `infrastructure/` | Integration (real DB) | Must verify queries against real DB — mocks hide migration failures |
| `presentation/` | Unit + API (Fastify `inject()`) | Unit: handler mapping. API: full HTTP cycle without a running server |
| `container/` | None | Pure wiring |

### SA Decision Fields — Testing

| Field | Value |
|-------|-------|
| Test database | _[fill in: shared CI DB / ephemeral per-run / other]_ |
| Coverage threshold | _[fill in or leave default]_ |

---

## 4. Error Handling

Single global error handler at `src/presentation/api/error-handler.plugin.ts`. Domain errors extend `AppError` and propagate automatically — **no try-catch outside the global handler**.

### Error Code Ranges

| Range | Domain |
|-------|--------|
| `0000` | Success |
| `0001` | Created |
| `0400` | Zod validation failure |
| `9000` | Unhandled server error |
| `1xxx` | _[SA: assign per domain — e.g. `1xxx` = Users]_ |
| `2xxx` | _[SA: assign per domain]_ |

---

## 5. Endpoint Format

| Convention | Value |
|------------|-------|
| URL versioning | `/v1/` prefix on every route |
| Resource naming | Plural noun — `/v1/users`, `/v1/tasks` |
| Path parameter | `:id` — `GET /v1/users/:id` |
| Response envelope | `{ code, message, data }` — `code` is a string, not HTTP status |

---

## SA Decision Fields — Summary

Fill in all fields marked `_[fill in: ...]_` before sending to PO.

| Area | Field | Status |
|------|-------|--------|
| Project | Project name | ☐ |
| Project | Output language | ☐ |
| Database | Host / managed service | ☐ |
| Database | PostgreSQL version | ☐ |
| Auth | Identity Provider | ☐ |
| Auth | Token storage strategy | ☐ |
| Error codes | Domain code ranges | ☐ |
| Testing | Test database setup | ☐ |

---

## Dev Companion

The Dev implementation guide for this stack (bootstrap steps, layer-by-layer code examples, anti-patterns) is at:

```
docs/roles/dev/stack-templates/node-fastify/DEV_PATTERNS.md
```

SA/Lead: include this path in the Lead Handoff so Dev knows where to find it.
