# QA Engineer Guide

> For workflow overview and setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

QA works in 4 phases. Phase 0 can start in parallel with SA — no need to wait for Lead or STACK_CONTEXT.md.

---

## Setup

### Option A — claude.ai Projects

Via the claude.ai web interface

#### Step 1 — Create a Project

1. Open [claude.ai](https://claude.ai) → **Projects** → **New project**
2. Name it, e.g. `[Project Name] — QA`
3. Open **Project Instructions** → Copy content from `ai/QA_PROJECT_INSTRUCTIONS.md` → Paste → Save

#### Step 2 — Upload Project Knowledge by Phase

**Phase 0** (received from PO after STEP 2):
| File | Required |
|------|----------|
| `PRD_[feature].md` | ✅ |
| `DECISION_LOG_[feature]_TODO.md` | ✅ |
| `DECISION_LOG_[feature]_RESOLVED.md` | If exists |

**Phase A** (received from Lead before SIT testing):
| File | Required |
|------|----------|
| `STACK_CONTEXT.md` | ✅ |
| `Task_[ID]_[title].md` | ✅ all tasks |
| PRD + DECISION_LOG files | ✅ (updated from Phase 0 if changes were made) |

---

### Option B — Claude Code

Via Claude Code CLI, Desktop App, or IDE Extension instead of claude.ai Projects

#### Step 1 — Setup workspace (one-time)

See setup details at [templates/option-b/README.md](../../templates/option-b/README.md) — use the `/setup` command to automatically create the directory structure and copy all required files

#### Step 2 — Start a QA session

```
/qa
```

Type `/qa` in Claude Code — Claude will read `ai/QA_PROJECT_INSTRUCTIONS.md` and focus on `docs/roles/qa/` and `docs/shared/`

#### Step 3 — Place files in the QA directory

Place knowledge files in `docs/roles/qa/` instead of uploading to Project Knowledge  
Add new files when moving to a new phase (Phase 0 → Phase A)

---

## Phase 0 — Early Preparation (parallel with SA design)

**Trigger:** PO sends PRD + DECISION_LOG to QA after STEP 2 is complete

**Can proceed without waiting for:** Lead, STACK_CONTEXT.md, task prompts

### P-STEP 1 — Draft Test Cases from PRD

- User stories → happy path test cases
- Business rules → edge case scenarios
- Error handling section → error path test cases
- NFR section → note performance/load scenarios for Phase B

Mark every test case as `[DRAFT — pending task breakdown]`

### P-STEP 2 — Test Data Requirements

Claude generates a test data request to send to Lead before Phase A:

```markdown
# Test Data Requirements — [Feature name]
## User accounts needed
## Sample data records
## External system mocks (if integrations exist)
## Environment access
- SIT URL, Staging URL
- Credentials (request from Lead)
```

Lead provisions test data before Phase A begins

---

## Phase A — SIT Testing (per task)

**Trigger:** Dev sends a Deploy Notification after deploying a task to SIT

> Do not start testing before receiving a Deploy Notification from Dev

### A-STEP 1 — Read Task Prompt + PRD

Extract:
- Done criteria from the task prompt (minimum pass bar)
- Relevant user stories from the PRD
- Error codes and edge cases from the PRD

### A-STEP 2 — Generate Test Cases

Claude creates a test case table:
- Happy path
- Edge cases
- Error cases
- Security cases (if applicable)

Preview → QA reviews → confirms → exports `TestCases_[TaskID].md`

### A-STEP 3 — Generate Claude Code Test Prompt

Claude creates a prompt for QA to paste in Claude Code to write + run automated tests in the SIT environment

Export `Prompt_SIT_[TaskID].md`

### A-STEP 4 — Bug Report (if test fails)

Claude drafts each bug entry:

```markdown
## BUG-[NNN] — [Short description]
Task: [Task ID] | Severity: Critical/High/Medium/Low | Status: Open

### Steps to reproduce
### Expected result
### Actual result
### Evidence (log/screenshot)
### Possible cause
```

Export `BugReport_[TaskID].md`

### A-STEP 5 — Task Test Summary

Claude summarises metrics + verdict → React Artifact with Download → send to Lead + Dev

**Verdict routing:**
| Verdict | Send to | For |
|---------|---------|-----|
| PASS | Lead | Unblock next task |
| FAIL | Lead + Dev | Dev fixes bug before re-test |
| BLOCKED | Lead only | Lead decides whether to escalate |

Dev fixes bug → QA re-tests only the failed cases

---

## Phase B — Staging Full Test Suite

**Trigger:** Dev has deployed all tasks to Staging

### B-STEP 0 — Regression Scope Check

Verify what existing functionality this feature may affect

**Default regression scope (if no Critical paths in STACK_CONTEXT):**
- Authentication / login flow (if auth middleware was changed)
- Every endpoint that the new feature calls or depends on
- Every DB table that was modified or extended

> If a regression case fails in Phase B → **STOP immediately** and notify Lead before proceeding — do not continue Phase B

### B-STEP 1 — Compile Full Test Suite

Merge all `TestCases_[TaskID].md` files into a master test suite

Always include **Security test cases** (Option B — if no SEC Engineer):
- Access endpoint without auth token → HTTP 401
- Access endpoint with expired token → HTTP 401
- SQL injection payload in text fields → HTTP 400 or sanitised response
- Response body does not include sensitive fields (password, token, PII)
- + feature-specific security cases from the PRD NFR

Export `TestSuite_[Feature].md`

### B-STEP 2 — Generate Claude Code Automated Test Prompt

Claude creates a prompt to run the full automated test suite on Staging

Export `Prompt_Staging_[Feature].md`

### B-STEP 3 — Test Report

Claude drafts a report with a verdict:
- **APPROVED FOR PRODUCTION**
- **FAILED — DO NOT DEPLOY**
- **CONDITIONAL** (list conditions)

React Artifact with Download → QA + Lead sign-off → **Lead notifies PO** (not QA directly)

---

## Phase C — Production Deployment

**Trigger:** Lead + QA sign-off on a TestReport with verdict APPROVED

### C-STEP 2 — Post-deployment Smoke Test

Run only P1 test cases in the production environment

- All P1 cases must pass within the configured timeout (default: 15 minutes)
- If P1 fails → **trigger rollback immediately** — diagnose after rollback, not before

### Rollback Triggers

| Condition | Action |
|-----------|--------|
| P1 smoke test fails | Lead rolls back immediately |
| Deployment hangs > timeout | Lead rolls back |
| Critical error in production logs within 15 minutes | Lead + QA assess → rollback if user-facing |

---

## Phase HF — Hotfix Smoke Test (Staging → Production)

Trigger: Lead notifies QA that a hotfix has been deployed to Staging, specifying the HF-ID and the scope to test

**Time allowed:** P1 = 30 minutes / P2 = 2 hours — QA must prioritise immediately

### HF-STEP 1 — Receive scope from Lead

Lead provides with the HotfixTask:
- What the bug fix was (reference BugIntake BR-[NNN])
- Critical paths to check

If Lead does not specify critical paths → use the Default regression scope from STACK_CONTEXT or the existing TestSuite

### HF-STEP 2 — Run smoke test on Staging

Run critical paths only — **do not run the full test suite**:
- The flow that was directly fixed (from Bug Intake)
- Critical paths that the fix may have affected

Record results as `HotfixSmokeTest_HF-[NNN].md`

### HF-STEP 3 — Report results to Lead (Staging)

| Result | Action |
|--------|--------|
| All TCs PASS | Notify Lead: "Hotfix HF-[NNN] smoke test passed on Staging" and attach the file — Lead proceeds to deploy to Production |
| Any TC FAIL | Notify Lead immediately — Lead fixes and re-deploys Staging first — **do not deploy to Production until it passes** |

### HF-STEP 4 — Production smoke check (after Lead deploys to Production)

Trigger: Lead notifies QA that Production deployment is done

Run **P1 test cases only** — use the same set from HF-STEP 2

**Time allowed:** P1 = 15 minutes / P2 = 1 hour

| Result | Action |
|--------|--------|
| All P1 PASS | Notify Lead: "Production smoke check passed — HF-[NNN]" |
| Any P1 FAIL | Notify Lead immediately — Lead decides to rollback immediately |

**Rule:** QA always reports to Lead — never directly to PO (Lead notifies PO via HotfixNotification)

---

## Files QA maintains

| File | When | Stored at |
|------|------|-----------|
| `TestCases_[TaskID].md` | After A-STEP 2 per task | `docs/qa/[feature]/` |
| `BugReport_[TaskID].md` | After A-STEP 4 per task | `docs/qa/[feature]/` |
| `TestSuite_[Feature].md` | After B-STEP 1 | `docs/qa/[feature]/` |
| `TestReport_[Feature].md` | After B-STEP 3 | `docs/qa/[feature]/` |
| `HotfixSmokeTest_HF-[NNN].md` | After HF-STEP 2 per hotfix | `docs/qa/hotfix/` |

Lead commits these files to the repo during PR merge

---

## QA Rules

- **QA decides the verdict** — Claude does not mark pass/fail or approve a feature without QA confirmation
- **Phase B Test Report → Lead always notifies PO** — QA does not notify PO directly
- If the environment is unreachable → **STOP** notify Lead/Dev immediately, do not assume
- If test results are ambiguous → **STOP** show the raw response for QA to decide
