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

ทุก hotfix (P1/P2) ต้องผ่าน **Retroactive Tier Tagging** จาก SA หลัง merge เสมอ — ไม่ใช่ optional "ถ้ามีเวลา" (ดู `ai/LEAD_PROJECT_INSTRUCTIONS.md` §Hotfix post-merge checklist และ `ai/SA_PROJECT_INSTRUCTIONS.md` §Retroactive Hotfix Triage) เพื่อจับกรณีที่ hotfix จริงๆ ควรเป็น Tier 2/3 แต่ไม่มีใครรู้เพราะข้ามการ triage ตอน merge เร่งด่วน

---

## 5. Environment (per-role, not project-wide)

`PROJECT_CONTEXT.md` เก็บ **default ของโปรเจกต์** + **override รายบุคคลต่อ role**:

```markdown
Environment (default): cli / claude.ai
Environment overrides:
  SA: [ไม่ระบุ = ใช้ default]
  Lead: [ไม่ระบุ = ใช้ default]
  Dev: cli   # fixed เสมอ — ห้าม override
  QA: [ไม่ระบุ = ใช้ default]
  SEC: [ไม่ระบุ = ใช้ default]   # เฉพาะ Option A (Security role: yes)
```

**Effective Environment ของแต่ละ role** = override ของ role นั้น (ถ้ามี) ไม่งั้นใช้ default — Dev fix เป็น `cli` เสมอ ไม่มีทาง override

**Pairwise rule สำหรับทุกจุด handoff** (ใช้แทน rule เดิมที่เช็คแค่ฝั่งเดียว):

| ฝั่งส่ง (effective) | ฝั่งรับ (effective) | วิธี output |
|---|---|---|
| cli | cli | Write tool → save ลง disk repo เดียวกัน (auto, ไม่ต้อง relay) |
| อื่นๆ (ฝั่งใดฝั่งหนึ่งหรือทั้งคู่เป็น claude.ai) | — | Artifact + Download button → แจ้งให้ upload เข้า Project ปลายทางเอง |
| ไม่รู้ effective Environment ของฝั่งรับ | — | ใช้ Artifact เป็น safe default (compatible เสมอ) |

แต่ละ role ถาม override ของตัวเอง **แค่ครั้งแรกที่เริ่มทำงานในโปรเจกต์นี้** ไม่ถามซ้ำทุก session — รายละเอียดการถามและ Handoff Check อยู่ใน `ai/SA_PROJECT_INSTRUCTIONS.md`, `LEAD_PROJECT_INSTRUCTIONS.md`, `QA_PROJECT_INSTRUCTIONS.md`, `SEC_PROJECT_INSTRUCTIONS.md` (Option A เท่านั้น) §Environment override + §Handoff Environment Check

---

## 6. Routing Principle — เมื่อไหร่ผ่าน PO, เมื่อไหร่ตรง

ระบบมี 2 รูปแบบ routing ที่ดูเหมือนไม่สมมาตร แต่ตั้งใจออกแบบตามจังหวะงาน:

**Planning-phase handoff → ผ่าน PO เสมอ** (ความถี่ต่ำ ต้องการ oversight)
- SA → PO → Lead (Solution Doc, ADR)
- SEC Phase A → ผ่าน PO (Solution Doc security review, ก่อนเริ่ม implement)
- เหตุผล: ช่วงวางแผนเกิดครั้งเดียวต่อ feature, PO เป็น single distribution channel ที่เห็นภาพรวมทั้งหมด เหมาะสำหรับจุดที่ต้องมี checkpoint ระดับ business

**Execution-phase interaction → ตรงระหว่าง operational roles** (ความถี่สูง ต้องการความเร็ว)
- Lead ↔ SA escalation (L-STEP 1.5)
- Lead ↔ SEC Phase B (PR-level code review)
- เหตุผล: ช่วง implement เกิดถี่ (ทุก task, ทุก PR) ถ้าผ่าน PO ทุกรอบจะกลายเป็นคอขวด — Lead กับ role ปฏิบัติการอื่นเป็นคนทำงานจริงในจังหวะนี้ ไม่จำเป็นต้องมี PO เป็นตัวกลางทุกครั้ง

**กฎสรุป:** ยิ่งความถี่ของ interaction สูง (เกิดหลายรอบต่อ feature) ยิ่งควรตัดผ่าน PO ออก ยิ่งความถี่ต่ำ (เกิดครั้งเดียวต่อ feature) ยิ่งควรผ่าน PO เพื่อรักษา oversight — ไม่ใช่กฎตายตัวรายบุคคล role
