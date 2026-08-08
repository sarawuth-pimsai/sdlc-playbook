# Last updated: YYYY-MM-DD | Version: 1 | Status: [Project Name]

# STACK_CONTEXT.md — [Project Name]

> SA เป็นเจ้าของไฟล์นี้ อัปเดตทุกครั้งที่ stack เปลี่ยน และ copy ไปที่ `docs/roles/po/STACK_CONTEXT.md` หลัง finalize

---

> **ไฟล์นี้คือ blank schema** — ใช้เมื่อไม่มี stack template ที่ตรงกับ project ของคุณ
>
> ถ้ามี template ที่ match — ใช้ template จาก `docs/roles/sa/stack-templates/` แทน (เริ่มได้เร็วกว่า)
> ดูรายการ templates ที่ [stack-templates/README.md](stack-templates/README.md)

---

## Purpose

[ระบุ project name และสรุป tech stack family ที่เลือก — 1-2 ประโยค]

---

## Versioning Principle

[ระบุนโยบาย versioning — เช่น "ใช้ latest stable เสมอ" หรือ "pin ทุก version เพราะ production constraint"]

---

## 1. Backend

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | [Go / Python / Node.js / Java / อื่น] | [version] |
| Web Framework | [framework] | [version] |
| Validation | [library] | [version] |
| Config | [library] | [version] |

### Architecture Pattern

[ระบุ pattern: Clean Architecture / Hexagonal / Layered / MVC / อื่น]

**Directory structure:**

```
[วาด directory structure ที่ใช้จริง]
```

**Layer responsibilities:**

[อธิบาย responsibility ของแต่ละ layer และ dependency direction]

### API Conventions

[URL format, versioning strategy, response envelope format, HTTP status codes, error code format]

### Constraints

[กฎที่ Dev และ SA ต้องปฏิบัติตามเสมอ — เขียนเฉพาะที่ non-obvious]

---

## 2. Frontend (ลบ section นี้ถ้าไม่มี frontend)

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | [Next.js / Vite+React / Vue / Angular / อื่น] | [version] |
| Language | [TypeScript / JavaScript] | [version] |
| Styling | [Tailwind / CSS Modules / Styled Components / อื่น] | [version] |
| Package Manager | [pnpm / npm / yarn] | [version] |

### Conventions

[routing pattern, state management, data fetching pattern, component organization]

### Constraints

[กฎสำคัญที่ Dev ต้องรู้]

---

## 3. Persistence

| Level | Technology | Use case | Location |
|-------|-----------|---------|---------|
| L1 Local | [library / plain map] | per-instance state ที่ไม่ shared | `internal/local/` |
| L2 Cache | [Redis / Memcached / อื่น] | shared state ข้าม instance | `internal/cache/` |
| L3 Persistence | [PostgreSQL / MySQL / MongoDB / อื่น] | source of truth | `internal/[driver]/` |

[ลบ row ที่ project ไม่ใช้]

### Primary Database Conventions

**[ชื่อ database engine]:**

| Item | Value |
|------|-------|
| Version | [version] |
| Driver | [package + version] |
| Migration tool | [tool] |

[naming conventions — table, column, index, FK]

**Constraints:**

[กฎสำคัญเกี่ยวกับ persistence]

---

## 4. Cross-cutting Concerns

### Authentication & Authorization

- Protocol: [OAuth2 + OIDC + PKCE (แนะนำ) / อื่น + เหตุผล]
- IdP: [Keycloak / Azure AD / Okta / อื่น] — ระบุใน ADR

**Backend packages:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| [package] | [version] | [purpose] |

**Frontend packages (ถ้ามี):**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| [package] | [version] | [purpose] |

**Constraints:**
- [ระบุ security constraints ที่สำคัญ]

### Observability

- Standard: OpenTelemetry (OTLP)
- Trace context: W3C TraceContext

**Backend packages:**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| [package] | [version] | [purpose] |

**Frontend packages (ถ้ามี):**

| Package | Version | ใช้สำหรับ |
|---------|---------|----------|
| [package] | [version] | [purpose] |

**Constraints:**
- ห้าม log PII — ใช้ masked หรือ hashed value

### Environment Variables

| Prefix | ใช้สำหรับ |
|--------|---------|
| `APP_` | Application config |
| `DB_` | Database connection |
| [เพิ่มตามที่ project ใช้] | — |

- ทุก secret ต้องใช้ environment variable — ห้าม hardcode

### PDPA (ระบุถ้า project เก็บ personal data)

- Personal data ที่เก็บ: [ระบุ fields]
- Retention period: [ระบุ]
- Consent mechanism: [ระบุ]

---

## 5. SA Decision Fields

SA ต้องระบุค่าต่อไปนี้ให้ชัดเจนใน Solution Doc ของแต่ละ feature:

| Field | ค่าที่เลือก | หมายเหตุ |
|-------|-----------|---------|
| [decision 1] | [value] | [เหตุผลถ้า non-obvious] |
| [decision 2] | [value] | — |

[เพิ่ม rows ตาม technology decisions ที่ SA ต้องระบุต่อ feature]

---

## 6. Stack Deviations

หาก feature ใดจำเป็นต้องเบี่ยงเบนจาก stack นี้ SA ต้อง:
1. เขียน ADR ระบุเหตุผลและ trade-off
2. แจ้ง PO ก่อนส่ง Solution Doc
3. อัปเดต STACK_CONTEXT.md ถ้า deviation นั้นกลายเป็น standard ใหม่

---

## Stack Versions (verified [date])

> SA ต้อง fill section นี้ด้วย version จริงที่ verify ผ่าน WebSearch ก่อนส่ง PO
> ห้าม fill จาก training data — ข้อมูลอาจล้าสมัย

| Package / Runtime | Version | Verified date |
|------------------|---------|--------------|
| [runtime / framework] | [version] | [date] |
