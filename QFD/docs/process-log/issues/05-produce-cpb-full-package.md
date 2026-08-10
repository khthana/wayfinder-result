# ผลิต: แพ็กเกจ CP-B ให้ครบ 4 ชิ้น (reference exemplar)

Type: task
Status: resolved
Blocked by: 04

## Question

เติมแพ็กเกจ deliverable ของ **CP-B (House of Quality + QR)** ให้ครบทั้ง 4 ชิ้นตามแม่แบบที่ lock แล้ว — ให้เป็น **ตัวอย่างอ้างอิงทองคำ (gold reference)** ที่ทุก checkpoint ที่เหลือจะ replicate ตาม.

Student Guideline เติมเต็มแล้วใน [`artifacts/04-template-cp-b-house-of-quality.md`](../artifacts/04-template-cp-b-house-of-quality.md); ticket นี้เติมอีก 3 ชิ้นให้ครบ (อ้าง criteria list C1–C7 เดิม, ใช้ตัวอย่าง EEG SleepInsight เดิม):

1. **Advisor Guideline (CP-B)** เต็ม: checklist ตรง C1–C7, คำถามกระตุ้น Socratic ต่อเกณฑ์, red flags, จุดดันกลับ, เชื่อมขั้นก่อน/หลัง
2. **Rubric (CP-B)** เต็ม: ตาราง C1–C7 × 4 ระดับ พร้อม descriptor ทุกช่อง + น้ำหนัก (traceability/process หนักสุด) — descriptor ผูกกฎ RTM เชิงกล
3. **Self-check (CP-B)** เต็ม: (i) Checklist ครบ C1–C7 + (ii) LLM Prompt ฉบับสมบูรณ์ที่ตรวจ traceability CR→TC→QR

**Output:** ไฟล์แพ็กเกจ CP-B ครบ 4 ชิ้น (linked asset) — เป็นแม่แบบจริงที่ replicate ได้.

**หลังเสร็จ:** graduate การผลิต CP-A / CP-C / CP-D / CP-E / CP-F (แต่ละอันเป็น task ticket, mirror CP-B) ออกจาก fog. เป็น execution (task) ตาม execution-override ของ effort นี้.

## Answer

Asset: [`artifacts/05-package-cp-b-house-of-quality.md`](../artifacts/05-package-cp-b-house-of-quality.md) — แพ็กเกจ CP-B ครบ 4 ชิ้น (gold reference)

เติม 3 projection ที่เหลือให้เต็มตามแม่แบบที่ lock (ticket 04), อ้าง criteria C1–C7 + ตัวอย่าง EEG SleepInsight:
- **Advisor Guideline:** สิ่งที่ต้องเห็น (map C1–C7) + คำถาม Socratic ต่อเกณฑ์ + red flags 6 ข้อ + จุดดันกลับ + เชื่อม CP-A/S6
- **Rubric:** ตาราง C1–C7 × 4 ระดับ พร้อม descriptor ทุกช่อง + น้ำหนัก 100 คะแนน (Traceability/Verifiability C3/C4/C7 = 50%, process C2/C5/C6 = 40%, ครบ C1 = 10%); จำนวนไม่ใช่ประตูตาย
- **Self-check:** (i) checklist C1–C7 ติ๊กเร็ว + (ii) LLM prompt ฉบับสมบูรณ์ตรวจ traceability CR→TC→QR รายองค์

มีหมายเหตุวิธี replicate ไป checkpoint อื่น (เปลี่ยนแค่ Cn, คงโครง 4 projection, เดินตัวอย่าง EEG ต่อ)

**graduate:** เปิดการผลิต CP-A/C/D/E/F ได้แล้ว (mirror CP-B) — ยังคงเป็น fog รวม รอ graduate เป็น task ticket ทีละอันตามกำลัง
