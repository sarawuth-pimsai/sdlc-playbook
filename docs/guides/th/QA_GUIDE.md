# คู่มือ QA Engineer

> ดูภาพรวม workflow และการ setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

QA ทำงาน 4 phases โดย Phase 0 เริ่มได้เลย parallel กับ SA — ไม่ต้องรอ Lead หรือ STACK_CONTEXT.md

---

## การติดตั้ง

### Option A — claude.ai Projects

ใช้ผ่าน claude.ai web interface

#### ขั้นตอนที่ 1 — สร้าง Project

1. เปิด [claude.ai](https://claude.ai) → **Projects** → **New project**
2. ตั้งชื่อ เช่น `[ชื่อโปรเจกต์] — QA`
3. เปิด **Project Instructions** → Copy เนื้อหาจาก `ai/QA_PROJECT_INSTRUCTIONS.md` → Paste → บันทึก

#### ขั้นตอนที่ 2 — อัปโหลด Project Knowledge ตาม Phase

**Phase 0** (ได้รับจาก PO หลัง STEP 2):
| ไฟล์ | จำเป็น |
|------|--------|
| `PRD_[feature].md` | ✅ |
| `DECISION_LOG_[feature]_TODO.md` | ✅ |
| `DECISION_LOG_[feature]_RESOLVED.md` | ถ้ามี |

**Phase A** (ได้รับจาก Lead ก่อน SIT test):
| ไฟล์ | จำเป็น |
|------|--------|
| `STACK_CONTEXT.md` | ✅ |
| `Task_[ID]_[title].md` | ✅ ทุก task |
| PRD + DECISION_LOG files | ✅ (อัปเดตจาก Phase 0 ถ้ามีการเปลี่ยนแปลง) |

---

### Option B — Claude Code

ใช้ผ่าน Claude Code CLI, Desktop App, หรือ IDE Extension แทน claude.ai Projects

#### ขั้นตอนที่ 1 — Setup workspace (ทำครั้งเดียว)

ดูรายละเอียดการ setup ที่ [templates/option-b/README.md](../../templates/option-b/README.md) — ใช้ `/setup` command เพื่อสร้าง directory structure และ copy ไฟล์ที่จำเป็นทั้งหมดอัตโนมัติ

#### ขั้นตอนที่ 2 — เริ่ม QA session

```
/qa
```

พิมพ์ `/qa` ใน Claude Code — Claude จะอ่าน `ai/QA_PROJECT_INSTRUCTIONS.md` และโฟกัสที่ `docs/roles/qa/` และ `docs/shared/`

#### ขั้นตอนที่ 3 — วางไฟล์ใน directory ของ QA

วางไฟล์ knowledge ใน `docs/roles/qa/` แทนการ upload เข้า Project Knowledge  
เพิ่มไฟล์ใหม่เมื่อขึ้น phase ใหม่ (Phase 0 → Phase A)

---

## Phase 0 — Early Preparation (parallel กับ SA design)

**Trigger:** PO ส่ง PRD + DECISION_LOG มาให้ QA หลัง STEP 2 เสร็จ

**ทำได้โดยไม่ต้องรอ:** Lead, STACK_CONTEXT.md, task prompts

### P-STEP 1 — Draft Test Cases จาก PRD

- User stories → happy path test cases
- Business rules → edge case scenarios
- Error handling section → error path test cases
- NFR section → note performance/load scenarios สำหรับ Phase B

Mark ทุก test case เป็น `[DRAFT — pending task breakdown]`

### P-STEP 2 — Test Data Requirements

Claude generate test data request ส่งให้ Lead ก่อน Phase A:

```markdown
# Test Data Requirements — [Feature name]
## User accounts needed
## Sample data records
## External system mocks (ถ้ามี integrations)
## Environment access
- SIT URL, Staging URL
- Credentials (ขอจาก Lead)
```

Lead provision test data ก่อน Phase A เริ่ม

---

## Phase A — SIT Testing (ทีละ task)

**Trigger:** Dev ส่ง Deploy Notification หลัง deploy task ถึง SIT

> ห้ามเริ่ม test ก่อนได้รับ Deploy Notification จาก Dev

### A-STEP 1 — อ่าน Task Prompt + PRD

Extract:
- Done criteria จาก task prompt (minimum pass bar)
- User stories ที่เกี่ยวข้องจาก PRD
- Error codes และ edge cases จาก PRD

### A-STEP 2 — Generate Test Cases

Claude สร้าง test case table:
- Happy path
- Edge cases
- Error cases
- Security cases (ถ้า applicable)

Preview → QA review → confirm → export `TestCases_[TaskID].md`

### A-STEP 3 — Generate Claude Code Test Prompt

Claude สร้าง prompt สำหรับ QA paste ใน Claude Code เพื่อ write + run automated tests ใน SIT environment

Export `Prompt_SIT_[TaskID].md`

### A-STEP 4 — Bug Report (ถ้า test fail)

Claude draft bug entries แต่ละรายการ:

```markdown
## BUG-[NNN] — [Short description]
Task: [Task ID] | Severity: Critical/High/Medium/Low | Status: Open

### Steps to reproduce
### Expected result
### Actual result
### Evidence (log/screenshot)
### Possible cause
```

Export `BugReport_[TaskID].md`

### A-STEP 5 — Task Test Summary

Claude สรุป metrics + verdict → React Artifact พร้อม Download → ส่ง Lead + Dev

**Verdict routing:**
| Verdict | ส่งให้ | เพื่อ |
|---------|--------|-------|
| PASS | Lead | Unblock task ถัดไป |
| FAIL | Lead + Dev | Dev แก้ bug ก่อน re-test |
| BLOCKED | Lead เท่านั้น | Lead ตัดสินใจ escalate |

Dev แก้ bug → QA re-test เฉพาะ failed cases เท่านั้น

---

## Phase B — Staging Full Test Suite

**Trigger:** Dev deploy ทุก task ถึง Staging แล้ว

### B-STEP 0 — Regression Scope Check

ตรวจว่า feature นี้กระทบ existing functionality อะไรบ้าง

**Default regression scope (ถ้าไม่มี Critical paths ใน STACK_CONTEXT):**
- Authentication / login flow (ถ้า auth middleware ถูกแก้)
- ทุก endpoint ที่ feature ใหม่เรียกหรือ depends on
- ทุก DB table ที่ถูก modified หรือ extended

> ถ้า regression case fail ใน Phase B → **STOP ทันที** แจ้ง Lead ก่อน — ห้าม continue Phase B

### B-STEP 1 — Compile Full Test Suite

Merge `TestCases_[TaskID].md` ทุกไฟล์เป็น master test suite

รวม **Security test cases** เสมอ (Option B — ถ้าไม่มี SEC Engineer):
- Access endpoint โดยไม่มี auth token → HTTP 401
- Access endpoint ด้วย expired token → HTTP 401
- SQL injection payload ใน text fields → HTTP 400 หรือ sanitised response
- Response body ไม่รวม sensitive fields (password, token, PII)
- + feature-specific security cases จาก PRD NFR

Export `TestSuite_[Feature].md`

### B-STEP 2 — Generate Claude Code Automated Test Prompt

Claude สร้าง prompt สำหรับรัน full automated test suite ใน Staging

Export `Prompt_Staging_[Feature].md`

### B-STEP 3 — Test Report

Claude draft report พร้อม verdict:
- **APPROVED FOR PRODUCTION**
- **FAILED — DO NOT DEPLOY**
- **CONDITIONAL** (list conditions)

React Artifact พร้อม Download → QA + Lead sign-off → **Lead แจ้ง PO** (ไม่ใช่ QA แจ้งโดยตรง)

---

## Phase C — Production Deployment

**Trigger:** Lead + QA sign-off บน TestReport ด้วย verdict APPROVED

### C-STEP 2 — Post-deployment Smoke Test

รัน P1 test cases เท่านั้นใน production environment

- ทุก P1 ต้องผ่านภายใน timeout ที่กำหนด (default: 15 นาที)
- ถ้า P1 fail → **trigger rollback ทันที** — diagnose หลัง rollback ไม่ใช่ก่อน

### Rollback Triggers

| เงื่อนไข | Action |
|---------|--------|
| P1 smoke test fail | Lead rollback ทันที |
| Deployment hang > timeout | Lead rollback |
| Critical error ใน production logs ภายใน 15 นาที | Lead + QA assess → rollback ถ้า user-facing |

---

## Phase HF — Hotfix Smoke Test (Staging → Production)

Trigger: Lead แจ้ง QA ว่า hotfix deploy สู่ Staging แล้ว พร้อมระบุ HF-ID และ scope ที่ต้องทดสอบ

**เวลาที่ให้:** P1 = 30 นาที / P2 = 2 ชั่วโมง — QA ต้อง prioritize ทันที

### HF-STEP 1 — รับ scope จาก Lead

Lead แจ้งมาพร้อม HotfixTask:
- Bug ที่ fix คืออะไร (อ้างอิง BugIntake BR-[NNN])
- Critical paths ที่ต้องตรวจ

ถ้า Lead ไม่ระบุ critical paths → ใช้ Default regression scope จาก STACK_CONTEXT หรือ TestSuite ที่มีอยู่

### HF-STEP 2 — Run smoke test บน Staging

Run critical paths เท่านั้น — **ไม่ run full test suite**:
- Flow ที่ fix ตรงๆ (จาก Bug Intake)
- Critical paths ที่ fix อาจกระทบ

บันทึกผลเป็น `HotfixSmokeTest_HF-[NNN].md`

### HF-STEP 3 — รายงานผลให้ Lead (Staging)

| ผล | ทำอะไร |
|---|---|
| ทุก TC PASS | แจ้ง Lead: "Hotfix HF-[NNN] smoke test ผ่านบน Staging ครับ" พร้อมแนบไฟล์ — Lead ดำเนิน deploy สู่ Production |
| มี TC FAIL | แจ้ง Lead ทันที — Lead fix และ re-deploy Staging ก่อน — **ห้าม deploy Production จนกว่าจะผ่าน** |

### HF-STEP 4 — Production smoke check (หลัง Lead deploy สู่ Production)

Trigger: Lead แจ้ง QA ว่า deploy สู่ Production แล้ว

Run **P1 test cases เท่านั้น** — ใช้ชุดเดิมจาก HF-STEP 2

**เวลาที่ให้:** P1 = 15 นาที / P2 = 1 ชั่วโมง

| ผล | ทำอะไร |
|---|---|
| ทุก P1 PASS | แจ้ง Lead: "Production smoke check ผ่านครับ HF-[NNN]" |
| มี P1 FAIL | แจ้ง Lead ทันที — Lead ตัดสินใจ rollback ทันที |

**กฎ:** QA รายงาน Lead เสมอ — ไม่รายงาน PO โดยตรง (Lead เป็นคนแจ้ง PO ผ่าน HotfixNotification)

---

## ไฟล์ที่ QA Maintain

| ไฟล์ | เมื่อไหร่ | เก็บที่ |
|------|----------|--------|
| `TestCases_[TaskID].md` | หลัง A-STEP 2 ต่อ task | `docs/qa/[feature]/` |
| `BugReport_[TaskID].md` | หลัง A-STEP 4 ต่อ task | `docs/qa/[feature]/` |
| `TestSuite_[Feature].md` | หลัง B-STEP 1 | `docs/qa/[feature]/` |
| `TestReport_[Feature].md` | หลัง B-STEP 3 | `docs/qa/[feature]/` |
| `HotfixSmokeTest_HF-[NNN].md` | หลัง HF-STEP 2 ต่อ hotfix | `docs/qa/hotfix/` |

Lead commit ไฟล์เหล่านี้เข้า repo ระหว่าง PR merge

---

## กฎ QA

- **QA เป็นคนตัดสิน verdict** — Claude ไม่ mark pass/fail หรือ approve feature โดยไม่มี QA confirm
- **Test Report Phase B → Lead แจ้ง PO เสมอ** ไม่ใช่ QA แจ้งโดยตรง
- ถ้า environment ไม่ reachable → **STOP** แจ้ง Lead/Dev ทันที อย่า assume
- ถ้า test result ambiguous → **STOP** แสดง raw response ให้ QA ตัดสิน
