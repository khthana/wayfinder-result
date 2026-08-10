# Lock: stage model + artifact ต่อขั้น + traceability spine (แทร็ก Full/Y3)

Type: grilling
Status: resolved
Blocked by: 01

## Question

เอาผลจาก research (01) มา **lock โมเดลกระบวนการฉบับ canonical (แทร็ก Full / ปี 3)** ให้เป็นแกนของทั้งแพ็กเกจ — ตัดสินใจร่วมกับอาจารย์ว่า:

1. **รายการขั้นตอน (stage list) สุดท้าย** มีกี่ขั้น อะไรบ้าง ชื่อเรียกแต่ละขั้น (เช่น 01 VoC → 02 จัดกลุ่ม/ถ่วงน้ำหนักความต้องการ → 03 House of Quality → 04 Quantifiable Requirements → 05 Functional Decomposition → 06 Architecture/Design → 07 Implementation → 08 Verification) — จำนวนขั้นต้อง "พอดี" กับ 3 เทอม
2. **artifact ที่นักศึกษาต้องส่งต่อขั้น** (เพราะไม่มีเทมเพลตเดิม): แต่ละขั้นได้ตาราง/ไดอะแกรม/เอกสารอะไร รูปแบบไหน
3. **traceability spine**: field/รหัสอ้างอิงที่ร้อยทุกขั้น (เช่น CR-01 → QR-03 → F-02 → C-05 → T-04) เพื่อให้ rubric และ self-check ตรวจได้
4. ขั้นไหนอยู่เทอมไหน (mapping กับ 3 เทอม + จุดส่งงาน/ตรวจ)

**หลัง lock:** clear fog "stage list" และ "โครงสร้าง artifact" ออกจาก Not yet specified แล้ว graduate แต่ละขั้นเป็น 4 ticket (student guideline / advisor guideline / rubric / self-check) — สร้างทีละ batch ตามกำลัง

ใช้ skill `/grilling` + `/domain-modeling`

## Answer

Locked model asset: [`artifacts/02-stage-model-s0-s10.md`](../artifacts/02-stage-model-s0-s10.md)

ตัดสินร่วมกับอาจารย์ (grilling 5 ข้อ):

1. **Stage list = 11 ขั้น S0–S10** คงความละเอียด (หลัก "แยกเรียนรู้ แต่รวมจุดส่ง"); S0–S3 แยกเป็นก้าวเรียน แต่ส่งรวมชิ้นเดียว
2. **House of Quality (S4) = สเตจเดียว มี 4 ก้าวย่อย** (4a HOWs→ISO 25010 / 4b relationship 9-3-1 / 4c roof trade-off เน้น AI / 4d target & priority) — ส่ง HoQ ครบใบเดียว
3. **ID scheme = 6 ระดับครบ** `CR → TC → QR → F → C → T` (แยก TC กับ QR เพราะ "ตั้งเป้าเป็นตัวเลข" เป็นทักษะแยก)
4. **เส้นแบ่งเทอม:** เทอม 1 = S0–S7 (จบที่ Architecture, ออกแบบบนกระดาษครบ = proposal); เทอม 2–3 = S8 build → S9 verify → S10 defense
5. **จุดตรวจ (graded checkpoint):** เทอม 1 มี 3 จุด — CP-A (S0–S3 Requirements Foundation) / CP-B (S4–S5 HoQ+QR) / CP-C (S6–S7 Design); เทอม 2–3 — CP-D build / CP-E verification+RTM / CP-F validation & defense

**graduate ต่อ:** fog "stage list" + "artifact structure" ปิดแล้ว (อยู่ใน stage-model asset). deliverable 4 ชิ้นต่อ checkpoint ยังไม่ graduate เต็ม — ต้อง lock "template ของ deliverable" ก่อน (ticket ใหม่) เพื่อไม่ให้ผลิตแบบไม่สม่ำเสมอ. ticket "Y4 ข้าม VoC" ปลดล็อกแล้ว
