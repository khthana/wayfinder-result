# แพ็กเกจ CP-C (Design: Functional Decomposition + Architecture, S6–S7) — ครบ 4 ชิ้น

Mirror gold reference [`05-package-cp-b-house-of-quality.md`](05-package-cp-b-house-of-quality.md). เดินตัวอย่าง **EEG SleepInsight** ต่อจาก QR ของ CP-B.
CP-C คือขั้นที่ **"QFD ไปยัง Functional Decomposition"** จริง — แปลง `QR`/`TC` เป็นโครงหน้าที่ (`F`) แล้วจับลง component (`C`).
แกน traceability: `QR/TC → F → C` — ทุก `F` ต้องชี้ขึ้นหา requirement, ทุก `QR` ต้องมี `F` รับผิดชอบ.

**Criteria list (สเปกเดียว) ของ CP-C — FD1–FD7:**

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

**เชื่อมขั้นก่อน/หลัง:** ← CP-B: ทุก `F` ต้องอ้าง `QR`/`TC` จริงจาก HoQ/QR · → CP-D (Build): `C` + secret คือหน่วยที่จะลงมือเขียน/เทรน; `F↔C` คือแผนงาน implement

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

═══════════════════════════════════════════════════

*เดินต่อจาก QR ของ CP-B; F/C ที่ได้เป็นตัวตั้งของ CP-D (Build). worked example EEG SleepInsight ต่อเนื่อง CR→TC→QR→F→C.*
