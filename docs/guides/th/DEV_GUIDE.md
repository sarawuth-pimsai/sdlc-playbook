# คู่มือ Developer

> ดูภาพรวม workflow และการ setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

Developer ใช้ **Claude Code** (CLI หรือ App) — ไม่ใช่ claude.ai Project

---

## การติดตั้ง Claude Code

### Option A — CLI (แนะนำ)

```bash
npm install -g @anthropic-ai/claude-code
```

ตรวจสอบ:
```bash
claude --version
```

### Option B — Desktop App

ดาวน์โหลดจาก [claude.ai/download](https://claude.ai/download)

### Option C — IDE Extension

ติดตั้ง extension สำหรับ VS Code หรือ JetBrains จาก marketplace

---

## Input ที่รับจาก Lead

Lead ส่งทีละไฟล์ตาม dependency order:

| ไฟล์ | หมายเหตุ |
|------|---------|
| `CLAUDE.md` | Lead commit เข้า repo root ก่อนเริ่ม — อ่านก่อนทุกครั้ง |
| `STACK_CONTEXT.md` | Tech stack, build/test commands |
| `Task_[ID]_[title].md` | Task prompt ทีละ task — ห้ามเริ่ม task ถัดไปจนกว่า done criteria ทุกข้อจะผ่าน |

---

## การ Resume task session (session ขาดกลางคัน)

ถ้าต้องกลับมาทำ task ที่ทำค้างไว้จาก session ก่อน:

1. เปิด `TASK_LOG.md` ก่อน — หา entry ของ task ID นั้น
   - Done criteria ทุกข้อติ๊ก `[x]` ครบแล้ว → task เสร็จแล้ว ไม่ต้อง implement ซ้ำ
   - Done criteria ยังไม่ครบ → อ่าน Deviations และ Notes ก่อนดำเนินการต่อ
2. รัน build และ test ก่อนเสมอ — อย่า assume ว่า repo ยัง clean
3. ทำต่อจากจุดที่ค้าง — ไม่ต้องเริ่มใหม่จากศูนย์

---

## วิธีเริ่ม Task

### ขั้นตอนที่ 1 — เตรียม Claude Code

```bash
# เปิด terminal ใน root ของ code repo
cd /path/to/my-project
claude
```

### ขั้นตอนที่ 2 — อ่าน CLAUDE.md ก่อนเสมอ

CLAUDE.md มี project conventions ที่ต้องตาม เช่น error shape, logging rules, context passing

### ขั้นตอนที่ 3 — สร้าง branch

สร้าง branch ตาม `### Branch setup` ใน task prompt ก่อนเริ่มเขียน code:

```bash
git checkout [base branch from task prompt]
git pull
git checkout -b [branch name from task prompt]
```

### ขั้นตอนที่ 4 — Paste Task Prompt

Paste **เนื้อหาทั้งหมด** ของ `Task_[ID]_[title].md` ลงใน Claude Code session ใหม่

> ห้าม summarize หรือตัดเนื้อหาออก — paste verbatim

### ขั้นตอนที่ 5 — Implement

Claude Code implement ตาม spec ใน task prompt

**ขอบเขตที่ทำได้:**
- Implement code ตาม task spec
- Refactor ภายใน scope ของ task
- Fix bugs ที่เกิดจาก code ใน task นี้
- ถามเพื่อทำความเข้าใจ code ที่มีอยู่

**ห้ามทำ:**
- เพิ่ม feature นอก task spec
- เริ่ม task ถัดไปก่อน done criteria ทุกข้อผ่าน
- เปลี่ยน architecture, data models, หรือ API contracts โดยไม่ปรึกษา Lead
- ติดต่อ SA โดยตรง — escalate ผ่าน Lead ก่อนเสมอ

---

## Done Criteria — ต้องรันด้วยตัวเองก่อน raise PR

```bash
# 1. Build ต้องผ่าน
[build command จาก STACK_CONTEXT.md]

# 2. Tests ต้องผ่าน
[test command จาก STACK_CONTEXT.md]

# 3. ตรวจสอบ done criteria แต่ละข้อ
# รัน curl/command ตามที่ระบุใน task prompt
```

> "Claude บอกว่า looks correct" ≠ done — ต้องรัน command จริง

---

## Self-check ก่อน raise PR (เกินกว่า build/test ผ่าน)

Build ผ่านและ test ผ่าน ไม่ได้แปลว่า code พร้อม — รันก่อนทุกครั้ง:

1. **Static analysis แบบเข้ม** — strict lint/analyze command จาก STACK_CONTEXT.md ต้องผ่านแบบ zero warning ไม่ใช่แค่ zero error
2. **Verify API ที่ใช้มีอยู่จริง** — เช็คกับ lockfile version จริงก่อนเชื่อว่า method/class มีอยู่ — ห้าม assume จาก training knowledge
3. **Error handling ครบ** — ทุก async call มี error path และเช็ค null/undefined ก่อนใช้งาน
4. **Resource cleanup** — controller/stream/listener ที่เปิดใหม่ ต้อง dispose ชัดเจน
5. **Self-review diff** — ไฟล์ที่แก้เกินขอบเขต task ไหม เพิ่ม dependency ใหม่โดยไม่จำเป็นไหม
6. **Test assert ของจริง** — test ที่เขียนเองต้อง assertion ผูกกับ behavior จริง ไม่ใช่ placeholder (เช่น `expect(true, true)`)
7. **Existing function check** — grep/search codebase หา function หรือ method ที่มี signature หรือ behavior คล้ายกับที่เพิ่งเขียน (เช่น validation, formatting, error building) — ถ้าเจอให้ reuse แทน ถ้าเลือกสร้างใหม่ต้องบันทึกเหตุผลใน TASK_LOG ใต้ "Deviations"

ข้อใดไม่ผ่าน → กลับไปแก้ก่อน ไม่ raise PR

---

## TASK_LOG.md — อัปเดตหลังทุก task

```markdown
## Task [ID] — [title]
Date completed    : [date]
STACK_CONTEXT ver : [Version N จาก STACK_CONTEXT.md header]
Deviations        : none / [description and reason]
Files changed     : [list]

Done criteria:
- [x] [build command] passes
- [x] [specific test/curl] returns [expected]
- [x] [other criteria]

Notes: [อะไรที่ Lead ควรรู้ — edge cases, assumptions, open questions]
```

Lead อ่าน TASK_LOG ก่อน review PR ทุกครั้ง — เขียนให้ครบและตรงไปตรงมา

---

## PR Checklist

ก่อน raise PR ตรวจ:

- [ ] Done criteria ทุกข้อผ่าน (รันแล้ว ไม่ใช่แค่ดู)
- [ ] Self-check 7 ข้อผ่านครบ (static analysis, API verification, error handling, resource cleanup, self-review diff, real test assertions, existing function check)
- [ ] `TASK_LOG.md` อัปเดตสำหรับ task นี้แล้ว
- [ ] ตาม conventions ใน `CLAUDE.md` (error shape, logging, context passing)
- [ ] ไม่มี hardcoded values ที่ควรมาจาก environment variables
- [ ] ไม่มีไฟล์ที่ควรอยู่ใน `.gitignore` ติดมาใน commit

---

## Deploy Notification ให้ QA

หลัง deploy ถึง SIT (หรือ Staging) → รัน deploy command จาก STACK_CONTEXT ก่อน → verify service up → ส่ง notification นี้ให้ QA:

```markdown
[DEPLOY NOTIFICATION]
Feature   : [feature name]
Task      : [Task ID] — [title]     (เขียน "All tasks" สำหรับ Staging deploy)
Target    : SIT / Staging
Date/time : [date and time]
URL       : [SIT_URL หรือ STAGING_URL จาก STACK_CONTEXT]

Changed endpoints / features:
- [list endpoints หรือ features ที่ deploy ใน build นี้]

Done criteria verified by Dev:
- [x] [criterion 1]
- [x] [criterion 2]

TASK_LOG updated : yes
Build passes     : yes

Ready for QA Phase A / Phase B ✓
```

> **กฎ:** ห้ามส่ง notification จนกว่า done criteria ทุกข้อจะผ่านแล้ว

---

## Hotfix Task

Lead ส่ง HotfixTask prompt ให้ Dev เหมือน normal task — **paste เนื้อหาทั้งหมดเป็น message แรกใน Claude Code session** task จะมี header `## HOTFIX — [HF-ID]` และ `Severity: P1 / P2`

### Branch discipline

- Branch **จาก production/main branch เสมอ** — ห้าม branch จาก dev หรือ feature branch
- ชื่อ branch: ใช้รูปแบบ `Hotfix branch` จาก STACK_CONTEXT.md เช่น `hotfix/HF-001-short-desc`
- PR target: production/main branch — ไม่ใช่ dev

### Minimal change rule — กฎสำคัญที่สุดของ hotfix

**แก้เฉพาะ root cause ที่ระบุใน "Fix scope" เท่านั้น ไม่มีข้อยกเว้น**

- ห้าม refactor code รอบข้าง
- ห้ามทำ "while I'm here" improvements
- ถ้า fix ต้องแตะ code นอก "Fix scope" → **STOP** แจ้ง Lead ก่อนทุกครั้ง

### TASK_LOG.md สำหรับ hotfix

ใช้ `HF-[NNN]` เป็น task ID:

```markdown
## HF-[NNN] — [title]
Date completed    : [date]
STACK_CONTEXT ver : [Version N]
Severity          : P1 / P2
Root cause        : [หนึ่งประโยค]
Deviations        : none / [อธิบาย — แจ้ง Lead ทันทีถ้ามี]
Files changed     : [list — ควรสั้นมากสำหรับ hotfix]
```

### Deploy Notification สำหรับ hotfix

Hotfix deploy ไปที่ **Staging ก่อนเสมอ** — QA ต้อง smoke test ผ่านบน Staging ก่อน Lead จึง deploy ต่อสู่ Production ส่ง notification ให้ **Lead และ QA** (ไม่ใช่ QA อย่างเดียว):

```markdown
[HOTFIX DEPLOY NOTIFICATION]
HF-ID     : HF-[NNN]
Severity  : P1 / P2
Target    : Staging
Date/time : [date and time]

Fix deployed:
- [สิ่งที่เปลี่ยน]

Done criteria verified:
- [x] Build passes
- [x] [fix criterion]
- [x] No regression on: [critical paths ที่ test locally]

TASK_LOG updated : yes
```

### PR Checklist เพิ่มเติมสำหรับ hotfix

นอกจาก normal PR checklist:

- [ ] Branch มาจาก production/main — ไม่ใช่ dev/feature
- [ ] แก้เฉพาะ code ใน "Fix scope" เท่านั้น
- [ ] "Do NOT change" items ใน task prompt ไม่ถูกแตะ
- [ ] ไม่มี dependency ใหม่ถูกเพิ่ม

---

## เมื่อ Task Prompt ไม่ครอบคลุมสถานการณ์

1. เลือก assumption ที่ conservative ที่สุด (prefer no-op over side effects)
2. บันทึกใน `TASK_LOG.md` ใต้ "Deviations": เกิดอะไร, ตัดสินใจอะไร, ทำไม
3. Implement ต่อ
4. Lead review assumption ใน PR — ถ้าผิดจะขอให้แก้ก่อน merge

---

## หยุดและรอ Lead ในกรณีเหล่านี้

| Level | เมื่อไหร่ | Action |
|-------|----------|--------|
| STOP | Task prompt ขัดกับ CLAUDE.md conventions | หยุด แสดง conflict ให้ Dev รู้ รอ Lead resolve |
| STOP | Done criteria ไม่สามารถ verify ได้ (environment ไม่ reachable) | หยุด แจ้ง Dev ให้ notify Lead รอ |
| PAUSE | Scope ของ task ใหญ่กว่าที่คาดหลังอ่าน code จริง | แสดง scope difference — ดำเนินการต่อหลัง Dev confirm เท่านั้น |
| PAUSE | Assumption ที่ต้องทำกระทบงานของ developer คนอื่นที่ทำ parallel | Flag ก่อน บันทึกใน TASK_LOG |
