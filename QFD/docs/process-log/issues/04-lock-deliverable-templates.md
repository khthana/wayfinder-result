# Lock: template ของ deliverable 4 ชิ้น (นำร่อง 1 checkpoint)

Type: prototype
Status: resolved
Blocked by: (none)

## Question

ก่อนผลิต deliverable ทุกขั้น (11 ขั้น × 4 ชิ้น = student guideline / advisor guideline / rubric / self-check) ต้อง **lock "แม่แบบ" ของแต่ละชิ้นให้สม่ำเสมอก่อน** ไม่งั้นผลิตออกมาไม่เป็นระบบเดียวกัน.

วิธี: **prototype เต็มชุด 4 ชิ้นสำหรับ 1 checkpoint จริง** (เสนอ CP-B House of Quality — เป็นขั้นที่ยากและมี 4 ก้าวย่อย ครบทุกความท้าทาย) โดยใช้ **หัวข้อจริงจากรายการหัวข้อนักศึกษา (`inputs/student-topics-cepp68.md`)** เป็นตัวอย่าง แล้วให้อาจารย์ react เพื่อ lock:

1. **Student guideline** ต่อขั้น มีหัวข้ออะไรบ้าง (เช่น: วัตถุประสงค์ขั้น / ทำอะไรทีละก้าว / **จำนวนที่ควรมี** เช่นกี่ TC / หัวข้อที่ต้องครบ / ตัวอย่างจริง / ช่องกรอก ID / ข้อผิดที่พบบ่อย) — รูปแบบ/ความยาว
2. **Advisor guideline** ต่อขั้น: อาจารย์ที่ปรึกษาควรถาม/ตรวจอะไร, red flags, คำถามกระตุ้น
3. **Rubric** ต่อขั้น: สเกลกี่ระดับ, เกณฑ์ (เน้น process + traceability), น้ำหนักคะแนน, descriptor แต่ละระดับ
4. **Self-check prompt** ต่อขั้น: โครงสร้าง prompt ให้นักศึกษาใช้ตรวจงานตัวเอง + ตรวจ traceability

**Output:** ไฟล์ prototype 4 ชิ้นสำหรับ CP-B (linked asset) + แม่แบบเปล่า (blank template) ของแต่ละชิ้นที่ lock แล้ว.

**หลัง lock:** graduate การผลิต deliverable ของ checkpoint ที่เหลือ (A, C, D, E, F) เป็น ticket ทีละ batch โดยเทียบแม่แบบนี้.

ใช้ skill `/prototype`. อ้างอิง: [`artifacts/02-stage-model-s0-s10.md`](../artifacts/02-stage-model-s0-s10.md)

## Answer

Template asset: [`artifacts/04-template-cp-b-house-of-quality.md`](../artifacts/04-template-cp-b-house-of-quality.md)

**LOCKED โครงสร้างแม่แบบ (prototype + react กับอาจารย์):**

1. **โครง "spine-first: criteria list เดียว → 4 projection"** — แต่ละขั้นมี criteria list เดียว (เช่น CP-B = C1–C7) แล้วฉายเป็น student guideline / advisor guideline / rubric / self-check ทำให้ 44 ชิ้นสม่ำเสมอและ trace ถึงกันเอง
2. **Student Guideline sectioning (LOCKED):** ทำไป-ทำไม / ส่งอะไร / ทำทีละก้าว+ตัวอย่างจริง / จำนวน (ช่วงปกติ + ให้อธิบายถ้าเกิน ไม่ใช่ประตูตาย) / ข้อผิดบ่อย / เช็คก่อนส่ง
3. **Self-check = ทั้ง 2 แบบ:** Checklist (ติ๊กเร็วก่อนส่ง) + LLM Prompt (ตรวจ traceability เชิงลึกด้วย AI)
4. **Rubric = 4 ระดับ** (4/3/2/1) สร้างบนกฎ RTM เชิงกล; น้ำหนัก **traceability/process หนักสุด** > ความถูกต้องเนื้อหา > ความครบจำนวน
5. **Exemplar เต็ม:** Student Guideline ของ CP-B (House of Quality) เดินตัวอย่างจริง EEG SleepInsight ครบ CR→TC→roof→target→QR (5 องค์); อีก 3 projection ยังเป็นโครงร่าง
6. **Blank shell** (Part 6) = โครงที่ทุกขั้นจะกรอก

**graduate ต่อ:** production ของ deliverable graduate ได้แล้ว. เสนอทำ **CP-B ให้ครบ 4 ชิ้นก่อน (reference exemplar)** แล้วค่อย replicate ไป CP-A/C/D/E/F. CR ในตัวอย่าง reverse-engineer = ซ้อมโจทย์ Y4 (ป้อน ticket 03)
