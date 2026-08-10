# Last updated: YYYY-MM-DD | Version: 1 | Status: [Project Name]

# STACK_CONTEXT.md — [Project Name]

> SA owns this file. Update every time the stack changes, and copy to `docs/roles/po/STACK_CONTEXT.md` after finalizing.

---

> **This file is a blank schema** — use it when no stack template matches your project.
>
> If a matching template exists — use the template from `docs/roles/sa/stack-templates/` instead (faster to start).
> See the list of templates at [stack-templates/README.md](stack-templates/README.md)

---

## Purpose

[State the project name and summarize the chosen tech stack family — 1-2 sentences]

---

## Project Settings

| Field | Value | Note |
|-------|-------|------|
| Output language | `en` | `en` = English (default) \| `th` = Thai — change to `th` for Thai-language teams |

---

## Versioning Principle

[State the versioning policy — e.g. "always use latest stable" or "pin every version due to production constraints"]

---

## 1. Backend

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Language | [Go / Python / Node.js / Java / other] | [version] |
| Web Framework | [framework] | [version] |
| Validation | [library] | [version] |
| Config | [library] | [version] |

### Architecture Pattern

[State the pattern: Clean Architecture / Hexagonal / Layered / MVC / other]

**Directory structure:**

```
[draw the actual directory structure used]
```

**Layer responsibilities:**

[Describe the responsibility of each layer and the dependency direction]

### API Conventions

[URL format, versioning strategy, response envelope format, HTTP status codes, error code format]

### Constraints

[Rules that Dev and SA must always follow — write only what is non-obvious]

---

## 2. Frontend (remove this section if there is no frontend)

### Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | [Next.js / Vite+React / Vue / Angular / other] | [version] |
| Language | [TypeScript / JavaScript] | [version] |
| Styling | [Tailwind / CSS Modules / Styled Components / other] | [version] |
| Package Manager | [pnpm / npm / yarn] | [version] |

### Conventions

[routing pattern, state management, data fetching pattern, component organization]

### Constraints

[Key rules Dev must know]

---

## 3. Persistence

| Level | Technology | Use case | Location |
|-------|-----------|---------|---------|
| L1 Local | [library / plain map] | per-instance state that is not shared | `internal/local/` |
| L2 Cache | [Redis / Memcached / other] | shared state across instances | `internal/cache/` |
| L3 Persistence | [PostgreSQL / MySQL / MongoDB / other] | source of truth | `internal/[driver]/` |

[Remove rows the project does not use]

### Primary Database Conventions

**[database engine name]:**

| Item | Value |
|------|-------|
| Version | [version] |
| Driver | [package + version] |
| Migration tool | [tool] |

[naming conventions — table, column, index, FK]

**Constraints:**

[Key rules about persistence]

---

## 4. Cross-cutting Concerns

### Authentication & Authorization

- Protocol: [OAuth2 + OIDC + PKCE (recommended) / other + reason]
- IdP: [Keycloak / Azure AD / Okta / other] — specify in ADR

**Backend packages:**

| Package | Version | Used for |
|---------|---------|---------|
| [package] | [version] | [purpose] |

**Frontend packages (if applicable):**

| Package | Version | Used for |
|---------|---------|---------|
| [package] | [version] | [purpose] |

**Constraints:**
- [List important security constraints]

### Observability

- Standard: OpenTelemetry (OTLP)
- Trace context: W3C TraceContext

**Backend packages:**

| Package | Version | Used for |
|---------|---------|---------|
| [package] | [version] | [purpose] |

**Frontend packages (if applicable):**

| Package | Version | Used for |
|---------|---------|---------|
| [package] | [version] | [purpose] |

**Constraints:**
- Do not log PII — use masked or hashed values

### Environment Variables

| Prefix | Used for |
|--------|---------|
| `APP_` | Application config |
| `DB_` | Database connection |
| [add as the project uses] | — |

- Every secret must use an environment variable — no hardcoding

### PDPA (fill in if the project stores personal data)

- Personal data stored: [list fields]
- Retention period: [specify]
- Consent mechanism: [specify]

---

## 5. SA Decision Fields

SA must specify the following values clearly in the Solution Doc for each feature:

| Field | Chosen value | Note |
|-------|-----------|---------|
| [decision 1] | [value] | [reason if non-obvious] |
| [decision 2] | [value] | — |

[Add rows for technology decisions SA must specify per feature]

---

## 6. Stack Deviations

If any feature requires deviating from this stack, SA must:
1. Write an ADR stating the reason and trade-offs
2. Notify PO before sending the Solution Doc
3. Update STACK_CONTEXT.md if the deviation becomes the new standard

---

## Stack Versions (verified [date])

> SA must fill this section with real versions verified via WebSearch before sending to PO.
> Do not fill from training data — information may be outdated.

| Package / Runtime | Version | Verified date |
|------------------|---------|--------------|
| [runtime / framework] | [version] | [date] |
