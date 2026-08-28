# Dev Guide — การบันทึก Doc + Memory เมื่อปิดงาน (สำหรับส่งต่องาน)

> · อัปเดต: 2026-08-28
> ที่มาของกฎ: `CLAUDE.md` (section "Memory policy" + "ปิดงานหนึ่งชิ้น"),
> `docs/guides/memory-policy.md`, `memory/README.md`, `memory/_fact.template.md`
> — ไฟล์นี้คือฉบับลงมือทำ พร้อมตัวอย่างจริงจากงานจริง

---

## หลักเดียวที่ต้องจำ

> **ปิดงานหนึ่งชิ้น (ไม่ต้องรอจบ session) = เอกสาร + memory ไปพร้อมงานใน commit เดียวกัน**
> "เดี๋ยวค่อย commit แยกทีหลัง" ไม่มีเจ้าของและไม่มี trigger — เคยทำให้ต้องมี commit ตามเก็บ

---

## 1. มีอะไรต้องบันทึก — เลือกที่ให้ถูก

| สิ่งที่จะจด | ที่ที่ถูก | รูปแบบ |
|---|---|---|
| decision ที่ตัดสินใจแล้ว (มีทางที่ไม่เลือก + เหตุผล + กับดัก) | `docs/decisions/<slug>.md` | ไฟล์เต็ม + **1 บรรทัด** สรุปใน `CLAUDE.md` section "Architecture Decisions" |
| feature ทั้งตัว (1 ไฟล์/feature, รวมประวัติการเปลี่ยนแปลง) | `docs/features/feature-<ชื่อ>.md` | ส่วนตามหัวข้อ + ตาราง "Commit provenance" (เวลา/commit hash/เนื้อหา/ไฟล์) |
| fact สั้น ที่ session ถัดไปต้องรู้ อ่านจบใน 30 วิ | `memory/<slug>.md` | frontmatter + เนื้อสั้น + **เพิ่ม index ใน `memory/MEMORY.md`** |
| งานเล็กที่ไม่เข้า feature ไหน | `docs/features/misc-features.md` | bullet |
| secret / IP / server path | `docs/private/` หรือ `memory/private/` | **gitignored ทั้งคู่ — ห้าม index ลง `MEMORY.md`** อ้างด้วย pointer เท่านั้น |

กติกาการไหลของขนาด: ถ้า bullet ใน `CLAUDE.md` **โตเกิน ~15 บรรทัด = promote เนื้อไป `docs/decisions/`** ทันที
(CLAUDE.md ถูกโหลดเต็มทุก session — โตแล้วคิดเงินเป็น context)

### ความต่างที่คนพลาดบ่อย

- `docs/decisions/` = ความคิด + ทางที่ไม่ได้เลือก + กับดัก (อ่าน **ก่อนแตะ** เรื่องนั้น)
- `docs/features/` = หน้าตาของ feature ปัจจุบัน + quirks ที่โค้ดเล่าเองไม่ได้ (อ่าน **ก่อนวางแผน** งานต่อ)
- `memory/` = fact สั้น 1 ไฟล์/1 fact — เรื่องใหญ่ไป `docs/` แล้ว memory เป็น pointer

---

## 2. Workflow จริง (checklist ตามลำดับ)

```
□ 1. งานสำเร็จ → ถามคำถาม 3 ข้อ:
     - เอกสารเดิมที่เกี่ยวข้องยังตรงกับความจริงไหม? (ผิด/ถูกแทน = mark superseded ไม่ลบทิ้ง)
     - มี decision/quirk ใหม่ที่ dev คนถัดไปต้องรู้ไหม? → doc
     - มี fact สั้นให้ session หน้าจำไหม? → memory
□ 2. เขียน/อัปเดตเอกสาร (ดู format ด้านล่าง)
□ 3. เขียน/อัปเดต memory + MEMORY.md index
□ 4. ลบ TODO(scope) ที่งานนี้ปิดแล้วออกจาก CLAUDE.md
□ 5. ตรวจ: prettier ไฟล์ .md ที่แตะ + grep originSessionId + ไม่มี secret
□ 6. commit เอกสาร + memory พร้อมงาน (หรือ commit docs แยกก็ได้ถ้างานใหญ่ — แต่ห้ามค้างข้าม session)
```

---

## 3. Format ของแต่ละที่

### 3.1 `memory/<slug>.md` — ตาม `memory/_fact.template.md` เป๊ะ

```markdown
---
name: <short-kebab-case-slug>
description: <one-line — ใช้ตัดสิน relevant ตอน recall>
metadata:
  type: user | feedback | project | reference
---

<ตัว fact — สั้น ตรงประเด็น>

**Why:** <เหตุผล/ที่มา — สำคัญสุด อย่าข้าม>
**How to apply:** <ใช้ยังไงในทางปฏิบัติ>

<ลิงก์ fact ที่เกี่ยวด้วย [[other-fact-name]]>
```

แล้วเพิ่ม 1 บรรทัดใน `memory/MEMORY.md`:

```markdown
- [ชื่อเรื่อง](slug.md) — hook สั้น ๆ ว่าทำไมต้องเปิด
```

ตัวอย่างจริงจากงาน (จด gotcha ของ Base UI):

```markdown
---
name: base-ui-select-value-needs-items-on-root
description: Base UI SelectValue โชว์ raw value หลังเลือก จนกว่าจะส่ง items ที่ Select root
metadata:
  type: project
---

`SelectValue` แมป value→label จาก prop `items` **ที่ `Select` root** เท่านั้น ...
```

### 3.2 `docs/features/feature-<ชื่อ>.md` — หัวใจคือ "ตรงกับความจริง ณ ตอนนี้ + ร่องรอย"

- บรรทัดแรก: วันที่/เวลาอัปเดตล่าสุด (ICT)
- โค้ดหลัก: path ที่เกี่ยว (FE/BE/permission/migration)
- เนื้อหาเป็น bullet ต่อข้อ พร้อม 🔴 นำหน้าข้อที่ **ผิดแล้วพังเงียบ ๆ** (อ่านก่อนแก้)
- 🔴 ข้อความที่ถูกแทน **ห้ามลบ** — mark superseded แล้วเขียนของใหม่ต่อ (เห็นประวัติการเปลี่ยน)
- จบด้วยตาราง **Commit provenance**: `เวลา | Commit hash | เนื้อหา | Reference files`
  — เพิ่มแถวทุกครั้งที่ feature เปลี่ยน

### 3.3 `CLAUDE.md` — พื้นที่มีจำกัด ใช้ประหยัด

- Architecture Decisions: **1 บรรทัดต่อเรื่อง** โยงลิงก์ไป `docs/decisions/` เสมอ
- feature ที่มี doc แล้วไม่ต้องเล่าซ้ำ — ลิงก์
- ตัวหนา + 🔴 สงวนให้ "ผิดแล้วพังเงียบ ๆ" เท่านั้น ไม่งั้นสัญญาณหลอก

---

## 4. กับดักที่เจอจริง (เจอทุกครั้งที่ commit memory)

1. **`originSessionId` ใน frontmatter** — harness ใส่ให้เองทุกไฟล์ และบางทีซ้อนใต้ `metadata:`
   (เยื้อง) ⇒ ก่อน commit ต้อง grep แบบ **ไม่ anchor `^`**:
   ```bash
   grep -rn "originSessionId" memory/
   ```
   เจอ = ลบทิ้ง (metadata ส่วนบุคคล ไม่ลง repo)
2. **Secret / path ภายใน** — ห้ามลงไฟล์ที่ git track แม้แต่ตัวเดียว; ย้ายไป `memory/private/`
   หรือ `docs/private/` แล้วอ้างด้วย pointer
3. **ลืม `MEMORY.md`** — index คือสิ่งเดียวที่ session ถัดไปโหลด ไฟล์ที่ไม่ถูก index = ของตาย
4. **merge หลาย branch** — ชนกันที่ `memory/MEMORY.md` เสมอ (ไฟล์ fact แยกกันไม่ชน)
   แก้ index ตอน merge เท่านั้น
5. **`Date.now()` ในตัวอย่างโค้ด?** — ไม่เกี่ยวกับ doc แต่จดไว้: ถ้า fact พูดถึงสิ่งที่เทียบเวลา
   ระบุให้ชัดว่าธงคิดฝั่งไหน (ดู `memory/expired-flag-computed-server-side.md`)

---

## 5. ตรวจก่อน commit (คำสั่งตรง ๆ)

```bash
# prettier ไฟล์เอกสารที่แตะ (repo บังคับ format ด้วย)
pnpm exec prettier --check CLAUDE.md docs/... memory/...

# metadata ส่วนบุคคล (ต้องว่าง)
grep -rn "originSessionId" memory/ docs/

# secret หลุดมาไหม (สแกนหยาบ ๆ)
grep -rniE "api[_-]?key|secret|password" memory/ docs/ | grep -v private

# status — ไฟล์ doc/memory ต้องไม่ค้าง untracked ข้าม session
git status --short
```

---

## 6. ตัวอย่าง end-to-end (จากงานจริง 2026-08-28)

งาน: หน้ารายการ/รายละเอียดผู้ใช้บน ops console + เติม/ลดเครดิต + ขยายเวลาหมดอายุ

1. **feature doc** — `docs/features/feature-administrator-operations.md` มีอยู่แล้ว
   ⇒ เพิ่ม bullet รอบใหม่ใน section "ผู้ใช้งาน" (พร้อม mark ข้อความเก่าที่ superseded)
   + เพิ่มแถวในตาราง Commit provenance
2. **decision ใหญ่รวมอยู่ใน** `docs/decisions/administrator-rbac.md` อยู่แล้ว
   ⇒ `CLAUDE.md` เพิ่ม "รอบสิบสาม" 1 ย่อหน้าต่อท้าย bullet เดิม + ลิงก์ feature doc
3. **memory ใหม่ 2 ไฟล์** (gotcha ที่เจอระหว่างทำ):
   - `base-ui-select-value-needs-items-on-root.md` — SelectValue โชว์ raw value จนกว่าส่ง items ที่ root
   - `expired-flag-computed-server-side.md` — ธง "หมดอายุ" ต้องคิดฝั่ง server เพราะ render เรียก `Date.now()` ไม่ได้
   + append ข้อมูลเสริมลงไฟล์เดิมที่เกี่ยว (`admin-credit-adjustment-preserves-wallet-breakdown.md`)
   + เพิ่ม 3 บรรทัดใน `MEMORY.md`
4. **commit เดียวจบ**: `docs(admin): เอกสาร+memory รอบ users/credit ...`

---

## 7. ข้อห้ามสรุป

- ❌ ลบเอกสาร/decision เก่าที่ถูกแทน (mark superseded เท่านั้น)
- ❌ TODO เปล่าไม่มี scope — format คือ `TODO(<scope>): ...`
- ❌ ข้ามขั้น prettier สำหรับ .md (pre-commit hook จะบล็อกเอง แก้ก่อนถึงจะ commit ผ่าน)
- ❌ เขียน fact ยาวเป็นความเรียงใน memory — 1 fact/ไฟล์ 30 วินาที; ใหญ่ไป docs/
- ❌ ปล่อย doc/memory ค้าง untracked ข้าม session