# คู่มือ Product Owner (PO)

> ดูภาพรวม workflow และการ setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

PO เป็น **hub กลาง** ของ SDLC Playbook — ประสานงานทุก role และเป็นเจ้าของ decision history ของทุก feature

**PO ไม่ตัดสินใจ tier** — ทำได้แค่เก็บรายละเอียด feature และ flag business-risk keyword เป็น context ส่งให้ SA ตัดสินใจ ดู [CORE_POLICY.md](../CORE_POLICY.md) §1

---

## การติดตั้ง

### Option A — claude.ai Projects

ใช้ผ่าน claude.ai web interface

#### ขั้นตอนที่ 1 — สร้าง Project

1. เปิด [claude.ai](https://claude.ai) → **Projects** → **New project**
2. ตั้งชื่อ เช่น `[ชื่อโปรเจกต์] — PO`
3. เปิด **Project Instructions** → Copy เนื้อหาจาก `ai/PROJECT_INSTRUCTIONS.md` → Paste → บันทึก

#### ขั้นตอนที่ 2 — อัปโหลด Project Knowledge

| ไฟล์ | เมื่อไหร่ | หมายเหตุ |
|------|----------|---------|
| `PROJECT_CONTEXT.md` | ✅ ก่อน session แรก | กำหนด `Security role:`, `UX/UI required:` และ `Environment:` — Claude สร้างให้ใน session แรก → re-upload |
| `STACK_CONTEXT.md` | หลังได้รับจาก SA | ต้องครบก่อน STEP 3 |
| `DECISION_LOG_[feature]_TODO.md` | ทุก session ที่มี TODO คงเหลือ | Re-upload ทุก session ใหม่ |
| `DECISION_LOG_[feature]_RESOLVED.md` | เมื่อมี resolved items | Upload ครั้งแรก แล้ว re-upload เฉพาะเมื่อมีเพิ่ม |
| `PATTERN_LIBRARY.md` | ถ้ามี | SA หรือ PO สร้าง |

> **Session แรก:** Claude จะถาม `Security role`, `UX/UI required` และ `Environment` แล้วสร้าง `PROJECT_CONTEXT.md` ให้อัตโนมัติ → ดาวน์โหลดแล้ว re-upload เข้า Project Knowledge

---

### Option B — Claude Code

ใช้ผ่าน Claude Code CLI, Desktop App, หรือ IDE Extension แทน claude.ai Projects

#### ขั้นตอนที่ 1 — Setup workspace (ทำครั้งเดียว)

ดูรายละเอียดการ setup ที่ [templates/option-b/README.md](../../templates/option-b/README.md) — ใช้ `/setup` command เพื่อสร้าง directory structure และ copy ไฟล์ที่จำเป็นทั้งหมดอัตโนมัติ

#### ขั้นตอนที่ 2 — เริ่ม PO session

```
/po
```

พิมพ์ `/po` ใน Claude Code — Claude จะอ่าน `ai/PROJECT_INSTRUCTIONS.md` และโฟกัสที่ `docs/roles/po/`

#### ขั้นตอนที่ 3 — วางไฟล์ใน directory ของ PO

วางไฟล์ knowledge ใน `docs/roles/po/` แทนการ upload เข้า Project Knowledge

---

## SESSION START — ทุกครั้งที่เปิด session ใหม่

1. Claude อ่านไฟล์ทั้งหมดใน Project Knowledge อัตโนมัติ (ทำ silently)
2. Claude แสดง **Session Welcome** ใน chat — สรุปไฟล์ที่พบ + ถามสถานะโปรเจกต์หรืองานที่ต้องการทำ
3. ตอบคำถามทีละข้อ — Claude routing ตาม input ทันที

> ถ้ากลับมาใน conversation thread เดิม → Claude ข้าม Welcome และแสดง feature status summary แทน

เมื่อ STACK_CONTEXT มี `Status: Confirmed` Claude จะถาม:

```
1. เพิ่ม Feature ใหม่
2. ต่อ Feature ที่ค้างไว้
3. ดู Decision Log
4. แจ้ง Bug / ปัญหา Production — รับรายงานและส่งต่อ Lead
```

เลือก **4** → Claude ถามต่อ: รายงาน Bug ใหม่ หรือ ติดตาม Bug ที่ส่ง BugIntake ไปแล้ว

---

## SDLC Flow ของ PO

### STEP 1 — วิเคราะห์ PRD

**Trigger:** Upload PRD เข้า session หรือ confirm PRD จาก interview

Claude จะ:
- Summarise feature (2-3 ประโยค)
- Score แต่ละ section (1-10)
- List gaps ตาม severity: HIGH / MED / LOW
- แยก tasks: UNBLOCKED vs BLOCKED (รอ PO ตัดสินใจ)

### STEP 1.5 — ชี้แจง PRD

Claude ถามทีละคำถามสำหรับ ambiguity 3 ประเภท:
- **Type A** — Ambiguous requirement (ตีความได้หลายทาง)
- **Type B** — Contradictory requirement (ขัดแย้งกัน)
- **Type C** — Missing critical detail (ขาดรายละเอียดสำคัญ)

ตอบคำถามทีละข้อ หรือกด **Skip** เพื่อบันทึกเป็น TODO

### STEP 1.6 — สแกน Business-risk keyword

ก่อนสแกน ถ้ามี `PATTERN_LIBRARY.md` ให้ Claude อ่าน section `## Escalated Keywords` ก่อนเสมอ (keyword ที่เคย escalate มาก่อนหน้า) แล้วรวมเข้ากับ scan รอบนี้ด้วย — ดู `ai/PROJECT_INSTRUCTIONS.md` §STEP 1.6

หลัง STEP 1.5 Claude สแกน PRD หา business-risk keyword (payment, auth, PII, external API, encryption/compliance ฯลฯ — ดูรายการเต็มใน [CORE_POLICY.md](../CORE_POLICY.md) §1) แล้วแนบผลสแกนเป็น context ใน SA Handoff — **นี่ไม่ใช่การตัดสิน tier** เป็นแค่ raw flag ให้ SA ใช้ประกอบการตัดสินใจที่ STEP 1.5 Tier Triage ของ SA

### STEP 2 — กำหนด Epics

Claude จัดกลุ่มงานเป็น Epics (high-level เท่านั้น — Lead ทำ task breakdown)

**หลัง confirm STEP 2:** Claude สร้าง SA Handoff อัตโนมัติ (รวม business-risk flags จาก STEP 1.6)
- ถ้ายังไม่มี `Security role` หรือ `UX/UI required` ใน PROJECT_CONTEXT.md → Claude ถามพร้อมกันก่อน (ดู `ai/PROJECT_INSTRUCTIONS.md` §Security role & UX/UI check)

**หลังส่ง SA Handoff:** แจ้ง QA ให้เริ่ม Phase 0 ได้เลยโดยไม่รอ SA

### STEP 3 — ตรวจ Stack + Triage Summary/Solution Doc

**Trigger:** SA ส่ง Triage Summary (Tier 1) หรือ Solution Doc + ADRs (Tier 2/3) + STACK_CONTEXT กลับมา

**บังคับเสมอ — ห้าม proceed ไป STEP 4 ถ้ายังไม่ได้รับผลจาก SA** ไม่ว่า tier ไหน (ดู [CORE_POLICY.md](../CORE_POLICY.md) §1) — ไม่มี soft-warning แล้วเดินหน้าต่ออีกต่อไป

Claude ตรวจ:
- STACK_CONTEXT.md ครบถ้วนไหม
- **STACK_CONTEXT.md version sync (hard gate):** เทียบ version header ของฉบับที่นี่กับฉบับล่าสุดใน SA Project Knowledge — ถ้าไม่ตรงกัน **block** ไม่สร้าง Lead Handoff จนกว่า PO จะ re-sync ไฟล์ (ดู `ai/PROJECT_INSTRUCTIONS.md` §STEP 3)
- Tier 1: มี Triage Summary หรือยัง — Tier 2/3: Solution Doc ครบทุก section (9 sections required + section 10 ถ้ามี UX/UI) หรือยัง
- ADR files ตรงกับที่ reference ใน Solution Doc หรือไม่ (ถ้ามี significant tech decision แต่ไม่มี ADR → PAUSE รอก่อน)
- Security Requirements (ถ้า `Security role: yes` → รอ SEC ส่ง Security_Requirements ก่อน)

### STEP 4 — สร้าง Lead Handoff

Claude generate `LEAD_HANDOFF_[feature].md` อัตโนมัติ

ไฟล์นี้รวม: PRD + Solution Doc + Epics + STACK_CONTEXT summary + DECISION_LOG + PATTERN_LIBRARY + ADRs + Security Requirements

ดาวน์โหลดแล้วส่งให้ Lead

---

## การจัดการ DECISION_LOG

Claude append DECISION_LOG อัตโนมัติหลังทุก confirmed answer

**กฎ:**
- `_TODO.md` — เฉพาะ items ที่ยังไม่ resolve → re-upload ทุก session ใหม่ที่มี TODO คงเหลือ
- `_RESOLVED.md` — archive ที่ resolve แล้ว → upload ครั้งแรก แล้ว re-upload เฉพาะเมื่อมี resolved items เพิ่ม
- ห้ามรวมสองไฟล์เป็นไฟล์เดียว

เมื่อ resolve TODO item → Claude ย้าย entry จาก `_TODO` ไป `_RESOLVED` และ export ทั้งสองไฟล์

---

## PATTERN_LIBRARY — บันทึก Patterns

หลัง STEP 4 เสร็จ Claude จะ detect patterns ที่ใช้ซ้ำได้จาก feature นั้น และถาม PO ว่าต้องการบันทึกไหม

หลัง confirm → Claude append ลง `PATTERN_LIBRARY.md` → ดาวน์โหลด → อัปโหลดกลับ Project

---

## Production Bug Intake

Trigger: ใครก็ได้ (แคชเชียร์, Ops, Dev) แจ้ง PO ว่ามีปัญหาบน production — หรือ PO เลือก option 4 ใน Welcome Dialog

**กฎ:** PO คือจุดรับรายงานเสมอ — Dev ห้ามแจ้ง Lead โดยตรง

### PO ทำทันที (< 5 นาที)

1. สร้าง `BugIntake_BR-[NNN]_[title].md` บันทึกใน `docs/roles/po/` (Option B) หรือ download เก็บไว้ (Option A)
2. แจ้ง Lead ทันที พร้อมส่งไฟล์ BugIntake
3. **รอ Lead ยืนยัน severity** — PO ไม่ตัดสิน severity เอง ไม่สั่ง Dev เอง

| Severity | ความหมาย | สิ่งที่ Lead จะทำ |
|----------|---------|-----------------|
| **P1** | Service down / data loss / security breach | ออก HotfixTask ทันที — ไม่รอ SA |
| **P2** | Functional bug, มี workaround | ออก HotfixTask; SA review async หลัง merge |
| **P3** | Minor bug, ไม่กระทบ user โดยตรง | ใส่ backlog — ใช้ normal pipeline |

### สิ่งที่ PO จะได้รับคืน

หลัง hotfix deploy สู่ Production และ QA smoke test ผ่าน → Lead ส่ง `HotfixNotification_HF-[NNN].md` ให้ PO บันทึกใน `docs/roles/po/` — ใช้เป็น audit trail ว่า bug นั้นได้รับการแก้ไขแล้ว

---

## Re-entry Flows

| สถานการณ์ | Action |
|----------|--------|
| SA ส่ง revised Solution Doc (หลัง PoC FAIL/PARTIAL) | กลับเข้า STEP 3 โดยตรง — ไม่ต้องรัน STEP 1-2 ซ้ำ |
| Lead ส่ง Issue Report → SA แก้ → ส่งกลับ PO | กลับเข้า STEP 3 โดยตรง |

---

## ไฟล์ BugIntake

Claude สร้างให้อัตโนมัติเมื่อ PO เลือก option 4 → รายงาน Bug ใหม่ และตอบคำถาม:

```markdown
# BugIntake_BR-[NNN]_[ชื่อ bug สั้นๆ].md

Date     : [วันที่]
Reporter : PO / [ผู้รายงาน]
Severity : รอ Lead ยืนยัน

## อาการที่พบ
[PO อธิบายจากสิ่งที่ได้รับแจ้ง — ไม่ต้องเป็น technical]

## สาขา / environment ที่พบปัญหา
[Production เท่านั้น? หรือ SIT/Staging ด้วย?]

## ขั้นตอนที่ reproduce ได้
[ถ้าทราบ]

## ผลกระทบ
[กี่สาขา / กี่ user / มี workaround ไหม]
```

---

## Hard Rules

1. **One conversation per feature** — ใช้ thread เดิมตลอดอายุ feature
2. STEP 1 และ STEP 2 ไม่ถูก block โดย STACK_CONTEXT.md ที่ยังไม่มี
3. ห้าม generate Claude Code prompts — นั่นคืองานของ Lead
4. ห้ามทำ task breakdown — PO ทำ Epics เท่านั้น
5. ทุก session ที่เพิ่ม decisions ให้ remind PO download DECISION_LOG ก่อน session ถัดไป
6. STACK_CONTEXT.md ต้องมาจาก SA เท่านั้น — ห้ามถาม PO เรื่อง tech stack

---

## Checklist ก่อนเริ่ม Feature ใหม่

- [ ] `PROJECT_CONTEXT.md` อยู่ใน Project Knowledge แล้ว — ต้องมี `Security role:`, `UX/UI required:` และ `Environment:` (`cli` หรือ `claude.ai`)
- [ ] `STACK_CONTEXT.md` อยู่ใน Project Knowledge แล้ว (ถ้ามีจากโปรเจกต์ก่อน)
- [ ] เปิด conversation thread ใหม่สำหรับ feature นี้โดยเฉพาะ
