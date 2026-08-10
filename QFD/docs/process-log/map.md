# Map: กระบวนการ QFD → Functional Decomposition สำหรับ Senior Project (CEPP)

label: wayfinder:map

## Destination

แพ็กเกจกระบวนการที่พร้อมใช้จริง สำหรับวิชา Senior Project (เตรียมโครงงาน/โครงงานวิศวกรรมคอมพิวเตอร์ ป.ตรี KMITL) ที่เปลี่ยนการวัดผลจาก **output-centric** ไปเป็น **process- + traceability-centric** โดยยึดสายโซ่ **VoC → QFD (House of Quality) → Quantifiable Requirements → Functional Decomposition → Implementation → Verification** ครบทั้ง 3 เทอม

แพ็กเกจประกอบด้วย (ผลิตจริงในเทอมนี้ — ดู Notes ข้อ execution override):
1. **Guideline สำหรับนักศึกษา** ต่อขั้นตอน — บอกจำนวน หัวข้อที่ควรมี ตัวอย่างจริง (เน้นโครงงาน Software + AI/กึ่งวิจัย)
2. **Guideline สำหรับอาจารย์ที่ปรึกษา** — ควรตรวจ/ถามอะไรต่อขั้นตอน
3. **Rubric** สำหรับตรวจแต่ละขั้นตอน (ให้คะแนน process + traceability ไม่ใช่ UX/UI หรือคุณภาพซอฟต์แวร์ปลายทาง)
4. **Self-check Prompt** สำหรับนักศึกษาตรวจงานตัวเองว่าตรงแนวทางและ traceable

รองรับ 2 แทร็ก: **Y3 = Full Function (เริ่มจาก VoC)** และ **Y4 = ข้าม VoC** (ได้หัวข้อแล้ว ไม่ย้อนกลับทำ VoC)

ปลายทางถึงเมื่อ: มีแพ็กเกจ 4 ชิ้นครบทุกขั้นตอน + worked example เดินจริง 1 ชุด — พร้อมแจกนักศึกษาและอาจารย์

**✅ ถึงปลายทางแล้ว (2026-07-14):** คู่มือนักศึกษา (`handbook-student.docx`) + คู่มืออาจารย์ที่ปรึกษา (`handbook-advisor.docx`) พร้อมแจกจริง — ครบแพ็กเกจ 4 ชิ้นทุก checkpoint, 3 worked example (EEG SleepInsight เต็ม 11 บท + Travel Chatbot สาธิต Y4 + 2 ตัวอย่างเสริมในภาคผนวก), rubric weighting ตัดสินแล้ว. ไม่มี ticket เปิดค้าง ไม่มี fog เหลือ.

## Notes

**Domain:** วิชา 01076014/01076016 (เตรียมโครงงานฯ) + วิชาโครงงาน 3 เทอม — เทอม 1 = วางแผน (หัวใจของแพ็กเกจนี้คือ QFD→FD), เทอม 2–3 = implement + verify. โครงงานส่วนใหญ่เป็น Software บางส่วนกึ่งวิจัยผสม AI. หัวข้อนักศึกษา 39 หัวข้ออยู่ใน [`inputs/student-topics-cepp68.md`](inputs/student-topics-cepp68.md) (ปิดบังชื่อแล้ว); proposal ตัวอย่าง 3 ไฟล์ (.docx) เคยวางไว้ในโฟลเดอร์ราก — ถือเป็น "before" cases ไม่ใช่ gold exemplar; **ลบออกจากโปรเจกต์แล้ว (2026-08-10)** ตอนจัดโครงสร้างขึ้น GitHub เพราะมีข้อมูลส่วนบุคคลของนักศึกษา (ต้นฉบับเก็บไว้นอก repo).

**Execution override:** effort นี้ผลิต deliverable จริง (แพ็กเกจ 4 ชิ้น) — เป็น `task`-type tickets ไม่ใช่แค่ตัดสินใจ. แต่ frontier ช่วงแรกยังเป็นการตัดสินใจ (research + lock model) ก่อนจึงจะผลิตได้.

**Defaults (ตกลงแล้ว ไม่ต้องถามซ้ำ):**
- ภาษา = ไทย + ศัพท์เทคนิคภาษาอังกฤษ
- Source = Markdown (แปลงเป็น handout/Word ภายหลังได้)
- ไม่มีเทมเพลต proposal ตายตัว — ออกแบบโครงสร้าง artifact ที่นักศึกษาส่งได้ใหม่
- Rubric วัด process + traceability เป็นหลัก

**Skills ที่ทุก session ควรใช้:** `/research` (ticket research), `/grilling` + `/domain-modeling` (ticket lock/ออกแบบ), `/prototype` (ลองรูปแบบ artifact/rubric)

**หมายเหตุเทคนิค:** อ่านเนื้อหาไทยจาก .docx ให้ extract ด้วย PowerShell เป็น `clean.txt` แล้วเปิดด้วย Read tool (bash grep จะเพี้ยน encoding). ตอนทำ worked example เลือกหัวข้อหลากหลาย 2–3 แบบ (pure software, AI/ML, IoT).

## Decisions so far

<!-- one line per closed ticket -->

- [Research: หลักการ QFD → Functional Decomposition ที่เหมาะกับโครงงาน Software + AI](issues/01-research-qfd-fd-pipeline.md) — ได้สายโซ่อ้างอิง (VoC→CR→HoQ→TC→QR→FD→component→verification), ข้อเสนอ pipeline 10 ขั้น (S0–S10) + ID scheme `CR/TC/QR/F/C/T` + worked example "StudyMate"; asset: [`artifacts/01-research-qfd-fd-foundations.md`](artifacts/01-research-qfd-fd-foundations.md)
- [Lock: stage model + artifact ต่อขั้น + traceability spine (แทร็ก Full/Y3)](issues/02-lock-stage-model.md) — **LOCKED**: 11 ขั้น S0–S10, HoQ=สเตจเดียว 4 ก้าวย่อย, ID 6 ระดับ `CR→TC→QR→F→C→T`, เทอม 1=S0–S7 (จบ Architecture), เทอม 2–3=build/verify/defense, จุดตรวจ CP-A/B/C (T1) + CP-D/E/F (T2–3); asset แกน: [`artifacts/02-stage-model-s0-s10.md`](artifacts/02-stage-model-s0-s10.md)
- [Lock: template ของ deliverable 4 ชิ้น (นำร่อง 1 checkpoint)](issues/04-lock-deliverable-templates.md) — **LOCKED** แม่แบบ: spine "criteria list เดียว → 4 projection", self-check=Checklist+LLM Prompt, rubric=4 ระดับเน้น traceability/process; Student Guideline CP-B เติมเต็ม (ตัวอย่าง EEG SleepInsight); asset: [`artifacts/04-template-cp-b-house-of-quality.md`](artifacts/04-template-cp-b-house-of-quality.md)
- [ผลิต: แพ็กเกจ CP-B ให้ครบ 4 ชิ้น (reference exemplar)](issues/05-produce-cpb-full-package.md) — แพ็กเกจ CP-B ครบ (Advisor Guideline + Rubric 100 คะแนน + Self-check checklist&prompt) เป็น **gold reference** ให้ checkpoint อื่น replicate; asset: [`artifacts/05-package-cp-b-house-of-quality.md`](artifacts/05-package-cp-b-house-of-quality.md)
- [ผลิต: แพ็กเกจ CP-A (Requirements Foundation, S0–S3)](issues/06-produce-cpa-package.md) — CP-A ครบ 4 ชิ้น (criteria A1–A7, ราก = ทุก CR สืบย้อน VoC); ตัวอย่าง EEG SleepInsight VoC→CR ต่อเข้า CP-B; asset: [`artifacts/06-package-cp-a-requirements-foundation.md`](artifacts/06-package-cp-a-requirements-foundation.md)
- [ผลิต: แพ็กเกจ CP-C (Design: FD + Architecture, S6–S7)](issues/07-produce-cpc-package.md) — CP-C ครบ 4 ชิ้น (criteria FD1–FD7, แกน F↔QR + F↔C + Parnas information-hiding); ตัวอย่าง EEG function tree→component ต่อจาก QR; asset: [`artifacts/07-package-cp-c-design.md`](artifacts/07-package-cp-c-design.md)
- [ผลิต: แพ็กเกจ CP-D (Implementation + ML Experimentation, S8)](issues/08-produce-cpd-package.md) — CP-D ครบ 4 ชิ้น (criteria BD1–BD7, build→C/F + experiment log + data quality/no-leakage + progress vs QR); asset: [`artifacts/08-package-cp-d-build-progress.md`](artifacts/08-package-cp-d-build-progress.md)
- [ผลิต: แพ็กเกจ CP-E (Verification & RTM Close-out, S9)](issues/09-produce-cpe-package.md) — CP-E ครบ 4 ชิ้น (criteria VE1–VE7, **ปิดวง RTM CR→…→T**, method เหมาะ, รายงานผลไม่ผ่านตรงไปตรงมา); asset: [`artifacts/09-package-cp-e-verification.md`](artifacts/09-package-cp-e-verification.md)
- [ผลิต: แพ็กเกจ CP-F (Validation & Defense, S10)](issues/10-produce-cpf-package.md) — CP-F ครบ 4 ชิ้น (criteria VA1–VA7, validation ผู้ใช้จริง + traceability walkthrough + reflection trade-off); **ปิด worked example EEG ครบ 11 ขั้น**; asset: [`artifacts/10-package-cp-f-validation-defense.md`](artifacts/10-package-cp-f-validation-defense.md)
- [Design: แทร็ก Y4 ข้าม VoC — รากทดแทนของ traceability](issues/03-y4-voc-skip-root.md) — รากทดแทน = **Assumed CR + หลักฐาน + ป้าย "สมมติ"**; Y4 ทำ CP-A ย่อแล้วกลับเข้า CP-B เดินหน้าปกติ (CP-B–F ไม่แก้); รูปแบบ = Addendum เดียวแทน CP-A; กัน reverse-justify ด้วยหลักฐาน+assumption log+เกณฑ์ความซื่อตรงใน rubric; asset: [`artifacts/11-addendum-cp-a-y4-voc-skip.md`](artifacts/11-addendum-cp-a-y4-voc-skip.md)
- [Packaging: รวมเล่มเป็น Handbook เดียว + export Word](issues/12-packaging.md) — **v1 superseded** โดย ticket 13 (แยกเป็น 2 เล่มหลัง feedback อ่านยาก); เทคนิค pandoc page-break (raw openxml block) ยังใช้ต่อ; asset: [`superseded/handbook-v1-combined.md`](superseded/handbook-v1-combined.md)/`.docx` (ประวัติ)
- [ปรับปรุงคู่มือให้อ่านง่ายขึ้น + แยกเป็น 2 เล่ม](issues/13-rewrite-readability-split-books.md) — เขียนใหม่เป็นเรื่องเล่าขยายความ (ทำไมสำคัญ→แนวคิด→ตัวอย่าง→deliverable→เกณฑ์), ลดยึดติดเทอม, แยก**คู่มือนักศึกษา** (11 บท + ภาคผนวก 2 ตัวอย่างเสริม) กับ **คู่มืออาจารย์ที่ปรึกษา** (9 บท); rubric weighting = เท่ากันทุก checkpoint; **พร้อมแจกจริงทั้งสองเล่ม**; assets: [`../../handbooks/handbook-student.md`](../../handbooks/handbook-student.md)/`.docx`, [`../../handbooks/handbook-advisor.md`](../../handbooks/handbook-advisor.md)/`.docx`

## Not yet specified

<!-- fog — in-scope, ยังไม่คมพอจะตั้ง ticket; graduate เมื่อ frontier คืบ -->

*(ไม่มี — fog ที่เหลือทั้งหมด graduate แล้วใน ticket 13)*

## Out of scope

<!-- นอกปลายทาง; ปิด ไม่ graduate -->

- การวัดผล output แบบเดิม (UX/UI ดีไหม, ผลงานสวยไหม) — คงไว้เป็นส่วนอื่นของวิชา ไม่ใช่โฟกัสของแพ็กเกจกระบวนการนี้
