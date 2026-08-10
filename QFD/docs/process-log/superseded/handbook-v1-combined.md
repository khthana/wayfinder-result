**คู่มือกระบวนการ QFD → Functional Decomposition**
**สำหรับวิชาโครงงานวิศวกรรมคอมพิวเตอร์ (Senior Project)**

วิชา 01076014/01076016 — เตรียมโครงงานวิศวกรรมคอมพิวเตอร์ + วิชาโครงงาน (3 เทอม)
เวอร์ชัน 1.0 — 14 กรกฎาคม 2569

---

## โครงสร้างคู่มือนี้

คู่มือนี้เปลี่ยนการวัดผลโครงงานจาก **output-centric** (ผลงานสวยไหม อธิบาย technical ได้ไหม) ไปเป็น **process- + traceability-centric** โดยยึดสายโซ่ **VoC → QFD (House of Quality) → Quantifiable Requirements → Functional Decomposition → Implementation → Verification** ครบทั้ง 3 เทอม

แต่ละขั้นตอน (checkpoint) มีเอกสารครบ 4 ชิ้น: **Student Guideline · Advisor Guideline · Rubric · Self-check (Checklist + LLM Prompt)**

**รองรับ 2 แทร็ก:**
- **นักศึกษาปีที่ 3 (Full track, เริ่มจาก VoC จริง):** อ่านบทที่ 1–2 แล้วอ่านบทที่ 3–8 ตามลำดับ
- **นักศึกษาปีที่ 4 (ข้าม VoC, ได้หัวข้อแล้ว):** อ่านบทที่ 1–2 แล้ว**ข้ามไปบทที่ 9 ก่อน** (ใช้แทนบทที่ 3) แล้วจึงกลับมาอ่านบทที่ 4–8 ตามปกติ

**สารบัญ:**
1. ภาพรวมกระบวนการ (Stage Model)
2. วิธีใช้คู่มือนี้ — 4 มุมมองของทุกขั้นตอน
3. CP-A — Requirements Foundation (เทอม 1)
4. CP-B — House of Quality + Quantifiable Requirements (เทอม 1)
5. CP-C — Design: Functional Decomposition + Architecture (เทอม 1)
6. CP-D — Implementation + ML Experimentation (เทอม 2)
7. CP-E — Verification & RTM Close-out (เทอม 3)
8. CP-F — Validation & Defense (เทอม 3)
9. แทร็กปีที่ 4: รากทดแทน VoC (ใช้แทนบทที่ 3)

ตัวอย่างที่ใช้ตลอดคู่มือ: **EEG SleepInsight** (โครงงาน AI จำแนกระยะการนอนจากสัญญาณ EEG) เดินครบทุกขั้นตอนของแทร็กปี 3 ตั้งแต่ VoC จนถึง defense; **Intelligent Travel Planning AI Chatbot** สาธิตรากทดแทนของแทร็กปี 4

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 1 — ภาพรวมกระบวนการ (Stage Model)

เอกสารแกน (canonical) ของทั้งคู่มือ — guideline / rubric / self-check ทุกบทยึดโครงนี้. Locked: 2026-07-14.

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

## 2. House of Quality (S4) — 4 ก้าวย่อยในสเตจเดียว

สอนทีละก้าว, rubric ตรวจแยกทีละห้อง, แต่ **ส่ง HoQ ครบใบเดียว จุดตรวจเดียว**.

| ก้าว | ชื่อ | ทำอะไร | ห้องของบ้าน |
|---|---|---|---|
| 4a | HOWs | แปลง `CR` เป็น technical characteristics `TC` (ยึดคลังศัพท์ **ISO/IEC 25010**) | เพดาน (ceiling) |
| 4b | Relationship matrix | ให้คะแนนความสัมพันธ์ `CR`×`TC` (Strong/Medium/Weak = 9/3/1) | ตัวบ้าน (body) |
| 4c | Roof (correlation) | หา trade-off ระหว่าง `TC` (`+` เสริม / `−` ขัดกัน) — **สำคัญมากกับงาน AI** เช่น accuracy↔latency↔cost↔interpretability | หลังคา (roof) |
| 4d | Target & priority | คำนวณ importance ของแต่ละ `TC` (weight × relationship) + ใส่ target value เบื้องต้น | ฐานบ้าน (basement) |

หมายเหตุ: roof ทำแบบ lightweight ได้ แต่ **อย่าตัดทิ้งสำหรับงาน AI** เพราะ trade-off เป็นตัวกำหนดดีไซน์จริง.

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
- `QR` ที่ไม่มี metric+operator+threshold+dataset+condition = ไม่ verifiable (ผิดเกณฑ์ ISO/IEC/IEEE 29148)

## 4. Mapping กับ 3 เทอม + จุดตรวจ (graded checkpoint)

**เทอม 1 = วางแผน "ออกแบบบนกระดาษครบ" (S0–S7)** จบด้วย proposal สมบูรณ์
**เทอม 2–3 = build + verify (S8–S10)**

| Checkpoint | ครอบขั้น | ส่งอะไร | เทอม | บทที่ |
|---|---|---|---|---|
| **CP-A · Requirements Foundation** | S0–S3 | framing + VoC log + KJ + weighted `CR` table | 1 | 3 |
| **CP-B · House of Quality + QR** | S4–S5 | HoQ ครบใบ + `QR` spec | 1 | 4 |
| **CP-C · Design** | S6–S7 | function tree (`F`) + architecture/module map (`C`) | 1 | 5 |
| **ปลายเทอม 1** | — | **Proposal ฉบับสมบูรณ์ + RTM ต้นทาง (CR→QR, และ QR→F→C)** | 1 | — |
| **CP-D · Build progress** | S8 | ระบบทำงานได้ (สาธิตความคืบหน้า กลาง/ปลาย T2) + model card | 2 | 6 |
| **CP-E · Verification & RTM close-out** | S9 | `T` tests ครบ + RTM ปิดวง + acceptance report | 3 | 7 |
| **CP-F · Validation & defense** | S10 | demo + results-vs-target + traceability walkthrough | 3 | 8 |

**แทร็ก Y4 (ข้าม VoC):** ต่างที่ราก — ดูบทที่ 9

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 2 — วิธีใช้คู่มือนี้: 4 มุมมองของทุกขั้นตอน

deliverable 4 ชิ้นของแต่ละขั้น **ไม่ใช่เอกสาร 4 ฉบับแยกกัน** แต่เป็น **4 มุมมองของสเปกเดียวกัน** — "งานที่ดีของขั้นนี้หน้าตาเป็นยังไง":

```
                 ┌─ Student Guideline   = สเปก ในมุม "ผลิตอะไร กี่ชิ้น ตัวอย่าง"
Criteria List ──┼─ Advisor Guideline   = สเปก ในมุม "ตรวจอะไร red flag ตรงไหน"
(ของขั้นนั้น)     ├─ Rubric              = สเปก ในมุม "ให้คะแนนเป็นระดับ"
                 └─ Self-check          = สเปก ในมุม "นักศึกษายืนยันเองทีละข้อ"
```

**แต่ละบท (CP-A ถึง CP-F) กำหนด criteria list ครั้งเดียว แล้วฉายเป็น 4 มุม** — นี่คือสิ่งเดียวที่ทำให้ deliverable ทั้งเล่มสม่ำเสมอ และทำให้เอกสารเอง **trace ถึงกันได้** ซึ่งเป็นวินัยเดียวกับที่คู่มือนี้สอนนักศึกษา

**หลักที่ใช้ร่วมกันทุกบท:**

- **หลักจำนวน:** เกณฑ์จำนวนทุกตัวเป็น **ช่วงปกติ + "ถ้าน้อย/มากกว่านี้ให้อธิบายเหตุผล"** ไม่ใช่ประตูตายตัว — เพื่อไม่ให้นักศึกษาปั่นจำนวนมาให้ครบ (ซึ่งขัดกับการวัด process จริง)
- **Rubric:** สเกล 4 ระดับ (4 ดีเยี่ยม / 3 ผ่านดี / 2 ต้องแก้ / 1 ยังไม่ถึง) เสมอ สร้างบนกฎตรวจ RTM เชิงกล (orphan = scope creep ฯลฯ) เพื่อให้ traceability ตรวจได้ในไม่กี่นาที ไม่ใช่ความรู้สึก — น้ำหนักคะแนนเน้น **traceability/process หนักสุดเสมอ**
- **Self-check:** มีทั้ง 2 แบบคู่กันทุกขั้น — **(i) Checklist** ติ๊กเร็วก่อนส่ง และ **(ii) LLM Prompt** ให้นักศึกษาวางข้อมูลใส่ผู้ช่วย AI เพื่อตรวจ traceability เชิงลึก

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 3 — CP-A: Requirements Foundation (S0–S3, เทอม 1)

เดินตัวอย่าง **EEG SleepInsight** — CR list ที่ได้ป้อนเข้าบทที่ 4 (CP-B) พอดี.
CP-A = **รากของ traceability ทั้งเส้น** (VoC → CR): ถ้ารากไม่แน่น ทั้งสายโซ่ CR→TC→QR→F→C→T ลอยหมด.

> **หมายเหตุตัวอย่าง:** VoC ของ EEG SleepInsight ในบทนี้ *สร้างขึ้นเพื่อสาธิต* (หัวข้อจริงไม่ได้เก็บ VoC มาก่อน) — จุดนี้เองคือเหตุผลที่ **แทร็กปีที่ 4 ต้องมีรากทดแทน** (ดูบทที่ 9)

**Criteria list (สเปกเดียว) ของ CP-A:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **A1** | Problem statement ชัด: ระบุปัญหา/โอกาส + บริบท + ทำไมควรแก้ | ◑ |
| **A2** | Stakeholder list ครบ: ผู้มีส่วนได้เสีย + ผู้ใช้เป้าหมาย + บทบาทของแต่ละกลุ่ม | ✅ นับกลุ่ม |
| **A3** | VoC log ดิบ: เสียงผู้ใช้ "ในภาษาผู้ใช้" พร้อมแหล่งที่มา (ใคร/วิธีเก็บ) | ✅ นับ + มีแหล่ง |
| **A4** | ทุก `CR` **สืบย้อนไป VoC item ได้** (ไม่มี CR ที่คิดเอง) — รากของ traceability | ✅ RTM (VoC↔CR) |
| **A5** | `CR` เขียนเป็น "ความต้องการ" ไม่ใช่ solution/เทคโนโลยี | ◑ ตรวจถ้อยคำ |
| **A6** | ทุก `CR` มีน้ำหนัก + ระบุวิธีถ่วงน้ำหนักโปร่งใส (direct 1–5 / 100-point / AHP) | ✅ นับ + มีวิธี |
| **A7** | `CR` ครอบคลุมธีมหลักของ VoC และไม่ซ้ำซ้อน (completeness + ไม่มีธีม VoC ตกหล่น) | ✅ RTM (VoC→CR coverage) |

จำนวนปกติ (เกิน/ต่ำให้อธิบายเหตุผล): stakeholder ≥ 3 กลุ่ม · VoC item 10–30 · `CR` 5–12

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (CP-A)

**ขั้นนี้ทำไปทำไม:** วางรากของโครงงาน — รู้ว่าทำให้ใคร แก้ปัญหาอะไร และผู้ใช้ "พูดว่าต้องการอะไร" แล้วกลั่นเป็นรายการความต้องการที่ถ่วงน้ำหนักแล้ว เพื่อเป็นตัวตั้งของ House of Quality

**ส่งอะไร:** (1) Problem statement + stakeholder list · (2) VoC log · (3) affinity/KJ diagram · (4) ตาราง `CR` ถ่วงน้ำหนัก

**ทำทีละก้าว:**

**S0 — Framing:** เขียน problem statement 1 ย่อหน้า (ปัญหา/โอกาส + บริบท + ทำไมสำคัญ) และ list stakeholder พร้อมบทบาท
> 🟢 EEG SleepInsight — Problem: *"การประเมินคุณภาพการนอนที่แม่นยำต้องทำ polysomnography ในห้องแล็บ ซึ่งแพง ใช้เวลา และติดอิเล็กโทรดหลายจุด ทำให้คนทั่วไปเข้าถึงยาก จึงต้องการระบบ AI ที่จำแนกระยะการนอนจากสัญญาณ EEG จำนวน channel น้อยได้เชื่อถือพอ"*
> Stakeholders: ผู้ใช้ทั่วไปที่อยากติดตามการนอน · แพทย์/นักเทคนิคการนอน (ผู้ใช้ผลเชิงคลินิก) · นักวิจัยด้าน sleep/EEG

**S1 — VoC capture:** เก็บเสียงผู้ใช้ "ดิบ" ในภาษาเขา ระบุว่าใครพูด/เก็บมายังไง (อย่าเพิ่งตีความ)
> 🟢 VoC ดิบ (ตัวอย่าง): V1 *"อยากรู้ว่าคืนนี้หลับลึกพอไหม โดยไม่ต้องไปนอนที่ รพ."* · V2 *"อุปกรณ์ที่บ้านต้องใส่ง่าย ไม่ใช่ติดสาย 10 เส้น"* · V3 *"ผลต้องอ่านรู้เรื่อง ไม่ใช่กราฟที่ต้องเป็นหมอถึงเข้าใจ"* · V4 *"ต้องแม่นพอที่หมอจะเอาไปดูต่อได้"* · V5 *"อยากเห็นผลตอนเช้าเลย ไม่ต้องรอหลายวัน"* · V6 *"เครื่องต้องทำงานได้ทั้งคืนไม่หลุด"*

**S2 — Affinity → CR:** จัดกลุ่ม VoC ที่พูดเรื่องเดียวกันด้วย KJ แล้วตั้งชื่อกลุ่มเป็น `CR` (เป็น "ความต้องการ" ไม่ใช่ solution)
> 🟢 affinity → CR (พร้อมที่มา):

| `CR` | มาจาก VoC |
|---|---|
| CR-01 จำแนกระยะนอนแม่นยำเชื่อถือได้ | V4 |
| CR-02 อุปกรณ์ใส่ง่าย ใช้ channel น้อย | V2 |
| CR-03 เห็นผลไว (เกือบทันที) | V1, V5 |
| CR-04 ผลอ่านเข้าใจง่ายสำหรับคนทั่วไป | V3 |
| CR-05 ทำงานเสถียรตลอดคืน | V6 |

**S3 — Weight:** ให้น้ำหนักความสำคัญแต่ละ `CR` ระบุวิธี (ที่นี่ direct rating 1–5 จากการถามผู้ใช้)
> 🟢 CR-01 = 5 · CR-05 = 4 · CR-02 = 4 · CR-03 = 3 · CR-04 = 3 *(ตรงกับที่บทที่ 4 ใช้)*

**จำนวนที่ควรมี:** stakeholder ≥ 3 กลุ่ม · VoC item 10–30 · `CR` 5–12 (ช่วงปกติ — น้อย/มากให้อธิบาย)

**ข้อผิดที่พบบ่อย:** ❌ CR ที่ "คิดเอง" ไม่มี VoC รองรับ · ❌ เขียน CR เป็น solution ("ใช้ deep learning") แทนความต้องการ · ❌ VoC เป็นการตีความของผู้ทำ ไม่ใช่คำผู้ใช้จริง · ❌ ลืมระบุวิธี/ที่มาของน้ำหนัก

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (CP-A)

**สิ่งที่ต้องเห็น (map A1–A7):** problem statement มีบริบท+เหตุผล (A1) · stakeholder ≥3 กลุ่มมีบทบาท (A2) · VoC เป็นคำผู้ใช้จริงมีแหล่ง (A3) · ทุก CR โยงกลับ VoC (A4) · CR เป็นความต้องการไม่ใช่ solution (A5) · ทุก CR มีน้ำหนัก+วิธี (A6) · CR ครอบคลุม VoC ไม่ซ้ำ (A7)

**คำถามกระตุ้น (Socratic):**
- (A4 — รากสำคัญสุด) *"CR ตัวนี้มาจากที่ผู้ใช้พูดว่าอะไร? ชี้ VoC item ให้ดูหน่อย"* → ถ้าชี้ไม่ได้ = CR ลอย
- (A5) *"นี่คือสิ่งที่ผู้ใช้ 'ต้องการ' หรือ 'วิธีที่เราจะทำ'? ถ้าเปลี่ยนเทคโนโลยี ความต้องการนี้ยังอยู่ไหม?"*
- (A3) *"ประโยคนี้เป็นคำผู้ใช้ หรือเป็นการสรุปของเรา?"*
- (A7) *"มีเสียงผู้ใช้เรื่องไหนที่ยังไม่กลายเป็น CR ไหม?"*
- (A6) *"น้ำหนักนี้มาจากไหน — ถามใคร กี่คน หรือเดาเอง?"*

**Red flags:** 🚩 VoC น้อยเกินไป (< 10) หรือมาจากคนกลุ่มเดียว · 🚩 CR = ชื่อ feature/เทคโนโลยี · 🚩 CR จำนวนมากแต่ VoC น้อย (แปลว่าแต่งเพิ่ม) · 🚩 ทุก CR น้ำหนักเท่ากันหมด (ไม่ได้ priritize จริง) · 🚩 stakeholder มีแต่ "ผู้ใช้" กลุ่มเดียว ลืมผู้มีส่วนได้เสียอื่น

**จุดที่มักต้องดันกลับ:**
1. CR ที่สืบย้อน VoC ไม่ได้ (A4) — **ดันกลับเสมอ** เพราะมันคือรากของทั้งสายโซ่
2. CR ที่เป็น solution — ดึงกลับเป็นความต้องการก่อน

**เชื่อมขั้นก่อน/หลัง:** ← ไม่มี (นี่คือราก) · → บทที่ 4 (CP-B): `CR` ถ่วงน้ำหนักคือ WHATs ที่จะเข้า House of Quality; CR ที่นี่ไม่แน่น HoQ พังตาม

═══════════════════════════════════════════════════

## Projection C · RUBRIC (CP-A) — 4 ระดับ

**หลัก:** สร้างบนกฎ RTM (VoC↔CR) · น้ำหนัก traceability-root หนักสุด · จำนวนไม่ใช่ประตูตาย

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **A4 · ทุก CR สืบย้อน VoC (25)** | ทุก `CR` ชี้ VoC item ได้ชัด ไม่มี CR ลอย | CR ลอย ≤1 และไม่ใช่ตัวน้ำหนักสูง | CR ลอย 2–3 | CR ส่วนใหญ่ไม่มี VoC รองรับ |
| **A7 · CR ครอบคลุม VoC ไม่ซ้ำ (20)** | ธีม VoC หลักกลายเป็น CR ครบ ไม่ซ้ำซ้อน | ตกหล่น 1 ธีมเล็ก | ตกหล่นธีมสำคัญ/มีซ้ำ | CR ไม่สะท้อน VoC |
| **A3 · VoC ดิบมีคุณภาพ+แหล่ง (20)** | VoC เป็นคำผู้ใช้จริง หลากหลายกลุ่ม ระบุแหล่งครบ | ส่วนใหญ่ดี มีบางส่วนเป็นการตีความ | VoC น้อย/กลุ่มเดียว/ไม่มีแหล่ง | ไม่มี VoC จริง |
| **A5 · CR เป็นความต้องการ ไม่ใช่ solution (15)** | ทุก CR เป็นความต้องการล้วน | มี 1–2 เอียงไป solution | หลายตัวเป็น solution | CR = feature list |
| **A6 · น้ำหนัก + วิธีโปร่งใส (10)** | ทุก CR มีน้ำหนัก + ระบุวิธี/ที่มา | มีน้ำหนักครบ วิธีคลุมเครือ | น้ำหนักบางส่วน/ไม่มีวิธี | ไม่มีน้ำหนัก |
| **A1+A2 · Framing + stakeholder (10)** | problem ชัดมีบริบท + stakeholder ≥3 มีบทบาท | ครบแต่ตื้น | ขาดบริบท/stakeholder ไม่ครบ | ไม่ชัด |

**รวม 100** — Traceability-root (A4/A7) = **45%** · คุณภาพ VoC/CR (A3/A5) = **35%** · process (A6/A1/A2) = **20%**

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (CP-A)

### D.1 Checklist (ติ๊กเร็วก่อนส่ง)

```
Requirements Foundation — เช็คก่อนส่ง (CP-A)
[ ] A1  Problem statement มีปัญหา + บริบท + เหตุผลที่ควรแก้
[ ] A2  Stakeholder ≥ 3 กลุ่ม แต่ละกลุ่มระบุบทบาท
[ ] A3  VoC เป็นคำผู้ใช้จริง (ไม่ใช่การตีความของเรา) + ระบุใครพูด/เก็บยังไง
[ ] A4  CR ทุกตัวชี้กลับ VoC item ได้ (ไม่มี CR ที่คิดเอง)
[ ] A5  CR ทุกตัวเป็น "ความต้องการ" ไม่ใช่ solution/เทคโนโลยี
[ ] A6  CR ทุกตัวมีน้ำหนัก + บอกวิธีถ่วงน้ำหนัก
[ ] A7  ธีม VoC หลักกลายเป็น CR ครบ ไม่มีตกหล่น/ซ้ำซ้อน
[ ] เดินเส้นได้: VoC item → CR (อย่างน้อย 1 เส้นเต็ม)
```

### D.2 LLM Prompt (ตรวจ traceability-root เชิงลึก)

```
คุณคือผู้ช่วยตรวจ "Requirements Foundation" (VoC → Customer Requirements) ของโครงงานวิศวกรรม
ตรวจเข้มงวดตามเกณฑ์ รายงานเป็นตาราง PASS/FAIL + เหตุผล + จุดที่ต้องแก้

ข้อมูลของฉัน:
- Problem statement: [วาง]
- Stakeholder list (กลุ่ม + บทบาท): [วาง]
- VoC log ดิบ (คำผู้ใช้ + แหล่งที่มา): [วาง]
- Affinity/CR mapping (CR + VoC ที่มา): [วาง]
- ตาราง CR + น้ำหนัก + วิธีถ่วงน้ำหนัก: [วาง]

ตรวจทีละข้อ:
1. (A3) VoC ประโยคไหนเป็น "การตีความของผู้ทำ" ไม่ใช่คำผู้ใช้จริง — ชี้ตัว
2. (A4) มี CR ตัวไหน "ลอย" คือชี้กลับ VoC item ไม่ได้ไหม — ชี้ตัว (นี่คือเกณฑ์สำคัญสุด)
3. (A5) CR ตัวไหนเขียนเป็น solution/เทคโนโลยี แทนความต้องการ — ชี้ตัว + เสนอวิธีเขียนใหม่
4. (A7) มีธีมใน VoC เรื่องไหนที่ยังไม่กลายเป็น CR (ตกหล่น) ไหม / มี CR ซ้ำซ้อนไหม
5. (A6) CR ทุกตัวมีน้ำหนักและระบุวิธีถ่วงน้ำหนักไหม — ชี้ที่ขาด
6. (A2) stakeholder ครบพอไหม หรือมองข้ามผู้มีส่วนได้เสียกลุ่มใด

สุดท้าย:
- วาด traceability 1 เส้นเต็ม: VoC "…" → CR-?? และชี้ CR ที่ไม่มีรากรองรับ
- สรุป 3 อย่างที่ต้องแก้ก่อนเป็นอันดับแรก
อย่าแต่งข้อมูลเพิ่มเอง
```

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 4 — CP-B: House of Quality + Quantifiable Requirements (S4–S5, เทอม 1)

เดินตัวอย่าง **EEG SleepInsight** ต่อจากบทที่ 3.

**Criteria list (สเปกเดียว) ของ CP-B:**

| # | เกณฑ์ (criterion) | ตรวจเชิงกลได้? |
|---|---|---|
| **C1** | ทุก `CR` ยกมาจาก CP-A พร้อม **น้ำหนัก** (ครบ ไม่ตกหล่น) | ✅ นับได้ |
| **C2** | ทุก `TC` (HOWs) **วัดได้** และใช้คำจากคลัง **ISO/IEC 25010** (functional suitability, performance efficiency, usability, reliability, …) | ◑ ตรวจนิยาม |
| **C3** | ทุก `TC` **trace ขึ้นหา ≥1 `CR`** (ไม่มี TC ลอย) | ✅ RTM |
| **C4** | ทุก `CR` มี **≥1 `TC` ที่สัมพันธ์แรง (9)** — ไม่มี CR ที่ไม่มีใครตอบ | ✅ RTM |
| **C5** | **Roof** บันทึก trade-off ระหว่าง `TC` โดยเฉพาะคู่ของงาน AI (accuracy↔latency↔channels↔interpretability) | ◑ ตรวจว่ามี |
| **C6** | แต่ละ `TC` มี **target value** เบื้องต้น (ช่อง "how much") | ✅ นับได้ |
| **C7** | `TC` สำคัญ (importance สูง) ถูกแปลงเป็น **`QR` ที่ verifiable** = metric + operator + threshold + dataset + condition ครบ 5 องค์ (S5) | ✅ ตรวจ 5 องค์ |

จำนวนปกติ: `CR` 5–12 · `TC` 6–15 · roof trade-off ≥2 คู่ · `QR` (ขั้น S5) อย่างน้อย 1 ต่อ TC ที่ importance สูงสุด 3–5 ตัว (เกิน/ต่ำกว่าให้อธิบายเหตุผล)

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (CP-B)

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

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (CP-B)

### สิ่งที่ต้องเห็นในงาน (map ตรง C1–C7)

| เกณฑ์ | ต้องเห็นอะไร |
|---|---|
| C1 | ตาราง `CR` ที่มีคอลัมน์น้ำหนัก และ `CR` ทุกตัวโยงกลับ CP-A ได้ (ไม่มี CR โผล่ใหม่กลางทาง) |
| C2 | `TC` เป็นสิ่งที่ "วัดเป็นตัวเลข/หน่วยได้" และจับคู่กับหมวด ISO/IEC 25010 ได้ (ไม่ใช่ชื่อ feature) |
| C3 | ทุก `TC` มีเส้นโยงลงหา `CR` อย่างน้อยหนึ่ง — ชี้ได้ว่า TC นี้เกิดเพราะ CR ตัวไหน |
| C4 | ทุก `CR` มีอย่างน้อยหนึ่งช่องความสัมพันธ์ระดับแรง (9) — ไม่มี CR ที่ "ไม่มีใครตอบ" |
| C5 | roof มีเครื่องหมาย +/− และมีคู่ trade-off ของงาน AI (เช่น ความแม่น↔latency/จำนวน channel) |
| C6 | ทุก `TC` มีช่อง target value กรอกแล้ว (ตัวเลข + หน่วย/เงื่อนไข) |
| C7 | `TC` สำคัญสุด 3–5 ตัวถูกเขียนเป็น `QR` ที่มีครบ 5 องค์ |

### คำถามกระตุ้น (Socratic — ถามทีละเกณฑ์)

- **C1/C4:** "CR ตัวไหนที่ยังไม่มี TC ตอบเลย? แล้วมันสำคัญ (น้ำหนักสูง) ไหม?" → ถ้า CR น้ำหนักสูงไม่มี TC = ช่องโหว่ใหญ่
- **C2:** "TC ตัวนี้วัดยังไง หน่วยอะไร? ถ้าวัดไม่ได้ มันคือ TC จริงหรือเป็นแค่ feature?"
- **C3:** "TC ตัวนี้เกิดมาจาก CR ข้อไหน? ถ้าตอบไม่ได้ ทำไมเราถึงทำมัน?"
- **C5 (สำคัญกับ AI):** "TC สองตัวไหนที่พอดันตัวหนึ่งดีขึ้น อีกตัวแย่ลง? แล้วโครงงานจะเลือกทางไหน?"
- **C6/C7:** "ตัวเลขเป้าของ TC นี้มาจากไหน? อ้างอิงอะไร (งานวิจัย/baseline/ความต้องการผู้ใช้)? แล้วจะพิสูจน์ว่าได้ตามเป้ายังไง?"

### Red flags (สัญญาณต้องเบรก)

- 🚩 `TC` ลอกมาจาก feature list ตรง ๆ ("ระบบ login", "หน้า dashboard") — นี่คือ function ไม่ใช่ technical characteristic
- 🚩 `TC` ที่วัดไม่ได้ ("ใช้งานง่าย", "แม่นยำ") โดยไม่มีมิติ/หน่วย
- 🚩 relationship matrix เต็มไปด้วย 9 ทุกช่อง (ให้คะแนนมั่ว ไม่ได้คิดจริง)
- 🚩 roof ว่างเปล่า หรือไม่มี trade-off ของงาน AI ทั้งที่เป็นโครงงาน AI
- 🚩 `QR` ที่ขาด dataset หรือ condition → ทดสอบไม่ได้จริงในเทอม 3
- 🚩 จำนวนพอดีเป๊ะแบบผิดธรรมชาติ (เช่น 10 CR, 10 TC กลม ๆ) — อาจปั่นจำนวน

### จุดที่มักต้องดันกลับให้แก้

1. `QR` ไม่ verifiable (ขาดองค์ประกอบ) — **ดันกลับเสมอ** เพราะเทอม 2–3 ทั้งหมดพิงตรงนี้
2. TC ที่เป็น solution สำเร็จรูป (ระบุเทคโนโลยีเจาะจง เช่น "ใช้ ResNet") แทนที่จะเป็นคุณลักษณะที่วัดได้ — ดึงกลับเป็นคุณลักษณะ (ความแม่น) ก่อน แล้วค่อยเลือกเทคโนโลยีในบทที่ 5–6

### เชื่อมขั้นก่อน/หลัง

- **← บทที่ 3 (CP-A):** `CR` + น้ำหนักต้องมาจาก CP-A ครบ; ถ้าพบ CR ใหม่ ให้ย้อนไปเติมที่ CP-A ไม่ใช่แทรกที่นี่
- **→ บทที่ 5 (CP-C):** `QR`/`TC` ที่ได้ต้องพร้อมเป็น "ตัวตั้ง" ของ Functional Decomposition — ทุก `F` จะต้องอ้าง `QR`/`TC` ตัวใดตัวหนึ่ง

═══════════════════════════════════════════════════

## Projection C · RUBRIC (CP-B) — เกณฑ์ให้คะแนน 4 ระดับ

**หลัก:** สร้างบนกฎ RTM เชิงกล (ตรวจได้ในไม่กี่นาที) · น้ำหนัก traceability + verifiability หนักสุด · จำนวนไม่ใช่ประตูตาย

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **C3+C4 · Traceability ไม่มี orphan (25)** | ทุก `TC`↔`CR` โยงสองทางครบ ไม่มี orphan ทั้งสองฝั่ง | orphan รวม ≤1 และไม่ใช่ CR น้ำหนักสูง | orphan 2–3 หรือมี CR สำคัญไม่มี TC | โครงความสัมพันธ์ขาด/ไม่มีเส้นโยง |
| **C7 · QR verifiable 5 องค์ (25)** | ทุก `QR` ครบ metric+operator+threshold+dataset+condition; วัดได้จริง | ขาด ≤1 องค์ ใน ≤1 QR | หลาย QR ขาดองค์ประกอบ | QR วัดไม่ได้/ไม่มี |
| **C2 · TC วัดได้ + ISO 25010 (15)** | `TC` ทุกตัววัดได้ + จับหมวด ISO 25010 ชัด | ส่วนใหญ่วัดได้ มี 1–2 คลุมเครือ | หลายตัวเป็น feature/วัดไม่ได้ | TC เป็นชื่อ feature ล้วน |
| **C5 · Roof trade-off (เน้น AI) (15)** | roof ครบ มี trade-off ของ AI + ระบุทางเลือกที่จะเดิน | มี trade-off หลักแต่ไม่ลึก | roof มีแต่ไม่แตะ trade-off AI | ไม่มี roof |
| **C1 · CR ครบ + น้ำหนัก (10)** | `CR` ครบจาก CP-A ทุกตัวมีน้ำหนัก สมเหตุผล | ครบ แต่บางน้ำหนักไม่มีเหตุผล | ขาด CR บางตัว/น้ำหนักมั่ว | ไม่มีน้ำหนัก |
| **C6 · Target values (10)** | ทุก `TC` มี target + อ้างอิงที่มา | ส่วนใหญ่มี target | target บางส่วน/ไม่มีที่มา | ไม่มี target |

**รวม 100 คะแนน** — Traceability + Verifiability (C3/C4/C7) = **50%**, เทคนิค process (C2/C5/C6) = **40%**, ความครบ (C1) = **10%**
**หมายเหตุจำนวน:** ถ้าจำนวน CR/TC/QR อยู่นอกช่วงปกติ *แต่มีเหตุผลอธิบาย* → ไม่หักคะแนน; หักเฉพาะเมื่อ "น้อยจนขาด" หรือ "มากจนเฟ้อไม่มีเหตุผล"

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (CP-B)

### D.1 Checklist (ติ๊กเร็วก่อนส่ง)

```
House of Quality + QR — เช็คก่อนส่ง (CP-B)
[ ] C1  CR ทุกตัวมาจาก CP-A และมีน้ำหนักครบ
[ ] C2  TC ทุกตัววัดเป็นตัวเลข/หน่วยได้ และจับคู่หมวด ISO/IEC 25010 ได้
[ ] C3  ไม่มี TC "ลอย" — TC ทุกตัวโยงลงหา CR อย่างน้อย 1 ตัว
[ ] C4  ไม่มี CR "ไร้คนตอบ" — CR ทุกตัวมี TC สัมพันธ์แรง (9) อย่างน้อย 1
[ ] C5  Roof มี trade-off ของงาน AI อย่างน้อย 2 คู่ (เช่น ความแม่น↔latency)
[ ] C6  TC ทุกตัวมี target value (ตัวเลข + หน่วย/เงื่อนไข)
[ ] C7  QR สำคัญ 3–5 ตัว ครบ 5 องค์: metric + operator + threshold + dataset + condition
[ ] เดินเส้น traceability 1 เส้นเต็มได้: CR → TC → QR (ไม่มีจุดขาด)
```

### D.2 LLM Prompt (ตรวจ traceability เชิงลึกด้วย AI)

> วิธีใช้: คัดลอก prompt ด้านล่าง วางข้อมูล HoQ/QR ของตัวเองลงในวงเล็บ แล้วส่งให้ผู้ช่วย AI

```
คุณคือผู้ช่วยตรวจ "House of Quality + Quantifiable Requirements" ของโครงงานวิศวกรรมคอมพิวเตอร์
ตรวจอย่างเข้มงวดตามเกณฑ์ด้านล่าง รายงานผลเป็นตาราง PASS/FAIL + เหตุผลสั้น + จุดที่ต้องแก้

ข้อมูลของฉัน:
- รายการ CR (พร้อมน้ำหนัก): [วาง]
- รายการ TC (พร้อมหมวด ISO 25010): [วาง]
- Relationship matrix CR×TC (9/3/1): [วาง]
- Roof / correlation ระหว่าง TC: [วาง]
- Target values ของแต่ละ TC: [วาง]
- รายการ QR: [วาง]

ตรวจทีละข้อ:
1. (C1) CR ทุกตัวมีน้ำหนักครบไหม — ชี้ตัวที่ขาด
2. (C2) TC ตัวไหน "วัดไม่ได้" หรือเป็นแค่ชื่อ feature ไม่ใช่คุณลักษณะที่วัดได้ — ชี้ตัว
3. (C3) มี TC ตัวไหน "ลอย" คือไม่โยงลงหา CR เลยไหม — ชี้ตัว
4. (C4) มี CR ตัวไหนที่ไม่มี TC สัมพันธ์แรง (9) ตอบไหม โดยเฉพาะ CR น้ำหนักสูง — ชี้ตัว
5. (C5) Roof มี trade-off ของงาน AI (เช่น ความแม่น↔latency↔จำนวน channel↔ความอธิบายได้) ไหม — ถ้าไม่มี เตือน
6. (C6) TC ทุกตัวมี target value ไหม — ชี้ตัวที่ขาด
7. (C7) QR แต่ละตัวครบ 5 องค์ (metric + operator + threshold + dataset + condition) ไหม — ชี้องค์ที่ขาดรายตัว

สุดท้าย:
- วาด traceability chain ที่สมบูรณ์อย่างน้อย 1 เส้น: CR-?? → TC-?? → QR-?? และชี้ว่ามีเส้นไหนขาดตรงไหน
- สรุป 3 อย่างที่ควรแก้ก่อนเป็นอันดับแรก
อย่าแต่งข้อมูลเพิ่มเอง ถ้าข้อมูลไม่พอให้บอกว่าต้องการอะไร
```

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 5 — CP-C: Design — Functional Decomposition + Architecture (S6–S7, เทอม 1)

เดินตัวอย่าง **EEG SleepInsight** ต่อจากบทที่ 4.
CP-C คือขั้นที่ **"QFD ไปยัง Functional Decomposition"** จริง — แปลง `QR`/`TC` เป็นโครงหน้าที่ (`F`) แล้วจับลง component (`C`).
แกน traceability: `QR/TC → F → C` — ทุก `F` ต้องชี้ขึ้นหา requirement, ทุก `QR` ต้องมี `F` รับผิดชอบ.

**Criteria list (สเปกเดียว) ของ CP-C:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **FD1** | มี overall function แล้วแตกเป็น sub-function เป็นลำดับชั้น (function tree/IDEF0); ทุกโหนดเป็น "หน้าที่ (กริยา+กรรม)" ไม่ใช่ module/เทคโนโลยี | ◑ ตรวจถ้อยคำ |
| **FD2** | ทุก `F` **trace ขึ้นหา `QR`/`TC`** ที่มัน satisfies (ไม่มี F ลอย = scope creep) | ✅ RTM (F→QR) |
| **FD3** | ทุก `QR` ถูก allocate ลงอย่างน้อย 1 `F` (ไม่มี QR ที่ไม่มี function รับผิดชอบ) | ✅ RTM (QR→F) |
| **FD4** | FR = realized โดย leaf function; **NFR = constraint/target บน function/architecture ไม่ใช่ leaf** | ◑ ตรวจการวาง |
| **FD5** | Architecture: ทุก `F` จับลง `C` (component/module); ทุก `C` implement ≥1 `F` (ไม่มี orphan สองทาง) | ✅ RTM (F↔C) |
| **FD6** | **Parnas information-hiding**: แต่ละ module ระบุ "secret" (design decision ที่จะเปลี่ยน เช่น โมเดล AI, รูปแบบข้อมูล) ที่ซ่อนหลัง interface | ◑ ตรวจว่าระบุ secret |
| **FD7** | ระดับการแตกสมเหตุผล (ไม่ลึก/ตื้นเกิน); leaf ทำงานเดียวชัด ไม่มี node ที่ทำหลายอย่างปน | ◑ |

จำนวนปกติ (เกิน/ต่ำให้อธิบาย): `F` node 8–25 · ความลึก 2–3 ชั้น · `C` component 4–10

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (CP-C)

**ขั้นนี้ทำไปทำไม:** แปลง "จะวัดผ่านด้วยอะไร" (`QR`/`TC`) เป็น "ระบบต้อง *ทำ* อะไรบ้าง" (function tree) แล้วจับหน้าที่ลงเป็น module — เพื่อให้เขียนโค้ดได้โดยทุกส่วน**ตอบ requirement ที่มีจริง**

**ส่งอะไร:** (1) function tree/IDEF0 (`F` nodes) · (2) ตาราง allocation `F → QR/TC` · (3) architecture/module map (`C`) + `F↔C` · (4) รายการ NFR-as-constraint + secret ของแต่ละ module

**ทำทีละก้าว:**

**S6a — Overall function:** เขียนหน้าที่รวมของระบบเป็นประโยคเดียว (กริยา + กรรม)
> 🟢 EEG: *"ประเมินระยะการนอนจากสัญญาณ EEG แล้วแสดงผลที่เข้าใจง่ายให้ผู้ใช้"*

**S6b — Decompose เป็น sub-function** (Pahl & Beitz: แตกตามการไหลของข้อมูล/สัญญาณ) และ **โยงแต่ละ F ขึ้นหา QR/TC**
> 🟢 function tree (พร้อม satisfies):

| `F` | หน้าที่ | satisfies |
|---|---|---|
| F-01 | นำเข้าสัญญาณ EEG (channel น้อย) | TC-02 / QR (≤2 channel) |
| F-02 | ประมวลผลสัญญาณเบื้องต้น (filter, ตัด epoch 30s, จัดการ artifact) | TC-05 / QR-05 (robustness) |
| F-03 | สกัดคุณลักษณะ (feature extraction) | TC-01 (สนับสนุน) |
| F-04 | จำแนกระยะการนอน 5 ระยะ | **TC-01 / QR-01 (macro-F1 ≥ 0.80)** |
| ↳ F-04.1 | เตรียม input ให้โมเดล | — (ย่อยของ F-04) |
| ↳ F-04.2 | รันโมเดลจำแนก *(ซ่อนตัวโมเดล — Parnas)* | QR-01 |
| ↳ F-04.3 | แปลง output เป็น label 5 ระยะ + ความมั่นใจ | QR-01 |
| F-05 | สร้าง hypnogram + สรุปคุณภาพการนอน | TC-04 / QR-04 (usability) |
| F-06 | แสดงผลให้ผู้ใช้ | CR-04 / TC-04 |

**S6c — NFR เป็น constraint (ไม่ใช่ leaf):**
> 🟢 latency ≤ 1s/epoch (จาก TC-03/QR) = **constraint บน F-04.2** ไม่ใช่ function แยก · uptime ตลอดคืน = constraint บนสถาปัตยกรรมรวม

**S7a — จับ F ลง component (`C`):**
> 🟢 C-01 Signal Ingest (F-01) · C-02 Preprocessing (F-02, F-03) · C-03 Sleep-Stage Classifier (F-04) · C-04 Result/Hypnogram Builder (F-05) · C-05 UI (F-06)

**S7b — ระบุ "secret" ของแต่ละ module (Parnas information-hiding):**
> 🟢 C-03 secret = *สถาปัตยกรรมโมเดล + weights* (จะเปลี่ยนบ่อย → ซ่อนหลัง interface `classify(epoch)→(label,conf)`) · C-02 secret = *พารามิเตอร์ filter/วิธี handle artifact* · เปลี่ยนโมเดลใน C-03 ไม่กระทบ C อื่น

**จำนวนที่ควรมี:** `F` 8–25 · ลึก 2–3 ชั้น · `C` 4–10 (ช่วงปกติ — น้อย/มากให้อธิบาย)

**ข้อผิดที่พบบ่อย:** ❌ node เป็น module/เทคโนโลยี ("React frontend") แทนหน้าที่ · ❌ F ลอย ไม่มี QR/TC รองรับ (scope creep) · ❌ QR บางตัวไม่มี F รับผิดชอบ · ❌ เอา NFR (latency, security) มาเป็น leaf function · ❌ ไม่ระบุ secret ของ module (จับ F ลง C มั่ว)

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (CP-C)

**สิ่งที่ต้องเห็น (map FD1–FD7):** function tree ที่โหนดเป็นหน้าที่ (FD1) · ทุก F โยง QR/TC (FD2) · ทุก QR มี F (FD3) · NFR เป็น constraint (FD4) · F↔C ครบสองทาง (FD5) · ระบุ secret ต่อ module (FD6) · การแตกพอดี (FD7)

**คำถามกระตุ้น (Socratic):**
- (FD2) *"function นี้เกิดขึ้นเพราะ QR/TC ตัวไหน? ถ้าตอบไม่ได้ ทำไมเราต้องเขียนมัน?"*
- (FD3) *"QR-01 (ความแม่น) ถูกทำให้เกิดจริงที่ function ไหน?"*
- (FD4) *"latency เป็น 'สิ่งที่ระบบทำ' หรือ 'ข้อจำกัดของการทำ'? แล้วมันบีบ function ตัวไหน?"*
- (FD6) *"ถ้าปีหน้าเปลี่ยนโมเดล AI ต้องแก้กี่ module? ตัวไหนซ่อนโมเดลไว้?"* → ถ้าต้องแก้หลายที่ = information hiding ยังไม่ดี
- (FD1) *"โหนดนี้เป็นหน้าที่ (ทำอะไร) หรือเป็นวิธี (ทำด้วยอะไร)?"*

**Red flags:** 🚩 tree เป็น diagram ของ tech stack ไม่ใช่หน้าที่ · 🚩 F ลอยไม่มี requirement · 🚩 QR ตกหล่นไม่มี F · 🚩 NFR กลายเป็น leaf · 🚩 module ไม่มี secret / โมเดล AI กระจายอยู่หลาย module · 🚩 แตกลึกเป็นสิบชั้นแต่ leaf ยังกว้าง

**จุดที่มักต้องดันกลับ:**
1. F ลอย (FD2) และ QR ไม่มี F (FD3) — ดันกลับ เพราะทำลาย traceability กลางเส้น
2. โมเดล AI ไม่ถูกซ่อน (FD6) — จะเจ็บตอนเปลี่ยนโมเดลในเทอม build

**เชื่อมขั้นก่อน/หลัง:** ← บทที่ 4 (CP-B): ทุก `F` ต้องอ้าง `QR`/`TC` จริงจาก HoQ/QR · → บทที่ 6 (CP-D): `C` + secret คือหน่วยที่จะลงมือเขียน/เทรน; `F↔C` คือแผนงาน implement

═══════════════════════════════════════════════════

## Projection C · RUBRIC (CP-C) — 4 ระดับ

**หลัก:** สร้างบนกฎ RTM (F→QR, QR→F, F↔C) · น้ำหนัก traceability กลางเส้น + information-hiding หนักสุด

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **FD2+FD3 · F↔QR ไม่มี orphan (25)** | ทุก F โยง QR/TC และทุก QR มี F ครบสองทาง | orphan ≤1 ไม่ใช่ตัวสำคัญ | orphan 2–3 | หลาย F ลอย/หลาย QR ไม่มีเจ้าภาพ |
| **FD5 · F↔C mapping (20)** | ทุก F ลง C และทุก C มี F ครบ | orphan ≤1 | orphan 2–3 | mapping ขาด/ไม่มี |
| **FD6 · Information-hiding (20)** | ทุก module ระบุ secret; โมเดล AI/ข้อมูลผันแปรถูกซ่อนดี | secret ส่วนใหญ่ชัด | ระบุ secret ตื้น/โมเดลรั่วบางส่วน | ไม่มี secret / decompose ตามลำดับงาน |
| **FD1 · หน้าที่ ไม่ใช่ module (15)** | ทุกโหนดเป็นหน้าที่แท้ | ส่วนใหญ่ใช่ มี 1–2 ปน tech | หลายโหนดเป็น module/tech | tree เป็น tech stack |
| **FD4 · NFR เป็น constraint (10)** | NFR ทุกตัววางเป็น constraint บน F/arch ชัด | ส่วนใหญ่ถูก | NFR บางตัวเป็น leaf | NFR ปนเป็น function |
| **FD7 · การแตกพอดี (10)** | leaf งานเดียวชัด ลึกเหมาะ | ส่วนใหญ่ดี | ลึก/ตื้นเกินบางกิ่ง | โครงมั่ว |

**รวม 100** — Traceability กลางเส้น (FD2/FD3/FD5) = **45%** · Information-hiding + คุณภาพ decomposition (FD6/FD1/FD7) = **45%** · NFR (FD4) = **10%**

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (CP-C)

### D.1 Checklist (ติ๊กเร็วก่อนส่ง)

```
Design: Functional Decomposition + Architecture — เช็คก่อนส่ง (CP-C)
[ ] FD1  ทุกโหนดใน tree เป็น "หน้าที่" (กริยา+กรรม) ไม่ใช่ module/เทคโนโลยี
[ ] FD2  ทุก F ชี้ขึ้นหา QR/TC ที่มัน satisfies (ไม่มี F ลอย)
[ ] FD3  ทุก QR มี F อย่างน้อย 1 ตัวรับผิดชอบ (ไม่มี QR ตกหล่น)
[ ] FD4  NFR (latency/security/uptime) วางเป็น constraint บน F/architecture ไม่ใช่ leaf
[ ] FD5  ทุก F จับลง component C; ทุก C implement F อย่างน้อย 1 (ไม่มี orphan)
[ ] FD6  แต่ละ module ระบุ "secret" ที่ซ่อน; โมเดล AI ถูกซ่อนหลัง interface เดียว
[ ] FD7  leaf แต่ละตัวทำงานเดียวชัด ระดับการแตกไม่ลึก/ตื้นเกิน
[ ] เดินเส้นได้เต็ม: QR-?? → F-?? → C-?? (อย่างน้อย 1 เส้น)
```

### D.2 LLM Prompt (ตรวจ traceability กลางเส้นเชิงลึก)

```
คุณคือผู้ช่วยตรวจ "Functional Decomposition + Architecture" ของโครงงานวิศวกรรม
ตรวจเข้มงวดตามเกณฑ์ รายงานเป็นตาราง PASS/FAIL + เหตุผล + จุดที่ต้องแก้

ข้อมูลของฉัน:
- Overall function + function tree (F nodes + ลำดับชั้น): [วาง]
- ตาราง allocation F → QR/TC: [วาง]
- รายการ QR/TC (จาก CP-B): [วาง]
- Architecture/module map + F↔C: [วาง]
- NFR + secret ของแต่ละ module: [วาง]

ตรวจทีละข้อ:
1. (FD1) โหนดไหนเป็น "module/เทคโนโลยี" แทนที่จะเป็น "หน้าที่" — ชี้ตัว
2. (FD2) มี F ตัวไหน "ลอย" ไม่โยง QR/TC ไหม — ชี้ตัว (scope creep)
3. (FD3) มี QR ตัวไหนที่ไม่มี F รับผิดชอบเลยไหม — ชี้ตัว (requirement ตกหล่น)
4. (FD4) NFR ตัวไหนถูกวางผิดเป็น leaf function แทน constraint — ชี้ตัว
5. (FD5) มี F ที่ไม่ได้จับลง C หรือ C ที่ไม่มี F ไหม — ชี้ orphan
6. (FD6) โมเดล AI/รูปแบบข้อมูลที่จะเปลี่ยนบ่อย ถูกซ่อนใน module เดียวไหม — ถ้ากระจายหลาย module เตือน + เสนอ boundary
7. (FD7) กิ่งไหนแตกลึก/ตื้นผิดปกติ

สุดท้าย:
- วาด traceability เต็ม 1 เส้น: QR-?? → F-?? → C-?? และชี้จุดขาด
- สรุป 3 อย่างที่ต้องแก้ก่อน
อย่าแต่งข้อมูลเพิ่มเอง
```

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 6 — CP-D: Implementation + ML Experimentation (S8, เทอม 2)

เดินตัวอย่าง **EEG SleepInsight** ต่อจาก `F`/`C` ของบทที่ 5.
CP-D = ลงมือสร้างจริง แต่คู่มือนี้วัด **process**: การสร้างต้อง **ยึดติดกับ design (F/C)** และ **เดินเข้าหา QR target อย่าง traceable + reproducible**.

**Criteria list (สเปกเดียว) ของ CP-D:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **BD1** | ทุก component/feature ที่สร้าง **trace กลับ `C`/`F`** ของ CP-C (ไม่มีของสร้างนอกแผน; ถ้ามีของใหม่ ย้อนไปเติม design ก่อน) | ✅ RTM (build→C) |
| **BD2** | แกน AI: มี **experiment log** — ลองอะไร/ทำไม/ผล, อยู่ในโมดูลที่ถือโมเดล | ◑ มี log |
| **BD3** | **Model card** (ถ้ามี AI): intended use, ข้อมูลที่ใช้, metric ที่วัดได้, ข้อจำกัด | ✅ มี/ไม่มี |
| **BD4** | **Data quality** ระบุ+ตรวจ (ISO/IEC 25012); ประกาศ train/val/test split ไม่มี leakage | ◑ |
| **BD5** | ความคืบหน้า**เดินเข้าหา QR target** (มีตัวเลขปัจจุบันเทียบ QR bar) — traceable progress | ✅ ตัวเลข vs QR |
| **BD6** | **Reproducibility**: seed คงที่, ข้อมูล versioned, eval set frozen, โค้ด version control | ◑ |
| **BD7** | รักษา **information-hiding** ของ CP-C (interface ที่ซ่อนโมเดลไม่รั่ว — เปลี่ยนโมเดลได้จริง) | ◑ |

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (CP-D)

**ขั้นนี้ทำไปทำไม:** สร้างระบบตาม design โดยไม่หลุดจาก requirement — และสำหรับงาน AI ทำการทดลองอย่างมีระบบเพื่อไล่ให้ถึง QR target ที่ตั้งไว้

**ส่งอะไร:** (1) ระบบ/โค้ดที่ทำงานได้ (สาธิตความคืบหน้า) · (2) experiment log · (3) model card (ถ้ามี AI) · (4) ตาราง build→C/F + ความคืบหน้าเทียบ QR

**ทำทีละก้าว:**
- **สร้างตาม C/F:** หยิบ component จาก CP-C มา implement; ทุก feature ที่เขียนต้องชี้กลับ `C`/`F` ได้
- **ทดลอง AI ในที่ของมัน:** การเทรน/ทดลองอยู่ **ภายในโมดูลที่ถือโมเดล** (EEG: ภายใน C-03) — ไม่กระจายออกไปที่อื่น
- **บันทึก experiment log:** แต่ละครั้งลองอะไร ทำไม ได้ metric เท่าไร
- **ทำ model card + ประกาศข้อมูล:** dataset, split, data quality, ข้อจำกัด

> 🟢 EEG SleepInsight (ต่อจากบทที่ 5):
> - build: C-01..C-05 ตามบทที่ 5; C-03 คือที่เทรนโมเดล
> - experiment log (ย่อ): E1 baseline CNN 1-channel → macro-F1 0.71 · E2 เพิ่ม spectral feature → 0.74 · E3 2-channel + augment → **0.76** (เป้า QR-01 = 0.80, ยังห่าง 0.04 → ระบุแผนไล่ต่อ)
> - model card: ใช้กับ EEG ผู้ใหญ่ระหว่างนอน; ข้อมูล Sleep-EDF (subject-independent split, ไม่มี leakage); metric macro-F1/κ; ข้อจำกัด: ยังไม่ทดสอบกับผู้ป่วยโรคการนอน
> - data quality (ISO 25012): completeness (ตัด epoch ที่สัญญาณขาด >50%), consistency (sampling rate เดียว)

**ข้อผิดที่พบบ่อย:** ❌ สร้าง feature นอกแผนที่ไม่มีใน C/F (scope creep) · ❌ ไม่มี experiment log (เทรนมั่วจำผลไม่ได้) · ❌ data leakage (test ปนกับ train) · ❌ อ้างผลโดยไม่ประกาศ dataset/split · ❌ โมเดลรั่วออกนอกโมดูล (แก้ที่เดียวไม่ได้)

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (CP-D)

**สิ่งที่ต้องเห็น (map BD1–BD7):** build ผูก C/F (BD1) · experiment log (BD2) · model card (BD3) · data quality + split (BD4) · ความคืบหน้า vs QR (BD5) · reproducibility (BD6) · interface ไม่รั่ว (BD7)

**คำถามกระตุ้น (Socratic):**
- (BD1) *"feature ที่เพิ่งเขียนนี้อยู่ใน component ไหนของ design? ถ้าไม่มี ทำไมถึงสร้าง?"*
- (BD5) *"ตอนนี้ metric เท่าไร ห่างจาก QR target แค่ไหน แผนไล่ต่อคืออะไร?"*
- (BD4) *"test set แยกจาก train ตั้งแต่เมื่อไร มั่นใจว่าไม่ leak ยังไง?"*
- (BD2) *"ทำไมถึงเลือกโมเดลตัวนี้ ลองอะไรมาก่อนบ้าง ผลต่างกันยังไง?"*
- (BD7) *"ถ้าเปลี่ยนโมเดลตอนนี้ ต้องแตะโค้ดกี่ไฟล์?"*

**Red flags:** 🚩 โค้ดมี feature ที่ไม่มีใน design · 🚩 ไม่มี experiment log/ผลลอย · 🚩 metric สูงผิดปกติ (สงสัย leakage) · 🚩 ไม่ประกาศ dataset/split · 🚩 hard-code ผลลัพธ์ · 🚩 โมเดลเทรนใหม่ทีต้องแก้หลายที่

**จุดดันกลับ:** data leakage (BD4) — ดันกลับเสมอ ทำให้ QR verification เชื่อไม่ได้ · feature นอกแผน (BD1) — ให้ย้อนเติม design

**เชื่อมขั้น:** ← บทที่ 5 (CP-C): build ตาม `C`/`F` · → บทที่ 7 (CP-E): ระบบ + frozen eval set พร้อมให้ verify เทียบ QR

═══════════════════════════════════════════════════

## Projection C · RUBRIC (CP-D) — 4 ระดับ

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **BD1 · build ผูก C/F ไม่มีนอกแผน (20)** | ทุกส่วนที่สร้าง trace กลับ C/F | ของนอกแผน ≤1 เล็กน้อย | นอกแผนหลายจุด | build ไม่ผูก design |
| **BD5 · ความคืบหน้า vs QR (20)** | มีตัวเลขปัจจุบันเทียบ QR ทุกตัว + แผนไล่ | มีตัวเลขส่วนใหญ่ | ตัวเลขบางส่วน/ไม่เทียบ QR | ไม่รู้ว่าห่างเป้าแค่ไหน |
| **BD4 · data quality + no leakage (20)** | split ประกาศชัด ไม่ leak + data quality ตาม 25012 | ส่วนใหญ่ดี | split คลุมเครือ/ข้อสงสัย leak | leakage ชัด/ไม่ประกาศ |
| **BD2+BD3 · experiment log + model card (20)** | log ครบเหตุผล + model card ครบ | มีแต่ตื้น | ขาดอย่างใดอย่างหนึ่ง | ไม่มี |
| **BD6 · reproducibility (10)** | seed/version/frozen eval ครบ | ส่วนใหญ่มี | บางส่วน | ทำซ้ำไม่ได้ |
| **BD7 · information-hiding คงอยู่ (10)** | เปลี่ยนโมเดลแตะที่เดียว | เกือบ | รั่วบางส่วน | โมเดลกระจาย |

**รวม 100** — process traceability (BD1/BD5) = 40% · ความน่าเชื่อของผล (BD4/BD2/BD3) = 40% · วิศวกรรมซอฟต์แวร์ (BD6/BD7) = 20%

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (CP-D)

### D.1 Checklist
```
Implementation + ML Experiment — เช็คก่อนส่ง (CP-D)
[ ] BD1  ทุก feature/ส่วนที่สร้าง ชี้กลับ C/F ของ design ได้ (ไม่มีของนอกแผน)
[ ] BD2  มี experiment log: ลองอะไร ทำไม ได้ผลเท่าไร
[ ] BD3  มี model card: intended use / data / metric / ข้อจำกัด
[ ] BD4  ประกาศ train/val/test split ชัด ไม่มี leakage + ระบุ data quality
[ ] BD5  รู้ metric ปัจจุบันเทียบกับ QR target ทุกตัว + มีแผนไล่ให้ถึง
[ ] BD6  seed คงที่ / ข้อมูล versioned / eval set frozen / โค้ดใน version control
[ ] BD7  เปลี่ยนโมเดลแตะแค่โมดูลเดียว (interface ไม่รั่ว)
```

### D.2 LLM Prompt
```
คุณคือผู้ช่วยตรวจเฟส "Implementation + ML Experimentation" ของโครงงานวิศวกรรม (เน้น process + traceability)
รายงานเป็นตาราง PASS/FAIL + เหตุผล + จุดที่ต้องแก้

ข้อมูลของฉัน:
- design (C/F จาก CP-C): [วาง]
- รายการสิ่งที่สร้างจริง/feature: [วาง]
- experiment log: [วาง]
- model card + dataset/split: [วาง]
- QR target + metric ปัจจุบัน: [วาง]

ตรวจ:
1. (BD1) feature ไหนที่สร้างแล้วไม่มีใน C/F — ชี้ (scope creep)
2. (BD4) มีความเสี่ยง data leakage ไหม (test ปน train / เลือก split หลังเห็นผล) — ชี้
3. (BD5) metric ปัจจุบันห่าง QR target แต่ละตัวเท่าไร — ทำตาราง
4. (BD2/BD3) experiment log / model card ขาดอะไร
5. (BD6) จุดที่ทำซ้ำไม่ได้ (seed/version/eval set)
6. (BD7) โมเดลถูกซ่อนในโมดูลเดียวไหม — ถ้ารั่ว ชี้จุด
สรุป 3 อย่างที่ต้องแก้ก่อน. อย่าแต่งข้อมูลเพิ่ม
```

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 7 — CP-E: Verification & RTM Close-out (S9, เทอม 3)

เดินตัวอย่าง **EEG SleepInsight** ต่อ.
CP-E = **ปิดวง traceability**: ทุก `QR` ถูกพิสูจน์ด้วย `T` (test/verification) และ **RTM ปิดครบ CR→TC→QR→F→C→T ไม่มี orphan**. นี่คือหัวใจของการวัดแบบ process — พิสูจน์ว่าคำสัญญาทุกข้อถูกทำจริงและวัดได้.

**Criteria list (สเปกเดียว) ของ CP-E:**

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

**เชื่อมขั้น:** ← บทที่ 6 (CP-D): ระบบ + frozen eval set · → บทที่ 8 (CP-F): RTM + ผล → นำเสนอ/validate กับผู้ใช้

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

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 8 — CP-F: Validation & Defense (S10, เทอม 3)

เดินตัวอย่าง **EEG SleepInsight** ต่อ — ปิด loop.
CP-F = **ปิดคุณค่ากลับหาผู้ใช้**: verification (บทที่ 7) พิสูจน์ว่า "สร้างถูกตามสเปก"; **validation** ตอบว่า "สเปกนั้นแก้ปัญหาผู้ใช้จริงไหม" + เดิน traceability ให้กรรมการเห็นทั้งเส้น.

**Criteria list (สเปกเดียว) ของ CP-F:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **VA1** | **Validation กับผู้ใช้/stakeholder จริง** — ระบบตอบ `CR` เดิมไหม (ไม่ใช่แค่ผ่าน test เชิงเทคนิค) | ◑ |
| **VA2** | **ตาราง results-vs-target**: `QR` target vs actual ครบทุกตัว | ✅ |
| **VA3** | **Traceability walkthrough**: เดินเส้น CR→…→T ให้เห็น ≥ 2–3 เส้น | ✅ |
| **VA4** | **ข้อจำกัด + งานอนาคต** ซื่อตรง (รู้ว่าอะไรยังไม่ถึง) | ◑ |
| **VA5** | **Demo ทำงานจริง** (method: Demonstration) ครอบ use case หลัก | ◑ |
| **VA6** | **Reflection ต่อ trade-off (roof)**: ที่เลือกไว้ตอน HoQ ผลเป็นไง | ◑ |
| **VA7** | นำเสนอ **เชื่อมกลับ VoC/CR เดิม** (ปิด loop เชิงคุณค่า ไม่ใช่โชว์ฟีเจอร์) | ◑ |

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (CP-F)

**ขั้นนี้ทำไปทำไม:** ปิดโครงงานด้วยการพิสูจน์ **คุณค่า** — ไม่ใช่แค่ "ระบบทำงาน" แต่ "ระบบแก้สิ่งที่ผู้ใช้ต้องการตั้งแต่ต้นได้จริง" และแสดง traceability ทั้งเส้นให้กรรมการเชื่อ

**ส่งอะไร:** (1) รายงาน validation กับผู้ใช้ · (2) ตาราง results-vs-target · (3) สไลด์/สคริปต์ traceability walkthrough · (4) demo + ข้อจำกัด/งานอนาคต

**ทำทีละก้าว:**
- **Validate กับผู้ใช้:** ให้ stakeholder จริงลองใช้/ประเมินว่าตอบ CR เดิมไหม (เช่น ให้ผู้ใช้/แพทย์ดูผล)
- **ทำตาราง results-vs-target:** ทุก QR เทียบ target vs actual จากบทที่ 7
- **เตรียม traceability walkthrough:** เลือก 2–3 CR สำคัญ เดินให้เห็นตั้งแต่ VoC → CR → TC → QR → F → C → T
- **สะท้อน trade-off + ข้อจำกัด:** roof ที่เลือก (เช่น แม่น vs channel) ผลออกมายังไง อะไรยังไม่ถึง

> 🟢 EEG SleepInsight:
> - results-vs-target: QR-01 target F1≥0.80 / actual **0.79** (ไม่ถึง เล็กน้อย — ระบุตรง) · channel ≤2 ✅ · latency ✅ · SUS ✅
> - validation: ให้ผู้ใช้ 5 คน + นักเทคนิคการนอน 1 คนลองใช้ → hypnogram อ่านเข้าใจ (ตอบ CR-04); นักเทคนิคเห็นว่าใช้คัดกรองเบื้องต้นได้แต่ยังไม่แทนการอ่านจริง
> - walkthrough: V4 → CR-01 → TC-01 → QR-01 → F-04 → C-03 → T-01 (พร้อมหลักฐานทุกจุด)
> - trade-off reflection: เลือก 2-channel เพื่อ CR-02 (เข้าถึงง่าย) แลกกับความแม่นที่หลุด 0.80 ไป 0.01 — เป็น trade-off ที่ยอมรับได้ตามน้ำหนัก CR
> - ข้อจำกัด/อนาคต: ยังไม่ทดสอบกลุ่มผู้ป่วย; แผน: เก็บข้อมูลเพิ่ม, personalization

**ข้อผิดที่พบบ่อย:** ❌ โชว์แต่ฟีเจอร์ ไม่เชื่อมกลับ CR/VoC · ❌ ไม่มี validation กับผู้ใช้จริง (มีแต่ verification) · ❌ ซ่อน QR ที่ไม่ถึงเป้า · ❌ เดิน traceability ไม่ได้เมื่อกรรมการถาม · ❌ ไม่มี reflection ต่อ trade-off

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (CP-F)

**สิ่งที่ต้องเห็น (map VA1–VA7):** validation ผู้ใช้จริง (VA1) · results-vs-target ครบ (VA2) · walkthrough เดินได้ (VA3) · ข้อจำกัดซื่อตรง (VA4) · demo จริง (VA5) · reflection trade-off (VA6) · เชื่อมกลับ VoC/CR (VA7)

**คำถามกระตุ้น (Socratic) — ซ้อมก่อน defense จริง:**
- (VA3) *"หยิบ CR ที่กรรมการน่าจะถามมา 1 ตัว เดินให้ผมดูตั้งแต่ผู้ใช้พูดจนถึง test"*
- (VA1) *"ใครลองใช้บ้าง เขาบอกว่ามันแก้ปัญหาเขาจริงไหม"*
- (VA6) *"ตอน HoQ เลือก trade-off อะไรไว้ ผลออกมาคุ้มไหม"*
- (VA4) *"อะไรที่ยังไม่ถึงเป้า แล้วจะพูดกับกรรมการยังไง"*

**Red flags:** 🚩 นำเสนอเป็น feature tour ไม่เชื่อม CR · 🚩 ไม่มีผู้ใช้จริงประเมิน · 🚩 เลี่ยงพูด QR ที่ไม่ถึง · 🚩 เดิน traceability สะดุด · 🚩 ไม่มี reflection

**จุดเน้น:** ให้คุณค่ากับความซื่อตรง (VA4) — โครงงานที่ QR ไม่ถึงเป้าแต่**รู้ว่าทำไมและเดิน traceability ได้** ดีกว่าโครงงานที่อ้างว่าผ่านหมดแต่เดินเส้นไม่ได้

**เชื่อมขั้น:** ← บทที่ 7 (CP-E): RTM + ผล · → จบโครงงาน (ปิด loop VoC→คุณค่า)

═══════════════════════════════════════════════════

## Projection C · RUBRIC (CP-F) — 4 ระดับ

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **VA3 · Traceability walkthrough (25)** | เดิน CR→…→T ได้ลื่น ≥3 เส้น ตอบคำถามเจาะได้ | เดินได้ 2 เส้น มีสะดุดเล็กน้อย | เดินได้บางส่วน | เดินไม่ได้ |
| **VA1+VA7 · Validation เชื่อมกลับ CR/VoC (25)** | ผู้ใช้จริงประเมิน + เชื่อมคุณค่ากลับ VoC ชัด | มี validation + เชื่อมบางส่วน | validation ตื้น/เชื่อมหลวม | เป็น feature tour |
| **VA2 · Results-vs-target (20)** | ทุก QR เทียบ target/actual ครบ ตรงไปตรงมา | ส่วนใหญ่ครบ | ขาดบางตัว | ไม่มี/เลี่ยง |
| **VA4+VA6 · ซื่อตรง + reflection trade-off (20)** | ข้อจำกัดชัด + reflection trade-off ลึก | มีแต่ตื้น | คลุมเครือ | ไม่มี/ซ่อน |
| **VA5 · Demo จริง (10)** | demo ครอบ use case หลัก ทำงานจริง | ทำงานเป็นส่วนใหญ่ | สาธิตจำกัด | ไม่ทำงาน |

**รวม 100** — Traceability + คุณค่ากลับผู้ใช้ (VA3/VA1/VA7) = 50% · ผล + ความซื่อตรง (VA2/VA4/VA6) = 40% · demo (VA5) = 10%

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (CP-F)

### D.1 Checklist
```
Validation & Defense — เช็คก่อนนำเสนอ (CP-F)
[ ] VA1  มีผู้ใช้/stakeholder จริงลองใช้และประเมินว่าตอบ CR ไหม
[ ] VA2  ตาราง results-vs-target ครบทุก QR (target vs actual จริง)
[ ] VA3  ซ้อมเดิน traceability CR→…→T ได้ลื่น ≥ 2–3 เส้น
[ ] VA4  ระบุข้อจำกัด + งานอนาคตตรงไปตรงมา (รวม QR ที่ยังไม่ถึง)
[ ] VA5  demo ครอบ use case หลัก ทำงานจริง
[ ] VA6  reflection: trade-off (roof) ที่เลือกตอน HoQ ผลเป็นไง คุ้มไหม
[ ] VA7  การนำเสนอเชื่อมกลับ VoC/CR เดิม (ไม่ใช่แค่โชว์ฟีเจอร์)
```

### D.2 LLM Prompt
```
คุณคือผู้ช่วยซ้อม defense โครงงานวิศวกรรม (เน้น validation + traceability)
รายงานเป็นตาราง PASS/FAIL + เหตุผล + สิ่งที่กรรมการน่าจะถาม

ข้อมูลของฉัน:
- results-vs-target (QR: target vs actual): [วาง]
- รายงาน validation กับผู้ใช้: [วาง]
- traceability chains ที่เตรียมเดิน: [วาง]
- ข้อจำกัด + reflection trade-off: [วาง]

ตรวจ:
1. (VA7/VA1) การนำเสนอเชื่อมกลับ VoC/CR หรือเป็นแค่ feature tour — ชี้จุดที่หลุด
2. (VA3) traceability chain ที่เตรียม มีจุดไหนขาด/เดินสะดุด — ชี้
3. (VA2) QR ตัวไหนไม่ถึงเป้า และถูกพูดถึงตรง ๆ ไหม
4. (VA6) reflection ต่อ trade-off ลึกพอไหม
5. ตั้งคำถามแบบกรรมการ 5 ข้อที่เจาะ traceability/คุณค่า แล้วบอกว่าฉันควรตอบแนวไหน
อย่าแต่งข้อมูลเพิ่ม
```

```{=openxml}
<w:p><w:r><w:br w:type="page"/></w:r></w:p>
```

# บทที่ 9 — แทร็กปีที่ 4: รากทดแทน VoC (ใช้แทนบทที่ 3)

**ใช้แทนเฉพาะบทที่ 3 (CP-A)** สำหรับแทร็กปีที่ 4 (ได้หัวข้อ/เริ่มสร้างแล้ว ไม่ย้อนทำ VoC). **บทที่ 4–8 (CP-B–F) ใช้เหมือนแทร็กปีที่ 3 ทุกประการ ไม่แก้.**

**หลักการ:** VoC คือรากของ traceability. แทร็กปีที่ 4 ไม่มี VoC จริง → สร้าง **"รากสมมติที่ซื่อตรง"**: Assumed Customer Requirements ที่ทุกตัวมีหลักฐานรองรับ และ **ติดป้ายชัดว่าเป็นสมมติ** (ไม่อ้างว่าเก็บ VoC จริง). กันไม่ให้กลายเป็น "justify ย้อนหลังให้ดูดี".

**Criteria list (สเปกเดียว) ของแทร็กปีที่ 4:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **YA1** | Problem statement + **Assumed Stakeholder** (ผู้ใช้เป้าหมายที่สมมติจากหัวข้อ) ชัด | ◑ |
| **YA2** | **Assumed CR list** — เขียนเป็นความต้องการ (ไม่ใช่ solution) | ◑ |
| **YA3** | **ทุก Assumed CR อ้างหลักฐาน ≥1** (feature ที่ทำแล้ว / proxy user / product คู่เทียบ / งานวิจัย) — ไม่มี CR เดาลอย | ✅ RTM (CR→evidence) |
| **YA4** | **ทุก CR ติดป้าย "สมมติ (assumed)" ชัด** — ไม่พรางเป็น VoC จริง | ✅ ตรวจป้าย |
| **YA5** | ทุก CR มีน้ำหนัก + ระบุ**ฐานการให้น้ำหนัก** (จากหลักฐานไหน) | ✅ |
| **YA6** | **Assumption log** — ทะเบียนข้อสมมติ + **ระดับความเสี่ยงถ้าผิด** (สูง/กลาง/ต่ำ) + ผลกระทบ | ✅ มี log |
| **YA7** | *(แนะนำ ไม่บังคับ)* validate อย่างน้อย 1 CR ความเสี่ยงสูงกับ proxy user จริง | ◑ optional |

จำนวนปกติ: Assumed CR 5–12 · หลักฐานต่อ CR ≥1 · assumption log ครอบ CR ความเสี่ยงสูงทุกตัว

═══════════════════════════════════════════════════

## Projection A · STUDENT GUIDELINE (แทร็กปีที่ 4)

**ขั้นนี้ทำไปทำไม:** สร้าง "ราก" ให้ traceability ทั้งเส้นมีที่ยึด — แม้ไม่ได้เก็บ VoC จริง ก็ต้องรู้ว่าทำให้ใคร แก้อะไร โดย**ซื่อตรงว่าอะไรคือข้อเท็จจริงและอะไรคือข้อสมมติ**

**ส่งอะไร:** (1) Problem statement + Assumed Stakeholder · (2) ตาราง Assumed CR (ติดป้ายสมมติ + หลักฐาน + น้ำหนัก) · (3) Assumption log (+ ความเสี่ยง)

**ทำทีละก้าว:**
- **ตั้งกรอบจากหัวข้อ:** เขียนปัญหา/ผู้ใช้เป้าหมายที่หัวข้อนี้ตั้งใจแก้ (สมมติได้ แต่ให้สมเหตุผล)
- **derive Assumed CR + หลักฐาน:** แต่ละความต้องการต้องชี้หลักฐานได้ว่าทำไมเชื่อว่าผู้ใช้ต้องการ — ห้ามเดาลอย
- **ติดป้าย "สมมติ" ทุกตัว** และทำ **assumption log** ระบุความเสี่ยงถ้าสมมติผิด

> 🟢 ตัวอย่าง: **Intelligent Travel Planning AI Chatbot** (ปี 4 สร้าง chatbot ไปแล้วบางส่วน)
> - Problem/Assumed Stakeholder: นักท่องเที่ยวอิสระที่อยากวางแผนทริปเร็วโดยไม่ต้องเปิดหลายเว็บ
> - Assumed CR (ติดป้าย [สมมติ] + หลักฐาน):

| Assumed CR | น้ำหนัก | หลักฐานรองรับ |
|---|---|---|
| [สมมติ] CR-01 ได้แผนเที่ยวที่ตรงงบและวัน | 5 | product คู่เทียบ (Wanderlog มีฟีเจอร์นี้), feature ที่ทีมทำแล้ว |
| [สมมติ] CR-02 คุยเป็นภาษาไทยธรรมชาติ | 4 | proxy user 2 คนบอกอยากพิมพ์ไทย, งานวิจัย Thai NLP |
| [สมมติ] CR-03 แนะนำที่ตรงความชอบ | 4 | analogous product (Google Travel), assumption |
| [สมมติ] CR-04 ตอบไว ไม่รอนาน | 3 | best practice chatbot latency |

> - Assumption log: A1 "ผู้ใช้ยอมพิมพ์ไทยยาว" — เสี่ยง**กลาง** (ถ้าผิด ต้องเพิ่มปุ่มเลือก) · A2 "งบเป็นเกณฑ์หลัก" — เสี่ยง**สูง** (กระทบ CR-01 น้ำหนักสูงสุด → ควร validate)

**→ ต่อบทที่ 4 (CP-B):** เอา Assumed CR (ถ่วงน้ำหนัก) เข้า House of Quality เหมือนแทร็กปีที่ 3 ทุกประการ. ถ้าระบบที่สร้างไว้แล้วขัดกับสิ่งที่ HoQ/design forward บอก = **บันทึกเป็น gap** แล้วอธิบาย/แก้ (ไม่ซ่อน)

**ข้อผิดที่พบบ่อย:** ❌ เขียน Assumed CR แบบเดาลอยไม่มีหลักฐาน · ❌ พราง assumption เป็น "ผลสำรวจผู้ใช้" ทั้งที่ไม่ได้ทำ · ❌ ไม่มี assumption log/ไม่ประเมินความเสี่ยง · ❌ ปรับ CR ให้เข้ากับของที่สร้างไว้แล้ว (reverse-justify) แทนที่จะยอมรับ gap

═══════════════════════════════════════════════════

## Projection B · ADVISOR GUIDELINE (แทร็กปีที่ 4)

**สิ่งที่ต้องเห็น (map YA1–YA7):** problem+assumed stakeholder (YA1) · assumed CR เป็นความต้องการ (YA2) · หลักฐานต่อ CR (YA3) · ติดป้ายสมมติ (YA4) · น้ำหนัก+ฐาน (YA5) · assumption log+ความเสี่ยง (YA6) · validate เสี่ยงสูง (YA7 optional)

**คำถามกระตุ้น (Socratic):**
- (YA3) *"CR ตัวนี้ — อะไรทำให้เชื่อว่าผู้ใช้ต้องการ? ชี้หลักฐานให้ดู"* → ถ้าตอบ "รู้สึกว่า" = เดาลอย
- (YA4) *"อันไหนคือข้อเท็จจริง อันไหนคือที่เราสมมติ? แยกให้ชัด"*
- (YA6) *"ถ้าข้อสมมตินี้ผิด กระทบอะไรบ้าง แล้วเราจะรู้ได้ยังไงว่าผิด?"*
- (gap) *"มีอะไรที่สร้างไว้แล้วแต่ไม่มี CR รองรับไหม / CR ไหนที่ของจริงยังไม่ตอบ?"*

**Red flags:** 🚩 Assumed CR ไม่มีหลักฐาน · 🚩 assumption ถูกเขียนเป็น "เก็บ VoC มาแล้ว" · 🚩 CR ทุกตัวพอดีกับ feature ที่สร้างไว้เป๊ะ (reverse-justify) · 🚩 ไม่มี assumption log · 🚩 CR เสี่ยงสูงไม่มีใครตั้งคำถาม

**จุดเน้น:** ให้คุณค่ากับ**ความซื่อตรง** — แทร็กปีที่ 4 ที่บอกตรงว่า "นี่คือข้อสมมติ ความเสี่ยง X" ดีกว่าที่แสร้งว่ามี VoC. ถ้าพบ reverse-justify (แต่ง CR ให้เข้ากับของที่ทำ) ให้ดันกลับ

**เชื่อมขั้น:** ← ไม่มี VoC (ใช้รากสมมติแทน) · → บทที่ 4 (CP-B): Assumed CR = WHATs เข้า HoQ เหมือนแทร็กปีที่ 3

═══════════════════════════════════════════════════

## Projection C · RUBRIC (แทร็กปีที่ 4) — 4 ระดับ (มีเกณฑ์ "ความซื่อตรง")

| เกณฑ์ (น้ำหนัก) | 4 ดีเยี่ยม | 3 ผ่านดี | 2 ต้องแก้ | 1 ยังไม่ถึง |
|---|---|---|---|---|
| **YA3 · ทุก Assumed CR มีหลักฐาน (25)** | ทุก CR อ้างหลักฐานชัด ตรวจสอบได้ | CR เดาลอย ≤1 | เดาลอย 2–3 | ส่วนใหญ่ไม่มีหลักฐาน |
| **YA4+ซื่อตรง · ติดป้ายสมมติตรงไปตรงมา (25)** | แยกข้อเท็จจริง/ข้อสมมติชัดทุกตัว ไม่พราง | ส่วนใหญ่ชัด | ป้ายไม่ครบ/คลุมเครือ | พราง assumption เป็น VoC จริง |
| **YA6 · Assumption log + ความเสี่ยง (20)** | log ครบ + ประเมินความเสี่ยง/ผลกระทบดี | มีแต่ตื้น | log ไม่ครบ | ไม่มี |
| **YA2+YA5 · CR เป็นความต้องการ + น้ำหนักมีฐาน (20)** | CR เป็นความต้องการล้วน + น้ำหนักอิงหลักฐาน | ส่วนใหญ่ดี | ปน solution/น้ำหนักลอย | CR = feature list |
| **YA1 · Framing + assumed stakeholder (10)** | ชัด สมเหตุผล | ครบแต่ตื้น | คลุมเครือ | ไม่ชัด |

**รวม 100** — หลักฐาน+ความซื่อตรง (YA3/YA4) = **50%** · การจัดการข้อสมมติ (YA6/YA2/YA5) = **40%** · framing (YA1) = **10%**
**โบนัส (optional):** validate CR เสี่ยงสูงกับ proxy user จริง (YA7) — บวกได้ ไม่หักถ้าไม่ทำ

═══════════════════════════════════════════════════

## Projection D · SELF-CHECK (แทร็กปีที่ 4)

### D.1 Checklist
```
Requirements Foundation (ปี 4 — รากสมมติ) — เช็คก่อนส่ง
[ ] YA1  มี problem statement + assumed stakeholder ที่สมเหตุผล
[ ] YA2  Assumed CR เขียนเป็น "ความต้องการ" ไม่ใช่ solution
[ ] YA3  ทุก Assumed CR อ้างหลักฐาน ≥1 (feature/proxy/product คู่เทียบ/งานวิจัย) ไม่มีเดาลอย
[ ] YA4  ทุก CR ติดป้าย "สมมติ" ชัด — ไม่พรางว่าเป็น VoC จริง
[ ] YA5  ทุก CR มีน้ำหนัก + บอกฐานการให้น้ำหนัก
[ ] YA6  มี assumption log + ประเมินความเสี่ยงถ้าสมมติผิด
[ ] (optional) YA7 validate CR เสี่ยงสูง ≥1 ตัวกับ proxy user จริง
[ ] ไม่ได้ปรับ CR ให้เข้ากับของที่สร้างไว้ (ถ้ามี gap บันทึกตรง ๆ)
```

### D.2 LLM Prompt
```
คุณคือผู้ช่วยตรวจ "รากสมมติของ requirement" (แทร็กนักศึกษาปี 4 ที่ข้าม VoC)
จุดสำคัญคือความซื่อตรง: แยกข้อเท็จจริงกับข้อสมมติ และกันการ justify ย้อนหลัง
รายงานเป็นตาราง PASS/FAIL + เหตุผล + จุดที่ต้องแก้

ข้อมูลของฉัน:
- Problem statement + assumed stakeholder: [วาง]
- ตาราง Assumed CR (+ ป้ายสมมติ + หลักฐาน + น้ำหนัก): [วาง]
- Assumption log (+ ความเสี่ยง): [วาง]
- (ถ้ามี) รายการ feature/ระบบที่สร้างไปแล้ว: [วาง]

ตรวจ:
1. (YA3) CR ตัวไหน "เดาลอย" ไม่มีหลักฐานรองรับ — ชี้ตัว
2. (YA4) มี CR ตัวไหนเขียนเหมือน "เก็บ VoC จริงมาแล้ว" ทั้งที่เป็นข้อสมมติ — ชี้ (พราง)
3. (reverse-justify) CR ตัวไหนดู "ปรับให้เข้ากับ feature ที่สร้างไว้เป๊ะ" น่าสงสัยว่า justify ย้อนหลัง — ชี้
4. (YA6) ข้อสมมติเสี่ยงสูงตัวไหนที่ยังไม่มีใน log หรือไม่ได้ประเมินผลกระทบ
5. (gap) มี feature ที่สร้างแล้วแต่ไม่มี CR รองรับ / CR ไหนที่ระบบยังไม่ตอบ
สรุป 3 อย่างที่ต้องแก้ก่อน. อย่าแต่งข้อมูลเพิ่ม
```

*หลังบทนี้ นักศึกษาปีที่ 4 เดินบทที่ 4–8 ด้วยแม่แบบเดียวกับแทร็กปีที่ 3 ทุกประการ.*
