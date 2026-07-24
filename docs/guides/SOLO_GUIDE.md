# Solo Developer Guide — SDLC Playbook

คู่มือนี้สำหรับ developer ที่ทำงานคนเดียวและต้องสวมหมวกหลาย role

---

## ทำไมต้องแยก session แม้จะเป็นคนเดียว

การเปลี่ยน role = การเปลี่ยน mental context:

- **PO session** — คิดแบบ user, ห้ามคิดเรื่อง implementation
- **Lead session** — คิดแบบ architect, แตก scope ให้ชัดก่อนเขียน code
- **Dev session** — implement ตาม spec เท่านั้น, ไม่ออกนอก task

ถ้าอยู่ใน session เดียว Claude จะ shortcut ข้ามขั้นตอน เช่น รู้ PRD แล้วเขียน code เลย โดยไม่ผ่าน task breakdown — ผลคือ scope creep และ implementation ที่ไม่ตรง requirement

---

## Roles ที่แนะนำสำหรับ solo

### Minimum (โปรเจกต์เล็ก, ไม่ซับซ้อน)

```
PO session → Lead session → Dev session (CLAUDE.md)
```

| Role | ทำอะไร | เวลาที่ใช้ |
|------|---------|-----------|
| PO | วิเคราะห์ PRD, สร้าง Lead Handoff | 30-60 นาที ต่อ feature |
| Lead | รับ handoff, แตก tasks, generate Task_[ID].md | 30-60 นาที ต่อ feature |
| Dev | implement ทีละ task จนครบ | ส่วนใหญ่ของเวลา |

### แนะนำ (โปรเจกต์มี external integrations หรือ data model ซับซ้อน)

```
PO session → SA session → PO session → Lead session → Dev session
```

เพิ่ม SA เพื่อ design architecture ก่อน — ป้องกัน rework ที่ Lead/Dev

### Full (โปรเจกต์ production-critical)

```
PO → SA → PO → Lead → Dev → QA → Dev (fix) → QA (retest)
```

---

## Setup สำหรับ solo (Option B)

ทำตาม [Option B Setup Guide](../option-b/README.md) ก่อน โดยใช้ `/setup` command:

### 1. Setup ครั้งแรก

```bash
# copy แค่ setup.md ไฟล์เดียว
mkdir -p .claude/commands
cp <path-to-sdlc-playbook>/templates/option-b/commands/setup.md .claude/commands/setup.md
```

เปิด Claude Code แล้วพิมพ์ `/setup` — Claude จะสร้าง directory structure และ copy ไฟล์ที่จำเป็นทั้งหมดให้อัตโนมัติ

### 2. เตรียม session ต่อ role

หลัง setup เสร็จ ใช้ slash commands แยก session ตาม role:

```
/po    → วิเคราะห์ PRD
/sa    → ออกแบบ solution (ถ้าต้องการ)
/lead  → แตก tasks
       → (Dev ไม่ต้องพิมพ์ command — CLAUDE.md load อัตโนมัติ)
/qa    → ทำ test (ถ้าต้องการ)
```

### 2. Checkpoint ระหว่าง role

การ copy-paste handoff file ไปยัง session ถัดไปคือ **forced checkpoint** — บังคับให้คุณอ่านสิ่งที่ Claude gen ก่อน proceed

อย่ามองว่า overhead — มองว่าเป็นการ review ตัวเองในฐานะ role ถัดไป

---

## แนวทางการทำงาน

### เริ่ม feature ใหม่

1. เปิด Claude Code → พิมพ์ `/po`
2. Upload หรือ paste PRD content
3. ทำตาม STEP 1-4 จนได้ `LEAD_HANDOFF_[feature].md`
4. บันทึก handoff ใน `docs/roles/lead/`
5. พิมพ์ `/lead` ในหน้าต่างใหม่ (หรือ session เดิมหลัง reset)
6. ทำตาม L-STEP 1-4 จนได้ `Task_[ID]_[title].md` ทุกไฟล์
7. บันทึก task files ใน `docs/shared/tasks/`
8. เปิด session ใหม่ (CLAUDE.md load อัตโนมัติ)
9. Paste content จาก Task_[ID].md ที่ต้องการ implement

### ต่อ feature ที่ค้างไว้

- กลับเข้า **session เดิม** ของ role นั้น (ถ้าใช้ conversation thread เดิม)
- หรือเปิด session ใหม่ + พิมพ์ slash command + upload knowledge files ที่เกี่ยวข้อง

---

## Tips

**อย่าสลับ role ใน session เดียวกัน** — ถ้าคิด "อ้าว น่าจะ implement เลย" ระหว่าง PO session → จด note ไว้ แล้วไปทำใน Dev session แทน

**PRD สั้นก็ไม่เป็นไร** — PO interview mode ใน `PROJECT_INSTRUCTIONS.md` ช่วย draft PRD จากการตอบคำถามได้ ไม่ต้องมีเอกสารเต็ม

**ใช้ TASK_LOG.md จริงจัง** — เป็น single source of truth ว่า task ไหนทำแล้ว done criteria ผ่านไหม — ช่วยมากเมื่อกลับมาทำต่อหลังหยุดพักหลายวัน

**Solo ไม่จำเป็นต้องมี STACK_CONTEXT.md แยก** — ถ้ารู้ stack ดีอยู่แล้ว Lead สามารถ embed stack info ลงใน CLAUDE.md โดยตรง และข้าม SA Stack Setup flow

---

## Artifact ownership (solo version)

เมื่อทำคนเดียว คุณเป็นเจ้าของทุก artifact แต่ยังต้องสร้างตาม convention เพราะ Claude อ่าน format เหล่านี้:

| Artifact | สร้างตอนไหน | เก็บที่ไหน |
|---|---|---|
| PRD | PO session | `docs/roles/po/` |
| LEAD_HANDOFF_[feature].md | PO session | `docs/roles/lead/` |
| Task_[ID]_[title].md | Lead session | `docs/shared/tasks/` |
| CLAUDE.md | Lead session | root ของ project |
| TASK_LOG.md | Dev session (update ทุก task) | `docs/shared/` |
