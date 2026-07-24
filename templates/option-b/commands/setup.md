คุณทำหน้าที่เป็น **Setup Assistant** สำหรับ SDLC Playbook (Option B)

## งานของคุณ

สร้าง directory structure และ copy ไฟล์ที่จำเป็นสำหรับ Option B workflow ในโปรเจกต์นี้

## ขั้นตอน

### STEP 1 — หา sdlc-playbook

ค้นหา path ของ sdlc-playbook ตามลำดับ:
1. ตรวจสอบ `~/.sdlc-playbook-path` (ถ้ามี ให้อ่าน path จากไฟล์นี้)
2. ตรวจสอบ `~/Workspace/ai/sdlc-playbook`
3. ตรวจสอบ `~/sdlc-playbook`
4. ถ้าไม่พบ → ถาม user ว่า sdlc-playbook อยู่ที่ไหน แล้วบันทึก path นั้นลงใน `~/.sdlc-playbook-path`

### STEP 2 — สร้าง directory structure

รัน bash commands ต่อไปนี้:

```bash
mkdir -p docs/roles/po
mkdir -p docs/roles/sa/adr
mkdir -p docs/roles/lead
mkdir -p docs/roles/qa
mkdir -p docs/shared/tasks
mkdir -p .claude/commands
```

### STEP 3 — Copy ไฟล์จาก sdlc-playbook

```bash
# Copy role instruction files
cp -r <SDLC_PLAYBOOK_PATH>/ai ./ai

# Copy slash commands (ยกเว้น setup.md ตัวเอง)
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/po.md .claude/commands/po.md
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/sa.md .claude/commands/sa.md
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/lead.md .claude/commands/lead.md
cp <SDLC_PLAYBOOK_PATH>/templates/option-b/commands/qa.md .claude/commands/qa.md
```

แทน `<SDLC_PLAYBOOK_PATH>` ด้วย path จริงที่หาได้ใน STEP 1

### STEP 4 — สร้าง CLAUDE.md เริ่มต้น

สร้างไฟล์ `CLAUDE.md` ที่ root ของโปรเจกต์ (ถ้ายังไม่มี) ด้วย content:

```markdown
# CLAUDE.md

คุณทำหน้าที่เป็น **Developer** สำหรับโปรเจกต์นี้

## Instructions
อ่านและทำตาม `ai/DEV_PROJECT_INSTRUCTIONS.md` ทุกข้อ

## Knowledge files ของคุณ
อ่านไฟล์ต่อไปนี้ใน `docs/shared/` silently ก่อนเริ่ม task:
- `tasks/Task_[ID]_[title].md` — task ที่ได้รับจาก Lead
- `TASK_LOG.md` — ดู task ที่ทำแล้วและ done criteria

## กฎสำคัญ
- Implement ตาม task spec เท่านั้น
- อัปเดต `docs/shared/TASK_LOG.md` ทุกครั้งที่ task เสร็จ
- ไม่ออกนอก scope ของ task ที่ได้รับ
```

### STEP 5 — สร้าง TASK_LOG.md เริ่มต้น

สร้างไฟล์ `docs/shared/TASK_LOG.md` ถ้ายังไม่มี:

```markdown
# Last updated: <TODAY_DATE> | Version: 1

# TASK_LOG.md

| Task ID | Title | Status | Done Criteria Met | Notes |
|---------|-------|--------|-------------------|-------|
```

### STEP 6 — แสดงสรุป

หลังจากทำเสร็จ แสดง:
1. directory structure ที่สร้างแล้ว (ใช้ `find . -type d | sort`)
2. ไฟล์ที่ copy มา
3. Next steps สำหรับผู้ใช้:

```
Setup เสร็จแล้ว ✓

Next steps:
1. พิมพ์ /po เพื่อเริ่ม PO session และวิเคราะห์ feature แรก
2. หรือพิมพ์ /sa ถ้าต้องการออกแบบ architecture ก่อน
```
