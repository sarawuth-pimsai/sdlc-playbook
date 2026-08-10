# Tech Lead Guide

> For workflow overview and setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

Lead has two responsibilities: (1) break tasks + generate Claude Code prompts, and (2) review Dev PRs before merge.

---

## Setup

### Option A — claude.ai Projects

Via the claude.ai web interface

#### Step 1 — Create a Project

1. Open [claude.ai](https://claude.ai) → **Projects** → **New project**
2. Name it, e.g. `[Project Name] — Lead`
3. Open **Project Instructions** → Copy content from `ai/LEAD_PROJECT_INSTRUCTIONS.md` → Paste → Save

#### Step 2 — Upload Project Knowledge

All files come from the **Lead Handoff** sent by PO:

| File | Required | Note |
|------|----------|------|
| `LEAD_HANDOFF_[feature].md` | ✅ | Main file — includes PRD + Solution Doc + Epics |
| `STACK_CONTEXT.md` | ✅ | Embedded in Lead Handoff or sent separately |
| `ADR_[NNN].md` | If present | SA sends via PO |
| `DECISION_LOG_[feature]_TODO.md` | If present | |
| `DECISION_LOG_[feature]_RESOLVED.md` | If present | |

---

### Option B — Claude Code

Via Claude Code CLI, Desktop App, or IDE Extension instead of claude.ai Projects

#### Step 1 — Setup workspace (one-time)

See setup details at [templates/option-b/README.md](../../templates/option-b/README.md) — use the `/setup` command to automatically create the directory structure and copy all required files

#### Step 2 — Start a Lead session

```
/lead
```

Type `/lead` in Claude Code — Claude will read `ai/LEAD_PROJECT_INSTRUCTIONS.md` and focus on `docs/roles/lead/` and `docs/shared/`

#### Step 3 — Place files in the Lead directory

Place knowledge files in `docs/roles/lead/` instead of uploading to Project Knowledge

---

## Lead Workflow — Task Breakdown

### L-STEP 1 — Read and Cross-check

Claude reads the Lead Handoff and extracts:
- PRD requirements
- SA Solution Doc constraints that Dev must not override
- Architecture decisions that affect task sequencing
- Epics from PO STEP 2
- Open items from DECISION_LOG

If open items are not yet resolved → Claude asks Lead: block task generation or use a placeholder?

### L-STEP 1.5 — Tier Escalation Check

Before breaking tasks, verify that the Triage Summary/Solution Doc received covers what Lead actually sees in the code/requirements.

**Block immediately if:**
- A task needs to change a data schema not mentioned in the original document
- A task needs to add a new external dependency not in the original Feature Brief
- The feature is Tier 1 but Lead sees it actually has more impact

When triggered → send an **Escalation Request directly to SA** (bypassing PO) and wait for a response before proceeding — Lead must not make architectural decisions in SA's place, except in Hotfix Flow P1/P2 (see §Hotfix Flow and [CORE_POLICY.md](../CORE_POLICY.md) §2)

**Before sending the Escalation Request**, always add an entry to `PATTERN_LIBRARY.md` section `## Escalated Keywords` (the keyword the original scan missed, the feature, the tier it should actually be), then send the updated file to PO to attach to the next SA Handoff — no need to wait for SA's escalation response first (see `ai/LEAD_PROJECT_INSTRUCTIONS.md` §L-STEP 1.5)

### L-STEP 2 — Break into Epics and Tasks

Claude proposes a task board — rules for each task:
- Single responsibility
- Completable in 1–2 days by one developer
- At least 2 verifiable done criteria

Claude shows a task board preview → Lead reviews → confirms before proceeding

**Claude will PAUSE if:**
- Task dependency order is unclear
- Task scope > 3 story points (suggests splitting)
- SA constraint conflicts with a PRD requirement

### L-STEP 2.5 — Dependency Graph + Lane Assignment

After the task board is confirmed, Claude performs 3 steps before generating prompts:

1. **Build Dependency Graph** — from `Depends on`/`Blocks` of every task
2. **Check file overlap** — tasks touching the same file → always forced to sequence; tasks not touching the same file + no dependency → split into independent lanes, can run in parallel
3. **Cross-lane utility scan** — review "What to create/modify" across all lanes; if there is a helper/utility that 2 or more lanes will share (e.g. validation, formatting, error builder) → extract it as a separate shared task and set `Depends on` for all lanes using it — that task must merge before those lanes start

**Lanes are named by domain** (e.g. `lane-auth`, `lane-api-gateway`) — not tied to person names, since lane assignments can change

Claude shows the Dependency Graph + lane assignment → Lead reviews → confirms before proceeding to L-STEP 3

**PAUSE trigger:** if the cross-lane scan finds a helper that multiple lanes may share but Lead is unsure whether to extract → Claude asks Lead first: "Found [utility name] that may be shared between [lane A] and [lane B] — would you like to extract it as a shared task first, or let each lane implement it separately?"

See details in `ai/LEAD_PROJECT_INSTRUCTIONS.md` §L-STEP 2.5 and [CORE_POLICY.md](../CORE_POLICY.md) §3

### L-STEP 3 — Generate Claude Code Prompts

Claude generates 1 prompt per task using the standard template. Each prompt includes:
- Project context (stack, framework, conventions)
- Task context (why this task exists)
- What to create/modify (files + function signatures)
- Skeleton reference (minimal correct code)
- Done criteria (verifiable commands/curl)
- What NOT to implement in this task

Claude shows a preview → Lead reviews all done criteria → confirms → Claude creates a React Artifact with a Download button per file

### L-STEP 4 — Generate CLAUDE.md for the Code Repo

Before drafting CLAUDE.md, check the **CI/CD gate checklist** (skip if Provider = none): does lint/analyze run automatically on PR? does the test suite run automatically? is there a coverage gate? does a build check block merge? — for any missing items, do not generate CI config (risk of being wrong for the provider) but add them as Open TODOs for Lead to discuss with ops instead

Claude drafts `CLAUDE.md` from STACK_CONTEXT.md + Solution Doc constraints

**Lead must commit CLAUDE.md to the repo root before Dev starts the first task**

---

## Lead Workflow — PR Review

### R-STEP 1 — Read TASK_LOG

Read `TASK_LOG.md` entries for tasks in this PR:
- Did Dev follow the spec?
- Any deviations? If so, are they justified?
- Any open TODOs that block merge?

### R-STEP 2 — Review Code in Claude Code

**Open a new session** for this review (not the same session Dev used for implementation) — paste only the review prompt, changed files, task spec, and CLAUDE.md; do not paste the Dev's history to avoid biasing the review from prior reasoning

Claude generates a review prompt for Lead to run in **Claude Code** (not claude.ai chat)

Review checklist:
1. Correctness — does the code match the spec?
2. Conventions — does it follow CLAUDE.md?
3. Error handling — are all error codes in the spec covered?
4. Security (Option B) — hardcoded secrets, missing input validation, sensitive data in response, auth checks, error message leaks
5. Test coverage — are the done criteria covered by tests?
6. Duplication — does any new function/method duplicate logic already in the codebase? Check utility functions, validation helpers, formatters, error builders — grep for similar names or signatures in other files before confirming this code is genuinely new

Output format: Must fix / Should fix / Consider / Verdict: APPROVE or REQUEST CHANGES

### R-STEP 3 — Update CLAUDE.md (if needed)

If this PR introduces a new pattern or convention → Claude asks Lead → Lead confirms → update CLAUDE.md

---

## Sending files to Dev and QA

**Send to Dev:**
- `Task_[ID]_[title].md` one at a time in dependency order
- `CLAUDE.md` (committed to repo before Dev starts)
- `STACK_CONTEXT.md`

**Send to QA (Phase A setup):**
- `STACK_CONTEXT.md`
- `PRD_[feature].md`
- `DECISION_LOG_[feature]_TODO.md` and `DECISION_LOG_[feature]_RESOLVED.md`
- All task prompt files (same set as Dev)

---

## Hotfix Flow

### Receiving BugIntake from PO

PO creates `BugIntake_BR-[NNN]_[title].md` and sends it to Lead — Lead **assesses severity only**, not Dev or Ops

| Severity | Condition | Path |
|----------|-----------|------|
| **P1** | Service down / data loss / security breach | Lead issues HotfixTask directly — no SA sign-off required |
| **P2** | Functional bug, workaround available | Lead issues HotfixTask directly — no SA sign-off required |
| **P3** | Minor bug | Add to backlog — use normal pipeline |

**Rule:** escalate to SA after merge — not before (P1/P2 speed over process) — but after merging, every hotfix must go through **Retroactive Tier Tagging** from SA without exception (see §Post-merge Checklist and `ai/SA_PROJECT_INSTRUCTIONS.md` §Retroactive Hotfix Triage)

### Lead issues HotfixTask to Dev

Send HotfixTask prompt to Dev like a normal task — Dev pastes the whole file as the first message in a Claude Code session with header: `## HOTFIX — [HF-ID]`

### Post-merge Checklist (after hotfix merges)

After hotfix merges and Dev deploys to Staging:

- [ ] Dev updates `TASK_LOG.md` with a `HF-[NNN]` entry
- [ ] Lead notifies QA to run **Phase HF smoke test on Staging** (see QA_GUIDE.md §Phase HF)
- [ ] QA smoke test **passes on Staging** → Lead then deploys to Production (do not deploy to Production before QA passes Staging)
- [ ] Lead notifies QA to run **Production smoke check** (P1 cases only)
- [ ] Production smoke check passes → Lead creates `HotfixNotification_HF-[NNN].md` and sends to PO
- [ ] PO forwards HF summary to SA for **Retroactive Tier Tagging** — mandatory for every hotfix (see `ai/SA_PROJECT_INSTRUCTIONS.md` §Retroactive Hotfix Triage) — SA records the result back in `TASK_LOG.md` at the original `HF-[NNN]` entry
- [ ] If fix deviates from Solution Doc / ADR → Lead creates an ADR Amendment
- [ ] If fix reveals an architectural gap → Lead sends an Issue Report to SA

---

## Direct SA Communication

Lead can send an Issue Report directly to SA (bypassing PO) for:
- Technical clarification that does not affect PRD requirements
- Implementation-level questions in the Solution Doc

Use the Issue Report format (see SA_GUIDE.md) so SA can correctly assess impact

---

## Pre-Task-Breakdown Checklist

- [ ] `LEAD_HANDOFF_[feature].md` has been uploaded to the Lead Project
- [ ] `STACK_CONTEXT.md` is present and all fields are complete
- [ ] Solution Doc is complete with all required sections (9 required + section 10 if UX/UI)
- [ ] Version header of all files matches the Lead Handoff date

## Pre-Task-Prompt Checklist (before sending to Dev)

- [ ] Task board confirmed
- [ ] All done criteria are verifiable (by real commands, not just "code looks right")
- [ ] `CLAUDE.md` committed to repo root
- [ ] Task prompts sent one at a time in dependency order
- [ ] Every task has a Lane assigned in the Parallel metadata block
- [ ] Tasks touching the same file have been sequenced — no two tasks on the same file are still marked as parallel
- [ ] Assigned Dev is listed for every lane (or "unassigned")
- [ ] Cross-lane utility scan complete — any shared utilities found have been extracted as separate tasks, and the lanes using them have a dependency pointing to those tasks
