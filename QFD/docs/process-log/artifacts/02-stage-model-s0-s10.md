# Stage Model (LOCKED) — กระบวนการ QFD → Functional Decomposition, แทร็ก Full / ปี 3

เอกสารแกน (canonical) ของทั้งแพ็กเกจ. guideline / rubric / self-check ทุกชิ้นยึดโครงนี้.
Locked: 2026-07-14 (ticket "Lock: stage model + artifact ต่อขั้น + traceability spine"). อ้างอิงหลักการ: [`01-research-qfd-fd-foundations.md`](01-research-qfd-fd-foundations.md).

---

## 1. รายการขั้นตอน (11 ขั้น S0–S10)

หลักออกแบบ: **"แยกเป็นขั้นเรียนรู้ แต่รวมเป็นจุดส่ง"** — สอน/ตรวจ process ละเอียด แต่คุมจำนวนจุดส่งงาน (graded checkpoint) ให้พอดี.

| # | ขั้น (stage) | สิ่งที่นักศึกษาทำ | Artifact ที่ส่ง | ID ที่เกิด |
|---|---|---|---|---|
| S0 | Project framing & stakeholder identification | ระบุปัญหา ผู้มีส่วนได้เสีย ผู้ใช้เป้าหมาย | Problem statement + stakeholder list | — |
| S1 | Voice of Customer capture | เก็บเสียงผู้ใช้ดิบ (สัมภาษณ์/สังเกต/แบบสอบถาม) | Raw VoC log | — |
| S2 | Affinity grouping & structuring | จัดกลุ่ม VoC ด้วย KJ/affinity → รายการความต้องการ | KJ/affinity diagram → `CR` list | `CR` |
| S3 | Prioritize customer requirements | ถ่วงน้ำหนักความสำคัญของ `CR` (direct 1–5 / 100-point; AHP = optional) | Weighted `CR` table | `CR` (+weight) |
| **S4** | **House of Quality** (มี 4 ก้าวย่อย — ดู §2) | สร้างบ้านคุณภาพครบใบ | HoQ ใบเดียว | `TC` |
| S5 | Quantifiable Requirements | แปลง `TC` เป็นเกณฑ์วัดผ่าน/ไม่ผ่าน | `QR` spec (metric + operator + threshold + dataset + condition) | `QR` |
| S6 | Functional Decomposition | แตกฟังก์ชันรวมเป็น function tree; ทุกโหนดชี้ขึ้นหา `QR`/`TC` | Function tree / IDEF0-style → `F` nodes | `F` |
| S7 | Architecture / module mapping | จับ `F` ลง component/module (Parnas information-hiding); allocate FR/NFR | Component/module map + FR/NFR allocation | `C` |
| S8 | Implementation + ML experimentation | สร้างระบบ; งาน AI: เทรน/ทดลองภายใน function ที่ถือโมเดล | Working system + model card (ถ้ามี AI) | — |
| S9 | Verification & RTM close-out | ทดสอบทุก `QR`; ปิด RTM | `T` tests (Test/Demo/Inspect/Analyze) + RTM ครบ + acceptance report | `T` |
| S10 | Validation & defense | สาธิต, เทียบผลกับเป้า, เดิน traceability | Demo + results-vs-target + traceability walkthrough | — |

> **ก้าวย่อยเพื่อการเรียนรู้ (ไม่ใช่จุดส่งแยก):** S0–S3 แยกเป็น 4 ก้าวเรียน แต่ **ส่งรวมเป็นชิ้นเดียว** ที่ CP-A. ทำนองเดียวกัน S4 มี 4 ก้าวย่อยแต่ส่ง HoQ ใบเดียว.

---

## 2. House of Quality (S4) — 4 ก้าวย่อยในสเตจเดียว

สอนทีละก้าว, rubric ตรวจแยกทีละห้อง, แต่ **ส่ง HoQ ครบใบเดียว จุดตรวจเดียว**.

| ก้าว | ชื่อ | ทำอะไร | ห้องของบ้าน |
|---|---|---|---|
| 4a | HOWs | แปลง `CR` เป็น technical characteristics `TC` (ยึดคลังศัพท์ **ISO/IEC 25010**) | เพดาน (ceiling) |
| 4b | Relationship matrix | ให้คะแนนความสัมพันธ์ `CR`×`TC` (Strong/Medium/Weak = 9/3/1) | ตัวบ้าน (body) |
| 4c | Roof (correlation) | หา trade-off ระหว่าง `TC` (`+` เสริม / `−` ขัดกัน) — **สำคัญมากกับงาน AI** เช่น accuracy↔latency↔cost↔interpretability | หลังคา (roof) |
| 4d | Target & priority | คำนวณ importance ของแต่ละ `TC` (weight × relationship) + ใส่ target value เบื้องต้น | ฐานบ้าน (basement) |

หมายเหตุ: roof ทำแบบ lightweight ได้ แต่ **อย่าตัดทิ้งสำหรับงาน AI** เพราะ trade-off เป็นตัวกำหนดดีไซน์จริง.

---

## 3. Traceability spine — ID scheme 6 ระดับ

แกนกลางที่ใช้วัด process. **ทุก artifact มีรหัส และทุกแถวต้องอ้าง parent ของตัวเอง.**

```
CR-xx  Customer Requirement (ถ่วงน้ำหนัก)      ← ราก (จาก VoC)
  └─ TC-xx  Technical Characteristic (HoQ)      อ้าง ≥1 CR
       └─ QR-xx  Quantifiable Requirement        อ้าง TC ของมัน
            └─ F-xx  Function / sub-function      อ้าง QR/TC ที่มัน satisfies
                 └─ C-xx  Component / Module      อ้าง F ที่มัน implement
                      └─ T-xx  Test / Verification (+method)  อ้าง QR ที่มัน verify
```

**กฎตรวจ RTM เชิงกล (ใช้ในทั้ง rubric และ self-check):**
- `CR` ที่ไม่มี `T` ปลายทาง = คำสัญญาที่ไม่ได้พิสูจน์ (unverified promise)
- `T` ที่ไม่มี `QR` ต้นทาง = งานเกิน (gold-plating)
- `F` ที่ไม่มี `QR`/`TC` = scope creep / งานตาย
- `QR` ที่ไม่มี metric+operator+threshold+dataset+condition = ไม่ verifiable (ผิดเกณฑ์ 29148)

---

## 4. Mapping กับ 3 เทอม + จุดตรวจ (graded checkpoint)

**เทอม 1 = วางแผน "ออกแบบบนกระดาษครบ" (S0–S7)** จบด้วย proposal สมบูรณ์
**เทอม 2–3 = build + verify (S8–S10)**

| Checkpoint | ครอบขั้น | ส่งอะไร | เทอม |
|---|---|---|---|
| **CP-A · Requirements Foundation** | S0–S3 | framing + VoC log + KJ + weighted `CR` table | 1 |
| **CP-B · House of Quality + QR** | S4–S5 | HoQ ครบใบ + `QR` spec | 1 |
| **CP-C · Design** | S6–S7 | function tree (`F`) + architecture/module map (`C`) | 1 |
| **ปลายเทอม 1** | — | **Proposal ฉบับสมบูรณ์ + RTM ต้นทาง (CR→QR, และ QR→F→C)** | 1 |
| **CP-D · Build progress** | S8 | ระบบทำงานได้ (สาธิตความคืบหน้า กลาง/ปลาย T2) + model card | 2 |
| **CP-E · Verification & RTM close-out** | S9 | `T` tests ครบ + RTM ปิดวง + acceptance report | 3 |
| **CP-F · Validation & defense** | S10 | demo + results-vs-target + traceability walkthrough | 3 |

---

## 5. หมายเหตุสำหรับการผลิต deliverable ต่อจากนี้

- แต่ละขั้น/checkpoint ต้องมีครบ 4 ชิ้น: **student guideline / advisor guideline / rubric / self-check prompt** (ผลิตทีละ batch ตามกำลัง — ยังอยู่ใน fog จนกว่าจะ lock "template ของ deliverable")
- **จำนวน & หัวข้อ** ที่นักศึกษาควรมีต่อขั้น (เช่น กี่ CR, กี่ TC) จะกำหนดใน student guideline ของขั้นนั้น
- **แทร็ก Y4 (ข้าม VoC)** ต่างที่ราก — ดู ticket แยก "Design: แทร็ก Y4 ข้าม VoC"
- **Worked example** เดินจริง end-to-end ยังต้องผลิต (research เริ่ม "StudyMate" ไว้เป็น seed; Notes ให้เลือกหัวข้อจริงหลากหลายจากรายการหัวข้อนักศึกษา (`inputs/student-topics-cepp68.md`))
