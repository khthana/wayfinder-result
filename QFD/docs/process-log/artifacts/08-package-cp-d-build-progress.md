# แพ็กเกจ CP-D (Implementation + ML Experimentation, S8) — ครบ 4 ชิ้น

Mirror gold reference [`05-package-cp-b-house-of-quality.md`](05-package-cp-b-house-of-quality.md). เดินตัวอย่าง **EEG SleepInsight** ต่อจาก `F`/`C` ของ CP-C. เทอม 2 (→ต้น 3).
CP-D = ลงมือสร้างจริง แต่คอร์สนี้วัด **process**: การสร้างต้อง **ยึดติดกับ design (F/C)** และ **เดินเข้าหา QR target อย่าง traceable + reproducible**.

**Criteria list (สเปกเดียว) ของ CP-D — BD1–BD7:**

| # | เกณฑ์ | ตรวจเชิงกล? |
|---|---|---|
| **BD1** | ทุก component/feature ที่สร้าง **trace กลับ `C`/`F`** ของ CP-C (ไม่มีของสร้างนอกแผน; ถ้ามีของใหม่ ย้อนไปเติม design ก่อน) | ✅ RTM (build→C) |
| **BD2** | แกน AI: มี **experiment log** — ลองอะไร/ทำไม/ผล, อยู่ใน module ที่ถือโมเดล | ◑ มี log |
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
- **ทดลอง AI ในที่ของมัน:** การเทรน/ทดลองอยู่ **ภายใน module ที่ถือโมเดล** (EEG: ภายใน C-03) — ไม่กระจายออกไปที่อื่น
- **บันทึก experiment log:** แต่ละครั้งลองอะไร ทำไม ได้ metric เท่าไร
- **ทำ model card + ประกาศข้อมูล:** dataset, split, data quality, ข้อจำกัด

> 🟢 EEG SleepInsight (ต่อจาก CP-C):
> - build: C-01..C-05 ตาม CP-C; C-03 คือที่เทรนโมเดล
> - experiment log (ย่อ): E1 baseline CNN 1-channel → macro-F1 0.71 · E2 เพิ่ม spectral feature → 0.74 · E3 2-channel + augment → **0.76** (เป้า QR-01 = 0.80, ยังห่าง 0.04 → ระบุแผนไล่ต่อ)
> - model card: ใช้กับ EEG ผู้ใหญ่ระหว่างนอน; ข้อมูล Sleep-EDF (subject-independent split, ไม่มี leakage); metric macro-F1/κ; ข้อจำกัด: ยังไม่ทดสอบกับผู้ป่วยโรคการนอน
> - data quality (ISO 25012): completeness (ตัด epoch ที่สัญญาณขาด >50%), consistency (sampling rate เดียว)

**ข้อผิดที่พบบ่อย:** ❌ สร้าง feature นอกแผนที่ไม่มีใน C/F (scope creep) · ❌ ไม่มี experiment log (เทรนมั่วจำผลไม่ได้) · ❌ data leakage (test ปนกับ train) · ❌ อ้างผลโดยไม่ประกาศ dataset/split · ❌ โมเดลรั่วออกนอก module (แก้ที่เดียวไม่ได้)

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

**เชื่อมขั้น:** ← CP-C: build ตาม `C`/`F` · → CP-E: ระบบ + frozen eval set พร้อมให้ verify เทียบ QR

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
[ ] BD7  เปลี่ยนโมเดลแตะแค่ module เดียว (interface ไม่รั่ว)
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
6. (BD7) โมเดลถูกซ่อนใน module เดียวไหม — ถ้ารั่ว ชี้จุด
สรุป 3 อย่างที่ต้องแก้ก่อน. อย่าแต่งข้อมูลเพิ่ม
```

*ระบบ + frozen eval set ที่ได้ ส่งต่อ CP-E เพื่อ verify เทียบ QR.*
