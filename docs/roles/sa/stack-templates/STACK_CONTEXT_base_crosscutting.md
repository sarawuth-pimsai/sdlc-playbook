# STACK_CONTEXT — Base Cross-cutting Concerns

> **This file is a reference for template authors and PE**
> SA using a ready-made template does not need to read this file — cross-cutting concerns are already embedded in each template file.
> Use this file when: creating a new template, creating an org template, or verifying that a template covers all required concerns.

---

## Cross-cutting Concerns Required by Every Stack

These sections are **universal** — applicable to every tech stack.
Stack-specific packages must be added in each individual template.

---

## Authentication & Authorization

### Protocol (Universal)

- **OAuth2 + OIDC** (Authorization Code + PKCE) — standard for web apps and SPAs
- IdP: SA chooses per project (Keycloak / Azure AD / Okta / other) — specify in ADR

### Principles

- **PKCE is mandatory for SPA and mobile** — do not use Implicit flow
- Access tokens must be validated through the JWKS endpoint — do not trust without verifying the signature
- Tokens/sessions must never be logged and must never be returned in error responses
- Refresh token rotation — revoke the old token every time rotation occurs

> **Stack-specific packages:** each template lists the packages used in §4 Cross-cutting

---

## Observability (OpenTelemetry)

### Standard (Universal)

- Signals: **Traces + Metrics + Logs** via OpenTelemetry
- Export protocol: **OTLP** (HTTP or gRPC) — do not tie directly to vendor-specific formats
- Trace context propagation: **W3C TraceContext** standard (`traceparent` header)
- Every service must send the `service.name` resource attribute

### Log Levels

| Level | Environment |
|-------|------------|
| `DEBUG` | Development only |
| `INFO` | Production default |
| `WARN` | Degraded behavior that is still functional |
| `ERROR` | Critical — requires attention |

### PII Masking (mandatory)

- Do not log: email, phone, national ID, address, health data
- Use masked values: `usr_***@***.com` or a hashed identifier instead

> **Stack-specific packages:** each template lists OTel packages in §4 Cross-cutting

---

## Environment Variables

### Prefix Conventions

| Prefix | Used for | Example |
|--------|---------|---------|
| `APP_` | Application config | `APP_PORT`, `APP_ENV`, `APP_LOG_LEVEL` |
| `DB_` | Database connection | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER` |
| `REDIS_` | Cache connection | `REDIS_HOST`, `REDIS_PORT` |
| `AUTH_` | Auth/IdP config | `AUTH_ISSUER`, `AUTH_CLIENT_ID` |

### Rules

- Every secret must use an environment variable — no hardcoding in code or config files
- Do not commit `.env` files with real values to the repository
- `APP_ENV` values: `development` / `staging` / `production`

---

## PDPA (Thailand — Personal Data Protection Act B.E. 2562)

### Personal Data to Handle Carefully

Personal data under PDPA: full name, national ID, phone number, email, address, health data, biometric data.

### Requirements for Every Feature That Stores Personal Data

- Specify the **retention period** in the Solution Doc — must be explicit (e.g. 3 years, 7 years)
- Must have a **deletion mechanism** — delete data when the retention period expires
- Must have a **consent mechanism** — record when and for what the user gave consent
- Do not export personal data out of the system without an audit trail

---

## Stack Deviations Process (Universal)

If any feature requires deviating from the stack specified in `STACK_CONTEXT.md`, SA must:

1. Write an ADR stating the reason and trade-offs
2. Notify PO before sending the Solution Doc
3. Update `STACK_CONTEXT.md` if the deviation becomes the new standard

If the deviation affects an org template (PE-managed) — SA must also notify PE, not just PO.
