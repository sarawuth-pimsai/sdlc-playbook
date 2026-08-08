# STACK_CONTEXT — Base Cross-cutting Concerns

> **ไฟล์นี้เป็น reference สำหรับ template authors และ PE**
> SA ที่ใช้ template สำเร็จรูปไม่ต้องอ่านไฟล์นี้ — cross-cutting ถูก embed ใน template แต่ละไฟล์แล้ว
> ใช้ไฟล์นี้เมื่อ: สร้าง template ใหม่, สร้าง org template, หรือต้องการ verify ว่า template ครอบคลุม concerns ครบ

---

## Cross-cutting Concerns ที่ทุก stack ต้องมี

Section เหล่านี้เป็น **universal** — ใช้ได้กับทุก tech stack
Stack-specific packages ต้องเพิ่มเติมใน template แต่ละตัว

---

## Authentication & Authorization

### Protocol (Universal)

- **OAuth2 + OIDC** (Authorization Code + PKCE) — standard สำหรับ web app และ SPA
- IdP: SA เลือกต่อ project (Keycloak / Azure AD / Okta / อื่น) — ระบุใน ADR

### Principles

- **PKCE บังคับสำหรับ SPA และ mobile** — ห้ามใช้ Implicit flow
- Access token ต้อง validate ผ่าน JWKS endpoint — ห้าม trust โดยไม่ verify signature
- Token/session ห้าม log และห้าม return ใน error response
- Refresh token rotation — revoke old token ทุกครั้งที่ rotate

> **Stack-specific packages:** template แต่ละตัวระบุ packages ที่ใช้ใน §4 Cross-cutting

---

## Observability (OpenTelemetry)

### Standard (Universal)

- Signal: **Traces + Metrics + Logs** ผ่าน OpenTelemetry
- Export protocol: **OTLP** (HTTP หรือ gRPC) — ห้าม tie ถึง vendor-specific format โดยตรง
- Trace context propagation: **W3C TraceContext** standard (`traceparent` header)
- ทุก service ต้องส่ง `service.name` resource attribute

### Log Levels

| Level | Environment |
|-------|------------|
| `DEBUG` | Development เท่านั้น |
| `INFO` | Production default |
| `WARN` | Degraded behavior ที่ยัง functional |
| `ERROR` | Critical — ต้องการ attention |

### PII Masking (บังคับ)

- ห้าม log: email, phone, national ID, address, ข้อมูลสุขภาพ
- ใช้ masked value: `usr_***@***.com` หรือ hashed identifier แทน

> **Stack-specific packages:** template แต่ละตัวระบุ OTel packages ใน §4 Cross-cutting

---

## Environment Variables

### Prefix Conventions

| Prefix | ใช้สำหรับ | ตัวอย่าง |
|--------|---------|---------|
| `APP_` | Application config | `APP_PORT`, `APP_ENV`, `APP_LOG_LEVEL` |
| `DB_` | Database connection | `DB_HOST`, `DB_PORT`, `DB_NAME`, `DB_USER` |
| `REDIS_` | Cache connection | `REDIS_HOST`, `REDIS_PORT` |
| `AUTH_` | Auth/IdP config | `AUTH_ISSUER`, `AUTH_CLIENT_ID` |

### Rules

- ทุก secret ต้องใช้ environment variable — ห้าม hardcode ใน code หรือ config file
- ห้าม commit `.env` ที่มีค่า real เข้า repository
- `APP_ENV` values: `development` / `staging` / `production`

---

## PDPA (Thailand — พระราชบัญญัติคุ้มครองข้อมูลส่วนบุคคล พ.ศ. 2562)

### Personal Data ที่ต้องระวัง

ข้อมูลส่วนบุคคลตาม PDPA: ชื่อ-นามสกุล, เลขบัตรประชาชน, เบอร์โทรศัพท์, email, ที่อยู่, ข้อมูลสุขภาพ, ข้อมูลชีวมาตร

### Requirements ต่อทุก feature ที่เก็บ personal data

- ระบุ **retention period** ใน Solution Doc — ต้องชัดเจน (เช่น 3 ปี, 7 ปี)
- ต้องมี **deletion mechanism** — ลบข้อมูลเมื่อครบ retention period
- ต้องมี **consent mechanism** — บันทึกว่า user ให้ consent เมื่อไหร่ และ consent อะไร
- ห้าม export personal data ออกนอกระบบโดยไม่มี audit trail

---

## Stack Deviations Process (Universal)

หาก feature ใดจำเป็นต้องเบี่ยงเบนจาก stack ที่ระบุใน `STACK_CONTEXT.md` SA ต้อง:

1. เขียน ADR ระบุเหตุผลและ trade-off
2. แจ้ง PO ก่อนส่ง Solution Doc
3. อัปเดต `STACK_CONTEXT.md` ถ้า deviation นั้นกลายเป็น standard ใหม่

หาก deviation กระทบ org template (PE-managed) — SA ต้องแจ้ง PE ด้วย ไม่ใช่แค่ PO
