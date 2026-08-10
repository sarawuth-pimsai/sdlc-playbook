# Security Engineer (SEC) Guide

**Option A only** — The SEC Project is used only when `Security role: yes` is set in `PROJECT_CONTEXT.md`.

If the team has no Security Engineer → use **Option B** where security checkpoints are already embedded in the SA / Lead / QA instructions — no need to set up a SEC Project.

---

## Setup

### Option A — claude.ai Projects

Via the claude.ai web interface

#### Step 1 — Create a Project

1. Open [claude.ai](https://claude.ai) → **Projects** → **New project**
2. Name it, e.g. `[Project Name] — Security`
3. Open **Project Instructions** → Copy content from `ai/SEC_PROJECT_INSTRUCTIONS.md` → Paste → Save

#### Step 2 — Upload Project Knowledge

| File | When |
|------|------|
| `STACK_CONTEXT.md` | Received from PO with Phase A package |
| `PRD_[feature].md` | Received from PO |
| `Solution_Doc_[feature].md` | Received from PO (from SA) — must be present before starting Phase A |
| `PROJECT_CONTEXT.md` | Received from PO with Solution Doc — read Environment default + override (see [CORE_POLICY.md](../CORE_POLICY.md) §5) |

**SEC asks for its own Environment override only once when first starting work on this project** (same as SA/Lead/QA), then performs a pairwise check before sending files back each time — Phase A compared against PO, Phase B compared against Lead (see `ai/SEC_PROJECT_INSTRUCTIONS.md` §Environment override + §Handoff Environment Check)

SEC has no Option B setup of its own — for teams using Option B (Claude Code), security checkpoints are embedded directly in SA/Lead/QA instructions instead (no `/sec` slash command or separate `docs/roles/sec/`). See `templates/option-b/README.md`

---

## Phase A — Solution Doc Security Review

**Trigger:** PO sends `Solution_Doc_[feature].md` to SEC after receiving it from SA

### A-STEP 1 — Analyse Architecture for Security

Review the Solution Doc in 7 categories:

| Category | What to check |
|----------|---------------|
| Authentication | Is auth defined for every endpoint? Who can call what? |
| Authorization | Are role/permission checks specified per operation? |
| Data exposure | Does the API response include fields that should not be public? |
| Input validation | Are all user input fields validated/sanitised before use? |
| PDPA / Data sensitivity | Does the feature collect or process personal data? Is a retention period defined? |
| Secrets management | Are all credentials using environment variables? No hardcoded values? |
| New dependencies | Are there new libraries? If so — any known CVEs? |

### A-STEP 2 — Output Security Requirements

Claude produces `Security_Requirements_[feature].md`:

```markdown
# Security Requirements — [Feature name]
Version: 1.0 | Date: [date] | Author: SEC

## Risk summary
| Risk | Severity | Mitigation required |

## Implementation requirements
[List of requirements that are verifiable in Phase B code review]
1. [e.g. Every /admin endpoint must validate JWT and assert role = "admin"]
2. [e.g. User email must not appear in the response body or application logs]

## PDPA considerations
[Data collected, retention policy, consent — or "none identified"]

## Open questions for SA/PO
[Items that need clarification before requirements can be finalised]
```

Preview → SEC reviews → confirms → exports → **sends to PO**

PO includes this file in the Lead Handoff → Lead embeds it in task prompts

### A-STEP 3 — Blocking Risk

If a risk is found that makes the implementation unsafe without an architecture redesign:
- **STOP** → explain the risk and the relevant Solution Doc section
- SEC notifies PO + SA before implementation begins
- **Never output partial requirements** — complete the full review first, then flag all issues together

---

## Phase B — Code Review (per PR)

**Trigger:** Dev raises a PR → Lead sends changed files + task prompt to SEC

### B-STEP 1 — Review Changed Files

Check 7 dimensions:

1. **Auth/authorization** — Do the Security Requirements from Phase A pass in the code?
2. **Input validation** — Are inputs sanitised before use in DB queries, shell commands, responses?
3. **Data exposure** — Does the response body include sensitive fields (passwords, tokens, PII)?
4. **Injection risks** — SQL injection, command injection, path traversal
5. **Secrets** — No hardcoded credentials, API keys, or env values in code or config files
6. **Error messages** — Do error responses leak internal details (stack traces, DB schema, internal paths)?
7. **New packages** — Do any new dependencies have known CVEs?

### B-STEP 2 — Output Security Review

```markdown
# Security Review — [Task ID] [Task title]
Date: [date] | Reviewer: SEC

## Must fix (blocks merge)
- [finding — file path, line number, description]

## Should fix (request changes)
- [finding — recommendation]

## Consider (non-blocking)
- [suggestion]

## Verdict: APPROVED / REQUEST CHANGES / ESCALATE TO SA
```

Preview → SEC confirms → exports `Security_Review_[TaskID].md` → **sends to Lead**

Lead reads and decides: merge or ask Dev to fix before re-review

If verdict is **ESCALATE TO SA** → Lead notifies SA + PO before any merge

---

## File Flow

```
PO ──► SEC    Solution_Doc_[feature].md
SEC ──► PO    Security_Requirements_[feature].md  (Phase A output)
              PO includes in Lead Handoff

Lead ──► SEC  changed files + task prompt (per PR)
SEC ──► Lead  Security_Review_[TaskID].md  (Phase B output)
```

---

## SEC Rules

- **SEC decides the security verdict** — Claude does not approve a PR or clear a security risk without SEC confirmation
- Phase A → always send Security Requirements to PO (not directly to Lead)
- Phase B → always send Security Review to Lead
- If a critical vulnerability is found in Phase B (auth bypass, data leak, injection) → **STOP** notify Lead immediately, do not approve

---

## Option B — Security Checkpoints (when no SEC Engineer)

When `Security role: no`, security is handled at 3 points:

| Point | Who | What |
|-------|-----|------|
| SA Solution Doc Section 6 | SA | Security checklist: auth, authorization, input validation, data exposure, secrets, PDPA |
| Lead PR Review (R-STEP 2) | Lead | Hardcoded secrets, missing input validation, sensitive data in response, auth checks, error message leaks |
| QA Phase B TestSuite (B-STEP 1) | QA | Security test cases: unauth access, expired token, SQL injection, sensitive fields in response |
