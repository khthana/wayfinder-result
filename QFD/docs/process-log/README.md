# Process log — ที่มาของคู่มือ

โฟลเดอร์นี้คือ **บันทึกกระบวนการ** ตอนสร้างแพ็กเกจ ไม่ใช่เอกสารที่แจกนักศึกษา
(เอกสารที่แจกอยู่ที่ [`handbooks/`](../../handbooks/))

เก็บไว้เพราะยังใช้ประโยชน์ได้ 2 อย่าง: ตามรอยว่า**ทำไม**ถึงตัดสินใจแบบนี้ และหยิบ**แพ็กเกจ CP รายชิ้น**ไปแจกแยกเป็น checkpoint ได้โดยไม่ต้องแกะจากเล่มรวม

| โฟลเดอร์ | คืออะไร |
| --- | --- |
| [`map.md`](map.md) | ภาพรวม effort — ปลายทาง, ข้อตกลง, และสรุปการตัดสินใจทุกใบ |
| [`inputs/`](inputs/) | โจทย์ตั้งต้นและรายการหัวข้อนักศึกษาที่ใช้เลือกตัวอย่าง |
| [`issues/`](issues/) | ticket 12 ใบ — ปัญหาที่ตั้ง + ข้อสรุปที่ปิด |
| [`artifacts/`](artifacts/) | ผลผลิตของแต่ละ ticket — research summary, stage model, แพ็กเกจ CP-A…F รายชิ้น |
| [`superseded/`](superseded/) | ฉบับที่ถูกแทนที่แล้ว เก็บไว้อ้างอิงประวัติ |

เลขนำหน้าไฟล์ใน `artifacts/` ตรงกับเลข ticket ใน `issues/` (เช่น `05-package-cp-b-*.md` มาจาก `05-produce-cpb-*.md`)

## แผนที่ artifact → checkpoint

| ไฟล์ | ครอบคลุม |
| --- | --- |
| `01-research-qfd-fd-foundations.md` | งานวิจัย/แหล่งอ้างอิงตั้งต้น (VoC→CR→HoQ→TC→QR→FD→C→T) |
| `02-stage-model-s0-s10.md` | stage model 11 ขั้น + ID scheme + จุดตรวจ CP-A…F |
| `04-template-cp-b-house-of-quality.md` | แม่แบบ deliverable 4 ชิ้น (นำร่องที่ CP-B) |
| `05-package-cp-b-house-of-quality.md` | **gold reference** — CP-B ครบ 4 ชิ้น |
| `06-package-cp-a-requirements-foundation.md` | CP-A (S0–S3) |
| `07-package-cp-c-design.md` | CP-C (S6–S7) |
| `08-package-cp-d-build-progress.md` | CP-D (S8) |
| `09-package-cp-e-verification.md` | CP-E (S9) |
| `10-package-cp-f-validation-defense.md` | CP-F (S10) |
| `11-addendum-cp-a-y4-voc-skip.md` | Addendum แทน CP-A สำหรับแทร็ก Y4 |

## หมายเหตุเรื่องข้อมูลส่วนบุคคล

โฟลเดอร์นี้**ไม่มีข้อมูลส่วนบุคคลของนักศึกษาแล้ว** — จัดการไปเมื่อ 2026-08-10 ตอนเตรียมขึ้น GitHub:

- ไฟล์ proposal ต้นฉบับ (.docx 3 ไฟล์ มีชื่อ-นามสกุล) นำออกจากโปรเจกต์ อาจารย์เก็บต้นฉบับไว้นอก repo
- directory listing ดิบ (`file.txt` เดิม) แปลงเป็น [`inputs/student-topics-cepp68.md`](inputs/student-topics-cepp68.md) โดยตัดชื่อ-นามสกุลและรหัสนักศึกษาออก เหลือเฉพาะรหัสหัวข้อกับชื่อเรื่อง
- `.gitignore` ที่ราก repo บล็อก `CEPP68-*.docx` และ `*Proposal*.docx` กันเผลอ commit ซ้ำ

ถ้าจะเพิ่มไฟล์จากงานจริงของนักศึกษาเข้ามาอีก ให้ปิดบังชื่อก่อนเสมอ
