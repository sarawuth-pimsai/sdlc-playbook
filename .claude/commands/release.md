คุณทำหน้าที่เป็น Release assistant สำหรับ repo `sdlc-playbook` นี้เอง (ไม่ใช่สำหรับ feature project ที่ใช้ playbook)

## งานของคุณ

สร้าง **semver tag** จาก `main` แล้วสร้าง **GitHub release** จาก tag นั้น — ใช้หลัง PR `develop` → `main` (จาก `/ship`) ถูก merge เข้า `main` แล้ว

ทำงานผ่าน `origin/main` โดยตรง **ไม่ checkout branch ใดๆ** — ไม่แตะ working directory หรือ local branch ปัจจุบันของ user เลย ปลอดภัยที่จะรันได้ไม่ว่า user จะ checkout branch อะไรอยู่ตอนนั้น

**ต่างจาก `/ship`:** ขั้นตอนอื่นทำต่อเนื่องได้โดยไม่หยุดถาม แต่ **การเลือก version ต้องหยุดถาม user ยืนยันเสมอ** (ดู STEP 2) เพราะเป็น judgment call ที่ผิดแล้วแก้ยาก (tag/release เป็นของ public ที่คนอื่น pin อ้างอิงได้) — ไม่ใช่ mechanical operation แบบ commit/push

---

## ก่อนเริ่ม — Safety checks

1. `git fetch origin --tags` — sync tags และ `origin/main` ล่าสุดก่อนเสมอ
2. หา tag ล่าสุด: `git tag -l --sort=-v:refname | head -1` (ถ้าไม่มี tag เลย → นี่คือ release แรก ใช้ `v1.0.0` เป็นข้อเสนอ base)
3. เช็คว่ามี commit ใหม่บน `origin/main` หลัง tag ล่าสุดไหม: `git log [last-tag]..origin/main --oneline`
   - **ถ้าไม่มี commit ใหม่เลย** → STOP แจ้ง user ว่า `origin/main` ไม่มีอะไรใหม่กว่า tag ล่าสุด (`[last-tag]`) ไม่ต้อง release
4. เช็คว่า `gh` auth พร้อมใช้งาน (`gh auth status`) — ถ้าไม่พร้อม ดู §เมื่อเจอปัญหา

---

## STEP 1 — วิเคราะห์การเปลี่ยนแปลงตั้งแต่ tag ล่าสุด

อ่าน commit ทั้งหมดตั้งแต่ tag ล่าสุดถึง `origin/main`:

```
git log [last-tag]..origin/main --no-merges --pretty=format:'%H%n%s%n%b%n---END---'
```

สำหรับแต่ละ commit ให้เข้าใจว่าเปลี่ยนอะไร กระทบ role ไหนบ้าง (PO/SA/Lead/Dev/QA/SEC/CORE_POLICY/docs) — ใช้ commit body ที่มักมี bullet list เหตุผลอยู่แล้วเป็นหลัก ถ้า commit message สั้นเกินไปให้ดู `git show [hash] --stat` เพิ่ม

---

## STEP 2 — เสนอ version + release notes → ต้องได้ user ยืนยันก่อนเสมอ

### เลือก bump type ตาม semver โดยดูจาก pattern ที่ repo นี้ใช้จริงมาก่อน (ดูตัวอย่างใน `gh release view` ของ tag เก่าๆ ถ้าไม่แน่ใจ):

| Bump | เมื่อไหร่ |
|------|----------|
| **MAJOR** (`vX.0.0`) | เปลี่ยนสถาปัตยกรรมหลักที่กระทบทุก role พร้อมกัน หรือ breaking change ต่อ workflow เดิม (เช่น `CORE_POLICY.md` ใหม่ทั้งไฟล์, ย้าย ownership ของการตัดสินใจข้าม role) |
| **MINOR** (`vX.Y.0`) | เพิ่ม feature/capability ใหม่ที่ backward-compatible (workflow ใหม่, field ใหม่, role ได้ step ใหม่) |
| **PATCH** (`vX.Y.Z`) | แก้บั๊ก, ปิด gap ความไม่สอดคล้องข้าม role, แก้ wording/drift — ไม่มี capability ใหม่ |

ถ้า commit ตั้งแต่ tag ล่าสุดมีทั้ง feature ใหม่และ bug fix ปนกัน → ใช้ bump สูงสุดที่พบ (MINOR ทับ PATCH)

### เขียน release notes ตาม style เดิมของ repo (ดูตัวอย่างจาก `gh release view v2.2.0`/`v2.2.1` เป็น reference):

```markdown
## What's changed
<!-- หรือ "## Highlights since v[prev]" ถ้าเป็น MAJOR bump -->

### [หมวดตาม role/พื้นที่ที่เปลี่ยน — เช่น "PO — ...", "Hotfix flow", "Docs"]
- [bullet อธิบายการเปลี่ยนแปลง + เหตุผล/gap ที่ปิด — ไม่ใช่แค่ "what" แต่บอก "why" ด้วย]

**Full diff**: https://github.com/sarawuth-pimsai/sdlc-playbook/compare/[last-tag]...v[new-version]

🤖 Generated with [Claude Code](https://claude.com/claude-code)
```

### แสดงให้ user เห็นก่อนทำอะไรต่อ:

```
Tag ล่าสุด: [last-tag]
เสนอ version ใหม่: v[X.Y.Z] ([MAJOR/MINOR/PATCH])
เหตุผล: [1-2 ประโยคสรุปว่าทำไมเลือก bump type นี้]

--- Release notes ที่ร่างไว้ ---
[full draft]
---

ยืนยัน tag + release เป็น v[X.Y.Z] เลยไหมครับ? (หรือแจ้ง version ที่ต้องการแทน)
```

**STOP รอ user ตอบเสมอ — ห้ามข้ามขั้นตอนนี้ไม่ว่ากรณีใด** ถ้า user ขอเปลี่ยน version หรือแก้ notes → ปรับแล้วแสดงใหม่อีกรอบก่อนไปต่อ

---

## STEP 3 — สร้าง tag + push

หลัง user ยืนยันแล้วเท่านั้น:

1. `git tag -a v[X.Y.Z] origin/main -m "v[X.Y.Z]"` — tag commit ที่ `origin/main` ชี้อยู่โดยตรง ไม่ต้อง checkout
2. `git push origin v[X.Y.Z]`
3. แสดงผลลัพธ์ push ให้ user เห็น

## STEP 4 — สร้าง GitHub release

1. `gh release create v[X.Y.Z] --title "v[X.Y.Z]" --notes "[release notes ที่ยืนยันแล้วจาก STEP 2]"`
2. รายงาน URL ของ release ให้ user

---

## เมื่อเจอปัญหา — หยุดแล้วถาม ห้ามแก้เองแบบเงียบๆ

- **`gh` ไม่ได้ติดตั้ง หรือยังไม่ auth** → แจ้ง user ให้จัดการเอง (`gh auth login`) ห้ามพยายาม `sudo` install/login เอง
- **Tag ชื่อนี้มีอยู่แล้ว** (เช่น user พิมพ์ version ผิดซ้ำของเก่า) → STOP แจ้ง user ห้าม force overwrite tag เดิมโดยไม่ถามก่อน
- **`origin/main` มี commit ที่ดูเหมือนยังไม่ผ่าน PR review ปกติ** (push ตรงเข้า main โดยไม่มี merge commit) → แจ้ง user ให้ตรวจสอบก่อน ไม่ต้อง block แต่ต้องบอก
- **ไม่แน่ใจว่าควร bump MAJOR/MINOR/PATCH** → เสนอทางเลือกที่เป็นไปได้ทั้งหมดพร้อมเหตุผลแต่ละแบบ ให้ user เลือกแทนการเดาเอง
