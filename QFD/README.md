# QFD → Functional Decomposition สำหรับวิชาโครงงานวิศวกรรมคอมพิวเตอร์

แพ็กเกจกระบวนการสำหรับวิชา **Senior Project** (เตรียมโครงงานฯ 01076014/01076016 + วิชาโครงงาน 3 เทอม)
ภาควิชาวิศวกรรมคอมพิวเตอร์ สจล. — เปลี่ยนการวัดผลจาก *output-centric* ไปเป็น **process + traceability centric**

สายโซ่หลักที่ยึดตลอดเล่ม:

```
VoC → CR → House of Quality → QR → Functional Decomposition → Component → Test
```

รองรับ 2 แทร็ก — **Y3 (Full Function)** เริ่มจาก VoC และ **Y4 (ข้าม VoC)** สำหรับนักศึกษาที่ได้หัวข้อแล้ว

---

## เอกสารหลัก (ของที่แจกจริง)

| ไฟล์ | สำหรับ | เนื้อหา |
| --- | --- | --- |
| [`handbooks/handbook-student.md`](handbooks/handbook-student.md) · [`.docx`](handbooks/handbook-student.docx) | นักศึกษา | 11 บท ตาม S0–S10 + ภาคผนวก 2 ตัวอย่างเสริม |
| [`handbooks/handbook-advisor.md`](handbooks/handbook-advisor.md) · [`.docx`](handbooks/handbook-advisor.docx) | อาจารย์ที่ปรึกษา | 9 บท — ตรวจอะไร ถามอะไร ให้คะแนนอย่างไร |

ทุก checkpoint มีครบ 4 ชิ้น: **Student Guideline · Advisor Guideline · Rubric · Self-check (Checklist + LLM Prompt)**

Worked example เดินจริง: **EEG SleepInsight** ครบทั้ง 11 ขั้น (+ Travel Chatbot สาธิตแทร็ก Y4)

## Checkpoint

| Checkpoint | ขั้น | ส่งอะไร | เทอม |
| --- | --- | --- | --- |
| CP-A · Requirements Foundation | S0–S3 | framing + VoC log + KJ + weighted `CR` | 1 |
| CP-B · House of Quality + QR | S4–S5 | HoQ ครบใบ + `QR` spec | 1 |
| CP-C · Design | S6–S7 | function tree (`F`) + architecture/module map (`C`) | 1 |
| CP-D · Build progress | S8 | ระบบทำงานได้ + experiment log + model card | 2 |
| CP-E · Verification & RTM close-out | S9 | `T` ครบ + RTM ปิดวง + acceptance report | 3 |
| CP-F · Validation & Defense | S10 | demo + results-vs-target + traceability walkthrough | 3 |

แทร็ก Y4 ใช้ [Addendum](docs/process-log/artifacts/11-addendum-cp-a-y4-voc-skip.md) แทน CP-A แล้วเข้า CP-B ตามปกติ (CP-B–F ไม่แก้)

## โครงสร้างโปรเจกต์

```
handbooks/            เอกสารหลัก 2 เล่ม (Markdown = source of truth, .docx = ฉบับแจก)
figures/              ภาพประกอบ House of Quality (ตัวอย่าง EEG SleepInsight)
docs/process-log/     บันทึกกระบวนการที่มาของคู่มือ — ดู README ในโฟลเดอร์
```

## แปลง Markdown → Word

```bash
pandoc handbooks/handbook-student.md -o handbooks/handbook-student.docx --toc --toc-depth=2
```

> `\newpage` ใช้กับ docx writer ของ pandoc ไม่ได้ (render เป็น literal text) — ใช้ raw-openxml block แทน:
> ` ```{=openxml}<w:p><w:r><w:br w:type="page"/></w:r></w:p>``` `

## สถานะ

เสร็จสมบูรณ์ พร้อมแจกจริง (2026-07-18) — ไม่มีงานค้าง รายละเอียดการตัดสินใจทั้งหมดอยู่ใน [`docs/process-log/map.md`](docs/process-log/map.md)
