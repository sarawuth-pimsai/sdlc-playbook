# Core Policy — shared across Solo and Enterprise mode

This file is the authoritative policy that every role/mode must follow. Other guides must not duplicate this logic — reference this file instead.

---

## 1. Tier Ownership

**SA always owns the tier decision** — no role (PO, Lead, or even a solo dev) can bypass SA. The only difference is how much time SA spends depending on the tier.

- PO can only collect feature details and flag business-risk keywords as context for SA — PO **does not decide the tier**
- SA decides the tier at STEP 1.5 (Tier Triage) — see details in `ai/SA_PROJECT_INSTRUCTIONS.md` §STEP 1.5
- Applies to both Solo and Enterprise mode without exception — the old Solo "Minimum" tier that let PO bypass SA **has been removed**. See `docs/guides/SOLO_GUIDE.md` §Recommended roles for solo

### Business-risk keyword list (used by both PO and SA)

```
BUSINESS_RISK_KEYWORDS = [
  "payment", "checkout", "billing",
  "auth", "authentication", "authorization", "login", "session token",
  "PII", "personal data", "email address", "phone number", "national ID",
  "external API", "third-party integration", "webhook",
  "encryption", "compliance", "GDPR", "PDPA"
]
```

A business-risk flag from PO does not auto-set the tier, but forces a minimum tier of Tier 3 — even if SA assesses the structure as low-impact.

---

## 2. Escalation

The Lead escalation gate (L-STEP 1.5 — Tier Escalation Check) applies identically to both modes — see `ai/LEAD_PROJECT_INSTRUCTIONS.md` §L-STEP 1.5

A Lead who finds that work exceeds the scope of the original Triage Summary/Solution Doc must escalate directly to SA (not through PO) — Lead must never make architectural decisions in SA's place under any circumstances, except in Hotfix Flow P1/P2 (see §4).

---

## 3. Parallel Execution

Parallel lane assignment (L-STEP 2.5 — Dependency Graph + Lane Assignment) applies to both modes — see `ai/LEAD_PROJECT_INSTRUCTIONS.md` §L-STEP 2.5

A solo dev with no second developer can skip this step (noted in `docs/guides/SOLO_GUIDE.md`).

---

## 4. Hotfix Flow

Hotfix flow (P1/P2/P3) in `ai/LEAD_PROJECT_INSTRUCTIONS.md` §Hotfix flow applies to both modes — Solo can use the same Lead session immediately, no separate flow needed (see `docs/guides/SOLO_GUIDE.md` §Hotfix Flow)

P1/P2 is the only exception where Lead makes technical/scope decisions without waiting for SA sign-off first — escalate to SA **only after merge**.

Every hotfix (P1/P2) must go through **Retroactive Tier Tagging** from SA after merge — this is not optional "when there's time" (see `ai/LEAD_PROJECT_INSTRUCTIONS.md` §Hotfix post-merge checklist and `ai/SA_PROJECT_INSTRUCTIONS.md` §Retroactive Hotfix Triage) to catch cases where a hotfix was actually Tier 2/3 but nobody knew because the triage was bypassed during the urgent merge.

---

## 5. Environment (per-role, not project-wide)

`PROJECT_CONTEXT.md` holds the **project default** and **per-role overrides**:

```markdown
Environment (default): cli / claude.ai
Environment overrides:
  SA: [not specified = use default]
  Lead: [not specified = use default]
  Dev: cli   # always fixed — cannot be overridden
  QA: [not specified = use default]
  SEC: [not specified = use default]   # Option A only (Security role: yes)
```

**Effective Environment per role** = that role's override (if set), otherwise the default — Dev is always fixed to `cli` and can never be overridden.

**Pairwise rule at every handoff point** (replaces the old single-side check):

| Sending side (effective) | Receiving side (effective) | How to output |
|---|---|---|
| cli | cli | Write tool → save directly to disk in the same repo (automatic, no relay needed) |
| Other (either or both sides are claude.ai) | — | Artifact + Download button → notify to upload to the destination Project manually |
| Receiving side effective Environment is unknown | — | Use Artifact as safe default (always compatible) |

Each role asks for its own override **only once when first starting work on this project** — not every session. Full details on how to ask and the Handoff Check are in `ai/SA_PROJECT_INSTRUCTIONS.md`, `LEAD_PROJECT_INSTRUCTIONS.md`, `QA_PROJECT_INSTRUCTIONS.md`, `SEC_PROJECT_INSTRUCTIONS.md` (Option A only) §Environment override + §Handoff Environment Check

---

## 6. Routing Principle — when to route through PO, when to route directly

The system has 2 routing patterns that look asymmetric but are intentionally designed around work cadence:

**Planning-phase handoffs → always through PO** (low frequency, requires oversight)
- SA → PO → Lead (Solution Doc, ADR)
- SEC Phase A → through PO (Solution Doc security review, before implementation starts)
- Reason: planning happens once per feature; PO is the single distribution channel with full visibility, appropriate as a business-level checkpoint

**Execution-phase interactions → directly between operational roles** (high frequency, requires speed)
- Lead ↔ SA escalation (L-STEP 1.5)
- Lead ↔ SEC Phase B (PR-level code review)
- Reason: implementation happens frequently (every task, every PR) — routing through PO every time would create a bottleneck. Lead and other operational roles are the ones doing the actual work at this stage; PO does not need to be an intermediary every time.

**Summary rule:** the higher the interaction frequency (many rounds per feature), the more PO should be cut out. The lower the frequency (once per feature), the more PO should be involved for oversight — this is not a fixed rule per individual role.
