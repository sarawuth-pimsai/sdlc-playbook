# คู่มือ Solution Architect (SA)

> ดูภาพรวม workflow และการ setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

SA รับผิดชอบวิเคราะห์ PRD ด้านเทคนิค ออกแบบ Solution และเป็นเจ้าของ `STACK_CONTEXT.md`

---

## การติดตั้ง

### Option A — claude.ai Projects

ใช้ผ่าน claude.ai web interface

#### ขั้นตอนที่ 1 — สร้าง Project

1. เปิด [claude.ai](https://claude.ai) → **Projects** → **New project**
2. ตั้งชื่อ เช่น `[ชื่อโปรเจกต์] — SA`
3. เปิด **Project Instructions** → Copy เนื้อหาจาก `ai/SA_PROJECT_INSTRUCTIONS.md` → Paste → บันทึก

#### ขั้นตอนที่ 2 — อัปโหลด Project Knowledge

| ไฟล์ | เมื่อไหร่ | หมายเหตุ |
|------|----------|---------|
| `STACK_CONTEXT.md` | ✅ ก่อน session แรก | SA สร้างและเป็นเจ้าของ |
| `SOLUTION_PATTERNS.md` | ✅ ก่อน session แรก | SA สร้างและ maintain เอง |
| `DECISION_LOG_[feature]_TODO.md` | เมื่อได้รับจาก PO | ส่งมาพร้อม SA Handoff |
| `DECISION_LOG_[feature]_RESOLVED.md` | เมื่อได้รับจาก PO | ส่งมาพร้อม SA Handoff |
| `PATTERN_LIBRARY.md` | เมื่อได้รับจาก PO | ส่งมาพร้อม SA Handoff |

---

### Option B — Claude Code

ใช้ผ่าน Claude Code CLI, Desktop App, หรือ IDE Extension แทน claude.ai Projects

#### ขั้นตอนที่ 1 — Setup workspace (ทำครั้งเดียว)

ดูรายละเอียดการ setup ที่ [templates/option-b/README.md](../../templates/option-b/README.md) — ใช้ `/setup` command เพื่อสร้าง directory structure และ copy ไฟล์ที่จำเป็นทั้งหมดอัตโนมัติ

#### ขั้นตอนที่ 2 — เริ่ม SA session

```
/sa
```

พิมพ์ `/sa` ใน Claude Code — Claude จะอ่าน `ai/SA_PROJECT_INSTRUCTIONS.md` และโฟกัสที่ `docs/roles/sa/`

#### ขั้นตอนที่ 3 — วางไฟล์ใน directory ของ SA

วางไฟล์ knowledge ใน `docs/roles/sa/` แทนการ upload เข้า Project Knowledge

---

## Input ที่รับจาก PO

### กรณี A — Stack Setup Request (feature แรก หรือ stack ยังไม่มี)

PO ส่งไฟล์ `SA_STACK_SETUP_REQUEST_[ProjectName].md`

**Action:**
1. เปิดไฟล์ — มี STACK_CONTEXT.md template + PRD context แนบมา
2. กรอกทุก field ใน template ตามการตัดสินใจ tech stack จริงของทีม
3. บันทึกเป็น `STACK_CONTEXT.md`
4. อัปโหลดเข้า SA Project Knowledge
5. ส่งไฟล์กลับให้ PO → PO อัปโหลดเข้า PO Project

### กรณี B — SA Handoff (มี STACK_CONTEXT.md แล้ว)

PO ส่งไฟล์ `SA_HANDOFF_[feature].md`

**ต้องมาพร้อม:**
- PRD content (embed ใน handoff)
- `DECISION_LOG_[feature]_TODO.md`
- `DECISION_LOG_[feature]_RESOLVED.md` (ถ้ามี)
- `PATTERN_LIBRARY.md` (ถ้ามี)

---

## Workflow ของ SA

### STEP 1 — อ่าน Context (silent)

Claude อ่านไฟล์ทั้งหมดก่อน แล้ว extract จาก SA Handoff:
- PRD content
- Epics จาก PO STEP 2
- Open TODO items ที่กระทบ architectural decisions

### STEP 2 — วิเคราะห์ PRD ด้านเทคนิค

- Summarise feature (2-3 ประโยค มุมมองเทคนิค)
- ระบุ data flows, integration points, external dependencies, constraints
- List technical risks (HIGH/MED/LOW)

ถ้าต้องการ clarification จาก PO → Claude สร้าง **HTML Artifact dialog**

### STEP 3 — เสนอ Solution Options

Claude เสนอ 2-3 options พร้อม pros/cons/complexity

**กฎ:** Claude ไม่เลือก option เอง — SA เป็นคนตัดสินใจ

### STEP 4 — Draft Solution Doc

หลัง SA เลือก option → Claude draft `Solution_Doc_[feature].md` (9 sections required)

**9 sections ที่ต้องครบ:**
1. Overview
2. Architecture
3. Tech decisions
4. API / Interface design
5. Data model changes
6. Non-functional considerations (รวม security checklist Option B)
7. Risks and mitigations
8. Open questions
9. PoC scope (ถ้าจำเป็น)

SA review draft ใน chat → confirm → Claude create HTML Artifact พร้อม Download button

### STEP 5 — Draft ADRs

สำหรับทุก tech decision ที่ "significant":
- เลือก technology ที่ต่างจาก STACK_CONTEXT.md defaults
- เปลี่ยน data model ที่กระทบมากกว่า 1 service
- Trade-off ที่ต้องเก็บ rationale ไว้
- Decision ที่ Lead ไม่สามารถ infer จาก code ได้เอง

**ADR numbering:** ใช้ global index `docs/adr/INDEX.md` — SA จอง number ก่อน draft

### STEP 6 — PoC Planning (ถ้ามี PoC scope ใน STEP 4)

Claude generate PoC prompts สำหรับ Lead ไปรัน spike

**ผล PoC:**
- **PASS** → mark assumption "validated" → ไปต่อ STEP 7
- **FAIL** → กลับ STEP 3 พร้อม evidence → ถ้ากระทบ PRD requirement → notify PO ก่อน
- **PARTIAL** → SA ตัดสินใจว่า partial result เพียงพอหรือไม่

### STEP 7 — Handoff Package

Claude compile handoff summary พร้อม distribution plan

**ส่งทุก artifact ให้ PO** (ไม่ส่งตรงให้ Lead):
- `Solution_Doc_[feature].md`
- `ADR_[NNN]_[title].md` (x N)
- PoC prompts (ถ้ามี)
- `STACK_CONTEXT.md` (ถ้า PO ส่ง Stack Setup Request มาด้วย)
- `SOLUTION_PATTERNS.md` (ถ้า update session นี้)

---

## STACK_CONTEXT.md — กฎการ Maintain

- SA **เป็นเจ้าของแต่เพียงผู้เดียว** — ห้าม PO แก้ไข
- เมื่อ stack เปลี่ยน → SA อัปเดตใน SA Project → increment version → notify PO ให้ re-upload เข้า PO Project
- ทุกครั้งที่ส่งไฟล์ให้ PO ระบุ version ใน message: เช่น `STACK_CONTEXT.md Version 2`

---

## Lead Issue Report (Lead → SA)

Lead ส่ง Issue Report ตรงถึง SA ได้ (ไม่ต้องผ่าน PO) สำหรับ technical clarification ที่ไม่กระทบ PRD

**SA วินิจฉัย impact:**
- **Small** → SA ออก ADR Amendment + update Solution Doc → ส่งให้ Lead ตรงได้
- **Large** (กระทบ PRD scope / data model ร่วม) → SA notify PO ก่อน → PO approve → SA revise → ส่ง PO → PO re-enters STEP 3

---

## SOLUTION_PATTERNS.md — Maintain

หลัง PO accept feature → SA review ว่ามี architectural pattern ที่ควรเก็บไว้ใช้ซ้ำหรือไม่

Claude ถาม: "Feature นี้มี [pattern] — บันทึกลง SOLUTION_PATTERNS.md ไหม?"

SA confirm → Claude append → export → SA อัปโหลดกลับเข้า SA Project

---

## Checklist ก่อนส่ง SA Handoff ให้ PO

- [ ] Solution Doc ครบทุก 9 sections (ไม่มี section ที่เป็นแค่ "TBD")
- [ ] ADR files ครบตาม reference ใน Solution Doc
- [ ] ADR numbers sequential ใน `docs/adr/INDEX.md`
- [ ] PoC prompts พร้อม (ถ้ามี PoC scope)
- [ ] `STACK_CONTEXT.md` version ระบุใน message
- [ ] ส่งทุก artifact ให้ PO — ไม่ส่งตรงให้ Lead
