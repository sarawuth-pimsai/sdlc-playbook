# Stack Template Library

Stack templates คือ `STACK_CONTEXT.md` ที่ pre-filled สำหรับ stack family ที่ใช้บ่อย
SA ใช้เป็น baseline แล้ว customize สำหรับ project — ไม่ต้องสร้างจากศูนย์

---

## วิธีใช้ (SA)

### 1. ตรวจสอบว่า PO ส่ง PE org template มาไหม

หาก PO แนบ `STACK_CONTEXT_[OrgName].md` มาพร้อม Stack Setup Request — ใช้ไฟล์นั้นเป็น baseline ได้เลย (PE ของ org นั้น validate แล้ว) ข้ามขั้นตอน 2-3

### 2. เลือก template ที่ตรงกับ stack family

| Template | เมื่อไหร่ควรเลือก |
|----------|-----------------|
| [STACK_CONTEXT_go_clean_arch.md](STACK_CONTEXT_go_clean_arch.md) | Backend Go + Clean Architecture + PostgreSQL/Redis + Next.js หรือ Vite frontend |
| _(เพิ่มตามที่ใช้จริงในองค์กร)_ | — |

ถ้าไม่มี template ที่ match — ใช้ blank schema ที่ `docs/roles/sa/STACK_CONTEXT.md` แทน

### 3. Copy template → เปลี่ยนชื่อเป็น `STACK_CONTEXT.md`

```
cp docs/roles/sa/stack-templates/STACK_CONTEXT_go_clean_arch.md  docs/roles/sa/STACK_CONTEXT.md
```

### 4. Customize สำหรับ project

สิ่งที่ต้องตรวจสอบและ fill in ทุกครั้ง:

- [ ] **Header** — เปลี่ยน `[Project Name]` และ date
- [ ] **IdP** — ระบุ Identity Provider จริงของ project (Keycloak / Azure AD / อื่น)
- [ ] **Version verification** — WebSearch ทุก package ก่อน finalize (ดู Version Verification rule ใน `ai/SA_PROJECT_INSTRUCTIONS.md`)
- [ ] **SA Decision Fields** — fill in ค่าที่เลือกสำหรับ project นี้
- [ ] **ลบ section ที่ไม่ใช้** — เช่น ถ้าไม่มี frontend ให้ลบ §2-3 ออก
- [ ] **เพิ่ม org-specific constraints** (ถ้า PE มี requirements เพิ่มเติม)

### 5. ส่งกลับ PO

SA อัปโหลด `STACK_CONTEXT.md` ที่ filled-in เข้า SA Project Knowledge → export → ส่ง PO → PO อัปโหลดเข้า PO Project

---

## วิธีใช้ (Platform Engineering — สร้าง org template)

PE ดูแล standard tech stack ขององค์กร สามารถ fork template เพื่อสร้าง org-specific baseline ที่ทุก project ในองค์กรใช้

### ขั้นตอนการสร้าง Org Template

1. **Copy template ที่ตรงกับ org stack**

   ```
   cp docs/roles/sa/stack-templates/STACK_CONTEXT_go_clean_arch.md  \
      STACK_CONTEXT_[OrgName].md
   ```

2. **ระบุ org constraints** ที่ทุก project ต้องใช้ ตัวอย่าง:

   ```markdown
   ## Org-specific Constraints ([OrgName])
   > Maintained by Platform Engineering — ห้ามแก้โดยไม่ผ่าน PE
   
   - DB: Aurora PostgreSQL 16 เท่านั้น (managed service — ห้ามใช้ self-hosted)
   - IdP: Azure AD B2C (กำหนดโดย IT Security)
   - Container runtime: Kubernetes on EKS (ไม่ใช้ docker-compose ใน production)
   - Secret management: AWS Secrets Manager — ห้าม hardcode ใน env file
   ```

3. **Version lock ที่ org approve แล้ว** — ระบุ version จริงแทน `latest` หรือ range

4. **ลบ SA Decision Fields ที่ PE ตัดสินใจแทน SA แล้ว** — เช่น ถ้า org ใช้ Aurora เสมอ ไม่ต้องให้ SA เลือก DB

5. **Distribute ให้ SA** — ส่งไฟล์ให้ PO ของแต่ละ project แนบมาพร้อม Stack Setup Request

### สิ่งที่ PE ควรระบุชัดเจนใน Org Template

| หมวด | ตัวอย่างที่ PE lock ได้ | ตัวอย่างที่ยังให้ SA เลือกได้ |
|------|----------------------|---------------------------|
| Infrastructure | DB engine, IdP, secret manager, container runtime | DB schema design, cache key pattern |
| Security | Auth protocol, token storage policy | Role/permission structure per feature |
| Observability | OTel collector endpoint, trace sampling rate | Signal ที่ instrument per service |
| Networking | API gateway, load balancer | Internal service communication pattern |

---

## การเพิ่ม Template ใหม่

เมื่อ org เริ่มใช้ stack family ใหม่ (เช่น Python FastAPI หรือ Node.js NestJS) และมีโอกาสใช้กับหลาย project:

1. Copy `STACK_CONTEXT_go_clean_arch.md` เป็น starting point สำหรับโครงสร้าง
2. เปลี่ยน §1-4 ให้ตรงกับ stack ใหม่
3. คง §5 Cross-cutting concerns — ส่วนใหญ่ reuse ได้ (Auth protocol, PDPA)
4. อัปเดตตารางในไฟล์นี้ (README.md)
5. ทดสอบกับ 1 project จริงก่อน promote เป็น official template

**กฎ:** อย่าสร้าง template จาก stack สมมติ — สร้างเฉพาะเมื่อมี project จริงที่ต้องใช้

---

## ไฟล์ใน directory นี้

| ไฟล์ | ประเภท | ครอบคลุม |
|------|--------|---------|
| `README.md` | คู่มือ | วิธีใช้ทั้งหมด |
| `STACK_CONTEXT_base_crosscutting.md` | Reference | Cross-cutting concerns (Auth protocol, OTel standard, PDPA) — ไม่มี stack-specific packages |
| `STACK_CONTEXT_go_clean_arch.md` | Template | Go + Clean Architecture + PostgreSQL + Redis + Next.js / Vite |
