# คู่มือ Platform Engineering (PE)

> ดูภาพรวม workflow และการ setup → [WORKFLOW_OVERVIEW.md](../WORKFLOW_OVERVIEW.md)

PE ดูแล standard tech stack ขององค์กร สร้างและ maintain **org template** ที่ทุก project ใช้เป็น baseline แทนที่จะให้แต่ละ SA สร้างจากศูนย์

---

## บทบาทของ PE ใน SDLC Playbook

PE ไม่ได้อยู่ใน SDLC workflow โดยตรง — PE ทำงาน "ก่อน" workflow เริ่ม:

```
PE สร้าง Org Template
        ↓
PO แนบ Org Template ใน Stack Setup Request → SA
        ↓
SA customize สำหรับ project → STACK_CONTEXT.md
        ↓
SDLC Workflow ปกติ (SA → PO → Lead → Dev → QA)
```

เมื่อ org template มีอยู่แล้ว SA ไม่ต้องตัดสินใจ technology choices ที่ PE lock ไว้แล้ว — ลด cognitive load และ enforce consistency

---

## การสร้าง Org Template (ครั้งแรก)

### ขั้นตอนที่ 1 — เลือก base template

```
docs/roles/sa/stack-templates/
  STACK_CONTEXT_go_clean_arch.md   ← Go fullstack (แนะนำถ้า org ใช้ Go)
  [template อื่นๆ ที่จะเพิ่มในอนาคต]
```

copy template ที่ตรงกับ org stack family มากที่สุด:

```bash
cp docs/roles/sa/stack-templates/STACK_CONTEXT_go_clean_arch.md \
   STACK_CONTEXT_[OrgName].md
```

### ขั้นตอนที่ 2 — เพิ่ม Org Constraints section

เพิ่ม section นี้ต่อจาก `## Purpose` ในไฟล์:

```markdown
## Org-specific Constraints — [OrgName]

> Maintained by Platform Engineering
> SA ห้ามแก้ไข section นี้โดยไม่ผ่าน PE

### Infrastructure (locked by PE)

| Component | Org Standard | หมายเหตุ |
|-----------|-------------|---------|
| Database | [เช่น Aurora PostgreSQL 16] | managed — ห้ามใช้ self-hosted |
| Cache | [เช่น ElastiCache Redis 7] | managed |
| Container | [เช่น Kubernetes on EKS] | production เท่านั้น |
| Secret management | [เช่น AWS Secrets Manager] | ห้าม .env ใน production |
| CI/CD | [เช่น GitHub Actions + ArgoCD] | — |

### Security (locked by PE)

| Requirement | Value |
|-------------|-------|
| IdP | [เช่น Azure AD B2C tenant: xxx] |
| Auth protocol | OAuth2 + OIDC + PKCE (ห้าม deviate) |
| Token storage (SPA) | HttpOnly cookie via BFF — ห้าม localStorage |
| Vulnerability scanning | [เช่น Snyk ทุก PR] |

### Observability (locked by PE)

| Item | Value |
|------|-------|
| OTel Collector endpoint | [internal endpoint] |
| Trace sampling | [เช่น 10% production, 100% staging] |
| Log destination | [เช่น CloudWatch / Elastic] |

### Network (locked by PE)

| Item | Value |
|------|-------|
| API Gateway | [เช่น AWS API Gateway / Kong] |
| Internal service discovery | [เช่น AWS Cloud Map] |
```

### ขั้นตอนที่ 3 — Lock SA Decision Fields ที่ PE ตัดสินใจแล้ว

ใน section `## SA Decision Fields` — เปลี่ยน fields ที่ PE lock แล้วจาก "SA เลือก" เป็นค่า fixed:

**ตัวอย่าง:**

```markdown
## SA Decision Fields

> Fields ที่ marked [PE LOCKED] — SA ไม่ต้องตัดสินใจ PE กำหนดให้แล้ว

| Field | Value | สถานะ |
|-------|-------|-------|
| Auth Method | OAuth2 + OIDC + PKCE via Azure AD B2C | [PE LOCKED] |
| PostgreSQL Driver | pgx/v5 | [PE LOCKED] |
| Migration Tool | golang-migrate | [PE LOCKED] |
| HTTP Router | SA เลือก — Fiber v3 (default) / Chi / Echo | SA decides |
| SPA State Management | SA เลือก — Zustand (default) / TanStack Query | SA decides |
```

### ขั้นตอนที่ 4 — Verify versions

**ห้าม fill version จาก memory** — verify ผ่าน package registry ก่อน finalize org template ทุกครั้ง

ระบุ verified date ใน Stack Versions table ที่ท้าย template

### ขั้นตอนที่ 5 — Distribute

ส่ง `STACK_CONTEXT_[OrgName].md` ให้ PO ทุก project ที่ใช้ org stack นี้
PO แนบมาพร้อม Stack Setup Request เมื่อส่งให้ SA

---

## การ Maintain Org Template

### เมื่อไหร่ต้อง update

| เหตุการณ์ | Action |
|---------|--------|
| Infrastructure upgrade (DB version, K8s version) | อัปเดต version ใน template + bump version header |
| Security policy เปลี่ยน (IdP, token policy) | อัปเดต Security section + notify SA ทุก project ที่ active |
| New approved package | เพิ่มใน template + ระบุ use case |
| Package deprecated หรือ CVE critical | ลบออก / เปลี่ยน + notify urgently |
| SA ขอ deviation (กระทบ org standard) | พิจารณา ADR → ถ้า approve อัปเดต template |

### Version bump convention

Template version ใช้ integer เพิ่มทีละ 1 ตาม header:

```
# Last updated: YYYY-MM-DD | Version: N | Status: [OrgName] Org Standard
```

### Notify SA

เมื่อ update org template → แจ้ง SA ทุก project ที่กำลัง active พร้อมระบุ:
- Version เก่า → Version ใหม่
- สิ่งที่เปลี่ยน
- ว่า project ปัจจุบันต้องอัปเดต `STACK_CONTEXT.md` ของตัวเองหรือไม่

---

## การเพิ่ม Stack Template ใหม่ใน Playbook

เมื่อ org เริ่มใช้ stack family ใหม่ที่จะ apply กับหลาย project (เช่น Python FastAPI):

1. Copy template ที่ใกล้เคียงที่สุดจาก `docs/roles/sa/stack-templates/`
2. เปลี่ยน §1-4 ให้ตรงกับ stack ใหม่
3. คง §4 cross-cutting structure — เปลี่ยนเฉพาะ packages ที่ stack-specific
4. เพิ่ม row ในตารางใน [stack-templates/README.md](../roles/sa/stack-templates/README.md)
5. ทดสอบกับ 1 project จริงก่อน promote เป็น official template

**กฎ:** สร้างเฉพาะ stack ที่มี project จริงในองค์กรใช้แล้ว — template สมมติที่ไม่มีคนใช้กลายเป็น maintenance burden

---

## Checklist ก่อน Distribute Org Template

- [ ] Version header ครบ (`Last updated`, `Version`, `Status`)
- [ ] Org Constraints section ครบ (Infrastructure / Security / Observability)
- [ ] [PE LOCKED] fields ระบุชัดเจน — SA รู้ว่าอะไรที่ตัดสินใจได้เอง
- [ ] Versions ทุกตัว verify ผ่าน package registry แล้ว
- [ ] Stack Versions table fill in ครบพร้อม verified date
- [ ] ทดสอบกับ 1 real project ก่อน distribute วงกว้าง

---

## FAQ

**Q: PE ต้องใช้ claude.ai หรือ Claude Code ไหม?**
A: ไม่จำเป็น — PE ทำงานกับ Markdown files โดยตรง ไม่มี AI workflow เฉพาะ PE ใน playbook นี้

**Q: ถ้า project มีข้อยกเว้นจาก org standard ทำอย่างไร?**
A: SA เขียน ADR → PE review → ถ้า approve SA customize `STACK_CONTEXT.md` ของ project นั้น โดยระบุ deviation ชัดเจน ไม่กระทบ org template

**Q: Org template ควรเก็บไว้ที่ไหน?**
A: ใน private repository ขององค์กร — ไม่ควร commit ลง playbook repo เพราะมี internal infrastructure details (endpoints, tenant IDs, internal service names)

**Q: PE ต้อง create Claude Project แยกไหม?**
A: ไม่จำเป็น สำหรับ playbook version นี้ PE ทำงานกับไฟล์ Markdown โดยตรง
