# คู่มือ Tech Lead

> ดูภาพรวม workflow และการ setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

Lead มีหน้าที่ 2 อย่าง: (1) แตก task + generate Claude Code prompts และ (2) review PR ของ Dev ก่อน merge

---

## การติดตั้ง

### Option A — claude.ai Projects

ใช้ผ่าน claude.ai web interface

#### ขั้นตอนที่ 1 — สร้าง Project

1. เปิด [claude.ai](https://claude.ai) → **Projects** → **New project**
2. ตั้งชื่อ เช่น `[ชื่อโปรเจกต์] — Lead`
3. เปิด **Project Instructions** → Copy เนื้อหาจาก `ai/LEAD_PROJECT_INSTRUCTIONS.md` → Paste → บันทึก

#### ขั้นตอนที่ 2 — อัปโหลด Project Knowledge

ไฟล์ทั้งหมดมาจาก **Lead Handoff** ที่ PO ส่งให้:

| ไฟล์ | จำเป็น | หมายเหตุ |
|------|--------|---------|
| `LEAD_HANDOFF_[feature].md` | ✅ | ไฟล์หลัก — รวม PRD + Solution Doc + Epics |
| `STACK_CONTEXT.md` | ✅ | ฝังอยู่ใน Lead Handoff หรือส่งแยก |
| `ADR_[NNN].md` | ถ้ามี | SA ส่งผ่าน PO |
| `DECISION_LOG_[feature]_TODO.md` | ถ้ามี | |
| `DECISION_LOG_[feature]_RESOLVED.md` | ถ้ามี | |

---

### Option B — Claude Code

ใช้ผ่าน Claude Code CLI, Desktop App, หรือ IDE Extension แทน claude.ai Projects

#### ขั้นตอนที่ 1 — Setup workspace (ทำครั้งเดียว)

ดูรายละเอียดการ setup ที่ [templates/option-b/README.md](../../templates/option-b/README.md) — ใช้ `/setup` command เพื่อสร้าง directory structure และ copy ไฟล์ที่จำเป็นทั้งหมดอัตโนมัติ

#### ขั้นตอนที่ 2 — เริ่ม Lead session

```
/lead
```

พิมพ์ `/lead` ใน Claude Code — Claude จะอ่าน `ai/LEAD_PROJECT_INSTRUCTIONS.md` และโฟกัสที่ `docs/roles/lead/` และ `docs/shared/`

#### ขั้นตอนที่ 3 — วางไฟล์ใน directory ของ Lead

วางไฟล์ knowledge ใน `docs/roles/lead/` แทนการ upload เข้า Project Knowledge

---

## Workflow ของ Lead — Task Breakdown

### L-STEP 1 — อ่านและ Cross-check

Claude อ่าน Lead Handoff และ extract:
- PRD requirements
- SA Solution Doc constraints ที่ Dev ห้าม override
- Architecture decisions ที่กระทบ task sequencing
- Epics จาก PO STEP 2
- Open items จาก DECISION_LOG

ถ้า open items ยังไม่ resolve → Claude ถาม Lead: block task generation หรือใช้ placeholder?

### L-STEP 2 — Break เป็น Epics และ Tasks

Claude เสนอ task board — กฎของแต่ละ task:
- Single responsibility
- เสร็จได้ใน 1-2 วันต่อ developer คนเดียว
- มี done criteria อย่างน้อย 2 ข้อที่ verifiable ได้

Claude แสดง task board preview → Lead review → confirm ก่อนไปต่อ

**Claude PAUSE ถ้า:**
- Task dependency order ไม่ชัดเจน
- Task scope > 3 story points (แนะนำ split)
- SA constraint ขัดกับ PRD requirement

### L-STEP 3 — Generate Claude Code Prompts

Claude generate 1 prompt ต่อ task ตาม template มาตรฐาน แต่ละ prompt มี:
- Project context (stack, framework, conventions)
- Task context (why this task exists)
- What to create/modify (files + function signatures)
- Skeleton reference (minimal correct code)
- Done criteria (verifiable commands/curl)
- What NOT to implement in this task

Claude แสดง preview → Lead review done criteria ทุกข้อ → confirm → Claude สร้าง React Artifact พร้อม Download ทีละไฟล์

### L-STEP 4 — Generate CLAUDE.md สำหรับ Code Repo

Claude draft `CLAUDE.md` จาก STACK_CONTEXT.md + Solution Doc constraints

**Lead ต้อง commit CLAUDE.md เข้า repo root ก่อน Dev เริ่ม task แรก**

---

## Workflow ของ Lead — PR Review

### R-STEP 1 — อ่าน TASK_LOG

อ่าน `TASK_LOG.md` entries ของ tasks ใน PR นี้:
- Dev ทำตาม spec หรือไม่?
- มี deviation? ถ้ามี justified ไหม?
- มี open TODO ที่ block merge หรือไม่?

### R-STEP 2 — Review Code ใน Claude Code

Claude generate review prompt สำหรับ Lead รันใน **Claude Code** (ไม่ใช่ claude.ai chat)

Checklist ที่ review:
1. Correctness — code match spec ไหม
2. Conventions — ตาม CLAUDE.md ไหม
3. Error handling — ครบทุก error code ใน spec ไหม
4. Security (Option B) — hardcoded secrets, missing input validation, sensitive data in response, auth checks, error message leaks
5. Test coverage — done criteria ถูก cover ด้วย tests ไหม

Output format: Must fix / Should fix / Consider / Verdict: APPROVE or REQUEST CHANGES

### R-STEP 3 — อัปเดต CLAUDE.md (ถ้าจำเป็น)

ถ้า PR นี้แนะนำ pattern หรือ convention ใหม่ → Claude ถาม Lead → Lead confirm → update CLAUDE.md

---

## การส่งไฟล์ให้ Dev และ QA

**ส่งให้ Dev:**
- `Task_[ID]_[title].md` ทีละไฟล์ตาม dependency order
- `CLAUDE.md` (commit เข้า repo ก่อน Dev เริ่ม)
- `STACK_CONTEXT.md`

**ส่งให้ QA (Phase A setup):**
- `STACK_CONTEXT.md`
- `PRD_[feature].md`
- `DECISION_LOG_[feature]_TODO.md` และ `DECISION_LOG_[feature]_RESOLVED.md`
- Task prompt files ทุกไฟล์ (ชุดเดียวกับ Dev)

---

## Hotfix Flow

### รับ BugIntake จาก PO

PO สร้าง `BugIntake_BR-[NNN]_[title].md` แล้วส่งมาให้ Lead — Lead **ประเมิน severity** เท่านั้น ไม่ใช่ Dev หรือ Ops

| Severity | เงื่อนไข | Path |
|----------|---------|------|
| **P1** | Service down / data loss / security breach | Lead ออก HotfixTask ตรง — ไม่ต้องรอ SA sign-off |
| **P2** | Functional bug, มี workaround | Lead ออก HotfixTask; SA review async หลัง merge |
| **P3** | Minor bug | ใส่ backlog — ใช้ normal pipeline ปกติ |

**กฎ:** escalate ถึง SA หลัง merge — ไม่ใช่ก่อน (P1/P2 speed over process)

### Lead ออก HotfixTask ให้ Dev

ส่ง HotfixTask prompt ให้ Dev เหมือน normal task — Dev paste ทั้งไฟล์เป็น message แรกใน Claude Code session header: `## HOTFIX — [HF-ID]`

### Post-merge Checklist (หลัง hotfix merge)

หลัง hotfix merge และ Dev deploy สู่ Staging:

- [ ] Dev อัปเดต `TASK_LOG.md` ด้วย `HF-[NNN]` entry
- [ ] Lead แจ้ง QA ให้ run **Phase HF smoke test บน Staging** (ดู QA_GUIDE.md §Phase HF)
- [ ] QA smoke test **ผ่านบน Staging** → Lead จึง deploy สู่ Production (ห้าม deploy Production ก่อน QA ผ่าน Staging)
- [ ] Lead แจ้ง QA ให้ run **Production smoke check** (P1 cases เท่านั้น)
- [ ] Production smoke check ผ่าน → Lead สร้าง `HotfixNotification_HF-[NNN].md` ส่งให้ PO
- [ ] ถ้า fix เบี่ยงจาก Solution Doc / ADR → Lead สร้าง ADR Amendment
- [ ] ถ้า fix เผย architectural gap → Lead ส่ง Issue Report ให้ SA

---

## Direct SA Communication

Lead ส่ง Issue Report ตรงถึง SA ได้ (ไม่ต้องผ่าน PO) สำหรับ:
- Technical clarification ที่ไม่กระทบ PRD requirements
- Implementation-level questions ใน Solution Doc

ใช้ format Issue Report (ดู SA_GUIDE.md) เพื่อให้ SA ประเมิน impact ได้ถูกต้อง

---

## Checklist ก่อนเริ่ม Task Breakdown

- [ ] `LEAD_HANDOFF_[feature].md` อัปโหลดเข้า Lead Project แล้ว
- [ ] `STACK_CONTEXT.md` มีและครบทุก field
- [ ] Solution Doc ครบทุก section (7 required sections)
- [ ] Version header ของทุกไฟล์ตรงกับ Lead Handoff date

## Checklist ก่อนส่ง Task Prompts ให้ Dev

- [ ] Task board confirmed แล้ว
- [ ] Done criteria ทุกข้อ verifiable (ด้วย command จริง ไม่ใช่ "code looks right")
- [ ] `CLAUDE.md` committed เข้า repo root แล้ว
- [ ] ส่ง task prompts ทีละไฟล์ตาม dependency order
