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

**SA เป็นเจ้าของการตัดสินใจ tier เสมอ แม้ทำคนเดียวก็ข้ามไม่ได้** — Minimum tier ยังต้องผ่าน SA session เสมอ แต่ใช้ SA STEP 1.5 Tier Triage แบบสั้น (ไม่ใช่ full Solution Doc) ดู `ai/SA_PROJECT_INSTRUCTIONS.md` §STEP 1.5 และ [CORE_POLICY.md](../CORE_POLICY.md) §1

```
PO session → SA session (triage สั้น) → Lead session → Dev session (CLAUDE.md)
```

| Role | ทำอะไร | เวลาที่ใช้ |
|------|---------|-----------|
| PO | วิเคราะห์ PRD, scan business-risk keywords, สร้าง SA Handoff | 30-60 นาที ต่อ feature |
| SA | Tier Triage — ตัดสิน tier + เขียน Triage Summary สั้น (Fast Path ถ้า Tier 1) | 10-15 นาที ต่อ feature (Tier 1) |
| Lead | รับ Triage Summary/Solution Doc, แตก tasks, generate Task_[ID].md | 30-60 นาที ต่อ feature |
| Dev | implement ทีละ task จนครบ | ส่วนใหญ่ของเวลา |

หมายเหตุ: ถ้า SA ประเมินว่า feature กระทบมากกว่าที่คิด (เช่น แตะ data model ใหม่ หรือมี business-risk flag) → tier จะถูกยกระดับเป็น Tier 2/3 อัตโนมัติ และใช้เวลามากกว่านี้ตามปกติของ Solution Doc เต็ม

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
/sa    → Tier Triage (บังคับเสมอ) + ออกแบบ solution ถ้า Tier 2/3
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
3. ทำตาม STEP 1-2 จนได้ `SA_HANDOFF_[feature].md`
4. บันทึก handoff ใน `docs/roles/sa/`
5. พิมพ์ `/sa` ในหน้าต่างใหม่ — SA ทำ STEP 1.5 Tier Triage (บังคับเสมอ แม้ Minimum tier) จนได้ `Triage_Summary_[feature].md` (Tier 1) หรือ `Solution_Doc_[feature].md` (Tier 2/3)
6. บันทึกผลจาก SA ใน `docs/roles/po/` แล้วกลับไป `/po` เพื่อทำ STEP 3-4 จนได้ `LEAD_HANDOFF_[feature].md`
7. บันทึก handoff ใน `docs/roles/lead/`
8. พิมพ์ `/lead` ในหน้าต่างใหม่ (หรือ session เดิมหลัง reset)
9. ทำตาม L-STEP 1-4 จนได้ `Task_[ID]_[title].md` ทุกไฟล์
10. บันทึก task files ใน `docs/shared/tasks/`
11. เปิด session ใหม่ (CLAUDE.md load อัตโนมัติ)
12. Paste content จาก Task_[ID].md ที่ต้องการ implement

### ต่อ feature ที่ค้างไว้

- กลับเข้า **session เดิม** ของ role นั้น (ถ้าใช้ conversation thread เดิม)
- หรือเปิด session ใหม่ + พิมพ์ slash command + upload knowledge files ที่เกี่ยวข้อง

---

## Tips

**อย่าสลับ role ใน session เดียวกัน** — ถ้าคิด "อ้าว น่าจะ implement เลย" ระหว่าง PO session → จด note ไว้ แล้วไปทำใน Dev session แทน

**PRD สั้นก็ไม่เป็นไร** — PO interview mode ใน `PROJECT_INSTRUCTIONS.md` ช่วย draft PRD จากการตอบคำถามได้ ไม่ต้องมีเอกสารเต็ม

**ใช้ TASK_LOG.md จริงจัง** — เป็น single source of truth ว่า task ไหนทำแล้ว done criteria ผ่านไหม — ช่วยมากเมื่อกลับมาทำต่อหลังหยุดพักหลายวัน

**Solo ไม่จำเป็นต้องมี STACK_CONTEXT.md แยก** — ถ้ารู้ stack ดีอยู่แล้ว Lead สามารถ embed stack info ลงใน CLAUDE.md โดยตรง และข้าม SA Stack Setup flow (นี่คือเรื่อง STACK_CONTEXT.md เท่านั้น — **ไม่เกี่ยวกับ SA Tier Triage session ซึ่งข้ามไม่ได้เสมอ** ดู §Roles ที่แนะนำสำหรับ solo)

---

## Hotfix Flow (production bug ด่วน)

Solo ใช้ Hotfix Flow เดียวกับ enterprise ทุกประการ — ดู [LEAD_GUIDE.md §Hotfix Flow](LEAD_GUIDE.md#hotfix-flow) และ `ai/LEAD_PROJECT_INSTRUCTIONS.md` §Hotfix flow (ดู [CORE_POLICY.md](../CORE_POLICY.md) §4)

เปิด Lead session แล้วพิมพ์ระบุ severity (P1/P2) ตามเกณฑ์เดิม — ไม่ต้องเขียน flow แยกสำหรับ solo เพราะ mechanism เหมือนกันทุกอย่าง ต่างแค่คนเดียวสวม role Lead

---

## Artifact ownership (solo version)

เมื่อทำคนเดียว คุณเป็นเจ้าของทุก artifact แต่ยังต้องสร้างตาม convention เพราะ Claude อ่าน format เหล่านี้:

| Artifact | สร้างตอนไหน | เก็บที่ไหน |
|---|---|---|
| PRD | PO session | `docs/roles/po/` |
| SA_HANDOFF_[feature].md | PO session | `docs/roles/sa/` |
| Triage_Summary_[feature].md (Tier 1) / Solution_Doc_[feature].md (Tier 2/3) | SA session | `docs/roles/po/` |
| LEAD_HANDOFF_[feature].md | PO session | `docs/roles/lead/` |
| Task_[ID]_[title].md | Lead session | `docs/shared/tasks/` |
| CLAUDE.md | Lead session | root ของ project |
| TASK_LOG.md | Dev session (update ทุก task) | `docs/shared/` |
