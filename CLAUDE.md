# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repository is

This is **not an application** — there is no code to build, lint, or test. `sdlc-playbook` is a set of Claude system prompts (`ai/*.md`) that define how Claude should behave when acting as each role (PO, SA, Lead, Dev, QA, SEC) in a team's SDLC, plus human-readable guides that mirror them. Consumers copy these files into their own projects — either as Claude Project Instructions on claude.ai ("Option A") or as `CLAUDE.md` + slash commands for Claude Code ("Option B").

**Do not add lint scripts, test runners, CI tooling, or any package manager to this repo.** Its value is being a stack-agnostic, zero-dependency markdown template usable by any downstream project regardless of language — introducing tooling here (even maintainer-only) contradicts that. Consistency checks on `ai/*.md`/`docs/*.md` (duplicate sections, broken `§`-references, stale patterns) are done manually via `grep`, not via a committed script.

## Repository structure

```
ai/           System prompts, one per role — the actual "product" of this repo
docs/
  CORE_POLICY.md        Single source of truth for policy shared across roles (see below)
  WORKFLOW_OVERVIEW.md  Mermaid diagrams of the full flow, both Option A and B
  guides/                Human-readable guide per role, mirrors ai/*.md content for people, not Claude
  roles/                 Example/seed knowledge files (STACK_CONTEXT.md samples, ADR index, etc.)
templates/option-b/     Slash-command templates (/po, /sa, /lead, /qa, /setup) for Claude Code consumers
.claude/commands/       Slash commands for maintaining *this* repo (e.g. /ship, /release) — not templates for consumers
```

`ai/DEV_PROJECT_INSTRUCTIONS.md` is the odd one out: Dev always uses Claude Code directly (never a claude.ai Project), so consumers copy it in as their project's own `CLAUDE.md`.

## Working on this repo

All work happens directly on `develop`; `main` only receives merges via PR. Use the `/ship` slash command (`.claude/commands/ship.md`) to commit, push to `develop`, and open the `develop` → `main` PR in one flow — it encodes the safety checks (branch check, no force-push, ask before touching `.gitignore`) worked out for this repo, so prefer it over ad-hoc `git`/`gh` commands.

After a `/ship` PR is merged into `main`, use the `/release` slash command (`.claude/commands/release.md`) to tag a semver version from `main` and publish a GitHub release from it — it works against `origin/main` directly (no checkout) and always stops to get explicit confirmation on the version number before tagging, since that judgment call is hard to undo once public.

## Core architecture — read before editing any role file

The system is a PO-centered handoff pipeline: **PO → SA → SEC (Option A only) → PO → Lead → Dev/QA**. Before changing any `ai/*.md` file, understand these cross-cutting mechanisms — most bugs in this repo are inconsistencies introduced when one role's file is updated without updating the shared policy or the other roles that reference it:

- **`docs/CORE_POLICY.md` is the single source of truth** for policy that must be identical across roles and across Solo/Enterprise mode: tier ownership (§1), escalation (§2), parallel execution (§3), hotfix flow (§4), environment default+override (§5), and the PO-routing-vs-direct-routing principle (§6). Individual `ai/*.md` / `docs/guides/*.md` files should **reference** CORE_POLICY sections, not duplicate their logic. If you find yourself writing the same rule in two role files, it belongs in CORE_POLICY instead.
- **SA owns tier decisions, never PO.** SA's STEP 1.5 (Tier Triage) classifies each feature as Tier 1 (Micro — Triage Summary fast path, no full Solution Doc), Tier 2 (Standard — full Solution Doc), or Tier 3 (Full — full Solution Doc + mandatory SEC review). PO only scans PRDs for business-risk keywords and passes them as context; a business-risk flag forces a tier floor of 3 but never sets the tier itself. QA reads the same `Tier:` field (from the Solution Doc or Triage Summary header) at Phase A-STEP 0 to decide between a full test suite and a Lightweight Check / Smoke Test fast path — Tier 2/3 QA behavior must stay byte-for-byte unchanged when editing Tier 1 fast-path logic.
- **Lead has an escalation gate (L-STEP 1.5)** that goes directly to SA, bypassing PO, when implementation reveals scope the Triage Summary/Solution Doc didn't cover. QA has an equivalent escalation that feeds into this same Lead gate rather than being a separate mechanism — don't invent a second escalation path per role.
- **Routing follows a frequency principle (CORE_POLICY §6):** low-frequency, once-per-feature planning handoffs (SA→PO→Lead, SEC Phase A) go through PO for oversight; high-frequency, per-task/per-PR execution interactions (Lead↔SA escalation, Lead↔SEC Phase B, Lead↔Dev/QA) bypass PO for speed. When adding a new cross-role interaction, decide its frequency first, then its routing — don't default to "through PO."
- **Environment is per-role, not project-wide.** `PROJECT_CONTEXT.md` holds an `Environment (default)` plus per-role overrides (SA/Lead/QA/SEC can override; Dev is permanently fixed to `cli`). Every handoff point does a *pairwise* check — both the sender's and receiver's effective environment — before deciding to `Write` straight to disk (both `cli`) or produce a downloadable Artifact (anything else, or unknown receiver as the safe default).
- **Hard gates, not soft warnings.** Places where a role could historically "note a missing input and proceed anyway" (e.g. PO generating a Lead Handoff without SA's Solution Doc) have been deliberately converted to PAUSE/STOP blocks. When editing a `Check for X` section, don't reintroduce a silent-proceed path.

## Editing conventions

- Every shared knowledge file template (`STACK_CONTEXT.md`, `DECISION_LOG_*`, `PATTERN_LIBRARY.md`, `PROJECT_CONTEXT.md`) carries a `Last updated: YYYY-MM-DD | Version: N` header — keep this when touching templates.
- `ai/*.md` files are written in English; the "ภาษาที่ใช้" section in each mandates that everything Claude *outputs* to that role (questions, warnings, summaries) is in Thai. Keep this split when editing.
- Each role file ends with an "Ask-human triggers" / "Golden rule" section defining STOP/PAUSE/CHECK levels and which role is the sole decision-maker for that domain (e.g. "SA is the technical decision maker", "QA sets the verdict"). Never let one role silently make another's decision when adding new logic.
- `docs/guides/*.md` should stay in sync with the corresponding `ai/*.md` file whenever a workflow step changes — they're two audiences (human vs. Claude) for the same process, not independent documents.
