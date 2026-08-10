# Stack Template Library

Stack templates are pre-filled `STACK_CONTEXT.md` files for commonly used stack families.
SA uses them as a baseline and customizes for the project — no need to build from scratch.

---

## How to use (SA)

### 1. Check whether PO sent a PE org template

If PO attached a `STACK_CONTEXT_[OrgName].md` with the Stack Setup Request — use that file as the baseline directly (it has already been validated by that org's PE). Skip steps 2–3.

### 2. Choose the template that matches the stack family

| Template | When to choose |
|----------|-----------------|
| [STACK_CONTEXT_go_clean_arch.md](STACK_CONTEXT_go_clean_arch.md) | Backend Go + Clean Architecture + PostgreSQL/Redis + Next.js or Vite frontend |
| _(add more as the org adopts new stacks)_ | — |

If no template matches — use the blank schema at `docs/roles/sa/STACK_CONTEXT.md` instead.

### 3. Copy template → rename to `STACK_CONTEXT.md`

```
cp docs/roles/sa/stack-templates/STACK_CONTEXT_go_clean_arch.md  docs/roles/sa/STACK_CONTEXT.md
```

### 4. Customize for the project

Things to check and fill in every time:

- [ ] **Header** — update `[Project Name]` and date
- [ ] **IdP** — specify the actual Identity Provider for this project (Keycloak / Azure AD / other)
- [ ] **Version verification** — WebSearch every package before finalizing (see Version Verification rule in `ai/SA_PROJECT_INSTRUCTIONS.md`)
- [ ] **SA Decision Fields** — fill in the chosen values for this project
- [ ] **Remove unused sections** — e.g. if there is no frontend, remove §2-3
- [ ] **Add org-specific constraints** (if PE has additional requirements)

### 5. Send back to PO

SA uploads the filled-in `STACK_CONTEXT.md` to SA Project Knowledge → exports → sends to PO → PO uploads to PO Project.

---

## How to use (Platform Engineering — creating an org template)

PE manages the organisation's standard tech stack and can fork a template to create an org-specific baseline for every project in the organisation to use.

### Steps to create an Org Template

1. **Copy the template that matches the org stack**

   ```
   cp docs/roles/sa/stack-templates/STACK_CONTEXT_go_clean_arch.md  \
      STACK_CONTEXT_[OrgName].md
   ```

2. **Specify org constraints** that every project must follow. Example:

   ```markdown
   ## Org-specific Constraints ([OrgName])
   > Maintained by Platform Engineering — do not modify without going through PE
   
   - DB: Aurora PostgreSQL 16 only (managed service — do not use self-hosted)
   - IdP: Azure AD B2C (mandated by IT Security)
   - Container runtime: Kubernetes on EKS (no docker-compose in production)
   - Secret management: AWS Secrets Manager — no hardcoding in env files
   ```

3. **Lock versions that the org has approved** — specify real versions instead of `latest` or ranges

4. **Remove SA Decision Fields that PE has already decided** — e.g. if the org always uses Aurora, SA doesn't need to choose the DB

5. **Distribute to SA** — send the file to the PO of each project to attach with the Stack Setup Request

### What PE should specify clearly in the Org Template

| Category | Examples PE can lock | Examples SA can still decide |
|------|----------------------|---------------------------|
| Infrastructure | DB engine, IdP, secret manager, container runtime | DB schema design, cache key pattern |
| Security | Auth protocol, token storage policy | Role/permission structure per feature |
| Observability | OTel collector endpoint, trace sampling rate | Signals to instrument per service |
| Networking | API gateway, load balancer | Internal service communication pattern |

---

## Adding a New Template

When the org starts using a new stack family (e.g. Python FastAPI or Node.js NestJS) and it will be used across multiple projects:

1. Copy `STACK_CONTEXT_go_clean_arch.md` as a starting point for the structure
2. Change §1-4 to match the new stack
3. Keep §4 cross-cutting concerns — most parts are reusable (Auth protocol, PDPA)
4. Update the table in this file (README.md)
5. Test with 1 real project before promoting to an official template

**Rule:** do not create templates for hypothetical stacks — only create when a real project in the organisation actually needs it.

---

## Files in This Directory

| File | Type | Covers |
|------|--------|---------|
| `README.md` | Guide | All usage instructions |
| `STACK_CONTEXT_base_crosscutting.md` | Reference | Cross-cutting concerns (Auth protocol, OTel standard, PDPA) — no stack-specific packages |
| `STACK_CONTEXT_go_clean_arch.md` | Template | Go + Clean Architecture + PostgreSQL + Redis + Next.js / Vite |
