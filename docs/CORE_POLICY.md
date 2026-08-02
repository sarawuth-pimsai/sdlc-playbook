# Core Policy — ใช้ร่วมกันทั้ง Solo และ Enterprise mode

ไฟล์นี้คือ policy หลักที่ทุก role/mode ต้องทำตาม ห้าม guide อื่นเขียน logic เหล่านี้ซ้ำ — ให้ reference มาที่นี่แทน

---

## 1. Tier Ownership

**SA เป็นเจ้าของการตัดสินใจ tier เสมอ** — ไม่มี role ไหน (PO, Lead, หรือแม้แต่ solo dev คนเดียว) ข้าม SA ได้ ต่างกันแค่ SA ใช้เวลาสั้นหรือยาวตาม tier

- PO ทำได้แค่เก็บรายละเอียด feature + flag business-risk keyword เป็น context ส่งให้ SA — PO **ไม่ตัดสิน tier**
- SA ตัดสิน tier ที่ STEP 1.5 (Tier Triage) ดูรายละเอียดใน `ai/SA_PROJECT_INSTRUCTIONS.md` §STEP 1.5
- ใช้ได้กับทั้ง Solo และ Enterprise mode โดยไม่มีข้อยกเว้น — Solo "Minimum" tier เดิมที่ให้ PO ข้าม SA ได้ **ถูกยกเลิกแล้ว** ดู `docs/guides/SOLO_GUIDE.md` §Roles ที่แนะนำสำหรับ solo

### Business-risk keyword list (ใช้ทั้ง PO และ SA)

```
BUSINESS_RISK_KEYWORDS = [
  "payment", "checkout", "billing",
  "auth", "authentication", "authorization", "login", "session token",
  "PII", "personal data", "email address", "phone number", "national ID",
  "external API", "third-party integration", "webhook",
  "encryption", "compliance", "GDPR", "PDPA"
]
```

business-risk flag จาก PO ไม่ได้ auto-set tier แต่บังคับให้ tier ต่ำสุดคือ Tier 3 เสมอ แม้ SA จะประเมินว่าโครงสร้างไม่กระทบก็ตาม

---

## 2. Escalation

Lead escalation gate (L-STEP 1.5 — Tier Escalation Check) ใช้ทั้ง 2 mode เหมือนกันทุกตัวอักษร — ดู `ai/LEAD_PROJECT_INSTRUCTIONS.md` §L-STEP 1.5

Lead ที่พบว่างานเกินขอบเขต Triage Summary/Solution Doc เดิมต้อง escalate ตรงไปหา SA (ไม่ผ่าน PO) — ห้าม Lead ตัดสินใจเชิงสถาปัตยกรรมแทน SA ไม่ว่ากรณีใด ยกเว้นเข้าเงื่อนไข Hotfix Flow P1/P2 (ดู §4)

---

## 3. Parallel Execution

Parallel lane assignment (L-STEP 2.5 — Dependency Graph + Lane Assignment) ใช้ได้ทั้ง 2 mode — ดู `ai/LEAD_PROJECT_INSTRUCTIONS.md` §L-STEP 2.5

Solo ที่ทำคนเดียวสามารถข้าม step นี้ได้ถ้าไม่มี dev คนที่ 2 จริง (ระบุ note ใน `docs/guides/SOLO_GUIDE.md`)

---

## 4. Hotfix Flow

Hotfix flow (P1/P2/P3) ใน `ai/LEAD_PROJECT_INSTRUCTIONS.md` §Hotfix flow ใช้ได้กับทั้ง 2 mode — Solo ใช้ Lead session เดียวกันได้ทันที ไม่ต้องเขียนแยก (ดู `docs/guides/SOLO_GUIDE.md` §Hotfix Flow)

P1/P2 เป็นข้อยกเว้นเดียวที่ Lead ตัดสินใจเชิง technical/scope โดยไม่ต้องรอ SA sign-off ก่อน — escalate ถึง SA **หลัง merge** เท่านั้น
