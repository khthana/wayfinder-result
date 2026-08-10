# แม่แบบ deliverable 4 ชิ้น (LOCKED โครงสร้าง) — นำร่อง CP-B: House of Quality + QR

> **โครงสร้าง LOCKED แล้ว (2026-07-14):** (1) โครง spine-first "criteria list เดียว → 4 projection" + หัวข้อ Student Guideline ✅ · (2) Self-check = **ทั้ง Checklist + LLM Prompt** ✅ · (3) Rubric = **4 ระดับ เน้น traceability/process หนักสุด** ✅
> Student Guideline เติมเต็มจริงเป็น exemplar; Advisor/Rubric/Self-check ยังเป็นโครงร่าง จะเติมครบใน ticket "ผลิตแพ็กเกจ CP-B ให้ครบ".
> หัวข้อตัวอย่าง: **EEG SleepInsight** (จำแนกระยะการนอนจากสัญญาณ EEG ด้วย AI) — หัวข้อจริงจากรายการหัวข้อนักศึกษา (`inputs/student-topics-cepp68.md`). **หมายเหตุ: CR ในตัวอย่าง reverse-engineer จากหัวข้อ (ไม่มี VoC จริง) — เป็นการซ้อมโจทย์แทร็ก Y4.**

---

## Part 0 — หลักคิดของแม่แบบ: "สเปกเดียว 4 มุมมอง" (the recursion)

deliverable 4 ชิ้นของแต่ละขั้น **ไม่ใช่เอกสาร 4 ฉบับแยกกัน** แต่เป็น **4 มุมมองของสเปกเดียวกัน** — "งานที่ดีของขั้นนี้หน้าตาเป็นยังไง":

```
                 ┌─ Student Guideline   = สเปก ในมุม "ผลิตอะไร กี่ชิ้น ตัวอย่าง"
Criteria List ──┼─ Advisor Guideline   = สเปก ในมุม "ตรวจอะไร red flag ตรงไหน"
(ของขั้นนั้น)     ├─ Rubric              = สเปก ในมุม "ให้คะแนนเป็นระดับ"
                 └─ Self-check          = สเปก ในมุม "นักศึกษายืนยันเองทีละข้อ"
```

**กำหนด criteria list ครั้งเดียวต่อขั้น แล้ว generate ทั้ง 4 มุม** — นี่คือสิ่งเดียวที่ทำให้ 44 ชิ้นสม่ำเสมอ และทำให้ deliverable เอง **trace ถึงกันได้** (วินัยเดียวกับที่สอนนักศึกษา). แม่แบบที่จะ lock จริง ๆ คือ **(ก) ฟอร์แมตของ criteria list + (ข) ฟอร์แมตของ 4 projection**.

---

## Part 1 — Criteria List (สเปกเดียว) ของ CP-B / S4 House of Quality

ตารางนี้คือแหล่งความจริงเดียว. ทุก projection ด้านล่างอ้างเลข C1–C7 นี้.

| # | เกณฑ์ (criterion) | ตรวจเชิงกลได้? |
|---|---|---|
| **C1** | ทุก `CR` ยกมาจาก CP-A พร้อม **น้ำหนัก** (ครบ ไม่ตกหล่น) | ✅ นับได้ |
| **C2** | ทุก `TC` (HOWs) **วัดได้** และใช้คำจากคลัง **ISO/IEC 25010** (functional suitability, performance efficiency, usability, reliability, …) | ◑ ตรวจนิยาม |
| **C3** | ทุก `TC` **trace ขึ้นหา ≥1 `CR`** (ไม่มี TC ลอย) | ✅ RTM |
| **C4** | ทุก `CR` มี **≥1 `TC` ที่สัมพันธ์แรง (9)** — ไม่มี CR ที่ไม่มีใครตอบ | ✅ RTM |
| **C5** | **Roof** บันทึก trade-off ระหว่าง `TC` โดยเฉพาะคู่ของงาน AI (accuracy↔latency↔channels↔interpretability) | ◑ ตรวจว่ามี |
| **C6** | แต่ละ `TC` มี **target value** เบื้องต้น (ช่อง "how much") | ✅ นับได้ |
| **C7** | `TC` สำคัญ (importance สูง) ถูกแปลงเป็น **`QR` ที่ verifiable** = metric + operator + threshold + dataset + condition ครบ 5 องค์ (S5) | ✅ ตรวจ 5 องค์ |

> **หลัก "จำนวน":** เกณฑ์จำนวนทุกตัวเป็น **ช่วงปกติ + "ถ้าน้อย/มากกว่านี้ให้อธิบายเหตุผล"** ไม่ใช่ประตูตายตัว — เพื่อไม่ให้นักศึกษาปั่นจำนวนมาให้ครบ (ซึ่งขัดกับการวัด process จริง). ตัวอย่างช่วง CP-B: `CR` 5–12, `TC` 6–15, `QR` (ขั้น S5) อย่างน้อย 1 ต่อ TC ที่ importance สูงสุด 3–5 ตัว.

---

## Part 2 — Projection A · STUDENT GUIDELINE (เติมเต็ม + ตัวอย่างจริง)

### S4 House of Quality — คู่มือนักศึกษา

**ขั้นนี้ทำไปทำไม:** แปลง "เสียงผู้ใช้" (CR ที่ถ่วงน้ำหนักแล้ว) ให้กลายเป็น **คุณลักษณะทางเทคนิคที่วัดได้** พร้อมรู้ว่าตัวไหนสำคัญสุดและตัวไหนขัดกันเอง — เพื่อจะตั้งเป้าเป็นตัวเลข (QR) ในขั้นถัดไป

**ส่งอะไร:** House of Quality ครบใบเดียว (ตาราง 1 ใบ) + คำอธิบายสั้นใต้บ้าน

**ทำทีละก้าว (4 ก้าวย่อย):**

**ก้าว 4a — HOWs:** สำหรับ `CR` แต่ละตัว ถามว่า "จะวัด/ควบคุมมันด้วย *ตัวแปรทางเทคนิค* อะไร" เขียนเป็น `TC` โดยเลือกคำจาก ISO/IEC 25010
> 🟢 ตัวอย่าง (EEG SleepInsight):

| `CR` (น้ำหนัก) | → `TC` ที่ตอบ (ISO 25010) |
|---|---|
| CR-01 จำแนกระยะนอนแม่นยำเชื่อถือได้ (5) | TC-01 ความแม่นการจำแนก 5 ระยะ *(functional correctness)* |
| CR-02 ใช้กับ EEG อุปกรณ์หาง่าย channel น้อย (4) | TC-02 จำนวน EEG channel ที่ต้องใช้ *(portability/compatibility)* |
| CR-03 เห็นผลไว ติดตามการนอนได้ (3) | TC-03 latency ต่อ epoch 30 วินาที *(performance efficiency)* |
| CR-04 ผลอ่านเข้าใจง่าย (ไม่ใช่แพทย์) (3) | TC-04 ความชัดของ hypnogram/visualization *(usability)* |
| CR-05 ทำงานเสถียรตลอดคืน (4) | TC-05 ความทนต่อ artifact/สัญญาณขาด *(reliability)* |

**ก้าว 4b — Relationship matrix:** ให้คะแนนความสัมพันธ์ CR×TC เป็น Strong=9 / Medium=3 / Weak=1 / ว่าง=0
> 🟢 ตัวอย่าง (บางส่วน): CR-01×TC-01 = 9, CR-01×TC-02 = 3 (channel น้อยลง แม่นลง), CR-02×TC-02 = 9, CR-03×TC-03 = 9, CR-05×TC-05 = 9

**ก้าว 4c — Roof (trade-off):** จับคู่ `TC` ที่ **ขัดกัน (−)** หรือ **เสริมกัน (+)** — งาน AI ต้องมีอย่างน้อยคู่ accuracy↔ต้นทุน
> 🟢 ตัวอย่าง: TC-01 แม่น ↔ TC-02 channel น้อย = **−** (ขัดกัน) · TC-01 แม่น ↔ TC-03 latency = **−** (โมเดลใหญ่ช้าลง) · TC-01 แม่น ↔ TC-05 เสถียร = **+**

**ก้าว 4d — Target & priority:** คูณน้ำหนัก×ความสัมพันธ์รวมเป็น importance ของแต่ละ TC แล้วใส่ target เบื้องต้น
> 🟢 ตัวอย่าง: TC-01 importance สูงสุด → target: macro-F1 ≥ 0.80 · TC-02 → ≤ 2 channel · TC-03 → ≤ 1 วินาที/epoch

**แล้วต่อขั้น S5 (QR):** เอา TC ที่ importance สูงสุด 3–5 ตัวมาเขียนเป็น `QR` ให้ครบ **5 องค์: metric + operator + threshold + dataset + condition**
> 🟢 ตัวอย่าง — **QR-01** (trace: CR-01 → TC-01 → QR-01):
> *"บนชุดทดสอบ held-out แบบ subject-independent (≥ 10 คน, มี label จากผู้เชี่ยวชาญ) โมเดลจำแนก 5 ระยะการนอนได้ **macro-F1 ≥ 0.80** และ **Cohen's κ ≥ 0.75** โดยใช้ EEG **≤ 2 channel** และ latency **≤ 1 วินาที/epoch**"*
> — metric: macro-F1, κ · operator/threshold: ≥ 0.80 / ≥ 0.75 · dataset: held-out subject-independent ≥10 คน · condition: ≤2 channel, ≤1s/epoch ✅ ครบ 5 องค์

**จำนวนที่ควรมี (ช่วงปกติ — น้อย/มากกว่านี้ให้อธิบายเหตุผล):** `CR` 5–12 · `TC` 6–15 · roof trade-off ≥ 2 คู่ (งาน AI) · `QR` 3–5

**ข้อผิดที่พบบ่อย:** ❌ TC เป็นคำลอย ๆ วัดไม่ได้ ("ระบบดี") · ❌ มี CR ที่ไม่มี TC ตอบ · ❌ ลืม roof (โดยเฉพาะ accuracy↔latency) · ❌ QR ไม่มี dataset/condition → ทดสอบไม่ได้

**เช็คก่อนส่ง (ดู Projection D):** ทุก CR มีน้ำหนัก? ทุก TC วัดได้+ trace หา CR? มี roof trade-off ของ AI? QR ครบ 5 องค์?

---

## Part 3 — Projection B · ADVISOR GUIDELINE (โครงร่าง)

> สเปกเดียวกัน (C1–C7) ในมุม "อาจารย์ที่ปรึกษาตรวจ/ถามอะไร". *เติมจริงหลัง lock โครงสร้าง.*

**รูปทรงต่อขั้น:**
- **สิ่งที่ต้องเห็นในงาน** (map ตรงกับ C1–C7): ⟨checklist สั้น⟩
- **คำถามกระตุ้น (Socratic)** ต่อเกณฑ์: เช่น C5 → *"TC สองตัวไหนที่พอดันตัวหนึ่งแล้วอีกตัวแย่ลง?"*
- **Red flags:** ⟨เช่น TC ลอกมาจาก feature list ตรง ๆ ไม่ได้มาจาก CR⟩
- **จุดที่มักต้องดันกลับ:** ⟨QR ไม่ verifiable⟩
- **เชื่อมขั้นก่อน/หลัง:** CR มาจาก CP-A ครบไหม / QR พร้อมส่งต่อ S6 ไหม

---

## Part 4 — Projection C · RUBRIC (โครงร่าง — สร้างบนกฎ RTM เชิงกล)

> สเปกเดียวกัน (C1–C7) ในมุม "คะแนนเป็นระดับ". **สร้างบนกฎตรวจ RTM ของ stage model** (orphan = scope creep ฯลฯ) เพื่อให้ traceability ตรวจได้ในไม่กี่นาที ไม่ใช่ความรู้สึก. **LOCKED: 4 ระดับ · น้ำหนัก traceability/process หนักสุด.**

**สเกล 4 ระดับ (LOCKED): 4 ดีเยี่ยม / 3 ผ่านดี / 2 ต้องแก้ / 1 ยังไม่ถึง — กลุ่ม traceability (C3+C4 no-orphan) และ verifiability (C7) ได้น้ำหนักสูงสุด:**

| เกณฑ์ | 4 | 3 | 2 | 1 | น้ำหนัก |
|---|---|---|---|---|---|
| C1 CR ครบ+น้ำหนัก | ⟨…⟩ | | | | ? |
| C3+C4 ไม่มี orphan (RTM) | ทุก TC↔CR สองทาง ไม่มี orphan | orphan ≤1 | orphan 2–3 | โครงขาด | **สูง (traceability)** |
| C7 QR verifiable (5 องค์) | ทุก QR ครบ 5 องค์ | ขาด 1 องค์ ≤1 ตัว | หลายตัวขาด | วัดไม่ได้ | **สูง** |
| … | | | | | |

**น้ำหนัก (LOCKED):** กลุ่ม traceability + process หนักสุด > ความถูกต้องเนื้อหา > ความครบจำนวน (จำนวนไม่ใช่ประตูตาย)

---

## Part 5 — Projection D · SELF-CHECK (LOCKED: ใช้ทั้ง 2 แบบคู่กัน)

> สเปกเดียวกัน (C1–C7) ในมุม "นักศึกษายืนยันเอง". **LOCKED: มีทั้ง 2 แบบ** — Checklist เป็นการติ๊กเร็วก่อนส่ง, LLM Prompt ไว้ตรวจ traceability เชิงลึกกับ AI:

**แบบ (i) — Checklist ธรรมดา** (นักศึกษาติ๊กเอง — pass เร็ว):
```
□ C1 ทุก CR มีน้ำหนัก
□ C3 ทุก TC ชี้ขึ้นหา CR อย่างน้อย 1 (ไม่มี TC ลอย)
□ C7 ทุก QR มีครบ: metric / operator / threshold / dataset / condition
...
```

**แบบ (ii) — LLM Prompt** (นักศึกษาเอางานใส่ให้ AI ช่วยตรวจ traceability เชิงลึก):
```
คุณคือผู้ช่วยตรวจ House of Quality ของโครงงานวิศวกรรม
ต่อไปนี้คือ CR list, TC list, relationship matrix, roof, และ QR ของฉัน: [วาง]
ตรวจทีละข้อและรายงานเป็นตาราง PASS/FAIL + เหตุผล:
1. ทุก CR มีน้ำหนักครบไหม (C1)
2. มี TC ตัวไหน "ลอย" (ไม่ trace หา CR) ไหม — ชี้ตัว (C3)
3. มี CR ตัวไหนไม่มี TC สัมพันธ์แรง (9) ตอบไหม (C4)
4. roof มี trade-off ของงาน AI (accuracy↔latency/channels) ไหม (C5)
5. QR แต่ละตัวครบ 5 องค์ (metric+operator+threshold+dataset+condition) ไหม — ชี้ที่ขาด (C7)
สุดท้าย: วาด traceability chain CR→TC→QR ที่สมบูรณ์ 1 เส้น และชี้จุดที่ chain ขาด
```

> ✅ อาจารย์เลือก **ใช้ทั้ง 2 แบบ** — checklist กันลืมก่อนส่ง + LLM prompt ตรวจลึกและฝึกทักษะใช้ AI ตรวจงาน.

---

## Part 6 — แม่แบบเปล่า (blank shell) ที่จะ reuse ทุกขั้น

หลัง lock โครงสร้าง แต่ละขั้น (S0–S10) จะกรอกลง shell เดียวกันนี้:
1. **Criteria List** ของขั้น (ตาราง C1..Cn + ช่วงจำนวน)
2. **Student Guideline**: ทำไป-ทำไม / ส่งอะไร / ทำทีละก้าว + ตัวอย่างจริง / จำนวน / ข้อผิดบ่อย
3. **Advisor Guideline**: checklist / คำถามกระตุ้น / red flags / เชื่อมขั้น
4. **Rubric**: ตาราง Cn × ระดับ + น้ำหนัก
5. **Self-check**: Checklist (ติ๊กเร็ว) + LLM Prompt (ตรวจ traceability ลึก) — ทั้งคู่อ้าง Cn
