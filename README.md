# SDLC Playbook — AI-Assisted Development Workflow

ชุด Claude Project Instructions สำหรับทีมพัฒนา software ที่ใช้ AI เป็นผู้ช่วยในแต่ละ role ตลอด SDLC

ไฟล์ใน `ai/` คือ **system prompt** สำหรับแต่ละ role — ใช้ได้ 2 วิธี:

- **Option A** — อัปโหลดเป็น Claude Project Instructions บน claude.ai (web interface)
- **Option B** — วางเป็น `CLAUDE.md` แล้วรัน Claude Code (CLI/App/IDE Extension)

ไม่มี application ที่รันได้ใน repo นี้

---

## ภาพรวม

> 📊 ดู Workflow Diagrams แบบครบถ้วน → [docs/WORKFLOW_OVERVIEW.md](docs/WORKFLOW_OVERVIEW.md)



```
PO (hub) ──────► SA      SA Handoff → Solution Doc + ADRs + PoC prompts
         ──────► SEC     Solution Doc → Security Requirements  (Option A เท่านั้น)
         ──────► Lead    Lead Handoff = PRD + Solution Doc + Epics + Stack

Lead ───────────► Dev    Task_[ID].md prompts ทีละไฟล์
Dev ────────────► QA     Deploy Notification หลัง deploy SIT/Staging
Lead ───────────► QA     PRD + DECISION_LOG + task files (Phase A setup)
SEC ────────────► Lead   Security_Review_[TaskID].md ต่อ PR  (Option A เท่านั้น)
```

**PO เป็น hub กลาง** — ทุก artifact ผ่าน PO ก่อนกระจายต่อให้ roles อื่น

---

## โครงสร้าง Repository

```
ai/
  PROJECT_INSTRUCTIONS.md        → PO Claude Project (Option A) / /po command (Option B)
  SA_PROJECT_INSTRUCTIONS.md     → SA Claude Project / /sa command
  LEAD_PROJECT_INSTRUCTIONS.md   → Lead Claude Project / /lead command
  DEV_PROJECT_INSTRUCTIONS.md    → Developer (CLAUDE.md / Claude Code)
  QA_PROJECT_INSTRUCTIONS.md     → QA Claude Project / /qa command
  SEC_PROJECT_INSTRUCTIONS.md    → Security Engineer Claude Project (Option A only)
docs/
  guides/
    PO_GUIDE.md       → คู่มือ Product Owner
    SA_GUIDE.md       → คู่มือ Solution Architect
    LEAD_GUIDE.md     → คู่มือ Tech Lead
    DEV_GUIDE.md      → คู่มือ Developer
    QA_GUIDE.md       → คู่มือ QA Engineer
    SEC_GUIDE.md      → คู่มือ Security Engineer
    SOLO_GUIDE.md     → คู่มือ Solo Developer (ทำงานคนเดียว หลาย role)
templates/
  option-b/
    README.md         → setup guide สำหรับ Option B (role isolation)
    commands/
      setup.md        → slash command template สำหรับ /setup (สร้าง directory structure อัตโนมัติ)
      po.md           → slash command template สำหรับ /po
      sa.md           → slash command template สำหรับ /sa
      lead.md         → slash command template สำหรับ /lead
      qa.md           → slash command template สำหรับ /qa
```

---

## Security Options

| Option       | เงื่อนไข                | ผล                                                              |
| ------------ | ----------------------- | --------------------------------------------------------------- |
| **Option A** | ทีมมี Security Engineer | SEC Project ใช้งาน, SEC review Solution Doc + ทุก PR            |
| **Option B** | ไม่มี Security Engineer | Security checkpoints ฝังอยู่ใน SA / Lead / QA instructions แล้ว |

PO ตั้งค่านี้ใน `PROJECT_CONTEXT.md` (`Security role: yes / no`) ครั้งเดียวตอนเริ่มโปรเจกต์

---

## Option B — Claude Code (CLI/App/IDE Extension)

Option B ให้แต่ละ role ใช้ slash command แทนการ copy prompt เข้า claude.ai Project

### โครงสร้าง directory ที่แนะนำ (role isolation)

```
my-project/
  CLAUDE.md                    ← Dev instructions (Lead generate)
  ai/                          ← copy จาก sdlc-playbook/ai/
  .claude/
    commands/                  ← copy จาก sdlc-playbook/templates/option-b/commands/
      setup.md → /setup  ← รันครั้งแรกเพื่อสร้าง structure ทั้งหมดอัตโนมัติ
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
    shared/                    ← ทุก role เห็น (tasks/, TASK_LOG.md)
```

ดูรายละเอียดการ setup ที่ [templates/option-b/README.md](templates/option-b/README.md)

---

## Solo Developer

ถ้าทำงานคนเดียวและต้องสวมหมวกหลาย role ดูที่ [docs/guides/SOLO_GUIDE.md](docs/guides/SOLO_GUIDE.md)

---

## การติดตั้งครั้งแรก (First-time Setup — Option A)

> ส่วนนี้อธิบาย **Option A** (claude.ai Projects) สำหรับ **Option B** (Claude Code) ดูที่ [templates/option-b/README.md](templates/option-b/README.md)

### ขั้นตอนที่ 1 — สร้าง Claude Projects บน claude.ai

สร้าง Claude Project แยกต่อ role ตามตาราง:

| Project name (แนะนำ) | Instruction file                  | ใครใช้                       |
| -------------------- | --------------------------------- | ---------------------------- |
| `[Project] — PO`     | `ai/PROJECT_INSTRUCTIONS.md`      | Product Owner                |
| `[Project] — SA`     | `ai/SA_PROJECT_INSTRUCTIONS.md`   | Solution Architect           |
| `[Project] — Lead`   | `ai/LEAD_PROJECT_INSTRUCTIONS.md` | Tech Lead                    |
| `[Project] — Dev`    | `ai/DEV_PROJECT_INSTRUCTIONS.md`  | Developer (ดูหมายเหตุ)       |
| `[Project] — QA`     | `ai/QA_PROJECT_INSTRUCTIONS.md`   | QA Engineer                  |
| `[Project] — SEC`    | `ai/SEC_PROJECT_INSTRUCTIONS.md`  | Security Engineer (Option A) |

> **Developer:** `DEV_PROJECT_INSTRUCTIONS.md` ใช้กับ **Claude Code** (CLI/App) ไม่ใช่ claude.ai Project
> ดูรายละเอียดใน [DEV_GUIDE.md](docs/guides/DEV_GUIDE.md)

### ขั้นตอนที่ 2 — อัปโหลด Instruction File เข้า Project

สำหรับแต่ละ Claude Project:

1. เปิด Project Settings → **Project Instructions**
2. Copy เนื้อหาจาก instruction file → วางลงในช่อง Instructions
3. บันทึก

### ขั้นตอนที่ 3 — สร้าง PROJECT_CONTEXT.md

สร้างไฟล์ `PROJECT_CONTEXT.md` แล้วอัปโหลดเข้า **PO Project Knowledge**:

```markdown
# PROJECT_CONTEXT.md

# Last updated: YYYY-MM-DD | Version: 1

## Project settings

Security role: yes / no
Environment: cli / claude.ai
```

- `Security role: yes` → Option A: SEC Engineer ใช้งานใน workflow
- `Security role: no` → Option B: security checkpoints ฝังใน roles อื่นแล้ว
- `Environment: cli` → Claude ใช้ Write tool save ไฟล์ลง disk ทันทีทุกครั้งที่ generate artifact
- `Environment: claude.ai` → Claude สร้าง Artifact พร้อม Download button

### ขั้นตอนที่ 4 — SA สร้าง STACK_CONTEXT.md

PO ส่ง SA Stack Setup Request ให้ SA กรอก `STACK_CONTEXT.md`
(Claude ใน PO Project จะ generate template ให้อัตโนมัติเมื่อเริ่ม feature แรก)

เมื่อได้รับ `STACK_CONTEXT.md` จาก SA → อัปโหลดเข้า:

- **PO Project Knowledge**
- **SA Project Knowledge**

---

## SDLC Flow ภาพรวม

```
┌─────────────────────────────────────────────────────────────────────┐
│  1. PO เริ่ม session ใหม่                                           │
│     Welcome Wizard → ตรวจสอบไฟล์ → STEP 1 (วิเคราะห์ PRD)         │
│     → STEP 1.5 (ชี้แจง) → STEP 2 (Epics)                           │
│     → สร้าง SA Handoff อัตโนมัติ                                    │
├─────────────────────────────────────────────────────────────────────┤
│  2. SA รับ SA Handoff                                               │
│     วิเคราะห์ PRD → เสนอ options → draft Solution Doc              │
│     → ADRs → PoC (ถ้าจำเป็น) → ส่งทุก artifact กลับ PO            │
├─────────────────────────────────────────────────────────────────────┤
│  3. SEC รับ Solution Doc (Option A เท่านั้น)                        │
│     Phase A review → Security_Requirements → ส่ง PO               │
├─────────────────────────────────────────────────────────────────────┤
│  4. PO รัน STEP 3 + STEP 4                                          │
│     ตรวจ stack + Solution Doc → สร้าง Lead Handoff                  │
├─────────────────────────────────────────────────────────────────────┤
│  5. Lead รับ Lead Handoff                                           │
│     L-STEP 1-4: cross-check → task breakdown → Claude Code prompts  │
│     → CLAUDE.md → ส่ง task prompts ให้ Dev + setup files ให้ QA    │
├─────────────────────────────────────────────────────────────────────┤
│  6. Dev implement (ทีละ task)                                       │
│     Paste Task_[ID].md ใน Claude Code → implement → run done criteria │
│     → update TASK_LOG → deploy SIT → notify QA                     │
├─────────────────────────────────────────────────────────────────────┤
│  7. QA test                                                          │
│     Phase 0: draft test cases จาก PRD (parallel กับ SA)            │
│     Phase A: SIT test ต่อ task → TaskTestSummary → Dev fix bugs     │
│     Phase B: Staging full suite → TestReport → Lead sign-off        │
│     Phase C: Production smoke test → rollback ถ้า P1 fail           │
└─────────────────────────────────────────────────────────────────────┘
```

---

## กฎการจัดการไฟล์

### Version Header

ทุก shared file ต้องมี header บรรทัดแรก:

```
# Last updated: YYYY-MM-DD | Version: N
```

### DECISION_LOG — สองไฟล์ต่อ feature

| ไฟล์                                 | เนื้อหา                       | Re-upload                            |
| ------------------------------------ | ----------------------------- | ------------------------------------ |
| `DECISION_LOG_[feature]_TODO.md`     | เฉพาะ items ที่ยังไม่ resolve | ทุก session ใหม่ที่มี TODO คงเหลือ   |
| `DECISION_LOG_[feature]_RESOLVED.md` | Archive ที่ resolve แล้ว      | เมื่อมี resolved items เพิ่มเท่านั้น |

ห้ามรวมสองไฟล์เป็นไฟล์เดียว

### One thread per feature (Hard Rule)

ใช้ conversation thread เดียวต่อหนึ่ง feature ตลอดอายุของ feature นั้น
เปิด conversation ใหม่เฉพาะเมื่อเริ่ม feature ใหม่เท่านั้น

### ไฟล์ที่เก็บในโปรเจกต์จริง (code repo)

> ⚠️ **ข้อควรระวัง:** `STACK_CONTEXT.md` ในโปรเจกต์จริงจะมีข้อมูล tech stack ภายในองค์กร เช่น IdP endpoint, infrastructure details, internal service names — **ห้าม commit ไฟล์นี้ลง public repository** เพิ่ม `docs/STACK_CONTEXT.md` เข้า `.gitignore` หรือใช้ private repo เท่านั้น

```
my-project/
  CLAUDE.md               ← Lead สร้าง, commit ก่อน Dev เริ่ม task แรก
  docs/
    STACK_CONTEXT.md      ← ห้าม commit ลง public repo (ข้อมูล internal stack)
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

## คู่มือแต่ละ Role

| Role               | คู่มือ                                                 |
| ------------------ | ------------------------------------------------------ |
| **ภาพรวม Workflow** | [docs/WORKFLOW_OVERVIEW.md](docs/WORKFLOW_OVERVIEW.md) |
| Product Owner      | [docs/guides/PO_GUIDE.md](docs/guides/PO_GUIDE.md)     |
| Solution Architect | [docs/guides/SA_GUIDE.md](docs/guides/SA_GUIDE.md)     |
| Tech Lead          | [docs/guides/LEAD_GUIDE.md](docs/guides/LEAD_GUIDE.md) |
| Developer          | [docs/guides/DEV_GUIDE.md](docs/guides/DEV_GUIDE.md)   |
| QA Engineer        | [docs/guides/QA_GUIDE.md](docs/guides/QA_GUIDE.md)     |
| Security Engineer  | [docs/guides/SEC_GUIDE.md](docs/guides/SEC_GUIDE.md)   |
| Solo Developer     | [docs/guides/SOLO_GUIDE.md](docs/guides/SOLO_GUIDE.md) |

---

## Shared Files — เจ้าของและ Flow

| ไฟล์                                 | เจ้าของ | ไหลไปถึง                  |
| ------------------------------------ | ------- | ------------------------- |
| `STACK_CONTEXT.md`                   | SA      | PO → Lead → Dev / QA      |
| `DECISION_LOG_[feature]_TODO.md`     | PO      | SA, Lead, QA              |
| `DECISION_LOG_[feature]_RESOLVED.md` | PO      | SA, Lead, QA              |
| `PATTERN_LIBRARY.md`                 | PO      | SA, Lead                  |
| `SOLUTION_PATTERNS.md`               | SA      | SA-internal               |
| `PROJECT_CONTEXT.md`                 | PO      | PO-internal               |
| `Solution_Doc_[feature].md`          | SA      | PO → Lead, SEC            |
| `Security_Requirements_[feature].md` | SEC     | PO → Lead                 |
| `LEAD_HANDOFF_[feature].md`          | PO      | Lead                      |
| `Task_[ID]_[title].md`               | Lead    | Dev, QA                   |
| `TASK_LOG.md`                        | Dev     | Lead (อ่านก่อน PR review) |

## License

This software is licensed under the **Business Source License 1.1 (BSL 1.1)**.

### 🛑 Key Terms & Restrictions

- **Allowed Uses (Free Internal Production):** องค์กร บริษัท หรือบุคคลทั่วไป **สามารถนำซอฟต์แวร์นี้ไปติดตั้งและใช้งานจริงในระบบ Production ภายในองค์กรได้ฟรี** โดยไม่มีค่าใช้จ่าย
- **Commercial & SaaS Restrictions (ข้อห้าม):**
  - ❌ **ห้าม** นำซอฟต์แวร์นี้ไปเปิดให้บริการในรูปแบบ Managed Service หรือ **Software-as-a-Service (SaaS)** แก่บุคคลภายนอก
  - ❌ **ห้าม** นำซอฟต์แวร์นี้ไปรวม (Embed) หรือซ้อนไว้ในผลิตภัณฑ์ทางการค้าอื่น ๆ เพื่อนำไปขายต่อ (Resell/Sublicense) ให้กับลูกค้าของคุณ
- **Automatic Open Source Conversion:** ในวันที่ **July 17, 2029** ซอฟต์แวร์เวอร์ชันนี้จะเปลี่ยนสถานะเป็น Open Source ภายใต้ **GPL v3.0** โดยอัตโนมัติ และจะปลดล็อกข้อจำกัดทางการค้าทั้งหมด

### 💼 Commercial Partnerships & Special Licensing

หากบริษัทของคุณต้องการนำโปรเจกต์นี้ไปทำระบบ SaaS บริการลูกค้า หรือรวมเข้ากับซอฟต์แวร์ปิดเพื่อจำหน่ายต่อ กรุณาติดต่อ **Sarawuth Pimsai** โดยตรงผ่าน [GitHub Issues](https://github.com/sarawuth-pimsai/sdlc-playbook/issues) เพื่อขอซื้อสิทธิ์การใช้งานเชิงพาณิชย์ (Commercial License)
