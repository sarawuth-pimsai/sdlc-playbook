# Developer Guide

> For workflow overview and setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

Developer uses **Claude Code** (CLI or App) — not claude.ai Projects.

---

## Installing Claude Code

### Option A — CLI (recommended)

```bash
npm install -g @anthropic-ai/claude-code
```

Verify:
```bash
claude --version
```

### Option B — Desktop App

Download from [claude.ai/download](https://claude.ai/download)

### Option C — IDE Extension

Install the extension for VS Code or JetBrains from the marketplace

---

## Input received from Lead

Lead sends files one at a time in dependency order:

| File | Note |
|------|------|
| `CLAUDE.md` | Lead commits to repo root before starting — read every time before anything else |
| `STACK_CONTEXT.md` | Tech stack, build/test commands |
| `Task_[ID]_[title].md` | Task prompt one task at a time — do not start the next task until all done criteria for the current task pass |

---

## Resuming a task session (session interrupted mid-way)

If you need to return to a task left from a previous session:

1. Open `TASK_LOG.md` first — find the entry for that task ID
   - All done criteria ticked `[x]` → task is complete, no need to implement again
   - Done criteria not yet complete → read Deviations and Notes before continuing
2. Always run build and tests first — do not assume the repo is still clean
3. Continue from where you left off — no need to start from scratch

---

## Starting a Task

### Step 1 — Prepare Claude Code

```bash
# Open a terminal at the root of the code repo
cd /path/to/my-project
claude
```

### Step 2 — Read CLAUDE.md first

CLAUDE.md contains project conventions to follow, e.g. error shape, logging rules, context passing

### Step 3 — Create a branch

Create the branch from `### Branch setup` in the task prompt before starting to write code:

```bash
git checkout [base branch from task prompt]
git pull
git checkout -b [branch name from task prompt]
```

### Step 4 — Paste the Task Prompt

Paste **the entire content** of `Task_[ID]_[title].md` into a new Claude Code session

> Do not summarise or cut the content — paste verbatim

### Step 5 — Implement

Claude Code implements according to the spec in the task prompt

**Within scope:**
- Implement code per task spec
- Refactor within the scope of this task
- Fix bugs caused by code in this task
- Ask questions to understand existing code

**Not allowed:**
- Add features outside the task spec
- Start the next task before all done criteria pass
- Change architecture, data models, or API contracts without consulting Lead
- Contact SA directly — always escalate through Lead first

---

## Done Criteria — must run yourself before raising a PR

```bash
# 1. Build must pass
[build command from STACK_CONTEXT.md]

# 2. Tests must pass
[test command from STACK_CONTEXT.md]

# 3. Verify each done criterion
# Run curl/commands as specified in the task prompt
```

> "Claude says it looks correct" ≠ done — run the real commands

---

## Self-check before raising a PR (beyond build/test passing)

Build passing and tests passing does not mean code is ready — run before every PR:

1. **Strict static analysis** — strict lint/analyze command from STACK_CONTEXT.md must pass with zero warnings, not just zero errors
2. **Verify APIs exist** — check against the real lockfile version before trusting that a method/class exists — never assume from training knowledge
3. **Error handling complete** — every async call has an error path, and null/undefined is checked before use
4. **Resource cleanup** — any controller/stream/listener opened must be explicitly disposed
5. **Self-review diff** — did you change files outside the task scope? did you add unnecessary new dependencies?
6. **Test asserts real behavior** — tests you wrote must have assertions tied to real behavior, not placeholders (e.g. `expect(true, true)`)
7. **Existing function check** — grep/search the codebase for functions or methods with a similar signature or behavior to what you just wrote (e.g. validation, formatting, error building) — if found, reuse instead; if you choose to create new code, record the reason in TASK_LOG under "Deviations"

Any item that fails → go back and fix before raising a PR

---

## TASK_LOG.md — update after every task

```markdown
## Task [ID] — [title]
Date completed    : [date]
STACK_CONTEXT ver : [Version N from STACK_CONTEXT.md header]
Deviations        : none / [description and reason]
Files changed     : [list]

Done criteria:
- [x] [build command] passes
- [x] [specific test/curl] returns [expected]
- [x] [other criteria]

Notes: [anything Lead should know — edge cases, assumptions, open questions]
```

Lead reads TASK_LOG before reviewing every PR — write it completely and honestly

---

## PR Checklist

Before raising a PR, verify:

- [ ] All done criteria passed (run, not just read)
- [ ] All 7 self-check items passed (static analysis, API verification, error handling, resource cleanup, self-review diff, real test assertions, existing function check)
- [ ] `TASK_LOG.md` updated for this task
- [ ] Follows conventions in `CLAUDE.md` (error shape, logging, context passing)
- [ ] No hardcoded values that should come from environment variables
- [ ] No files that should be in `.gitignore` included in the commit

---

## Deploy Notification to QA

After deploying to SIT (or Staging) → run the deploy command from STACK_CONTEXT first → verify service is up → send this notification to QA:

```markdown
[DEPLOY NOTIFICATION]
Feature   : [feature name]
Task      : [Task ID] — [title]     (write "All tasks" for Staging deploy)
Target    : SIT / Staging
Date/time : [date and time]
URL       : [SIT_URL or STAGING_URL from STACK_CONTEXT]

Changed endpoints / features:
- [list endpoints or features deployed in this build]

Done criteria verified by Dev:
- [x] [criterion 1]
- [x] [criterion 2]

TASK_LOG updated : yes
Build passes     : yes

Ready for QA Phase A / Phase B ✓
```

> **Rule:** do not send the notification until all done criteria have passed

---

## Hotfix Task

Lead sends a HotfixTask prompt to Dev like a normal task — **paste the entire content as the first message in a Claude Code session**. The task will have the header `## HOTFIX — [HF-ID]` and `Severity: P1 / P2`

### Branch discipline

- Branch **always from the production/main branch** — never from dev or a feature branch
- Branch name: use the `Hotfix branch` format from STACK_CONTEXT.md, e.g. `hotfix/HF-001-short-desc`
- PR target: production/main branch — not dev

### Minimal change rule — the most important rule of a hotfix

**Fix only the root cause specified in "Fix scope" — no exceptions**

- Do not refactor surrounding code
- Do not make "while I'm here" improvements
- If the fix needs to touch code outside "Fix scope" → **STOP** notify Lead before proceeding

### TASK_LOG.md for hotfixes

Use `HF-[NNN]` as the task ID:

```markdown
## HF-[NNN] — [title]
Date completed    : [date]
STACK_CONTEXT ver : [Version N]
Severity          : P1 / P2
Root cause        : [one sentence]
Deviations        : none / [explain — notify Lead immediately if any]
Files changed     : [list — should be very short for a hotfix]
```

### Deploy Notification for hotfixes

Hotfix deploys to **Staging first** — QA must pass a smoke test on Staging before Lead deploys to Production. Send notification to **both Lead and QA** (not QA only):

```markdown
[HOTFIX DEPLOY NOTIFICATION]
HF-ID     : HF-[NNN]
Severity  : P1 / P2
Target    : Staging
Date/time : [date and time]

Fix deployed:
- [what was changed]

Done criteria verified:
- [x] Build passes
- [x] [fix criterion]
- [x] No regression on: [critical paths tested locally]

TASK_LOG updated : yes
```

### Additional PR Checklist for hotfixes

In addition to the normal PR checklist:

- [ ] Branch is from production/main — not dev/feature
- [ ] Fix touches only code in "Fix scope"
- [ ] "Do NOT change" items in the task prompt were not touched
- [ ] No new dependencies were added

---

## When the Task Prompt doesn't cover a situation

1. Choose the most conservative assumption (prefer no-op over side effects)
2. Record in `TASK_LOG.md` under "Deviations": what happened, what was decided, why
3. Continue implementing
4. Lead reviews the assumption in the PR — if wrong will ask you to fix before merge

---

## Stop and wait for Lead in these situations

| Level | When | Action |
|-------|------|--------|
| STOP | Task prompt conflicts with CLAUDE.md conventions | Stop, show the conflict, wait for Lead to resolve |
| STOP | Done criteria cannot be verified (environment unreachable) | Stop, notify Dev to contact Lead, wait |
| PAUSE | Task scope is larger than expected after reading the actual code | Show the scope difference — proceed only after Dev confirms |
| PAUSE | An assumption you need to make affects another developer's parallel work | Flag first, record in TASK_LOG |
