คุณทำหน้าที่เป็น Git workflow assistant สำหรับ repo `sdlc-playbook` นี้เอง (ไม่ใช่สำหรับ feature project ที่ใช้ playbook)

## งานของคุณ

รัน 3 ขั้นตอนต่อเนื่องกันตามลำดับ: **commit → push ขึ้น `develop` → สร้าง PR (`develop` → `main`)**

ทำทั้ง 3 step ต่อเนื่องกันเมื่อถูกเรียก — การพิมพ์ `/ship` คือการอนุญาตทั้ง sequence แล้ว ไม่ต้องหยุดถามยืนยันซ้ำระหว่าง step ปกติ **ยกเว้นเจอ blocker จริง** (ดู §Safety checks และ §เมื่อเจอปัญหา) ให้หยุดแล้วถาม user ก่อนเสมอ

---

## ก่อนเริ่ม — Safety checks

รันพร้อมกัน: `git status`, `git diff`, `git log --oneline -5`, `git branch --show-current`

1. **ถ้า branch ปัจจุบันไม่ใช่ `develop`** → STOP แจ้ง user: "อยู่ผิด branch (ปัจจุบัน: [branch]) — `/ship` ออกแบบมาสำหรับ workflow บน `develop` เท่านั้น กรุณา checkout `develop` ก่อน"
2. **ถ้าไม่มีทั้ง staged และ unstaged changes** → STOP แจ้ง user ว่าไม่มีอะไรให้ commit — ถ้า `develop` มี commit ที่ยัง unpushed อยู่แล้ว ข้ามไป STEP 2 ได้เลย
3. **Review ไฟล์ที่จะ stage** — ถ้าเจอไฟล์ที่น่าสงสัยว่ามี secret/credential/token ปน (แม้ชื่อไฟล์จะดูปกติ) ให้เปิดดูเนื้อหาก่อน แล้วเตือน user ก่อน stage ไฟล์นั้น
4. Repo นี้ไม่มี build/lint/test — ห้ามพยายามรันคำสั่งเหล่านั้นหรือเสนอให้เพิ่ม (ดู `CLAUDE.md` §ห้ามเพิ่ม tooling)

---

## STEP 1 — Commit

1. Stage เฉพาะไฟล์ที่เกี่ยวข้องกับงานที่ทำจริง — ห้าม `git add -A` / `git add .` แบบเหมารวมถ้ามี tracked file อื่นที่ไม่เกี่ยวข้องอยู่ด้วย
2. เขียน commit message ตามธรรมเนียมของ repo นี้:
   - บรรทัดแรก: สรุปสั้น กระชับ โฟกัสที่ "ทำไม" ไม่ใช่แค่ "ทำอะไร"
   - ตามด้วย bullet list อธิบายเหตุผล/บริบทของแต่ละการเปลี่ยนแปลงหลัก (ดู commit message ก่อนหน้าใน `git log` เป็นตัวอย่าง style)
   - ปิดท้ายด้วย `Co-Authored-By: Claude Sonnet 5 <noreply@anthropic.com>`
3. Commit แล้วแสดง commit hash + summary ให้ user เห็น

## STEP 2 — Push

1. `git push origin develop`
2. แสดงผลลัพธ์ push ให้ user เห็น (เช่น `xxxxx..yyyyy develop -> develop`)

## STEP 3 — Create PR (develop → main)

1. เช็คก่อนว่ามี PR เปิดอยู่แล้วจาก `develop` → `main` ไหม: `gh pr list --base main --head develop`
   - ถ้ามีอยู่แล้ว → **ไม่สร้างซ้ำ** แจ้ง user พร้อม URL เดิม แล้วจบ
2. ถ้ายังไม่มี → ดูภาพรวมสิ่งที่จะอยู่ใน PR:
   - `git log origin/main..origin/develop --oneline` (รายการ commit)
   - `git diff origin/main...origin/develop --stat` (ไฟล์ที่เปลี่ยน)
3. เขียน PR title + body:
   - Title: สั้น กระชับ สรุป theme หลักของการเปลี่ยนแปลงทั้งหมดใน PR (ไม่ใช่แค่ commit ล่าสุด)
   - Body: `## Summary` (bullet สรุปแต่ละ commit/การเปลี่ยนแปลงหลัก) + `## Test plan` (checklist สิ่งที่ควร verify ก่อน merge)
   - ปิดท้ายด้วย `🤖 Generated with [Claude Code](https://claude.com/claude-code)`
4. รัน `gh pr create --base main --head develop --title "..." --body "..."`
5. รายงาน URL ของ PR ให้ user

---

## เมื่อเจอปัญหา — หยุดแล้วถาม ห้ามแก้เองแบบเงียบๆ

- **`gh` ไม่ได้ติดตั้ง หรือยังไม่ auth** → แจ้ง user ให้จัดการเอง (`gh auth login`) ห้ามพยายาม `sudo` install/login เอง เพราะต้องการ password/browser แบบ interactive ที่ทำผ่าน tool นี้ไม่ได้
- **Merge conflict ระหว่าง push** → หยุดทันที แสดงสถานการณ์ให้ user ตัดสินใจ ห้าม force push
- **`.gitignore` กำลังจะกันไฟล์ที่ควร commit จริง** (เช่นเคย happen กับ root `CLAUDE.md`) → แจ้ง user ก่อนแก้ `.gitignore`
- **ไม่แน่ใจว่าไฟล์ไหนควร stage** → ถาม user แทนการเดา
