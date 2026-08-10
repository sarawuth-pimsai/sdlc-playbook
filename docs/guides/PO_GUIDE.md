# Product Owner (PO) Guide

> For workflow overview and setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

The PO is the **central hub** of the SDLC Playbook — coordinating all roles and owning the decision history for every feature.

**PO does not decide tiers** — the PO only collects feature details and flags business-risk keywords as context for SA to decide. See [CORE_POLICY.md](../CORE_POLICY.md) §1

---

## Setup

### Option A — claude.ai Projects

Via the claude.ai web interface

#### Step 1 — Create a Project

1. Open [claude.ai](https://claude.ai) → **Projects** → **New project**
2. Name it, e.g. `[Project Name] — PO`
3. Open **Project Instructions** → Copy content from `ai/PROJECT_INSTRUCTIONS.md` → Paste → Save

#### Step 2 — Upload Project Knowledge

| File | When | Note |
|------|------|------|
| `PROJECT_CONTEXT.md` | ✅ Before first session | Sets `Security role:`, `UX/UI required:` and `Environment:` — Claude creates it in the first session → re-upload |
| `STACK_CONTEXT.md` | After receiving from SA | Required before STEP 3 |
| `DECISION_LOG_[feature]_TODO.md` | Every session with remaining TODOs | Re-upload each new session |
| `DECISION_LOG_[feature]_RESOLVED.md` | When resolved items exist | Upload once, re-upload only when new items are added |
| `PATTERN_LIBRARY.md` | If exists | Created by SA or PO |

> **First session:** Claude will ask for `Security role`, `UX/UI required` and `Environment`, then automatically create `PROJECT_CONTEXT.md` → download and re-upload to Project Knowledge

---

### Option B — Claude Code

Via Claude Code CLI, Desktop App, or IDE Extension instead of claude.ai Projects

#### Step 1 — Setup workspace (one-time)

See setup details at [templates/option-b/README.md](../../templates/option-b/README.md) — use the `/setup` command to automatically create the directory structure and copy all required files

#### Step 2 — Start a PO session

```
/po
```

Type `/po` in Claude Code — Claude will read `ai/PROJECT_INSTRUCTIONS.md` and focus on `docs/roles/po/`

#### Step 3 — Place files in the PO directory

Place knowledge files in `docs/roles/po/` instead of uploading to Project Knowledge

---

## SESSION START — Every new session

1. Claude reads all files in Project Knowledge automatically (silently)
2. Claude shows a **Session Welcome** in chat — summarises found files + asks for project status or the work you want to do
3. Answer questions one at a time — Claude routes based on your input immediately

> If you return in the same conversation thread → Claude skips the Welcome and shows a feature status summary instead

When STACK_CONTEXT has `Status: Confirmed`, Claude will ask:

```
1. Add a new feature
2. Continue a pending feature
3. View Decision Log
4. Report a bug / production issue — capture report and forward to Lead
```

Select **4** → Claude asks further: new bug report, or follow up on an already-submitted BugIntake

---

## PO SDLC Flow

### STEP 1 — Analyse PRD

**Trigger:** Upload PRD to the session or confirm PRD from interview

Claude will:
- Summarise the feature (2–3 sentences)
- Score each section (1–10)
- List gaps by severity: HIGH / MED / LOW
- Separate tasks: UNBLOCKED vs BLOCKED (waiting for PO decision)

### STEP 1.5 — Clarify PRD

Claude asks one question at a time for 3 types of ambiguity:
- **Type A** — Ambiguous requirement (multiple interpretations)
- **Type B** — Contradictory requirement (conflicting)
- **Type C** — Missing critical detail

Answer questions one at a time, or press **Skip** to record as a TODO

### STEP 1.6 — Scan for Business-risk keywords

Before scanning, if `PATTERN_LIBRARY.md` exists Claude reads the `## Escalated Keywords` section first (previously escalated keywords) and includes them in this scan — see `ai/PROJECT_INSTRUCTIONS.md` §STEP 1.6

After STEP 1.5, Claude scans the PRD for business-risk keywords (payment, auth, PII, external API, encryption/compliance, etc. — see full list in [CORE_POLICY.md](../CORE_POLICY.md) §1) and attaches the scan results as context in the SA Handoff — **this is not a tier decision** — it is raw flags for SA to use when making the Tier decision at SA's STEP 1.5 Tier Triage

### STEP 2 — Define Epics

Claude groups work into Epics (high-level only — Lead does the task breakdown)

**After confirming STEP 2:** Claude automatically creates the SA Handoff (including business-risk flags from STEP 1.6)
- If `Security role` or `UX/UI required` are not yet in PROJECT_CONTEXT.md → Claude asks for them first (see `ai/PROJECT_INSTRUCTIONS.md` §Security role & UX/UI check)

**After sending SA Handoff:** notify QA to start Phase 0 without waiting for SA

### STEP 3 — Check Stack + Triage Summary/Solution Doc

**Trigger:** SA sends back Triage Summary (Tier 1) or Solution Doc + ADRs (Tier 2/3) + STACK_CONTEXT

**Always mandatory — do not proceed to STEP 4 until SA's output is received** regardless of tier (see [CORE_POLICY.md](../CORE_POLICY.md) §1) — no soft-warning-then-proceed path

Claude checks:
- Is STACK_CONTEXT.md complete?
- **STACK_CONTEXT.md version sync (hard gate):** compare the version header here against the latest version in SA Project Knowledge — if they don't match, **block** Lead Handoff generation until PO re-syncs the file (see `ai/PROJECT_INSTRUCTIONS.md` §STEP 3)
- Tier 1: Is Triage Summary present? Tier 2/3: Is Solution Doc complete (all 9 required sections + section 10 if UX/UI present)?
- Do ADR files match those referenced in the Solution Doc? (if there is a significant tech decision but no ADR → PAUSE and wait)
- Security Requirements (if `Security role: yes` → wait for SEC to send Security_Requirements first)

### STEP 4 — Create Lead Handoff

Claude automatically generates `LEAD_HANDOFF_[feature].md`

This file includes: PRD + Solution Doc + Epics + STACK_CONTEXT summary + DECISION_LOG + PATTERN_LIBRARY + ADRs + Security Requirements

Download and send to Lead

---

## Managing DECISION_LOG

Claude appends to DECISION_LOG automatically after every confirmed answer

**Rules:**
- `_TODO.md` — only unresolved items → re-upload every new session with remaining TODOs
- `_RESOLVED.md` — archive of resolved items → upload once, re-upload only when new resolved items are added
- Never merge the two files into one

When a TODO item is resolved → Claude moves the entry from `_TODO` to `_RESOLVED` and exports both files

---

## PATTERN_LIBRARY — Recording Patterns

After STEP 4 completes, Claude detects reusable patterns from that feature and asks PO whether to record them

After confirming → Claude appends to `PATTERN_LIBRARY.md` → download → re-upload to Project

---

## Production Bug Intake

Trigger: Anyone (cashier, Ops, Dev) reports a production issue to PO — or PO selects option 4 in the Welcome Dialog

**Rule:** PO is always the intake point — Dev must not report directly to Lead

### PO actions immediately (< 5 min)

1. Create `BugIntake_BR-[NNN]_[title].md` and save in `docs/roles/po/` (Option B) or download and keep (Option A)
2. Notify Lead immediately and send the BugIntake file
3. **Wait for Lead to confirm severity** — PO does not decide severity, does not instruct Dev directly

| Severity | Meaning | What Lead will do |
|----------|---------|-------------------|
| **P1** | Service down / data loss / security breach | Issue HotfixTask immediately — no SA wait |
| **P2** | Functional bug, workaround available | Issue HotfixTask; SA reviews async after merge |
| **P3** | Minor bug, no direct user impact | Add to backlog — use normal pipeline |

### What PO receives back

After hotfix deploys to Production and QA smoke test passes → Lead sends `HotfixNotification_HF-[NNN].md` to PO to record in `docs/roles/po/` — used as an audit trail that the bug was resolved

---

## Re-entry Flows

| Situation | Action |
|-----------|--------|
| SA sends revised Solution Doc (after PoC FAIL/PARTIAL) | Re-enter STEP 3 directly — no need to re-run STEP 1-2 |
| Lead sends Issue Report → SA revises → sends back to PO | Re-enter STEP 3 directly |

---

## BugIntake File

Claude creates it automatically when PO selects option 4 → reports a new bug and answers the questions:

```markdown
# BugIntake_BR-[NNN]_[short bug title].md

Date     : [date]
Reporter : PO / [reporter name]
Severity : Pending Lead confirmation

## Observed symptoms
[PO describes what was reported — no need to be technical]

## Branch / environment where issue was found
[Production only? Or SIT/Staging as well?]

## Steps to reproduce
[If known]

## Impact
[How many branches / users affected / is there a workaround?]
```

---

## Hard Rules

1. **One conversation per feature** — use the same thread throughout the feature's lifetime
2. STEP 1 and STEP 2 are never blocked by a missing STACK_CONTEXT.md
3. Never generate Claude Code prompts — that is Lead's job
4. Never do task breakdown — PO creates Epics only
5. Every session where decisions are added, remind PO to download DECISION_LOG before the next session
6. STACK_CONTEXT.md must come from SA only — never ask PO about tech stack

---

## Pre-Feature Checklist

- [ ] `PROJECT_CONTEXT.md` is in Project Knowledge — must have `Security role:`, `UX/UI required:` and `Environment:` (`cli` or `claude.ai`)
- [ ] `STACK_CONTEXT.md` is in Project Knowledge (if carried over from a previous project)
- [ ] A new conversation thread has been opened for this feature
