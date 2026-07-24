# Last updated: 2026-07-22 | Version: 15 | Status: Template

# STACK_CONTEXT.md — Go Fullstack Clean Architecture

> SA เป็นเจ้าของไฟล์นี้ อัปเดตทุกครั้งที่ stack เปลี่ยน และ copy ไปที่ `docs/roles/po/STACK_CONTEXT.md` หลัง finalize

---

## Purpose

ไฟล์นี้เป็น baseline tech stack สำหรับทุก project ที่ใช้ SDLC Playbook บน Go Fullstack Clean Architecture
SA ต้องอ่านก่อนเสนอ Solution Doc ทุกครั้ง — ห้ามเบี่ยงเบนจาก stack นี้โดยไม่ระบุใน ADR

---

## Versioning Principle

> **ใช้ version ล่าสุดเป็น default เสมอ** — ทุก software, framework, และ package
> หาก project มีข้อจำกัดที่ต้อง pin version เก่า ให้ระบุเหตุผลใน ADR

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

สองกลุ่ม symmetric — Business Domain และ Data Source

```
cmd/api/main.go          entry point — wire infrastructure clients (db, rdb) ที่นี่เท่านั้น
internal/[domain]/       domain module — wire services ภายใน domain (เช่น user/, order/)

internal/
  ── Business Domain ──────────────────────────────────────────
  entity/                domain model — ไม่มี external dependency
  params/                usecase input DTOs
  usecase/               business rule interfaces  (1 file = 1 usecase)
  service/               implements usecase — named interface per usecase, 1 file = 1 usecase

  ── Data Source ──────────────────────────────────────────────
  persistence/           L3 data source interfaces
  cache/                 L2 data source interfaces
  local/                 L1 data source interfaces
  dao/
    persistence/         input params สำหรับ persistence interface  (package daopersist)
    cache/               input params สำหรับ cache interface         (package daocache)
    local/               input params สำหรับ local interface         (package daolocal)
  model/
    persistence/         db row struct — db tags                     (package modelpersist)
    cache/               cache value struct — json tags               (package modelcache)
    local/               local value struct                           (package modellocal)
  postgres/              implements persistence interfaces
  redis/                 implements cache interfaces
  memory/                implements local interfaces
```

**Dependency direction (ห้ามย้อน):**
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
- Error code: `SCREAMING_SNAKE_CASE` เช่น `ERR_USER_NOT_FOUND`

### Constraints

- Business logic ห้ามอยู่ใน `dao/`, `postgres/`, `redis/`, `memory/` — ต้องอยู่ใน `usecase/` หรือ `service/` เท่านั้น
- `entity/` ห้าม import package ภายนอก
- `service/` ห้าม import `postgres/`, `redis/`, `memory/` โดยตรง — รู้จักแค่ interface contracts
- `cmd/api/main.go` เป็นที่เดียวที่ wire infrastructure clients (db, rdb)
- Domain module (`internal/[domain]/module.go`) เป็นที่ wire services ภายใน domain
- แต่ละ usecase ใน `service/` ต้อง define **named interface** สำหรับ dependencies ของตัวเอง (1 ชุดต่อ 1 usecase)
- Package naming: sub-packages ใช้ prefix = parent folder — `daopersist`, `daocache`, `daolocal`, `modelpersist`, `modelcache`, `modellocal`
- ห้าม run raw query ใน `usecase/` หรือ `service/` — ต้องผ่าน `persistence/cache/local` interfaces เท่านั้น

### Request Lifecycle — Type Contracts per Layer

แต่ละชั้นรับ-ส่ง type ที่กำหนดไว้เท่านั้น — ห้ามข้ามขอบเขต:

| ขั้นตอน | From | To | Input type | Output type |
|---------|------|----|------------|-------------|
| Handler → service | handler | service | `params.*` | `entity.*` |
| Service → persistence | service | `postgres/` | `daopersist.*` | `modelpersist.*` |
| Service → cache | service | `redis/` | `daocache.*` | `modelcache.*` |
| Mapper | `service/[domain]_mapper.go` | — | `modelpersist.*` / `modelcache.*` | `entity.*` |

- `postgres/` ห้าม return `entity.*` — ต้อง return `modelpersist.*` เท่านั้น
- `service/` ห้าม pass `entity.*` หรือ `params.*` เป็น input ให้ persistence — ต้องสร้าง `daopersist.*` struct ใหม่
- mapper (`service/[domain]_mapper.go`) เป็นที่เดียวที่แปลง `modelpersist.*` → `entity.*`

### Dual Interface Contract

มี interface สองชุดที่คนละบทบาท — ห้ามสลับกัน:

| ชุด | ที่อยู่ | บทบาท |
|-----|---------|-------|
| **Package interface** | `persistence/user.go` | Document full contract ของ `postgres/` — เป็น reference เท่านั้น **ห้าม import ใน `service/`** |
| **Local named interface** | ในแต่ละ `service/*.go` | Minimum subset ที่ usecase นั้นต้องการจริงๆ — ใช้จริงใน code |

`postgres/` satisfy ทั้งสองชุดผ่าน Go structural typing — `service/` import เฉพาะชุด local เท่านั้น

### Anti-patterns (ห้ามทำ)

```go
// ❌ service/ import persistence/ package โดยตรง
import "clean/internal/persistence"
type s struct { repo persistence.User }

// ✅ define local interface ใน service file เดียวกัน
type createUserPersistence interface {
    StoreUser(ctx context.Context, p daopersist.StoreUser) (modelpersist.User, error)
}
```

```go
// ❌ postgres/ รับหรือ return entity.* โดยตรง
func (r *repo) StoreUser(ctx context.Context, user entity.User) error

// ✅ postgres/ ใช้ daopersist.* input และ modelpersist.* output เสมอ
func (r *repo) StoreUser(ctx context.Context, p daopersist.StoreUser) (modelpersist.User, error)
```

```go
// ❌ broad interface รวมหลาย usecase (Hexagonal port style)
type UserRepository interface {
    Store(...) error
    FindByID(...) (*User, error)
}

// ✅ named interface per usecase
// service/create_user.go:      type createUserPersistence interface { StoreUser(...) }
// service/find_user_by_id.go:  type findUserByIDPersistence interface { FindUserByID(...) }
```

```go
// ❌ wire services ใน cmd/api/main.go โดยตรง
svc := service.NewCreateUserService(postgres.NewUserRepo(db), redis.NewUserCache(rdb))

// ✅ wire ใน internal/[domain]/module.go — main.go รู้จักแค่ NewModule()
userMod := user.NewModule(db, rdb)
```

---

## 2. Frontend Static — Next.js 16

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

- Server Components by default — ใช้ `"use client"` เฉพาะเมื่อจำเป็น
- Data fetching ทำใน Server Component — ห้าม fetch จาก Client Component โดยตรง
- Route: App Router (Next.js 16) — ห้ามใช้ Pages Router
- Environment variable: ใช้ `NEXT_PUBLIC_` prefix เฉพาะตัวที่ expose ถึง browser
- shadcn/ui: ใช้ `pnpm dlx shadcn@latest add <component>` เพื่อ add component — component อยู่ที่ `components/ui/`

### Constraints

> **สำคัญ:** Next.js 16 มี breaking changes จาก version ก่อนหน้า
> SA และ Dev ต้องอ่าน `web/www/node_modules/next/dist/docs/` ก่อนเขียน code ทุกครั้ง

---

## 3. Frontend SPA — Vite + React (Dynamic)

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

- State management: Zustand (latest) — SA เบี่ยงเบนได้ถ้า use case ต้องการ server state เป็นหลัก (TanStack Query)
- Routing: TanStack Router (latest) — type-safe, file-based routing
- API call: ผ่าน HTTP client ที่ centralise — ห้าม fetch ตรงใน component
- Environment variable: ใช้ `VITE_` prefix เฉพาะตัวที่ expose ถึง browser
- shadcn/ui: ใช้ `pnpm dlx shadcn@latest add <component>` เพื่อ add component — component อยู่ที่ `src/components/ui/`

### Constraints

- `web/console/` เป็น admin/internal tool — ไม่ใช่ public-facing
- ห้าม share code โดยตรงกับ `web/www/` — ถ้าต้องการ share ให้ extract เป็น package แยก

---

## 4. Persistence

> Persistence แบ่งเป็น 3 scope ตามลักษณะการใช้งาน SA ต้องระบุในแต่ละ feature ว่าใช้ scope ใด

### Scope Overview

| Level | Scope | ที่อยู่ใน Clean Architecture | ลักษณะ | Default |
|-------|-------|----------------------------|--------|---------|
| **L1** | **Local** | `internal/local/` | In-memory ของ instance นั้นๆ — ไม่ shared ข้าม instance | Go map / sync.Map |
| **L2** | **Cache** | `internal/cache/` | In-memory ศูนย์กลาง — shared ทุก instance, รองรับ horizontal scale | Redis |
| **L3** | **Persistence** | `internal/postgres/` (หรือ driver อื่น) | Primary data store — ข้อมูลถาวร, shared ทุก instance | PostgreSQL |

```
ตัวอย่างการเลือก scope:
  compiled template, parsed config → L1 Local      — instance-specific, regenerate ได้
  session token, rate limit count  → L2 Cache      — ต้องการ consistency ข้าม instance
  ข้อมูล order, user, product      → L3 Persistence — source of truth
```

---

### 4.1 (L1) Local — Per-instance In-Memory

ใช้เมื่อต้องการ in-memory ที่ **ไม่ต้องการ share ข้าม instance** — เช่น compiled template, parsed config, lookup table ที่ load ครั้งเดียว

| Item | Value |
|------|-------|
| Default package | `maypok86/otter` (latest) — S3-FIFO eviction, generics, TTL per item |
| ใช้ plain map แทนได้เมื่อ | ข้อมูลไม่ต้องการ eviction และ load ครั้งเดียวตอน startup เช่น parsed config, lookup table |
| ที่อยู่ | `internal/local/` |
| Lifecycle | เริ่มต้นเมื่อ app start, หายไปเมื่อ instance ปิด |

**Constraints:**
- ห้ามใช้ Local สำหรับข้อมูลที่ต้อง consistent ข้าม instance — ใช้ L2 Cache แทน
- ต้อง thread-safe เสมอ — ห้ามใช้ plain `map` โดยไม่มี mutex ใน concurrent context
- ข้อมูลใน Local ต้อง regenerate ได้โดยไม่สูญเสีย business data

---

### 4.2 (L2) Cache — Centralized In-Memory (Multi-instance)

ใช้เมื่อต้องการ in-memory ที่ **shared ข้าม instance** — เช่น session, rate limit, pub/sub, distributed lock

| Item | Value |
|------|-------|
| Default | Redis 8+ |
| Driver (Go) | `go-redis/v9` (แนะนำ) |
| Key pattern | `{service}:{entity}:{id}` — `auth:session:abc123` |
| TTL | ต้องระบุเสมอ — ห้าม set key โดยไม่มี TTL |

**Constraints:**
- Cache ไม่ใช่ source of truth — ต้อง invalidate หรือ rebuild ได้เสมอ
- ห้ามเก็บข้อมูลที่ต้องการ durability สูงใน cache เท่านั้น — ต้องมี L3 Persistence รองรับด้วย
- Data ใน cache ต้อง serializable — ห้าม store Go struct ที่มี unexported field

---

### 4.3 (L3) Persistence — Primary Data Store

Default คือ RDBMS (PostgreSQL) แต่ SA สามารถเลือก storage type อื่นได้ตาม use case โดยต้องระบุใน ADR

| Storage Type | Use case | ตัวอย่าง |
|-------------|----------|---------|
| RDBMS (default) | Relational data, transaction, ACID | PostgreSQL |
| NoSQL Document | Flexible schema, nested document | MongoDB |
| NoSQL Key-Value | Simple lookup, high-throughput | DynamoDB |
| File / Object Store | Binary, media, large document | S3-compatible |

**เมื่อใช้ PostgreSQL (default):**

| Item | Value |
|------|-------|
| Min version | 18+ |
| Driver (Go) | `pgx/v5` + `pgx/v5/stdlib` adapter |
| Connection pool | `pgxpool` — ห้าม open connection ตรงใน request handler |
| Query management | `sqlx` (latest) — struct scan, named query; ใช้คู่กับ `pgx/v5/stdlib` |
| Migration tool | `golang-migrate` (latest) |

**Naming conventions (PostgreSQL):**
- Table: `snake_case` พหูพจน์ — `user_profiles`, `order_items`
- Column: `snake_case` — `created_at`, `updated_at`
- Index: `idx_{table}_{column(s)}` — `idx_users_email`
- FK: `fk_{table}_{ref_table}` — `fk_orders_users`
- ทุก table ต้องมี `id`, `created_at`, `updated_at`

**Constraints:**
- L3 Persistence เป็น source of truth เสมอ
- Migration script ต้อง idempotent และ reversible (up/down)
- ห้าม run raw query ใน `usecase/` หรือ `service/` — ต้องผ่าน `persistence/cache/local` interfaces

---

## 5. Cross-cutting Concerns

### Authentication & Authorization

- Protocol: **OAuth2 + OIDC** (Authorization Code + PKCE)
- IdP: SA เลือกต่อ project (Keycloak / Azure AD / Okta / อื่น) — ระบุใน ADR

**Backend (Go) — packages:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| `golang-jwt/jwt` | v5 | JWT validation + claims parsing |
| `coreos/go-oidc` | v3 | JWKS verification + OIDC discovery (auto key rotation) |
| `golang.org/x/oauth2` | latest | OAuth2 flow (เมื่อ backend act as OAuth2 client) |

**Frontend SPA — package:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| `oidc-client-ts` | latest | Authorization Code + PKCE, token refresh, session — IdP-agnostic |

**Constraints:**
- Token/session ห้าม log และห้าม return ใน error response
- Access token ต้อง validate ผ่าน JWKS endpoint — ห้าม trust โดยไม่ verify signature
- ใช้ PKCE เสมอสำหรับ SPA — ห้ามใช้ Implicit flow

### Observability (OpenTelemetry)

ใช้ OpenTelemetry เป็น standard สำหรับ Traces, Metrics, และ Logs ทั้ง frontend และ backend

**Backend (Go) — core packages:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| `go.opentelemetry.io/otel` | 1.44+ | API core |
| `go.opentelemetry.io/otel/sdk` | 1.44+ | Trace + Metric SDK |
| `go.opentelemetry.io/otel/exporters/otlp/otlptrace/otlptracehttp` | 1.44+ | Export traces ผ่าน OTLP/HTTP |
| `go.opentelemetry.io/contrib/bridges/otelslog` | latest | slog → OTel log bridge |

> Framework instrumentation (otelfiber / otelhttp / otelecho / otelchi / otelpgx ฯลฯ) — SA เลือกตาม router และ driver ที่ใช้ (default: `otelfiber` จาก `github.com/gofiber/contrib/otelfiber`)

- Log level: `DEBUG` (dev), `INFO` (prod default), `ERROR` (critical)
- ห้าม log PII (email, phone, national ID) — ใช้ masked หรือ hashed value

**Frontend Static (Next.js) — core packages:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| `@opentelemetry/api` | 1.9.x | API core |
| `@opentelemetry/sdk-node` | 0.220.x | Server-side SDK |
| `@opentelemetry/exporter-trace-otlp-http` | 0.220.x | OTLP export |

- Entry point: `instrumentation.ts` ที่ root (Next.js built-in ตั้งแต่ 15+)
- ทางเลือก: `@vercel/otel` แทน manual setup (SA ระบุใน Decision Fields)

**Frontend SPA (Vite/Browser) — core packages:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| `@opentelemetry/api` | 1.9.x | API core |
| `@opentelemetry/sdk-trace-web` | 0.220.x | Browser trace SDK |
| `@opentelemetry/exporter-trace-otlp-http` | 0.220.x | OTLP export |
| `@opentelemetry/context-zone` | 0.220.x | Async context propagation |

> Browser instrumentation (fetch, XHR, document-load, user-interaction ฯลฯ) — SA เลือกตาม signal ที่ project ต้องการ

**Constraints:**
- Exporter protocol: OTLP (HTTP/gRPC) — ห้าม tie ถึง vendor-specific format โดยตรง
- Trace context propagation: W3C TraceContext standard (`traceparent` header)
- ทุก service ต้องส่ง `service.name` attribute ใน resource

### Environment Variables

- ทุก secret ต้องใช้ environment variable — ห้าม hardcode
- ใช้ `APP_` prefix สำหรับ application config — `APP_PORT`, `APP_ENV`
- ใช้ `DB_` prefix สำหรับ database — `DB_HOST`, `DB_PORT`, `DB_NAME`
- ใช้ `REDIS_` prefix สำหรับ cache — `REDIS_HOST`, `REDIS_PORT`

### PDPA (Thailand)

- Personal data ได้แก่: ชื่อ, เลขบัตร, เบอร์โทร, email, ที่อยู่, ข้อมูลสุขภาพ
- ทุก feature ที่เก็บ personal data ต้องระบุ retention period ใน Solution Doc
- ลบข้อมูลตาม retention period — ต้องมี mechanism ใน design

---

## 6. SA Decision Fields (ต้องระบุเมื่อสร้าง Solution Doc)

SA ต้องระบุค่าต่อไปนี้ให้ชัดเจนใน Solution Doc ของแต่ละ feature:

| Field | Description |
|-------|-------------|
| HTTP Router | Fiber v3 (default) / Chi / Echo / อื่น + เหตุผล |
| Validation Library | go-playground/validator/v10 (default) / อื่น |
| Config Library | caarlos0/env (default) / koanf (multi-source) / อื่น |
| Auth Method | OAuth2 + OIDC + PKCE (default) — IdP: SA เลือกต่อ project |
| SPA State Management | Zustand (default) / TanStack Query / อื่น |
| SPA Router | TanStack Router (default) / React Router v7 / อื่น |
| Migration Tool | golang-migrate (default) / goose / อื่น |
| PostgreSQL Driver | pgx/v5 (default) / lib/pq |
| OTel Backend Instrumentation | otelfiber (default) / otelhttp / otelchi / otelecho + otelpgx / redisotel |
| OTel Next.js Setup | @vercel/otel (simple) / manual sdk-node (full control) |
| OTel Browser Instrumentation | เลือก signal: fetch / document-load / user-interaction / auto-instrumentations-web |

---

## 7. Stack Deviations

หาก feature ใดจำเป็นต้องเบี่ยงเบนจาก stack นี้ SA ต้อง:
1. เขียน ADR ระบุเหตุผลและ trade-off
2. แจ้ง PO ก่อนส่ง Solution Doc
3. อัปเดต STACK_CONTEXT.md ถ้า deviation นั้นกลายเป็น standard ใหม่
