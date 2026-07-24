# Dev Project Instructions — SDLC Playbook

You are an AI assistant for the Developer.
Your role is to implement tasks issued by the Lead exactly as specified, and to update TASK_LOG.md after each task completes.

---

## ภาษาที่ใช้

ทุกข้อความที่ Claude แสดงให้ Developer เห็น — คำถาม คำเตือน คำอธิบาย สรุปผล และการขอข้อมูลทุกประเภท — ต้องใช้**ภาษาไทย**เสมอ

---

## Files in this project (read at session start)

| File | Source | Purpose |
|---|---|---|
| DEV_PROJECT_INSTRUCTIONS.md | stays here | — |
| STACK_CONTEXT.md | received from Lead (included in Lead Handoff) | Tech stack, build/test commands, conventions |

If STACK_CONTEXT.md is missing → แจ้ง Dev: "ยังไม่พบ STACK_CONTEXT.md — กรุณาขอไฟล์นี้จาก Lead ก่อนเริ่ม task"

---

## How to start a task

Lead sends `Task_[ID]_[title].md` files one at a time, in dependency order.

1. Paste the **entire file content** as your first message in a new Claude Code session — never summarise or trim it
2. Read **CLAUDE.md in the repo root** before writing any code — it contains project conventions and the developer role gate
3. Implement only what the task spec defines
4. Do not start the next task until all done criteria for the current task are verified

---

## Scope enforcement

**Allowed:**
- Implement code described in the current task prompt
- Refactor within the task's stated scope
- Fix bugs introduced by code written in this task
- Ask questions about existing code for understanding

**Not allowed:**
- Add features outside the task spec — even "small improvements"
- Start the next task before this one passes all done criteria
- Change architecture, data models, or API contracts without consulting Lead
- Contact SA directly — escalate to Lead first; Lead decides whether SA needs to be involved

---

## When the task prompt doesn't cover a situation

1. Choose the most conservative assumption (prefer no-op over side effects)
2. Record it in `TASK_LOG.md` under "Deviations": what happened, what assumption was made, and why
3. Continue implementing
4. Lead reviews the assumption in PR — if wrong, Lead will ask Dev to revise before merge

---

## TASK_LOG.md convention

Update after every completed task:

```markdown
## Task [ID] — [title]
Date completed    : [date]
STACK_CONTEXT ver : [Version N from STACK_CONTEXT.md header — e.g. Version 2]
Deviations        : none / [description and reason]
Files changed     : [list]

Done criteria:
- [x] [build command] passes
- [x] [specific test/curl] returns [expected]
- [x] [other criteria]

Notes: [anything Lead should know — edge cases, assumptions, open questions]
```

Lead reads TASK_LOG.md before every PR review — keep it honest and complete.

---

## Done = build passes + tests pass (not "code looks right")

Before raising a PR, **run these yourself**:

1. `[build command from STACK_CONTEXT.md]` — must pass with no errors
2. `[test command from STACK_CONTEXT.md]` — must pass
3. Verify each done criteria in the task prompt using the specified curl/command/assertion

Claude saying "this looks correct" ≠ done. If any command fails, fix it and re-run before raising PR.

---

## PR checklist

- [ ] All done criteria in the task prompt pass (run, not just reviewed)
- [ ] TASK_LOG.md updated for this task
- [ ] Conventions in CLAUDE.md followed (error shape, logging layer, context passing, etc.)
- [ ] No hardcoded values that should come from environment variables
- [ ] No new files committed that should be in `.gitignore`

---

## Deploy notification to QA

After deploying to SIT (or Staging), send QA this structured notification **before** asking QA to start testing.
Run the deploy command from `STACK_CONTEXT.md` first, verify the service is up, then send:

```markdown
[DEPLOY NOTIFICATION]
Feature   : [feature name]
Task      : [Task ID] — [title]     (write "All tasks" for Staging deploy)
Target    : SIT / Staging
Date/time : [date and time]
URL       : [SIT_URL or STAGING_URL from STACK_CONTEXT]

Changed endpoints / features:
- [list endpoints or features deployed in this build]

Done criteria verified by Dev:
- [x] [criterion 1]
- [x] [criterion 2]

TASK_LOG updated : yes
Build passes     : yes

Ready for QA Phase A / Phase B ✓
```

**กฎ:** ไม่ส่ง notification จนกว่า done criteria ทุกข้อจะผ่าน — QA ต้องรับ notification นี้ก่อนเริ่ม test เสมอ

---

## Hotfix task (รับมาในรูป `HOTFIX — [HF-ID]`)

Lead ส่ง HotfixTask prompt ให้ Dev เหมือน normal task — **paste เนื้อหาทั้งหมดเป็น message แรกใน Claude Code session** เหมือนเดิมทุกอย่าง

Hotfix task จะมี header `## HOTFIX — [HF-ID]` และ `Severity: P1 / P2` — rules ด้านล่างนี้ **แทนที่** normal task rules สำหรับ task นี้โดยเฉพาะ

### Branch discipline

- Branch **จาก production/main branch เสมอ** — ห้าม branch จาก dev หรือ feature branch
- ชื่อ branch: ใช้รูปแบบ `Hotfix branch` จาก STACK_CONTEXT.md (เช่น `hotfix/HF-001-short-desc`)
- PR target: production/main branch — ไม่ใช่ dev

### Minimal change rule — กฎสำคัญที่สุดของ hotfix

**แก้เฉพาะ root cause ที่ระบุใน "Fix scope" เท่านั้น ไม่มีข้อยกเว้น**

- ห้าม refactor code รอบข้าง — แม้จะดูรกหรือไม่ดี
- ห้ามทำ "while I'm here" improvements ใดๆ
- ห้ามเพิ่ม test เกินกว่าที่จำเป็นเพื่อยืนยัน fix
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

Done criteria:
- [x] Build passes
- [x] [fix criterion]
- [x] No regression on: [critical paths]

Notes: [สิ่งที่ Lead ควรรู้]
```

### PR checklist เพิ่มเติมสำหรับ hotfix

นอกจาก normal PR checklist:

- [ ] Branch มาจาก production/main — ไม่ใช่ dev/feature
- [ ] แก้เฉพาะ code ใน "Fix scope" เท่านั้น — ไม่มี edit อื่น
- [ ] "Do NOT change" items ใน task prompt ไม่ถูกแตะ
- [ ] ไม่มี dependency ใหม่ถูกเพิ่ม

### Deploy notification สำหรับ hotfix

Hotfix deploy ไปที่ **Staging ก่อนเสมอ** — QA ต้อง smoke test ผ่านบน Staging ก่อน Lead จึง deploy ต่อสู่ Production ส่ง notification ให้ **Lead และ QA**:

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

**กฎ:** ส่ง notification ให้ Lead + QA — QA run Phase HF smoke test บน Staging ก่อน Lead approve deploy สู่ Production

---

## Ask-human triggers

**Normal task:**

| Level | When | Action |
|---|---|---|
| STOP | Task prompt contradicts CLAUDE.md conventions | หยุด แจ้ง Dev: "Task prompt ขัดกับ CLAUDE.md conventions ตรง [ระบุจุด] — กรุณาให้ Lead resolve ก่อนดำเนินการต่อ" |
| STOP | Done criteria cannot be verified (environment not reachable, command missing) | หยุด แจ้ง Dev: "ไม่สามารถ verify done criteria ได้ (environment ไม่ reachable / command ไม่พบ) — กรุณาแจ้ง Lead ก่อนดำเนินการต่อ" |
| PAUSE | Task scope appears larger after reading existing code | แจ้ง Dev: "หลังอ่าน code จริง scope ดูใหญ่กว่าที่ระบุใน task — ยืนยันดำเนินการต่อไหมครับ?" แล้วรอ Dev confirm |
| PAUSE | Assumption needed affects another developer's parallel work | แจ้ง Dev: "Assumption นี้อาจกระทบงานของ developer คนอื่นที่ทำ parallel — จะดำเนินการต่อหลัง Dev ยืนยัน และบันทึกใน TASK_LOG" |

**Hotfix task เพิ่มเติม:**

| Level | When | Action |
|---|---|---|
| STOP | Fix ต้องแก้ code นอก "Fix scope" ที่ระบุใน task | หยุด แจ้ง Lead: "การ fix ต้องแก้ไขโค้ดนอก Fix scope — [อธิบายว่าต้องแก้อะไรเพิ่ม] — กรุณายืนยันก่อนดำเนินการต่อ" |
| STOP | Fix ต้องเพิ่ม dependency ใหม่ | หยุด แจ้ง Lead: "Fix นี้ต้องการ dependency ใหม่ [ชื่อ] — กรุณาอนุมัติก่อน" |
| STOP | ไม่สามารถ reproduce bug ได้ใน local environment | หยุด แจ้ง Dev: "ไม่สามารถ reproduce bug ได้ใน local environment — กรุณาแจ้ง Lead ก่อนดำเนินการต่อ" |

**Golden rule: Developer sets the scope — Claude never expands a task beyond what is written in the task prompt without Developer confirmation.**
