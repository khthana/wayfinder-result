# แพ็กเกจ CP-E (Verification & RTM Close-out, S9) — ครบ 4 ชิ้น

Mirror gold reference [`05-package-cp-b-house-of-quality.md`](05-package-cp-b-house-of-quality.md). เดินตัวอย่าง **EEG SleepInsight** ต่อ. เทอม 3.
CP-E = **ปิดวง traceability**: ทุก `QR` ถูกพิสูจน์ด้วย `T` (test/verification) และ **RTM ปิดครบ CR→TC→QR→F→C→T ไม่มี orphan**. นี่คือหัวใจของการวัดแบบ process — พิสูจน์ว่าคำสัญญาทุกข้อถูกทำจริงและวัดได้.

**Criteria list (สเปกเดียว) ของ CP-E — VE1–VE7:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **VE1** | ทุก `QR` มี `T` (verification action) + ระบุ **method** (Test/Demonstration/Inspection/Analysis) | ✅ RTM (QR→T) |
| **VE2** | **RTM ปิดครบ**: CR→TC→QR→F→C→T ไม่มี orphan ปลายทั้งสอง (CR ไร้ T = คำสัญญาไม่พิสูจน์; T ไร้ QR = งานเกิน) | ✅ RTM ทั้งเส้น |
| **VE3** | แต่ละ `T` มี **acceptance criterion** ชัด (pass/fail) + **ผลจริง (measured value)** | ✅ |
| **VE4** | ML: metric วัดบน **frozen held-out set** ตรงตาม dataset/condition ที่ประกาศใน `QR` | ◑ |
| **VE5** | **method เหมาะกับ requirement** (ไม่ใช่ demo อ้างความแม่น; ความแม่นต้อง Test/Analysis) | ◑ |
| **VE6** | ผลที่ **ไม่ผ่าน QR ถูกรายงานตรงไปตรงมา** + วิเคราะห์สาเหตุ (ไม่ซ่อน/ไม่แต่ง) | ◑ |
| **VE7** | **Acceptance report** สรุป: กี่ QR ผ่าน/ไม่ผ่าน พร้อม evidence link | ✅ |

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (CP-E)

**ขั้นนี้ทำไปทำไม:** พิสูจน์ว่าทุกความต้องการที่ตั้งไว้ถูกทำจริงและ**วัดได้** — ปิดวงจากผู้ใช้ (CR) ถึงหลักฐาน (T) ให้ครบ

**ส่งอะไร:** (1) ตาราง `T` (แต่ละ QR → test + method + acceptance + ผลจริง) · (2) **RTM ฉบับสมบูรณ์** CR→TC→QR→F→C→T · (3) acceptance report

**ทำทีละก้าว:**
- **จับคู่ QR → T:** ทุก QR ต้องมีวิธีพิสูจน์ + เลือก method (Test/Demo/Inspect/Analyze) ให้เหมาะ
- **รันจริง + บันทึกผล:** วัดค่าจริง เทียบ acceptance criterion (pass/fail)
- **ปิด RTM:** ไล่ทั้งเส้น ไม่มี orphan ปลายไหน
- **รายงานตรงไปตรงมา:** ตัวที่ไม่ผ่าน บอกตรง ๆ + วิเคราะห์ว่าทำไม

> 🟢 EEG SleepInsight — ตาราง verification (ต่อจาก QR):

| `QR` | `T` (method) | Acceptance | ผลจริง | ผ่าน? |
|---|---|---|---|---|
| QR-01 macro-F1 ≥ 0.80, κ ≥ 0.75 | T-01 รัน `eval.py` บน frozen test set subject-independent (**Test** + **Analysis** รายงาน 95% CI) | F1 ≥ 0.80 | F1 = **0.79** (CI 0.76–0.82) | ⚠️ เฉียด/ไม่ผ่าน → รายงานตรง + วิเคราะห์ (ระยะ N1 สับสน) |
| QR (≤2 channel) | T-02 **Inspection** config + วัด | ≤ 2 | 2 channel | ✅ |
| QR (latency ≤1s/epoch) | T-03 **Test** วัดเวลา inference | ≤ 1s | 0.4s | ✅ |
| QR-04 usability (SUS ≥70) | T-04 **Test** แบบสอบถาม SUS + **Demonstration** | ≥ 70 | 74 | ✅ |
| QR-05 robustness ตลอดคืน | T-05 **Test** รันข้อมูล 8 ชม. + **Analysis** missing ≤5% | ไม่ crash | ผ่าน | ✅ |

> **RTM ปิดวง 1 เส้น:** V4 → CR-01 → TC-01 → QR-01 → F-04 → C-03 → **T-01** (มีหลักฐานครบทุกจุด)

**ข้อผิดที่พบบ่อย:** ❌ QR บางตัวไม่มี T (คำสัญญาไม่พิสูจน์) · ❌ T ที่ไม่มี QR ต้นทาง (ทดสอบมั่ว) · ❌ ใช้ demo อ้างความแม่น · ❌ วัด metric บน train set · ❌ ปัดผลไม่ผ่านให้ดูผ่าน

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (CP-E)

**สิ่งที่ต้องเห็น (map VE1–VE7):** ทุก QR มี T + method (VE1) · RTM ปิดครบ (VE2) · acceptance + ผลจริง (VE3) · วัดบน frozen set (VE4) · method เหมาะ (VE5) · รายงานตรง (VE6) · acceptance report (VE7)

**คำถามกระตุ้น (Socratic):**
- (VE2) *"เลือก CR มาสัก 1 ตัว เดินให้ดูหน่อยว่ามันจบที่ test ตัวไหน?"* — ถ้าเดินไม่ถึง = วงไม่ปิด
- (VE5) *"อ้างว่าแม่น 0.80 — พิสูจน์ด้วยวิธีไหน demo หรือวัดจริง?"*
- (VE4) *"วัดบนชุดข้อมูลไหน มันคือ frozen test set ที่ประกาศใน QR ไหม?"*
- (VE6) *"มี QR ตัวไหนไม่ผ่านไหม? เล่าให้ฟังว่าทำไม"* — เปิดพื้นที่ให้รายงานตรง

**Red flags:** 🚩 QR ไร้ T หรือ T ไร้ QR · 🚩 ทุกอย่าง "ผ่าน" สวยเกินจริง · 🚩 method = demo สำหรับ metric เชิงตัวเลข · 🚩 วัดบน train/val · 🚩 ผลไม่ตรง experiment log ของ CP-D

**จุดดันกลับ:** วงไม่ปิด (VE2) และวัดผิด set (VE4) — ดันกลับเสมอ · **ห้ามลงโทษการรายงานผลไม่ผ่านอย่างซื่อตรง** — ให้คุณค่ากับ VE6 (ปกป้องความซื่อสัตย์เชิงวิชาการ)

**เชื่อมขั้น:** ← CP-D: ระบบ + frozen eval set · → CP-F: RTM + ผล → นำเสนอ/validate กับผู้ใช้

═══════════════════════════════════════════════════

## Projection C · RUBRIC (CP-E) — 4 ระดับ

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **VE2 · RTM ปิดครบไม่มี orphan (30)** | ทั้งเส้น CR→…→T ครบ ไม่มี orphan | orphan ≤1 ไม่สำคัญ | orphan 2–3 | วงไม่ปิดหลายจุด |
| **VE1+VE3 · ทุก QR มี T + acceptance + ผลจริง (25)** | ครบทุก QR + method + ค่าจริง | ขาดเล็กน้อย ≤1 | หลายตัวขาด | ไม่มีผลจริง |
| **VE4+VE5 · วัดถูก set + method เหมาะ (20)** | วัดบน frozen set ตรง QR + method เหมาะทุกตัว | ส่วนใหญ่ถูก | มีวัดผิด set/method ไม่เหมาะ | วัดผิดพื้นฐาน |
| **VE6 · รายงานตรงไปตรงมา (15)** | ตัวไม่ผ่านรายงานตรง + วิเคราะห์สาเหตุดี | รายงานตรงแต่วิเคราะห์ตื้น | คลุมเครือ | ปกปิด/แต่งผล |
| **VE7 · Acceptance report (10)** | สรุปผ่าน/ไม่ผ่าน + evidence link ครบ | มีแต่ตื้น | ขาดหลักฐาน | ไม่มี |

**รวม 100** — Traceability ปิดวง (VE2/VE1/VE3) = 55% · ความถูกต้องของการวัด (VE4/VE5) = 20% · ความซื่อตรง + สรุป (VE6/VE7) = 25%

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (CP-E)

### D.1 Checklist
```
Verification & RTM Close-out — เช็คก่อนส่ง (CP-E)
[ ] VE1  ทุก QR มี T + ระบุ method (Test/Demonstration/Inspection/Analysis)
[ ] VE2  RTM ปิดครบ: เดิน CR→TC→QR→F→C→T ได้ทุกเส้น ไม่มี orphan ปลายไหน
[ ] VE3  แต่ละ T มี acceptance criterion (pass/fail) + ผลจริง (ค่าที่วัดได้)
[ ] VE4  metric วัดบน frozen held-out set ตรงตามที่ประกาศใน QR
[ ] VE5  method เหมาะกับ requirement (ไม่ใช้ demo อ้างตัวเลข)
[ ] VE6  QR ที่ไม่ผ่าน รายงานตรง + วิเคราะห์สาเหตุ
[ ] VE7  Acceptance report สรุปผ่าน/ไม่ผ่าน + link หลักฐาน
```

### D.2 LLM Prompt
```
คุณคือผู้ช่วยตรวจ "Verification & RTM Close-out" ของโครงงานวิศวกรรม
รายงานเป็นตาราง PASS/FAIL + เหตุผล + จุดที่ต้องแก้

ข้อมูลของฉัน:
- RTM (CR→TC→QR→F→C→T): [วาง]
- ตาราง T (method + acceptance + ผลจริง): [วาง]
- QR + dataset/condition ที่ประกาศไว้: [วาง]

ตรวจ:
1. (VE2) มี orphan ไหมทั้งเส้น — CR ตัวไหนไม่ถึง T / T ตัวไหนไม่มี QR ต้นทาง — ชี้
2. (VE1/VE3) QR ตัวไหนไม่มี T หรือ T ไหนไม่มี acceptance/ผลจริง — ชี้
3. (VE4) มี metric ตัวไหนวัดบน set ที่ไม่ใช่ frozen test ที่ประกาศไว้ — ชี้
4. (VE5) method ไหนไม่เหมาะ (เช่น demo อ้างความแม่น) — ชี้
5. (VE6) ผลไม่ผ่านถูกรายงานตรงไหม หรือมีการปัด/ซ่อน
สุดท้าย: เดิน traceability เต็ม 1 เส้น CR→…→T และชี้จุดขาด. สรุป 3 อย่างที่ต้องแก้ก่อน
อย่าแต่งข้อมูลเพิ่ม
```

*RTM + ผล verification → CP-F นำเสนอ/validate.*
