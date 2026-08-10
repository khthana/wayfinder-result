# Research: หลักการ QFD → Functional Decomposition ที่เหมาะกับโครงงาน Software + AI

Type: research
Status: resolved
Blocked by: (none)

## Question

หากระบวนการอ้างอิงที่ถูกต้องตามหลักวิศวกรรม สำหรับสายโซ่ **VoC → QFD (House of Quality) → Quantifiable/Technical Requirements → Functional Decomposition → Implementation → Verification** โดยตอบให้ได้ว่า:

1. **ลำดับขั้น canonical** ของ QFD แบบดั้งเดิม (manufacturing) คืออะไร: VoC → Affinity/KJ diagram → weighted customer requirements → House of Quality (WHATs vs HOWs, roof/correlation, target values) → technical/engineering characteristics
2. **software-QFD literature ปรับ QFD ของ manufacturing อย่างไร** (เช่น Software QFD, QFD สำหรับ requirements engineering) — อะไรที่ต้องแปลง/ตัดออก/เพิ่มสำหรับงานซอฟต์แวร์
3. **Functional Decomposition** ต่อจาก technical requirements อย่างไร (functional tree / IDEF0 / feature→function→sub-function), และเชื่อมกับ software architecture/FR-NFR ที่นักศึกษาเขียนอยู่แล้วยังไง
4. **โครงงานกึ่งวิจัย/AI** วางตรงไหนในกระบวนการ (AI performance requirement = Quantifiable Requirement ประเภทหนึ่ง? การทดลอง/เทรนโมเดลอยู่ช่วง implement? metric เป็น verification?)
5. **Verification ปิด traceability loop** กลับไปที่ Quantifiable Requirements/QRs ได้อย่างไร (แต่ละ QR → test/acceptance criteria)
6. หลัก **traceability** ที่ยึดกระบวนการนี้เข้าด้วยกัน (customer req → HoQ → QR → function → component → test)

**Output:** ไฟล์สรุป markdown เป็น linked asset (เช่น `docs/process-log/artifacts/01-research-qfd-fd-foundations.md`) — เน้นแหล่งอ้างอิงปฐมภูมิ/วิชาการ เพื่อให้ ticket lock-model (02) เอาไปตัดสินเป็น stage model จริง

ใช้ skill `/research`

## Answer

Asset: [`artifacts/01-research-qfd-fd-foundations.md`](../artifacts/01-research-qfd-fd-foundations.md) — ค้นจาก primary sources ครบทั้ง 6 คำถาม

**สาระที่ต้องเอาไปใช้ใน ticket lock-model (02):**

1. **สายโซ่ canonical**: VoC → affinity/KJ → weighted customer requirements (CR) → House of Quality (WHATs×HOWs, roof correlation, target values) → prioritized technical characteristics (TC). อ้างอิง Hauser & Clausing (HBR 1988), Akao/Mizuno. Four-phase QFD มีอยู่จริงแต่ **ชื่อ phase ไม่ standard** — คอร์สควรเลือกชุดชื่อเดียวแล้วบอกว่ามีชื่ออื่น
2. **Software QFD** (Zultner, Herzwurm/Schockert/Mellis, Haag et al. CACM 1996): เก็บแกน VoC→matrix→characteristics ไว้ แต่ **ตัด** parts/process/production phases + มัก simplify roof; **เพิ่ม** การ map HOWs เข้ากับ **ISO/IEC 25010** quality characteristics และแปลง TC เป็น measurable requirement. **Blitz QFD** = เวอร์ชันลดรูปสำหรับทีมเล็ก (ตรงกับนักศึกษา solo/pair)
3. **Functional Decomposition**: function tree (Pahl & Beitz), IDEF0/FIPS 183 (node numbering = ID scheme พร้อมใช้), Parnas 1972 (information hiding — ซ่อน design decision ที่จะเปลี่ยน เช่นตัวโมเดล). FR → leaf functions; **NFR = cross-cutting constraint** ไม่ใช่ leaf. บังคับทุก function ชี้ขึ้นหา requirement ที่มัน satisfies
4. **AI/ML**: metric (F1, latency…) = **Quantifiable Requirement / target value ใน HoQ**. สูตร QR = metric + operator + threshold + dataset + condition. Training/experiment อยู่ช่วง **implement** ภายใน function ที่ถือโมเดล; metric บน frozen eval set = **acceptance/verification**. เพิ่ม data-quality req (ISO 25012), model card/datasheet; อนุญาต must-pass vs stretch target สำหรับงานกึ่งวิจัย
5. **Verification**: 4 วิธี (Test/Demonstration/Inspection/Analysis — INCOSE/15288). "verifiable" เป็นเกณฑ์ requirement ที่ดีตาม **29148**. RTM ผูก req↔test end-to-end
6. **Traceability spine + ID scheme** (ข้อเสนอคอร์ส): `CR-xx → TC-xx → QR-xx → F-xx → C-xx → T-xx` แต่ละแถวอ้าง parent → ตรวจ RTM เชิงกลได้ (orphan = scope creep / unverified promise)

**ข้อเสนอ pipeline 10 ขั้น (S0–S10) ข้าม 3 เทอม** พร้อม artifact ต่อขั้น + worked example "StudyMate" — เป็น input ตั้งต้นให้ 02 lock (ยังไม่ถือว่า locked จนกว่าจะ grill กับอาจารย์)

**หมายเหตุความซื่อตรง:** มาตรฐาน paywalled (ISO/IEEE/HBR/ACM) อ้างโดยไม่แต่ง clause/quote; ทุกส่วน synthesis ติดป้าย [COURSE SYNTHESIS]; ไม่ได้อ้าง Barnett & Raja / Liu et al. เพราะยืนยัน bibliographic ไม่ได้รอบนี้
