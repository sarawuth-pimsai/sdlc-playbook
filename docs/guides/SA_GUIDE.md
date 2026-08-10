# Solution Architect (SA) Guide

> For workflow overview and setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

SA is responsible for analysing PRDs from a technical perspective, designing solutions, and owning `STACK_CONTEXT.md`.

**SA is always the owner of tier decisions** — no role can bypass SA. See [CORE_POLICY.md](../CORE_POLICY.md) §1

---

## Setup

### Option A — claude.ai Projects

Via the claude.ai web interface

#### Step 1 — Create a Project

1. Open [claude.ai](https://claude.ai) → **Projects** → **New project**
2. Name it, e.g. `[Project Name] — SA`
3. Open **Project Instructions** → Copy content from `ai/SA_PROJECT_INSTRUCTIONS.md` → Paste → Save

#### Step 2 — Upload Project Knowledge

| File | When | Note |
|------|------|------|
| `STACK_CONTEXT.md` | ✅ Before first session | SA creates and owns this file |
| `SOLUTION_PATTERNS.md` | ✅ Before first session | SA creates and maintains this file |
| `DECISION_LOG_[feature]_TODO.md` | When received from PO | Sent with SA Handoff |
| `DECISION_LOG_[feature]_RESOLVED.md` | When received from PO | Sent with SA Handoff |
| `PATTERN_LIBRARY.md` | When received from PO | Sent with SA Handoff |

---

### Option B — Claude Code

Via Claude Code CLI, Desktop App, or IDE Extension instead of claude.ai Projects

#### Step 1 — Setup workspace (one-time)

See setup details at [templates/option-b/README.md](../../templates/option-b/README.md) — use the `/setup` command to automatically create the directory structure and copy all required files

#### Step 2 — Start an SA session

```
/sa
```

Type `/sa` in Claude Code — Claude will read `ai/SA_PROJECT_INSTRUCTIONS.md` and focus on `docs/roles/sa/`

#### Step 3 — Place files in the SA directory

Place knowledge files in `docs/roles/sa/` instead of uploading to Project Knowledge

---

## Input received from PO

### Case A — Stack Setup Request (first feature, or no stack yet)

PO sends the file `SA_STACK_SETUP_REQUEST_[ProjectName].md`

**Action:**
1. Check whether PO has attached a PE org template (`STACK_CONTEXT_[OrgName].md`)
   - **If attached** → use the org template as the baseline, skip steps 2–3
   - **If not attached** → proceed to step 2
2. **Interview SA first** — summarise the constraints from the Stack Setup Request for SA, then ask which template SA wants or how to customise (do not choose a template for SA automatically)
3. Select a template from `docs/roles/sa/stack-templates/` that SA has confirmed
4. Fill in any missing values + verify all versions via WebSearch (see Version Verification rule in `ai/SA_PROJECT_INSTRUCTIONS.md`)
5. Save as `STACK_CONTEXT.md`
6. Upload to SA Project Knowledge
7. Send file back to PO → PO uploads to PO Project

See available stack templates: [docs/roles/sa/stack-templates/README.md](../roles/sa/stack-templates/README.md)

### Case B — SA Handoff (STACK_CONTEXT.md already exists)

PO sends the file `SA_HANDOFF_[feature].md`

**Must come with:**
- PRD content (embedded in handoff)
- `DECISION_LOG_[feature]_TODO.md`
- `DECISION_LOG_[feature]_RESOLVED.md` (if exists)
- `PATTERN_LIBRARY.md` (if exists)

---

## SA Workflow

### STEP 1 — Read Context (silently)

Claude reads all files first, then extracts from the SA Handoff:
- PRD content
- Epics from PO STEP 2
- Open TODO items that affect architectural decisions

### STEP 1.5 — Tier Triage (SA only — before starting STEP 2)

SA evaluates 4 questions (does it touch a new data model? add a new integration? match an existing pattern in SOLUTION_PATTERNS.md? is it already covered by STACK_CONTEXT.md?) then decides the tier:

| Tier | Criteria | Path |
|------|----------|------|
| 1 — Micro | No new data model, no new integration, matches existing pattern | Fast Path — write only a short Triage Summary (10–15 lines), skip STEP 2–6 |
| 2 — Standard | Moderate data model / logic changes | STEP 2–7 normally with full Solution Doc |
| 3 — Full | Large impact, or ≥ 1 business-risk flag from PO | STEP 2–7 + mandatory SEC review |

**Rule:** A business-risk flag from PO does not auto-set the tier, but forces a minimum tier of Tier 3. See [CORE_POLICY.md](../CORE_POLICY.md) §1 and `ai/SA_PROJECT_INSTRUCTIONS.md` §STEP 1.5

### STEP 2 — Technical PRD Analysis (Tier 2/3 only)

- Summarise the feature (2–3 sentences, technical perspective)
- Identify data flows, integration points, external dependencies, constraints
- List technical risks (HIGH/MED/LOW)

If clarification from PO is needed → Claude creates an **HTML Artifact dialog**

### STEP 3 — Propose Solution Options (Tier 2/3 only)

Claude proposes 2–3 options with pros/cons/complexity

**Rule:** Claude does not choose an option — SA decides

### STEP 4 — Draft Solution Doc (Tier 2/3 only)

After SA selects an option → Claude drafts `Solution_Doc_[feature].md` (9 required sections + section 10 if PROJECT_CONTEXT.md has `UX/UI required: yes`)

**9 required sections (+ section 10 if UI):**
1. Overview
2. Architecture
3. Tech decisions
4. API / Interface design
5. Data model changes
6. Non-functional considerations (including security checklist for Option B)
7. Risks and mitigations
8. Open questions
9. PoC scope (if needed)
10. UX/UI considerations (only when `UX/UI required: yes` — see `ai/PROJECT_INSTRUCTIONS.md` §Security role & UX/UI check)

SA reviews the draft in chat → confirms → Claude creates an HTML Artifact with a Download button

### STEP 5 — Draft ADRs (Tier 2/3 only — Tier 1 unless there is a genuinely significant decision)

For every tech decision that is "significant":
- Chooses a technology that differs from STACK_CONTEXT.md defaults
- Changes a data model that affects more than 1 service
- Trade-off where rationale must be preserved
- Decision that Lead cannot infer from the code alone

**ADR numbering:** Use the global index `docs/adr/INDEX.md` — SA reserves a number before drafting

### STEP 6 — PoC Planning (Tier 2/3 only — if PoC scope in STEP 4)

Claude generates PoC prompts for Lead to run as a spike

**PoC outcomes:**
- **PASS** → mark assumption "validated" → proceed to STEP 7
- **FAIL** → return to STEP 3 with evidence → if PRD requirements are affected → notify PO first
- **PARTIAL** → SA decides whether the partial result is sufficient

### STEP 7 — Handoff Package

Claude compiles a handoff summary with a distribution plan

**Send all artifacts to PO** (not directly to Lead):
- Tier 1: `Triage_Summary_[feature].md`
- Tier 2/3: `Solution_Doc_[feature].md`
- `ADR_[NNN]_[title].md` (× N, if any)
- PoC prompts (if any)
- `STACK_CONTEXT.md` (if PO sent a Stack Setup Request)
- `SOLUTION_PATTERNS.md` (if updated this session)

---

## STACK_CONTEXT.md — Maintenance Rules

- SA is the **sole owner** — PO must not edit it
- When the stack changes → SA updates it in the SA Project → increments the version → notifies PO to re-upload to the PO Project
- Every time a file is sent to PO, state the version in the message: e.g. `STACK_CONTEXT.md Version 2`
- If the organisation has PE and the org template changes → PE will notify SA about what the project needs to sync

---

## Lead Issue Report (Lead → SA)

Lead can send an Issue Report directly to SA (bypassing PO) for:
- Technical clarification that does not affect PRD requirements
- Implementation-level questions in the Solution Doc

**SA diagnoses impact:**
- **Small** → SA issues an ADR Amendment + updates Solution Doc → can send directly to Lead
- **Large** (affects PRD scope / shared data model) → SA notifies PO first → PO approves → SA revises → sends to PO → PO re-enters STEP 3

---

## Retroactive Hotfix Triage (PO → SA)

After a hotfix (P1/P2) is merged, PO sends an HF summary to SA for a brief retroactive tier tagging (5–10 min) — **mandatory for every hotfix, no exceptions** (see [CORE_POLICY.md](../CORE_POLICY.md) §4)

SA evaluates the Tier retrospectively using the same criteria as STEP 1.5 Tier Triage, then records the result back into `TASK_LOG.md` at the `HF-[NNN]` entry (no new file) — if the result is Tier 2/3, immediately create a follow-up task in Lead's normal queue

---

## SOLUTION_PATTERNS.md — Maintenance

After PO accepts a feature → SA reviews whether there is an architectural pattern worth keeping for reuse

Claude asks: "This feature has [pattern] — record in SOLUTION_PATTERNS.md?"

SA confirms → Claude appends → exports → SA re-uploads to SA Project

---

## Pre-Handoff Checklist (before sending SA Handoff to PO)

- [ ] Solution Doc is complete with all 9 sections (+ section 10 if UX/UI) (no section is just "TBD")
- [ ] ADR files are complete as referenced in the Solution Doc
- [ ] ADR numbers are sequential in `docs/adr/INDEX.md`
- [ ] PoC prompts are ready (if PoC scope exists)
- [ ] `STACK_CONTEXT.md` version is stated in the message
- [ ] All artifacts sent to PO — not directly to Lead
