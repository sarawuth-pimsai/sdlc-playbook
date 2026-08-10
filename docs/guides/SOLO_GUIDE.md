# Solo Developer Guide — SDLC Playbook

This guide is for developers working alone who need to wear multiple role hats.

---

## Why separate sessions even when working solo

Changing role = changing mental context:

- **PO session** — think like a user, do not think about implementation
- **Lead session** — think like an architect, clarify scope before writing code
- **Dev session** — implement exactly to spec, do not go outside the task

In a single session, Claude will shortcut and skip steps — e.g. knowing the PRD and writing code directly without going through task breakdown — leading to scope creep and implementation that doesn't match requirements.

---

## Recommended roles for solo

### Minimum (small project, not complex)

**SA always owns the tier decision, even working alone — it cannot be skipped** — the Minimum tier still requires going through an SA session, but uses a short SA STEP 1.5 Tier Triage (not a full Solution Doc). See `ai/SA_PROJECT_INSTRUCTIONS.md` §STEP 1.5 and [CORE_POLICY.md](../CORE_POLICY.md) §1

```
PO session → SA session (short triage) → Lead session → Dev session (CLAUDE.md)
```

| Role | What | Time estimate |
|------|------|---------------|
| PO | Analyse PRD, scan business-risk keywords, create SA Handoff | 30–60 min per feature |
| SA | Tier Triage — decide tier + write short Triage Summary (Fast Path if Tier 1) | 10–15 min per feature (Tier 1) |
| Lead | Receive Triage Summary/Solution Doc, break into tasks, generate Task_[ID].md | 30–60 min per feature |
| Dev | Implement task by task until complete | Most of the time |

Note: If SA assesses that the feature has more impact than anticipated (e.g. touches a new data model or has a business-risk flag) → the tier will be upgraded to Tier 2/3 automatically and will take more time as per a full Solution Doc

### Recommended (project with external integrations or complex data model)

```
PO session → SA session → PO session → Lead session → Dev session
```

Add SA to design the architecture first — prevents rework at Lead/Dev

### Full (production-critical project)

```
PO → SA → PO → Lead → Dev → QA → Dev (fix) → QA (retest)
```

---

## Solo setup (Option B)

Follow the [Option B Setup Guide](../option-b/README.md) first, using the `/setup` command:

### 1. First-time setup

```bash
# copy just the setup.md file
mkdir -p .claude/commands
cp <path-to-sdlc-playbook>/templates/option-b/commands/setup.md .claude/commands/setup.md
```

Open Claude Code and type `/setup` — Claude will automatically create the directory structure and copy all required files

### 2. Prepare sessions per role

After setup is complete, use slash commands to separate sessions by role:

```
/po    → analyse PRD
/sa    → Tier Triage (always required) + design solution if Tier 2/3
/lead  → break into tasks
       → (Dev does not need to type a command — CLAUDE.md loads automatically)
/qa    → run tests (if needed)
```

### 2. Checkpoint between roles

Copy-pasting a handoff file to the next session is a **forced checkpoint** — it forces you to read what Claude generated before proceeding

Don't see it as overhead — see it as reviewing yourself in the next role's perspective

---

## Working approach

### Starting a new feature

1. Open Claude Code → type `/po`
2. Upload or paste PRD content
3. Follow STEP 1–2 until you have `SA_HANDOFF_[feature].md`
4. Save the handoff in `docs/roles/sa/`
5. Type `/sa` in a new window — SA runs STEP 1.5 Tier Triage (always required, even for Minimum tier) until you have `Triage_Summary_[feature].md` (Tier 1) or `Solution_Doc_[feature].md` (Tier 2/3)
6. Save the SA output in `docs/roles/po/` then return to `/po` to run STEP 3–4 until you have `LEAD_HANDOFF_[feature].md`
7. Save the handoff in `docs/roles/lead/`
8. Type `/lead` in a new window (or the same session after resetting)
9. Follow L-STEP 1–4 until you have all `Task_[ID]_[title].md` files
10. Save task files in `docs/shared/tasks/`
11. Open a new session (CLAUDE.md loads automatically)
12. Paste the content from the Task_[ID].md you want to implement

### Continuing a pending feature

- Return to the **same session** for that role (if using the same conversation thread)
- Or open a new session + type the slash command + upload the relevant knowledge files

---

## Tips

**Don't switch roles in the same session** — if you think "let me just implement this" during a PO session → make a note and do it in the Dev session instead

**A short PRD is fine** — PO interview mode in `PROJECT_INSTRUCTIONS.md` can draft a PRD from answering questions, no need for a full document

**Use TASK_LOG.md seriously** — it is the single source of truth for which tasks are done and whether done criteria passed — it helps enormously when coming back after several days away

**Solo doesn't need a separate STACK_CONTEXT.md** — if you already know the stack well, Lead can embed stack info directly in CLAUDE.md and skip the SA Stack Setup flow (this is about STACK_CONTEXT.md only — **it does not apply to the SA Tier Triage session, which can never be skipped** — see §Recommended roles for solo)

---

## Hotfix Flow (urgent production bug)

Solo uses exactly the same Hotfix Flow as enterprise — see [LEAD_GUIDE.md §Hotfix Flow](LEAD_GUIDE.md#hotfix-flow) and `ai/LEAD_PROJECT_INSTRUCTIONS.md` §Hotfix flow (see [CORE_POLICY.md](../CORE_POLICY.md) §4)

Open a Lead session and state the severity (P1/P2) using the same criteria — no separate flow is needed for solo because the mechanism is identical, the only difference being that one person wears the Lead hat

---

## Artifact ownership (solo version)

When working alone, you own all artifacts, but you still need to create them following the conventions because Claude reads these formats:

| Artifact | Created when | Stored where |
|----------|-------------|--------------|
| PRD | PO session | `docs/roles/po/` |
| SA_HANDOFF_[feature].md | PO session | `docs/roles/sa/` |
| Triage_Summary_[feature].md (Tier 1) / Solution_Doc_[feature].md (Tier 2/3) | SA session | `docs/roles/po/` |
| LEAD_HANDOFF_[feature].md | PO session | `docs/roles/lead/` |
| Task_[ID]_[title].md | Lead session | `docs/shared/tasks/` |
| CLAUDE.md | Lead session | project root |
| TASK_LOG.md | Dev session (update every task) | `docs/shared/` |
