# Last updated: 2026-08-08 | Version: 1 | Status: Template

# STACK_CONTEXT — Go Fullstack Clean Architecture

> **This is a stack template** — copy this file as the project's `STACK_CONTEXT.md` and customize it.
> See usage instructions at [README.md](README.md)

---

## Purpose

Template for projects using:
- **Backend:** Go + Clean Architecture + Fiber
- **Frontend Static:** Next.js (SSR/SSG)
- **Frontend SPA:** Vite + React (admin/internal tool)
- **Persistence:** PostgreSQL (L3) + Redis (L2) + In-memory (L1)

SA copies this template, fills in missing values, and removes sections the project does not use.

---

## Versioning Principle

> **Always use the latest stable version by default** — for all software, frameworks, and packages.
> If the project has constraints requiring an older pinned version, state the reason in an ADR.

---

## 1. Backend API — REST (Go)

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | Go | 1.26+ |
| HTTP Router | Fiber | 3.x |
| Validation | go-playground/validator | v10 |
| Config | caarlos0/env | latest |

### Clean Architecture Layers

Two symmetric groups — Business Domain and Data Source.

```
cmd/api/main.go          entry point — wire infrastructure clients (db, rdb) here only
internal/[domain]/       domain module — wire services within the domain (e.g. user/, order/)

internal/
  ── Business Domain ──────────────────────────────────────────
  entity/                domain model — no external dependencies
  params/                usecase input DTOs
  usecase/               business rule interfaces  (1 file = 1 usecase)
  service/               implements usecase — named interface per usecase, 1 file = 1 usecase

  ── Data Source ──────────────────────────────────────────────
  persistence/           L3 data source interfaces
  cache/                 L2 data source interfaces
  local/                 L1 data source interfaces
  dao/
    persistence/         input params for persistence interface  (package daopersist)
    cache/               input params for cache interface         (package daocache)
    local/               input params for local interface         (package daolocal)
  model/
    persistence/         db row struct — db tags                  (package modelpersist)
    cache/               cache value struct — json tags            (package modelcache)
    local/               local value struct                        (package modellocal)
  postgres/              implements persistence interfaces
  redis/                 implements cache interfaces
  memory/                implements local interfaces
```

**Dependency direction (must not be reversed):**
```
entity ← params ← usecase ← service ← persistence/cache/local ← dao/model ← postgres/redis/memory
```

### REST Conventions

- URL: lowercase kebab-case — `/api/v1/user-profiles`
- Versioning: path-based `/api/v1/...`
- Response envelope:
  ```json
  { "data": {...}, "error": null }
  { "data": null, "error": { "code": "ERR_XXX", "message": "..." } }
  ```
- HTTP status: `200` success, `201` created, `400` validation, `401` unauth, `403` forbidden, `404` not found, `422` business rule, `500` internal
- Error code: `SCREAMING_SNAKE_CASE` e.g. `ERR_USER_NOT_FOUND`

### Constraints

- Business logic must not live in `dao/`, `postgres/`, `redis/`, `memory/` — must be in `usecase/` or `service/` only
- `entity/` must not import any external packages
- `service/` must not import `postgres/`, `redis/`, `memory/` directly — it only knows interface contracts
- `cmd/api/main.go` is the only place to wire infrastructure clients (db, rdb)
- Domain module (`internal/[domain]/module.go`) is where services within the domain are wired
- Each usecase in `service/` must define a **named interface** for its own dependencies (1 set per usecase)
- Package naming: sub-packages use the parent folder as a prefix — `daopersist`, `daocache`, `daolocal`, `modelpersist`, `modelcache`, `modellocal`
- Raw queries must not run in `usecase/` or `service/` — must go through `persistence/cache/local` interfaces only

### Request Lifecycle — Type Contracts per Layer

Each layer receives and returns only its defined types — crossing boundaries is not allowed:

| Step | From | To | Input type | Output type |
|---------|------|----|------------|-------------|
| Handler → service | handler | service | `params.*` | `entity.*` |
| Service → persistence | service | `postgres/` | `daopersist.*` | `modelpersist.*` |
| Service → cache | service | `redis/` | `daocache.*` | `modelcache.*` |
| Mapper | `service/[domain]_mapper.go` | — | `modelpersist.*` / `modelcache.*` | `entity.*` |

- `postgres/` must not return `entity.*` — must return `modelpersist.*` only
- `service/` must not pass `entity.*` or `params.*` as input to persistence — must build a new `daopersist.*` struct
- The mapper (`service/[domain]_mapper.go`) is the only place to convert `modelpersist.*` → `entity.*`

### Dual Interface Contract

Two interface sets with different roles — must not be swapped:

| Set | Location | Role |
|-----|---------|-------|
| **Package interface** | `persistence/user.go` | Documents the full contract of `postgres/` — reference only, **must not be imported in `service/`** |
| **Local named interface** | inside each `service/*.go` | Minimum subset that this specific usecase actually needs — used in real code |

`postgres/` satisfies both sets via Go structural typing — `service/` imports only the local set.

### Anti-patterns (do not do)

```go
// ❌ service/ imports persistence/ package directly
import "clean/internal/persistence"
type s struct { repo persistence.User }

// ✅ define a local interface in the same service file
type createUserPersistence interface {
    StoreUser(ctx context.Context, p daopersist.StoreUser) (modelpersist.User, error)
}
```

```go
// ❌ postgres/ takes or returns entity.* directly
func (r *repo) StoreUser(ctx context.Context, user entity.User) error

// ✅ postgres/ always uses daopersist.* input and modelpersist.* output
func (r *repo) StoreUser(ctx context.Context, p daopersist.StoreUser) (modelpersist.User, error)
```

```go
// ❌ broad interface grouping multiple usecases (Hexagonal port style)
type UserRepository interface {
    Store(...) error
    FindByID(...) (*User, error)
}

// ✅ named interface per usecase
// service/create_user.go:      type createUserPersistence interface { StoreUser(...) }
// service/find_user_by_id.go:  type findUserByIDPersistence interface { FindUserByID(...) }
```

```go
// ❌ wire services directly in cmd/api/main.go
svc := service.NewCreateUserService(postgres.NewUserRepo(db), redis.NewUserCache(rdb))

// ✅ wire in internal/[domain]/module.go — main.go only knows NewModule()
userMod := user.NewModule(db, rdb)
```

---

## 2. Frontend Static — Next.js

> Remove this section if the project has no SSR/SSG frontend.

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | Next.js | 16.x |
| UI Library | React | 19.x |
| Styling | Tailwind CSS | 4.x |
| UI Components | shadcn/ui | latest |
| Language | TypeScript | 7.x |
| Package Manager | pnpm | latest |
| Location | `web/www/` | — |

### Conventions

- Server Components by default — use `"use client"` only when necessary
- Data fetching is done in Server Components — do not fetch directly from Client Components
- Routing: App Router — do not use Pages Router
- Environment variables: use `NEXT_PUBLIC_` prefix only for values exposed to the browser
- shadcn/ui: use `pnpm dlx shadcn@latest add <component>` — components live at `components/ui/`

### Constraints

> SA and Dev must read the release notes for the Next.js version being used before writing code every time (breaking changes per major version).

---

## 3. Frontend SPA — Vite + React

> Remove this section if the project has no SPA (admin/internal tool).

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Build Tool | Vite | 8.x |
| UI Library | React | 19.x |
| Styling | Tailwind CSS | 4.x |
| UI Components | shadcn/ui | latest |
| Language | TypeScript | 7.x |
| Package Manager | pnpm | latest |
| Location | `web/console/` | — |

### Conventions

- State management: Zustand (latest) — SA may deviate if the use case is primarily server state (TanStack Query)
- Routing: TanStack Router (latest) — type-safe, file-based routing
- API calls: via a centralised HTTP client — do not fetch directly inside components
- Environment variables: use `VITE_` prefix only for values exposed to the browser
- shadcn/ui: use `pnpm dlx shadcn@latest add <component>` — components live at `src/components/ui/`

### Constraints

- `web/console/` is an admin/internal tool — not public-facing
- Do not share code directly with `web/www/` — if sharing is needed, extract into a separate package

---

## 4. Persistence

> Persistence is divided into 3 scopes based on usage characteristics. SA must specify which scope each feature uses.

### Scope Overview

| Level | Scope | Location in Clean Architecture | Characteristics | Default |
|-------|-------|----------------------------|--------|---------|
| **L1** | **Local** | `internal/local/` | In-memory of that instance — not shared across instances | Go map / sync.Map |
| **L2** | **Cache** | `internal/cache/` | Centralised in-memory — shared by all instances, supports horizontal scaling | Redis |
| **L3** | **Persistence** | `internal/postgres/` (or other driver) | Primary data store — persistent, shared by all instances | PostgreSQL |

```
Scope selection examples:
  compiled template, parsed config → L1 Local      — instance-specific, can regenerate
  session token, rate limit count  → L2 Cache      — needs consistency across instances
  order, user, product data        → L3 Persistence — source of truth
```

### 4.1 (L1) Local — Per-instance In-Memory

| Item | Value |
|------|-------|
| Default package | `maypok86/otter` (latest) — S3-FIFO eviction, generics, TTL per item |
| When to use a plain map instead | Data requires no eviction and is loaded once at startup |
| Location | `internal/local/` |

**Constraints:**
- Do not use Local for data that must be consistent across instances — use L2 Cache instead
- Must always be thread-safe — do not use plain `map` without a mutex in a concurrent context

### 4.2 (L2) Cache — Redis

| Item | Value |
|------|-------|
| Default | Redis 8+ |
| Driver (Go) | `go-redis/v9` |
| Key pattern | `{service}:{entity}:{id}` — `auth:session:abc123` |
| TTL | Must always be specified — do not set a key without a TTL |

**Constraints:**
- Cache is not the source of truth — must always be able to invalidate or rebuild
- Data in cache must be serializable — do not store Go structs with unexported fields

### 4.3 (L3) Persistence — PostgreSQL

| Item | Value |
|------|-------|
| Min version | 18+ |
| Driver (Go) | `pgx/v5` + `pgx/v5/stdlib` adapter |
| Connection pool | `pgxpool` — do not open connections directly in request handlers |
| Query management | `sqlx` (latest) — struct scan, named query |
| Migration tool | `golang-migrate` (latest) |

**Naming conventions:**
- Table: `snake_case` plural — `user_profiles`, `order_items`
- Column: `snake_case` — `created_at`, `updated_at`
- Index: `idx_{table}_{column(s)}` — `idx_users_email`
- FK: `fk_{table}_{ref_table}` — `fk_orders_users`
- Every table must have `id`, `created_at`, `updated_at`

**Constraints:**
- Migration scripts must be idempotent and reversible (up/down)
- Do not run raw queries in `usecase/` or `service/`

> **SA may choose a different storage type depending on the use case** — must specify in an ADR.
> Options: MongoDB (document), DynamoDB (key-value), S3-compatible (object store)

---

## 5. Cross-cutting Concerns

### Authentication & Authorization

- Protocol: **OAuth2 + OIDC** (Authorization Code + PKCE)
- IdP: **[SA specifies — Keycloak / Azure AD / Okta / other]** — specify in ADR

**Backend (Go) — packages:**

| Package | Version | Used for |
|---------|---------|----------|
| `golang-jwt/jwt` | v5 | JWT validation + claims parsing |
| `coreos/go-oidc` | v3 | JWKS verification + OIDC discovery |
| `golang.org/x/oauth2` | latest | OAuth2 flow (when backend acts as OAuth2 client) |

**Frontend SPA — package:**

| Package | Version | Used for |
|---------|---------|----------|
| `oidc-client-ts` | latest | Authorization Code + PKCE, token refresh, session |

**Constraints:**
- Tokens/sessions must never be logged and must never be returned in error responses
- Access tokens must be validated through the JWKS endpoint
- Always use PKCE for SPAs — do not use Implicit flow

### Observability (OpenTelemetry)

**Backend (Go) — core packages:**

| Package | Version | Used for |
|---------|---------|----------|
| `go.opentelemetry.io/otel` | 1.44+ | API core |
| `go.opentelemetry.io/otel/sdk` | 1.44+ | Trace + Metric SDK |
| `go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp` | 1.44+ | Export traces via OTLP/HTTP |
| `go.opentelemetry.io/contrib/bridges/otelslog` | latest | slog → OTel log bridge |

> Framework instrumentation — SA chooses based on router and driver (default: `otelfiber`)

**Frontend Static (Next.js) — core packages:**

| Package | Version | Used for |
|---------|---------|----------|
| `@opentelemetry/api` | 1.9.x | API core |
| `@opentelemetry/sdk-node` | 0.220.x | Server-side SDK |
| `@opentelemetry/exporter-trace-otlp-http` | 0.220.x | OTLP export |

> Entry point: `instrumentation.ts` at the root (Next.js built-in since 15+).
> Alternative: `@vercel/otel` instead of manual setup.

**Frontend SPA (Vite/Browser) — core packages:**

| Package | Version | Used for |
|---------|---------|----------|
| `@opentelemetry/api` | 1.9.x | API core |
| `@opentelemetry/sdk-trace-web` | 0.220.x | Browser trace SDK |
| `@opentelemetry/exporter-trace-otlp-http` | 0.220.x | OTLP export |
| `@opentelemetry/context-zone` | 0.220.x | Async context propagation |

**Constraints:**
- Export protocol: OTLP — do not tie directly to vendor-specific formats
- Trace context: W3C TraceContext (`traceparent` header)
- Every service must send the `service.name` resource attribute
- Do not log PII — use masked or hashed values

### Environment Variables

| Prefix | Used for |
|--------|---------|
| `APP_` | Application config — `APP_PORT`, `APP_ENV` |
| `DB_` | Database — `DB_HOST`, `DB_PORT`, `DB_NAME` |
| `REDIS_` | Cache — `REDIS_HOST`, `REDIS_PORT` |

- Every secret must use an environment variable — no hardcoding

### PDPA (Thailand)

- Personal data: full name, national ID, phone, email, address, health data
- Every feature that stores personal data must specify a retention period in the Solution Doc
- Must have a deletion mechanism matching the retention period

---

## 6. SA Decision Fields

SA must specify the following values clearly in the Solution Doc for each feature:

| Field | Default | Alternatives |
|-------|---------|---------|
| HTTP Router | Fiber v3 | Chi / Echo / other + reason |
| Validation Library | go-playground/validator/v10 | other |
| Config Library | caarlos0/env | koanf (multi-source) / other |
| Auth Method | OAuth2 + OIDC + PKCE | — IdP: SA chooses per project |
| SPA State Management | Zustand | TanStack Query / other |
| SPA Router | TanStack Router | React Router v7 / other |
| Migration Tool | golang-migrate | goose / other |
| PostgreSQL Driver | pgx/v5 | lib/pq |
| OTel Backend Instrumentation | otelfiber | otelhttp / otelchi / otelecho + otelpgx / redisotel |
| OTel Next.js Setup | @vercel/otel (simple) | manual sdk-node (full control) |
| OTel Browser Instrumentation | — | fetch / document-load / user-interaction / auto-instrumentations-web |

---

## 7. Stack Deviations

If any feature requires deviating from this stack, SA must:
1. Write an ADR stating the reason and trade-offs
2. Notify PO before sending the Solution Doc
3. Update `STACK_CONTEXT.md` if the deviation becomes the new standard

---

## Stack Versions (verified [date])

> SA must fill this section with real versions verified via WebSearch before sending to PO.

| Package / Runtime | Version | Verified date |
|------------------|---------|--------------|
| Go | [verify] | — |
| Fiber | [verify] | — |
| Next.js | [verify] | — |
| React | [verify] | — |
| Tailwind CSS | [verify] | — |
| TypeScript | [verify] | — |
| PostgreSQL | [verify] | — |
| Redis | [verify] | — |
| pgx/v5 | [verify] | — |
| go-redis/v9 | [verify] | — |
| golang-migrate | [verify] | — |
| go.opentelemetry.io/otel | [verify] | — |
