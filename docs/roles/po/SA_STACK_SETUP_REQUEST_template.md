# Last updated: YYYY-MM-DD | Version: 1

# SA Stack Setup Request — [Project Name]

> PO ใช้ไฟล์นี้เพื่อส่ง request ให้ SA configure stack สำหรับ project ใหม่
> copy ไฟล์นี้แล้วแก้ไข [Project Name] และ context section ด้านล่าง จากนั้นส่งให้ SA

---

## Project Context

**Project name:** [ชื่อ project]

**เป้าหมายของ project:**
[อธิบาย 2–3 ประโยคว่า project นี้ทำอะไร และใครใช้]

**PRD summary (ถ้ามี):**
[แนบหรือ embed PRD ที่เกี่ยวข้อง หรือ link ไปยังเอกสาร]

**ข้อจำกัดที่ทราบแล้ว:**
- [ ] ต้องรองรับ concurrent users ประมาณ ___
- [ ] ต้อง deploy บน ___
- [ ] มี timeline: ___
- [ ] อื่นๆ: ___

---

## Stack Default

SA ใช้ `docs/roles/sa/STACK_CONTEXT.md` ใน project นี้เป็น baseline
ไม่ต้องระบุ tech choices ในไฟล์นี้ — SA จะ interview และ confirm ใน Stack Setup Flow

---

## Action Required

SA กรุณาดำเนินการ **Stack Setup Flow** ตาม `ai/SA_PROJECT_INSTRUCTIONS.md`
เมื่อเสร็จแล้วส่ง `STACK_CONTEXT.md` ที่ filled-in กลับมาให้ PO
