# Last updated: YYYY-MM-DD | Version: 1

# SA Stack Setup Request — [Project Name]

> PO ใช้ไฟล์นี้เพื่อส่ง request ให้ SA configure stack สำหรับ project ใหม่
> copy ไฟล์นี้ แก้ไข [Project Name] และ context section ด้านล่าง จากนั้นส่งให้ SA

---

## Project Context

**Project name:** [ชื่อ project]

**เป้าหมายของ project:**
[อธิบาย 2-3 ประโยคว่า project นี้ทำอะไร และใครใช้]

**PRD summary (ถ้ามี):**
[แนบหรือ embed PRD ที่เกี่ยวข้อง หรือ link ไปยังเอกสาร]

**ข้อจำกัดที่ทราบแล้ว:**
- [ ] ต้องรองรับ concurrent users ประมาณ ___
- [ ] ต้อง deploy บน ___
- [ ] มี timeline: ___
- [ ] อื่นๆ: ___

---

## Stack Guidance

### กรณี 1 — PE มี Org Template แล้ว

ถ้า Platform Engineering ขององค์กรมี org template อยู่แล้ว แนบไฟล์นั้นมาพร้อมกับ request นี้:

```
[แนบไฟล์ STACK_CONTEXT_[OrgName].md ที่ได้รับจาก PE]
```

SA จะใช้ org template เป็น baseline และ customize สำหรับ project นี้

### กรณี 2 — ไม่มี Org Template (SA เลือกเอง)

SA เลือก stack template จาก `docs/roles/sa/stack-templates/` ที่เหมาะสมกับ project
ดูรายการ templates ที่ [docs/roles/sa/stack-templates/README.md](../../roles/sa/stack-templates/README.md)

**Stack family ที่คาดว่าจะใช้ (ถ้าทราบ):**
[ระบุถ้ารู้ เช่น "น่าจะเป็น Go backend" หรือ "ไม่แน่ใจ — ให้ SA ประเมิน"]

---

## Action Required

SA กรุณาดำเนินการ **Stack Setup Flow** ตาม `ai/SA_PROJECT_INSTRUCTIONS.md`

**ขั้นตอน SA:**
1. รับไฟล์นี้จาก PO
2. ตรวจสอบว่า PO แนบ PE org template มาด้วยหรือไม่
3. ถ้ามี org template → ใช้เป็น baseline
4. ถ้าไม่มี → เลือก template จาก `docs/roles/sa/stack-templates/` ที่เหมาะสม
5. Customize สำหรับ project นี้ + verify versions ผ่าน WebSearch
6. บันทึกเป็น `STACK_CONTEXT.md` → ส่งกลับ PO

เมื่อได้รับ `STACK_CONTEXT.md` จาก SA → PO อัปโหลดเข้า PO Project Knowledge
