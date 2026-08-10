# ปรับปรุงคู่มือให้อ่านง่ายขึ้น + แยกเป็น 2 เล่ม (นักศึกษา / อาจารย์)

Type: task
Status: resolved
Blocked by: 12

## Question

อาจารย์ให้ feedback ว่า handbook v1 (ticket 12) "อ่านยากมาก เพราะเหมือนเอาที่สรุปจากที่คุยกันมาดื้อ ๆ เลย น่าจะต้องเขียนขยายความอธิบายรายละเอียดเพิ่ม" ต้องแก้ 3 เรื่อง:

1. **เขียนใหม่ให้อ่านง่าย** — เรียงลำดับใหม่เป็น "ทำไมสำคัญ → แนวคิด → ตัวอย่างเป็นเรื่องเล่า → deliverable → เกณฑ์" แทนขึ้นต้นด้วยตารางเกณฑ์แบบ spec dump
2. **แยกเป็น 2 เล่ม** — คู่มือนักศึกษา (ตัดส่วนอาจารย์ออก) กับคู่มืออาจารย์ที่ปรึกษา (คำถามกระตุ้น/red flags + rubric)
3. **ลดการยึดติดเทอม 1/2/3** ในการนำเสนอ (ย้ายไปเป็น reference table ครั้งเดียว ไม่ทำเป็น subtitle ทุกบท)

ทำ exemplar (บทที่ 1–3) ก่อนแล้วให้อาจารย์ react ยืนยันโครง ก่อนขยายไปบทที่เหลือทั้งหมด ตาม advisor recommendation

## Answer

Assets: [`handbook-student.md`](../../../handbooks/handbook-student.md) + `.docx`, [`handbook-advisor.md`](../../../handbooks/handbook-advisor.md) + `.docx`. Exemplar draft (ก่อน propagate): [`handbook-exemplar-v2.html`](../superseded/handbook-v2-exemplar-preview.html) — artifact ที่อาจารย์ดูและ confirm ก่อน.

**ตัดสิน (grilling 2 ข้อ กับ exemplar):**
1. โครงเรื่องเล่า+กล่องอ้างอิงท้ายบท → ใช้ได้ แต่ต้อง**แยกเล่ม** (ไม่ใช่กล่องอาจารย์ inline ในเล่มเดียว) + **ลดเทอม** + **ขยายเพิ่มอีก**
2. Rubric weighting ข้าม 3 เทอม → **ถ่วงน้ำหนักเท่ากันทุก checkpoint** (~16.7% × 6) — บันทึกไว้ท้ายคู่มือทั้งสองเล่ม (Student ท้ายบทที่ 8, Advisor เป็นบทที่ 9 แยก)

**คู่มือนักศึกษา** (`handbook-student`, 11 บท): ภาพรวม → วิธีใช้คู่มือ → CP-A..F (เขียนใหม่ทั้งหมด: ทำไมสำคัญ/แนวคิดขยายความ/เดินตัวอย่าง EEG SleepInsight เป็นเรื่องเล่าต่อเนื่องพร้อมเหตุผลการตัดสินใจ/deliverable/เกณฑ์คะแนน/self-check) → แทร็กไม่มี VoC (Y4) → **ภาคผนวก: ตัวอย่างย่อจาก 2 โครงงานอื่น** (API Gateway for LLM Services = ซอฟต์แวร์ล้วน, AIoT LPG Monitoring = IoT) — ปิด fog "worked example ชุด 2–3"

**คู่มืออาจารย์ที่ปรึกษา** (`handbook-advisor`, 9 บท): หลักการรวม → CP-A..F (สรุปย่อ + คำถามกระตุ้น + red flags + จุดดันกลับ + rubric ชุดเดียวกับนักศึกษา) → แทร็ก Y4 → **การรวมคะแนนและการให้คะแนนที่สม่ำเสมอ** (rubric weighting section)

ตรวจสอบ docx ทั้งสองเล่มแล้ว: TOC, page break, ตาราง, ข้อความไทย ไม่มีปัญหา (Student: 15 ตาราง 10 page-break; Advisor: 8 ตาราง 9 page-break)

**graduate:** ปิด fog "worked example ชุด 2–3" (ทำเป็นภาคผนวกย่อแล้ว) และ "rubric weighting ข้ามเทอม" (ตัดสินแล้ว: เท่ากันทุก checkpoint)
