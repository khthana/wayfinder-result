# Design: แทร็ก Y4 ข้าม VoC — รากทดแทนของ traceability

Type: grilling
Status: resolved
Blocked by: 02

## Question

แทร็กปี 4 ข้ามขั้น VoC (นักศึกษาได้หัวข้อแล้ว ไม่ย้อนกลับทำ VoC ใหม่). แต่ **VoC คือรากของ traceability** — ถ้าไม่มีราก สายโซ่ทั้งเส้นลอย และ traceability คือหลักวิศวกรรมที่อาจารย์ต้องการสอนพอดี ต้องตัดสินใจว่า:

1. **artifact ทดแทน VoC** ที่ Y4 ต้องทำแบบเบา ๆ แต่ documented คืออะไร — เช่น "Assumed Stakeholder + Problem Statement + สมมุติฐานความต้องการ" ที่ reverse-engineer จากหัวข้อที่มีอยู่ ให้ยังมีรากให้ QR/HoQ อ้างได้
2. Y4 **re-enter pipeline (จาก 02) ตรงขั้นไหน** และข้ามอะไรได้บ้างโดยไม่ทำลาย traceability spine
3. ต้องมี guideline/rubric/self-check **เวอร์ชันแยกของ Y4** หรือแค่ note ความต่างบนของ Full track (กระทบจำนวน ticket ที่จะ graduate)
4. จะกันไม่ให้ Y4 กลายเป็น "เขียนย้อนหลังให้ดูดี" (justification ปลอม) อย่างไร — เกณฑ์ความสมเหตุสมผลของรากทดแทน

**หลังตัดสิน:** บันทึกลง Decisions ว่าแต่ละ deliverable ต้องมีเวอร์ชัน Y4 อย่างไร (กระทบการ graduate fog)

ใช้ skill `/grilling` + `/domain-modeling`

## Answer

Y4 Addendum asset: [`artifacts/11-addendum-cp-a-y4-voc-skip.md`](../artifacts/11-addendum-cp-a-y4-voc-skip.md)

ตัดสินร่วมกับอาจารย์ (grilling 4 ข้อ):
1. **รากทดแทน = Assumed Requirements + หลักฐานต่อ CR** — ทุก assumed CR อ้างหลักฐาน ≥1 (feature ที่ทำแล้ว / proxy user / product คู่เทียบ / งานวิจัย)
2. **Re-entry = CP-A ย่อ → กลับเข้า CP-B เดินหน้าปกติ** — ต่างจาก Full แค่ CP-A; **CP-B–F ใช้แม่แบบเดิม 100%**. ถ้าของที่สร้างไว้ขัด design forward = บันทึกเป็น gap ให้แก้/อธิบาย
3. **รูปแบบ = Addendum เดียว แทนเฉพาะ CP-A** (ไม่ทำชุด Y4 แยกครบ)
4. **กัน reverse-justify (เลือกหลายข้อ):** บังคับอ้างหลักฐาน+ติดป้าย "สมมติ" ทุก CR · rubric มีเกณฑ์ **ความซื่อตรง** (พราง assumption เป็น VoC = ตก) · **assumption log + ความเสี่ยง**. *(validate proxy user = optional โบนัส ไม่บังคับ)*

Addendum ครบ 4 projection (criteria YA1–YA7) + ตัวอย่าง Travel Planning AI Chatbot. **ปิดช่องว่างแทร็ก Y4 — traceability มีรากที่ honest.**
