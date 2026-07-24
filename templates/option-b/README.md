# Option B Setup — Claude Code (CLI/App/IDE)

คู่มือนี้แก้ปัญหา role isolation ใน Option B ที่ทุก session เห็น knowledge files เดียวกัน

---

## ปัญหาของ Option B (default)

Claude Code ใช้ `CLAUDE.md` ไฟล์เดียวและ filesystem ร่วมกัน — ทุก session เห็น knowledge files ของทุก role พร้อมกัน ทำให้ Claude "รู้มากเกินไป" และมักข้ามขั้นตอน

## วิธีแก้ — แยก knowledge directory ต่อ role + slash commands

---

## ขั้นตอนที่ 1 — โครงสร้าง directory ใน project ของคุณ

```
my-project/
  CLAUDE.md                        ← Dev role instructions (Lead generate)
  ai/                              ← copy จาก sdlc-playbook/ai/
    PROJECT_INSTRUCTIONS.md
    SA_PROJECT_INSTRUCTIONS.md
    LEAD_PROJECT_INSTRUCTIONS.md
    QA_PROJECT_INSTRUCTIONS.md
  .claude/
    commands/                      ← copy จาก sdlc-playbook/templates/option-b/commands/
      po.md                        → /po slash command
      sa.md                        → /sa slash command
      lead.md                      → /lead slash command
      qa.md                        → /qa slash command
  docs/
    roles/
      po/                          ← knowledge files ของ PO เท่านั้น
        PROJECT_CONTEXT.md
        PATTERN_LIBRARY.md
        DECISION_LOG_[feature]_TODO.md
        DECISION_LOG_[feature]_RESOLVED.md
      sa/                          ← knowledge files ของ SA เท่านั้น
        STACK_CONTEXT.md
        SOLUTION_PATTERNS.md
        Solution_Doc_[feature].md
        adr/
          INDEX.md
          ADR-001_[title].md
      lead/                        ← knowledge files ของ Lead เท่านั้น
        LEAD_HANDOFF_[feature].md
      qa/                          ← knowledge files ของ QA เท่านั้น
        [feature]/
          TestCases_[TaskID].md
          TestSuite_[feature].md
          BugReport_[TaskID].md
          TestReport_[feature].md
    shared/                        ← files ที่ทุก role ต้องเห็น
      tasks/
        Task_[ID]_[title].md
      TASK_LOG.md
```

---

## ขั้นตอนที่ 2 — รัน /setup เพื่อสร้างโครงสร้างอัตโนมัติ

copy แค่ไฟล์เดียวก่อน:

```bash
mkdir -p my-project/.claude/commands
cp sdlc-playbook/templates/option-b/commands/setup.md my-project/.claude/commands/setup.md
```

จากนั้นเปิด Claude Code ใน `my-project/` แล้วพิมพ์:

```
/setup
```

Claude จะ:
1. หา sdlc-playbook path อัตโนมัติ (หรือถามถ้าไม่พบ)
2. สร้าง directory structure ทั้งหมด (`docs/roles/`, `docs/shared/`)
3. Copy `ai/` folder และ slash commands ทุกไฟล์
4. สร้าง `CLAUDE.md` (Dev role) และ `TASK_LOG.md` เริ่มต้น

> **Manual setup (ทางเลือก):** ถ้าต้องการ copy ด้วยมือ ดูคำสั่ง bash ใน [Setup Script](#manual-setup)

---

## ขั้นตอนที่ 3 — เริ่ม session ตาม role

เปิด Claude Code ใน `my-project/` แล้วพิมพ์ slash command ตาม role ที่ต้องการ:

| ต้องการทำอะไร | พิมพ์ |
|---|---|
| วิเคราะห์ PRD, สร้าง handoff | `/po` |
| ออกแบบ solution, สร้าง ADR | `/sa` |
| แตก task, สร้าง task prompts | `/lead` |
| implement task (default) | ไม่ต้องพิมพ์ command — Claude อ่าน CLAUDE.md อัตโนมัติ |
| เขียน/รัน test cases | `/qa` |

---

## Isolation ที่ได้

| Session | อ่านจาก | บันทึกไปที่ |
|---|---|---|
| `/po` | `docs/roles/po/` | `docs/roles/po/` |
| `/sa` | `docs/roles/sa/` | `docs/roles/sa/` |
| `/lead` | `docs/roles/lead/` + `docs/shared/` | `docs/roles/lead/`, `docs/shared/tasks/` |
| Dev (CLAUDE.md) | `CLAUDE.md`, `docs/shared/` | code, `docs/shared/TASK_LOG.md` |
| `/qa` | `docs/roles/qa/` + `docs/shared/` | `docs/roles/qa/` |

Claude จะโฟกัสเฉพาะ directory ของ role ตัวเอง — ไม่ได้ blocked จากไฟล์อื่น แต่ไม่มีเหตุผลจะอ่าน

---

## STACK_CONTEXT.md — special case

ไฟล์นี้ต้องมีในหลาย role:

| Role | Path |
|---|---|
| SA (เจ้าของ/สร้าง) | `docs/roles/sa/STACK_CONTEXT.md` |
| PO (ใช้ตรวจ stack) | `docs/roles/po/STACK_CONTEXT.md` |
| Lead (embedded ใน LEAD_HANDOFF) | embed อยู่ใน `docs/roles/lead/LEAD_HANDOFF_[feature].md` |
| Dev (embedded ใน CLAUDE.md) | Lead extract แล้วใส่ใน `CLAUDE.md` |

SA สร้างเสร็จแล้ว copy ไปวางที่ `docs/roles/po/` ด้วย

---

## Manual Setup

ถ้าไม่ต้องการใช้ `/setup` สามารถ copy ด้วยมือ:

```bash
# copy role instruction files
cp -r sdlc-playbook/ai/ my-project/ai/

# copy slash command templates (รวม setup.md)
mkdir -p my-project/.claude/commands
cp sdlc-playbook/templates/option-b/commands/*.md my-project/.claude/commands/

# สร้าง knowledge directories
mkdir -p my-project/docs/roles/po
mkdir -p my-project/docs/roles/sa/adr
mkdir -p my-project/docs/roles/lead
mkdir -p my-project/docs/roles/qa
mkdir -p my-project/docs/shared/tasks
```

---

## Version header

ทุก knowledge file ต้องมี header บรรทัดแรก:

```
# Last updated: YYYY-MM-DD | Version: N
```
