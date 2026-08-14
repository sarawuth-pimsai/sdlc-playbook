# SDLC Playbook — AI-Assisted Development Workflow

A set of Claude Project Instructions for software development teams using AI as a role-specific assistant throughout the SDLC.

Files in `ai/` are **system prompts** for each role — usable in two ways:

- **Option A** — upload as Claude Project Instructions on claude.ai (web interface)
- **Option B** — place as `CLAUDE.md` and run Claude Code (CLI/App/IDE Extension)

There is no runnable application in this repo.

> **Note:** The `/CLAUDE.md` at the root of this repo is a different file from the `CLAUDE.md` mentioned above — the root file provides guidance to Claude Code for people who maintain or edit the playbook itself. The `CLAUDE.md` that Dev actually uses is the one Lead generates fresh for each feature repo (derived from `ai/DEV_PROJECT_INSTRUCTIONS.md`).

---

## 🚀 Quick Start (5 minutes)

Want to try it before reading everything? Follow these steps (Option B — Claude Code):

1. Clone `sdlc-playbook` anywhere on your machine
2. In the project repo you want to use it in: `mkdir -p .claude/commands && cp sdlc-playbook/templates/option-b/commands/setup.md .claude/commands/`
3. Open Claude Code in the project repo and type `/setup` — Claude builds the entire structure automatically
4. Type `/po` and start answering questions

**No need to read CORE_POLICY.md, WORKFLOW_OVERVIEW.md, or any role guide first** — Claude will ask for what it needs (tier, environment, security role, etc.) at the right moments. Come back to read them later when you want to understand "why Claude asked that" or if you're working as a team and want to see the full system before starting.

> 👤 Working solo? See [docs/guides/SOLO_GUIDE.md](docs/guides/SOLO_GUIDE.md) — a condensed flow designed specifically for solo devs.
> 📖 Full setup details (role isolation, manual copy without `/setup`) → [templates/option-b/README.md](templates/option-b/README.md)
> 🌐 Using Option A (claude.ai Projects) instead? See "First-time Setup — Option A" below in this file.

---

## Overview

> 📊 See full Workflow Diagrams → [docs/WORKFLOW_OVERVIEW.md](docs/WORKFLOW_OVERVIEW.md)
> 🏷️ Are the `ai/*.md` files you copied older than the latest version? Check → [Releases](https://github.com/sarawuth-pimsai/sdlc-playbook/releases)



```
PO (hub) ──────► SA      SA Handoff → Triage Summary (Tier 1) or Solution Doc + ADRs + PoC prompts (Tier 2/3)
         ──────► SEC     Solution Doc → Security Requirements  (Option A + mandatory if Tier 3)
         ──────► Lead    Lead Handoff = PRD + Tier + Triage Summary/Solution Doc + Epics + Stack

Lead ───────────► Dev    Task_[ID].md prompts one file at a time per lane
Dev ────────────► QA     Deploy Notification after deploying to SIT/Staging
Lead ───────────► QA     PRD + DECISION_LOG + task files (Phase A setup)
SEC ────────────► Lead   Security_Review_[TaskID].md per PR  (Option A only)
Lead ◄─────────► SA      Escalation Request (direct, not through PO — see reason in CORE_POLICY.md §6)
```

**PO is the central hub for planning-phase handoffs** (SA/SEC ↔ Lead) — execution-phase interactions that happen frequently (Lead↔SA escalation, Lead↔SEC Phase B, Lead↔Dev/QA) route directly without going through PO. See full principle in [docs/CORE_POLICY.md](docs/CORE_POLICY.md) §6

---

## Repository Structure

```
CLAUDE.md                        → guidance for Claude Code when editing this playbook repo itself (not the Dev role file)
ai/
  PROJECT_INSTRUCTIONS.md        → PO Claude Project (Option A) / /po command (Option B)
  SA_PROJECT_INSTRUCTIONS.md     → SA Claude Project / /sa command
  LEAD_PROJECT_INSTRUCTIONS.md   → Lead Claude Project / /lead command
  DEV_PROJECT_INSTRUCTIONS.md    → Developer (CLAUDE.md / Claude Code)
  QA_PROJECT_INSTRUCTIONS.md     → QA Claude Project / /qa command
  SEC_PROJECT_INSTRUCTIONS.md    → Security Engineer Claude Project (Option A only)
docs/
  CORE_POLICY.md    → single source of truth: tier ownership, escalation, parallel execution,
                       hotfix flow, environment default+override, routing principle
  WORKFLOW_OVERVIEW.md → Mermaid diagrams: role-to-role flow, tier/escalation overview, hotfix flow
  roles/            → example knowledge files (STACK_CONTEXT.md, ADR INDEX.md)
                       (SA Stack Setup Request is not a static template — PO generates it
                        automatically each time from ai/PROJECT_INSTRUCTIONS.md §SA Stack Setup Request)
  guides/
    PO_GUIDE.md       → Product Owner guide
    SA_GUIDE.md       → Solution Architect guide
    LEAD_GUIDE.md     → Tech Lead guide
    DEV_GUIDE.md      → Developer guide
    QA_GUIDE.md       → QA Engineer guide
    SEC_GUIDE.md      → Security Engineer guide
    PE_GUIDE.md       → Platform Engineering guide (org template)
    SOLO_GUIDE.md     → Solo Developer guide (one person wearing multiple hats)
    th/               → Thai language versions of all guides above
  roles/
    sa/
      STACK_CONTEXT.md              → blank schema (use when no matching template exists)
      stack-templates/
        README.md                   → guide for choosing and using templates + PE workflow
        STACK_CONTEXT_base_crosscutting.md  → cross-cutting reference (Auth, OTel, PDPA)
        STACK_CONTEXT_go_clean_arch.md      → Go + Clean Architecture + PostgreSQL/Redis
templates/
  option-b/
    README.md         → setup guide for Option B (role isolation)
    commands/
      setup.md        → slash command template for /setup (auto-creates directory structure)
      po.md           → slash command template for /po
      sa.md           → slash command template for /sa
      lead.md         → slash command template for /lead
      qa.md           → slash command template for /qa
```

---

## Security Options

| Option | Condition | Result |
|--------|-----------|--------|
| **Option A** | Team has a Security Engineer | SEC Project active, SEC reviews Solution Doc + every PR |
| **Option B** | No Security Engineer | Security checkpoints are already embedded in SA / Lead / QA instructions |

PO sets this once in `PROJECT_CONTEXT.md` (`Security role: yes / no`) when starting the project.

---

## Option B — Claude Code (CLI/App/IDE Extension)

Option B lets each role use slash commands instead of copying prompts into a claude.ai Project.

### Recommended directory structure (role isolation)

```
my-project/
  CLAUDE.md                    ← Dev instructions (Lead generates)
  ai/                          ← copy from sdlc-playbook/ai/
  .claude/
    commands/                  ← copy from sdlc-playbook/templates/option-b/commands/
      setup.md → /setup  ← run first to auto-create the entire structure
      po.md    → /po
      sa.md    → /sa
      lead.md  → /lead
      qa.md    → /qa
  docs/
    roles/
      po/                      ← PO knowledge files (DECISION_LOG, PATTERN_LIBRARY)
      sa/                      ← SA knowledge files (STACK_CONTEXT, Solution_Doc, adr/)
      lead/                    ← Lead knowledge files (LEAD_HANDOFF)
      qa/                      ← QA knowledge files (TestCases, BugReports)
    shared/                    ← visible to all roles (tasks/, TASK_LOG.md)
```

See setup details at [templates/option-b/README.md](templates/option-b/README.md)

---

## Solo Developer

Working alone and wearing multiple role hats? See [docs/guides/SOLO_GUIDE.md](docs/guides/SOLO_GUIDE.md)

---

## Optional Companion Tools

| Tool | What it adds | Works with |
|------|-------------|------------|
| [Ponytail](https://github.com/dietrichgebert/ponytail) | Code minimalism at generation time — seven-rung decision ladder that prevents over-engineering (YAGNI, stdlib-first, reuse-first) | Option B (Claude Code) |

Ponytail operates at a different layer from this playbook (code generation discipline vs. workflow orchestration) and can be installed alongside it without conflict. Follow the install instructions in the Ponytail repo.

---

## First-time Setup — Option A

> This section describes **Option A** (claude.ai Projects). For **Option B** (Claude Code) see [templates/option-b/README.md](templates/option-b/README.md)

### Step 1 — Create Claude Projects on claude.ai

Create a separate Claude Project per role:

| Project name (recommended) | Instruction file | Who uses it |
|----------------------------|------------------|-------------|
| `[Project] — PO` | `ai/PROJECT_INSTRUCTIONS.md` | Product Owner |
| `[Project] — SA` | `ai/SA_PROJECT_INSTRUCTIONS.md` | Solution Architect |
| `[Project] — Lead` | `ai/LEAD_PROJECT_INSTRUCTIONS.md` | Tech Lead |
| `[Project] — Dev` | `ai/DEV_PROJECT_INSTRUCTIONS.md` | Developer (see note) |
| `[Project] — QA` | `ai/QA_PROJECT_INSTRUCTIONS.md` | QA Engineer |
| `[Project] — SEC` | `ai/SEC_PROJECT_INSTRUCTIONS.md` | Security Engineer (Option A) |

> **Developer:** `DEV_PROJECT_INSTRUCTIONS.md` is used with **Claude Code** (CLI/App), not a claude.ai Project.
> See details in [DEV_GUIDE.md](docs/guides/DEV_GUIDE.md)

### Step 2 — Upload Instruction File to each Project

For each Claude Project:

1. Open Project Settings → **Project Instructions**
2. Copy content from the instruction file → paste into the Instructions field
3. Save

### Step 3 — Create PROJECT_CONTEXT.md

Create a `PROJECT_CONTEXT.md` file and upload it to **PO Project Knowledge**:

```markdown
# PROJECT_CONTEXT.md

# Last updated: YYYY-MM-DD | Version: 1

## Project settings

Security role: yes / no
UX/UI required: yes / no
Environment (default): cli / claude.ai
Environment overrides:
  SA: [not specified = use default]
  Lead: [not specified = use default]
  Dev: cli   # always fixed — cannot be overridden
  QA: [not specified = use default]
  SEC: [not specified = use default]   # Option A only (Security role: yes)
```

- `Security role: yes` → Option A: SEC Engineer is active in the workflow
- `Security role: no` → Option B: security checkpoints are embedded in other roles
- `UX/UI required: yes` → SA needs a UI reference before starting architecture design (affects decisions such as offline-first, real-time) / `no` → SA skips all UX/UI considerations (e.g. backend service, internal API, cron job)
- `Environment (default)` is the project-level default — each role (SA/Lead/QA/SEC) can override to a different channel at the start of its first session. Dev is always fixed to `cli` and cannot be overridden.
- Each handoff point performs a pairwise check (both sender and receiver sides) to decide whether to save directly to disk or create an Artifact — see full details in [docs/CORE_POLICY.md](docs/CORE_POLICY.md) §5

### Step 4 — SA creates STACK_CONTEXT.md

PO sends `SA_STACK_SETUP_REQUEST_[ProjectName].md` to SA.

**If the organisation has Platform Engineering and an org template is ready** → attach `STACK_CONTEXT_[OrgName].md` with the request. SA will use the org template as a baseline — reduces setup time and enforces org standards automatically.

**If no org template exists yet** → SA selects a stack template from `docs/roles/sa/stack-templates/` that matches the stack family.

Once `STACK_CONTEXT.md` is received from SA → upload it to:

- **PO Project Knowledge**
- **SA Project Knowledge**

> See Platform Engineering workflow details at [docs/guides/PE_GUIDE.md](docs/guides/PE_GUIDE.md)

---

## SDLC Flow Overview

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. PO starts a new session                                         │
│     Welcome Wizard → check files → STEP 1 (analyse PRD)            │
│     → STEP 1.5 (clarify) → STEP 1.6 (scan business-risk keywords)  │
│     → STEP 2 (Epics) → auto-create SA Handoff                       │
├─────────────────────────────────────────────────────────────────────┤
│  2. SA receives SA Handoff                                          │
│     STEP 1.5 — Tier Triage (always required, SA decides, not PO)    │
│     Tier 1 → short Triage Summary (fast path)                       │
│     Tier 2/3 → propose options → draft Solution Doc (with Tier:     │
│       header) → ADRs → PoC (if needed) → send all artifacts to PO  │
├─────────────────────────────────────────────────────────────────────┤
│  3. SEC receives Solution Doc (Option A + mandatory if Tier 3)      │
│     Phase A review → Security_Requirements → send to PO (always     │
│     through PO)                                                     │
├─────────────────────────────────────────────────────────────────────┤
│  4. PO runs STEP 3 + STEP 4                                         │
│     Check stack + Triage Summary/Solution Doc (hard block if not    │
│     yet received) → create Lead Handoff                             │
├─────────────────────────────────────────────────────────────────────┤
│  5. Lead receives Lead Handoff                                      │
│     L-STEP 1 cross-check → L-STEP 1.5 Escalation Check             │
│       (scope exceeds doc → escalate directly to SA, not through PO) │
│     → L-STEP 2 task breakdown → L-STEP 2.5 Dependency Graph +      │
│       Lane Assignment + Cross-lane Utility Scan → generate prompts  │
│     → CLAUDE.md → send task prompts to Dev + setup files to QA     │
├─────────────────────────────────────────────────────────────────────┤
│  6. Dev implements (one task at a time per lane)                    │
│     Paste Task_[ID].md in Claude Code → implement → run done        │
│     criteria → update TASK_LOG (Lane + Assigned Dev)                │
│     → deploy SIT → notify QA                                        │
├─────────────────────────────────────────────────────────────────────┤
│  7. QA tests — A-STEP 0 always checks tier first                    │
│     Phase 0: draft test cases from PRD (parallel with SA)           │
│     Phase A: Tier 1 → Lightweight Check | Tier 2/3 → full           │
│       TestCases → TaskTestSummary → Dev fixes bugs                  │
│     Phase B: Tier 1 → Smoke Test | Tier 2/3 → full suite           │
│       → TestReport → Lead sign-off                                  │
│     Phase C: Production smoke test → rollback if P1 fails           │
└─────────────────────────────────────────────────────────────────────┘
```

> Some routing goes through PO (planning-phase, low frequency) and some routes directly between operational roles (execution-phase, high frequency) — see full principle in [docs/CORE_POLICY.md](docs/CORE_POLICY.md) §6

---

## File Management Rules

### Version Header

Every shared file must have this header on the first line:

```
# Last updated: YYYY-MM-DD | Version: N
```

### DECISION_LOG — two files per feature

| File | Content | Re-upload |
|------|---------|-----------|
| `DECISION_LOG_[feature]_TODO.md` | Only unresolved items | Every new session with remaining TODOs |
| `DECISION_LOG_[feature]_RESOLVED.md` | Archive of resolved items | Only when new resolved items are added |

Never merge the two files into one.

### One thread per feature (Hard Rule)

Use one conversation thread per feature throughout the feature's lifetime.  
Open a new conversation only when starting a new feature.

### Files stored in the actual project (code repo)

> ⚠️ **Caution:** `STACK_CONTEXT.md` in a real project will contain internal org information such as IdP endpoints, infrastructure details, and internal service names — **do not commit this file to a public repository**. Add `docs/STACK_CONTEXT.md` to `.gitignore` or use a private repo only.

```
my-project/
  CLAUDE.md               ← Lead creates, committed before Dev starts first task
  docs/
    STACK_CONTEXT.md      ← do not commit to public repo (internal stack details)
    DECISION_LOG_[feature]_TODO.md
    DECISION_LOG_[feature]_RESOLVED.md
    Solution_Doc_[feature].md
    adr/
      INDEX.md
      ADR-001_[title].md
    tasks/
      Task_[ID]_[title].md
    qa/
      [feature]/
        TestCases_[TaskID].md
        BugReport_[TaskID].md
        TestSuite_[Feature].md
        TestReport_[Feature].md
    TASK_LOG.md
```

---

## Role Guides

| Role | Guide |
|------|-------|
| **Workflow Overview** | [docs/WORKFLOW_OVERVIEW.md](docs/WORKFLOW_OVERVIEW.md) |
| Product Owner | [docs/guides/PO_GUIDE.md](docs/guides/PO_GUIDE.md) |
| Solution Architect | [docs/guides/SA_GUIDE.md](docs/guides/SA_GUIDE.md) |
| Tech Lead | [docs/guides/LEAD_GUIDE.md](docs/guides/LEAD_GUIDE.md) |
| Developer | [docs/guides/DEV_GUIDE.md](docs/guides/DEV_GUIDE.md) |
| QA Engineer | [docs/guides/QA_GUIDE.md](docs/guides/QA_GUIDE.md) |
| Security Engineer | [docs/guides/SEC_GUIDE.md](docs/guides/SEC_GUIDE.md) |
| Platform Engineering | [docs/guides/PE_GUIDE.md](docs/guides/PE_GUIDE.md) |
| Solo Developer | [docs/guides/SOLO_GUIDE.md](docs/guides/SOLO_GUIDE.md) |
| **Stack Templates** | [docs/roles/sa/stack-templates/README.md](docs/roles/sa/stack-templates/README.md) |

---

## Shared Files — Owners and Flow

| File | Owner | Flows to |
|------|-------|---------|
| `STACK_CONTEXT.md` | SA | PO → Lead → Dev / QA |
| `DECISION_LOG_[feature]_TODO.md` | PO | SA, Lead, QA |
| `DECISION_LOG_[feature]_RESOLVED.md` | PO | SA, Lead, QA |
| `PATTERN_LIBRARY.md` | PO | SA, Lead |
| `SOLUTION_PATTERNS.md` | SA | SA-internal |
| `PROJECT_CONTEXT.md` | PO | PO → SA, Lead, QA, SEC (Environment default + per-role override) |
| `Triage_Summary_[feature].md` (Tier 1) | SA | PO → Lead |
| `Solution_Doc_[feature].md` (Tier 2/3) | SA | PO → Lead, SEC |
| `Security_Requirements_[feature].md` | SEC | PO → Lead |
| `LEAD_HANDOFF_[feature].md` | PO | Lead |
| `Task_[ID]_[title].md` | Lead | Dev, QA |
| `TASK_LOG.md` | Dev | Lead (reads before PR review) |

## License

This software is licensed under the **Business Source License 1.1 (BSL 1.1)**.

### 🛑 Key Terms & Restrictions

- **Allowed Uses (Free Internal Production):** Any organisation, company, or individual **may install and use this software in production systems internally at no cost**.
- **Commercial & SaaS Restrictions:**
  - ❌ **Prohibited** — using this software to offer a Managed Service or **Software-as-a-Service (SaaS)** to external parties
  - ❌ **Prohibited** — embedding or wrapping this software in a commercial product to resell or sublicense to your customers
- **Automatic Open Source Conversion:** On **July 17, 2029**, this version of the software automatically converts to Open Source under **GPL v3.0**, removing all commercial restrictions.

### 💼 Commercial Partnerships & Special Licensing

If your company wants to build a SaaS system, offer a client service, or embed this software in a closed product for resale, contact **Sarawuth Pimsai** directly via [GitHub Issues](https://github.com/sarawuth-pimsai/sdlc-playbook/issues) to purchase a Commercial License.
