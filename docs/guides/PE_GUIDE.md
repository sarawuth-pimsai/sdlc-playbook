# Platform Engineering (PE) Guide

> For workflow overview and setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

PE manages the organisation's standard tech stack and creates and maintains **org templates** that every project uses as a baseline — instead of each SA building from scratch.

---

## PE's role in the SDLC Playbook

PE is not directly in the SDLC workflow — PE works "before" the workflow starts:

```
PE creates Org Template
        ↓
PO attaches Org Template in Stack Setup Request → SA
        ↓
SA customises for the project → STACK_CONTEXT.md
        ↓
Normal SDLC Workflow (SA → PO → Lead → Dev → QA)
```

When an org template exists, SA does not need to make the technology choices that PE has already locked — reducing cognitive load and enforcing consistency.

---

## Creating the Org Template (first time)

### Step 1 — Choose a base template

```
docs/roles/sa/stack-templates/
  STACK_CONTEXT_go_clean_arch.md   ← Go fullstack (recommended if the org uses Go)
  [other templates to be added in future]
```

Copy the template that best matches the org's stack family:

```bash
cp docs/roles/sa/stack-templates/STACK_CONTEXT_go_clean_arch.md \
   STACK_CONTEXT_[OrgName].md
```

### Step 2 — Add an Org Constraints section

Add this section after `## Purpose` in the file:

```markdown
## Org-specific Constraints — [OrgName]

> Maintained by Platform Engineering
> SA must not edit this section without going through PE

### Infrastructure (locked by PE)

| Component | Org Standard | Note |
|-----------|-------------|------|
| Database | [e.g. Aurora PostgreSQL 16] | managed — do not use self-hosted |
| Cache | [e.g. ElastiCache Redis 7] | managed |
| Container | [e.g. Kubernetes on EKS] | production only |
| Secret management | [e.g. AWS Secrets Manager] | no .env in production |
| CI/CD | [e.g. GitHub Actions + ArgoCD] | — |

### Security (locked by PE)

| Requirement | Value |
|-------------|-------|
| IdP | [e.g. Azure AD B2C tenant: xxx] |
| Auth protocol | OAuth2 + OIDC + PKCE (no deviations) |
| Token storage (SPA) | HttpOnly cookie via BFF — no localStorage |
| Vulnerability scanning | [e.g. Snyk on every PR] |

### Observability (locked by PE)

| Item | Value |
|------|-------|
| OTel Collector endpoint | [internal endpoint] |
| Trace sampling | [e.g. 10% production, 100% staging] |
| Log destination | [e.g. CloudWatch / Elastic] |

### Network (locked by PE)

| Item | Value |
|------|-------|
| API Gateway | [e.g. AWS API Gateway / Kong] |
| Internal service discovery | [e.g. AWS Cloud Map] |
```

### Step 3 — Lock SA Decision Fields that PE has already decided

In the `## SA Decision Fields` section — change fields that PE has locked from "SA chooses" to a fixed value:

**Example:**

```markdown
## SA Decision Fields

> Fields marked [PE LOCKED] — SA does not need to decide, PE has already set these

| Field | Value | Status |
|-------|-------|--------|
| Auth Method | OAuth2 + OIDC + PKCE via Azure AD B2C | [PE LOCKED] |
| PostgreSQL Driver | pgx/v5 | [PE LOCKED] |
| Migration Tool | golang-migrate | [PE LOCKED] |
| HTTP Router | SA chooses — Fiber v3 (default) / Chi / Echo | SA decides |
| SPA State Management | SA chooses — Zustand (default) / TanStack Query | SA decides |
```

### Step 4 — Verify versions

**Never fill versions from memory** — verify through the package registry before finalising the org template each time

Include the verified date in the Stack Versions table at the bottom of the template

### Step 5 — Distribute

Send `STACK_CONTEXT_[OrgName].md` to PO for every project using this org stack.  
PO attaches it with the Stack Setup Request when sending to SA.

---

## Maintaining the Org Template

### When to update

| Event | Action |
|-------|--------|
| Infrastructure upgrade (DB version, K8s version) | Update version in template + bump version header |
| Security policy changes (IdP, token policy) | Update Security section + notify SA for every active project |
| New approved package | Add to template + specify use case |
| Package deprecated or critical CVE | Remove / replace + notify urgently |
| SA requests a deviation (affects org standard) | Review ADR → if approved, update template |

### Version bump convention

Template version uses an integer incrementing by 1, in the header:

```
# Last updated: YYYY-MM-DD | Version: N | Status: [OrgName] Org Standard
```

### Notify SA

When the org template is updated → notify SA for every currently active project, specifying:
- Old version → New version
- What changed
- Whether the project's own `STACK_CONTEXT.md` needs to be updated

---

## Adding a new Stack Template to the Playbook

When the org starts using a new stack family that will apply to multiple projects (e.g. Python FastAPI):

1. Copy the closest existing template from `docs/roles/sa/stack-templates/`
2. Change §1–4 to match the new stack
3. Keep the §4 cross-cutting structure — change only the stack-specific packages
4. Add a row to the table in [stack-templates/README.md](../roles/sa/stack-templates/README.md)
5. Test with 1 real project before promoting to an official template

**Rule:** only create templates for stacks that real projects in the organisation actually use — hypothetical templates with no users become a maintenance burden.

---

## Pre-Distribution Checklist

- [ ] Version header is complete (`Last updated`, `Version`, `Status`)
- [ ] Org Constraints section is complete (Infrastructure / Security / Observability)
- [ ] `[PE LOCKED]` fields are clearly marked — SA knows what it can decide on its own
- [ ] All versions verified through the package registry
- [ ] Stack Versions table filled in with verified dates
- [ ] Tested with 1 real project before broad distribution

---

## FAQ

**Q: Does PE need to use claude.ai or Claude Code?**  
A: Not necessarily — PE works directly with Markdown files. There is no AI workflow specific to PE in this playbook.

**Q: What if a project needs an exception from the org standard?**  
A: SA writes an ADR → PE reviews → if approved, SA customises that project's `STACK_CONTEXT.md` with the deviation clearly stated — the org template is not affected.

**Q: Where should the org template be stored?**  
A: In the organisation's private repository — not committed to the playbook repo, as it contains internal infrastructure details (endpoints, tenant IDs, internal service names).

**Q: Does PE need to create a separate Claude Project?**  
A: Not for this version of the playbook — PE works directly with Markdown files.
