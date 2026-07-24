# คู่มือ Security Engineer (SEC)

**Option A เท่านั้น** — SEC Project ใช้งานเฉพาะเมื่อ `Security role: yes` ใน `PROJECT_CONTEXT.md`

ถ้าทีมไม่มี Security Engineer → ใช้ **Option B** โดย security checkpoints ฝังอยู่ใน SA / Lead / QA instructions แล้ว ไม่ต้องติดตั้ง SEC Project

---

## การติดตั้ง

### Option A — claude.ai Projects

ใช้ผ่าน claude.ai web interface

#### ขั้นตอนที่ 1 — สร้าง Project

1. เปิด [claude.ai](https://claude.ai) → **Projects** → **New project**
2. ตั้งชื่อ เช่น `[ชื่อโปรเจกต์] — Security`
3. เปิด **Project Instructions** → Copy เนื้อหาจาก `ai/SEC_PROJECT_INSTRUCTIONS.md` → Paste → บันทึก

#### ขั้นตอนที่ 2 — อัปโหลด Project Knowledge

| ไฟล์ | เมื่อไหร่ |
|------|----------|
| `STACK_CONTEXT.md` | ได้รับจาก PO พร้อม Phase A package |
| `PRD_[feature].md` | ได้รับจาก PO |
| `Solution_Doc_[feature].md` | ได้รับจาก PO (จาก SA) — ต้องมีก่อนเริ่ม Phase A |

---

### Option B — Claude Code

ใช้ผ่าน Claude Code CLI, Desktop App, หรือ IDE Extension แทน claude.ai Projects

#### ขั้นตอนที่ 1 — ตั้งค่า Workspace

```bash
mkdir ~/[project]-sec
cd ~/[project]-sec
cp /path/to/sdlc-workflow/ai/SEC_PROJECT_INSTRUCTIONS.md CLAUDE.md
```

รัน `claude` จาก directory นั้น — Claude Code อ่าน `CLAUDE.md` อัตโนมัติเมื่อเริ่ม session

#### ขั้นตอนที่ 2 — วางไฟล์ใน Directory

วางไฟล์ทุกรายการจากตาราง Option A ลง directory เดียวกัน แทนการ upload เข้า Project Knowledge  
Claude Code สามารถอ่านไฟล์ใน working directory ได้โดยตรง

---

## Phase A — Solution Doc Security Review

**Trigger:** PO ส่ง `Solution_Doc_[feature].md` ให้ SEC หลังได้รับจาก SA

### A-STEP 1 — วิเคราะห์ Architecture ด้าน Security

Review Solution Doc ใน 7 categories:

| Category | สิ่งที่ตรวจ |
|----------|------------|
| Authentication | Auth ถูก define ทุก endpoint ไหม? ใครเรียกอะไรได้บ้าง? |
| Authorization | Role/permission check ระบุต่อ operation ชัดเจนไหม? |
| Data exposure | API response รวม fields ที่ไม่ควร public ไหม? |
| Input validation | User inputs ทุก field ถูก validate/sanitise ก่อนใช้ไหม? |
| PDPA / Data sensitivity | Feature เก็บหรือ process personal data ไหม? มี retention period กำหนดไหม? |
| Secrets management | Credentials ใช้ environment variables ทั้งหมดไหม? ไม่มี hardcoded? |
| New dependencies | มี library ใหม่ไหม? ถ้ามี — มี known CVE ไหม? |

### A-STEP 2 — Output Security Requirements

Claude produce `Security_Requirements_[feature].md`:

```markdown
# Security Requirements — [Feature name]
Version: 1.0 | Date: [date] | Author: SEC

## Risk summary
| Risk | Severity | Mitigation required |

## Implementation requirements
[รายการ requirements ที่ verifiable ได้ใน Phase B code review]
1. [e.g. ทุก /admin endpoint ต้อง validate JWT และ assert role = "admin"]
2. [e.g. User email ต้องไม่ปรากฏใน response body หรือ application logs]

## PDPA considerations
[Data collected, retention policy, consent — หรือ "none identified"]

## Open questions for SA/PO
[Items ที่ต้อง clarify ก่อน requirements จะ finalise]
```

Preview → SEC review → confirm → export → **ส่งให้ PO**

PO รวมไฟล์นี้เข้า Lead Handoff → Lead ฝังใน task prompts

### A-STEP 3 — Blocking Risk

ถ้าพบ risk ที่ทำให้ implementation ไม่ปลอดภัยโดยไม่มี architecture redesign:
- **STOP** → อธิบาย risk และ Solution Doc section ที่เกี่ยวข้อง
- SEC notify PO + SA ก่อน implementation เริ่ม
- **ห้าม output partial requirements** — complete review ทั้งหมดก่อนแล้ว flag ทุกปัญหาพร้อมกัน

---

## Phase B — Code Review (ต่อ PR)

**Trigger:** Dev raise PR → Lead ส่ง changed files + task prompt ให้ SEC

### B-STEP 1 — Review Changed Files

ตรวจ 7 ด้าน:

1. **Auth/authorization** — Security Requirements จาก Phase A ผ่านใน code ไหม?
2. **Input validation** — inputs ถูก sanitise ก่อนใช้ใน DB queries, shell commands, responses ไหม?
3. **Data exposure** — response body รวม sensitive fields (passwords, tokens, PII) ไหม?
4. **Injection risks** — SQL injection, command injection, path traversal
5. **Secrets** — ไม่มี hardcoded credentials, API keys, env values ใน code หรือ config files
6. **Error messages** — error responses leak internal details (stack traces, DB schema, internal paths) ไหม?
7. **New packages** — dependency ใหม่มี known CVE ไหม?

### B-STEP 2 — Output Security Review

```markdown
# Security Review — [Task ID] [Task title]
Date: [date] | Reviewer: SEC

## Must fix (blocks merge)
- [finding — file path, line number, description]

## Should fix (request changes)
- [finding — recommendation]

## Consider (non-blocking)
- [suggestion]

## Verdict: APPROVED / REQUEST CHANGES / ESCALATE TO SA
```

Preview → SEC confirm → export `Security_Review_[TaskID].md` → **ส่งให้ Lead**

Lead อ่านและตัดสิน: merge หรือขอ Dev แก้ก่อน re-review

ถ้า verdict **ESCALATE TO SA** → Lead notify SA + PO ก่อน merge ใดๆ

---

## Flow ของ Files

```
PO ──► SEC    Solution_Doc_[feature].md
SEC ──► PO    Security_Requirements_[feature].md  (Phase A output)
              PO รวมเข้า Lead Handoff

Lead ──► SEC  changed files + task prompt (ต่อ PR)
SEC ──► Lead  Security_Review_[TaskID].md  (Phase B output)
```

---

## กฎ SEC

- **SEC ตัดสิน security verdict** — Claude ไม่ approve PR หรือ clear security risk โดยไม่มี SEC confirm
- Phase A → ส่ง Security Requirements ให้ PO เสมอ (ไม่ส่งตรง Lead)
- Phase B → ส่ง Security Review ให้ Lead เสมอ
- ถ้าพบ critical vulnerability (auth bypass, data leak, injection) ใน Phase B → **STOP** notify Lead ทันที ห้าม approve

---

## Option B — Security Checkpoints (ถ้าไม่มี SEC Engineer)

เมื่อ `Security role: no` security ถูกจัดการใน 3 จุด:

| จุด | ใคร | อะไร |
|----|-----|------|
| SA Solution Doc Section 6 | SA | Security checklist: auth, authorization, input validation, data exposure, secrets, PDPA |
| Lead PR Review (R-STEP 2) | Lead | Hardcoded secrets, missing input validation, sensitive data in response, auth checks, error message leaks |
| QA Phase B TestSuite (B-STEP 1) | QA | Security test cases: unauth access, expired token, SQL injection, sensitive fields in response |
