# 05 — ตัดสิน: contribution เดียวที่วัดผลได้ของโครงงานนี้

Type: grilling
Status: resolved
Blocked by: 01, 02, 03, 04

## Question

จากหลักฐานที่ตั๋ว 01-04 หามาได้ **contribution ข้อเดียวที่โครงงานนี้จะอ้างคืออะไร**?

senior project คนเดียวรับ contribution ได้อันเดียว ตัวเลือกที่เห็นตอนนี้ (จะแคบลงหลังงานวิจัยเสร็จ):
- (ก) routing/pruning policy ข้าม repo ตามที่เสนอเดิม
- (ข) **benchmark/dataset** สำหรับ multi-repo code QA — ถ้าตั๋ว 03 พบว่ายังไม่มี อันนี้อาจแข็งกว่าและเสี่ยงน้อยกว่า
- (ค) การศึกษาเชิงเปรียบเทียบ (embedding RAG vs agentic grep vs hybrid) บน multi-repo — งานประเภทวัดผล ไม่ต้องประดิษฐ์ของใหม่ เหมาะกับคนเดียวมาก
- (ง) อย่างอื่นที่โผล่มาจากงานวิจัย

ต้องตัดสินพร้อมเหตุผลว่า **ทำไมถึงไม่เลือกอีกสามข้อ** และ contribution ที่เลือกจะ falsifiable อย่างไร

## Answer

**เลือก: Cross-repo entity disambiguation** — การแก้ปัญหาชื่อชนกันข้าม repository ใน code QA

### ข้ออ้างฉบับที่ป้องกันได้

> ในระบบ polyrepo/microservice สัญลักษณ์ชื่อเดียวกัน (`UserService`, `Order`, `Config`, `handleRequest`) ปรากฏในหลาย repository โดยมีความหมายต่างกัน retrieval คืนมาทั้งหมด แล้ว agent เอามาปนกันจนตอบผิด **โครงงานนี้วัดขนาดของปัญหานี้ในระบบจริง เสนอวิธีแก้ระดับ component และวัดผลว่าช่วยได้จริงหรือไม่**

### ทำไมข้อนี้รอด ในขณะที่ข้ออื่นไม่รอด

| ตัวเลือก | สถานะ |
|---|---|
| routing/pruning policy (ข้อเสนอเดิม) | **ตาย** — arXiv 2512.05908 ทำแล้วและชนะ Copilot/Cursor; arXiv 2606.22417 ตอบคำถาม "index ช่วยไหม" แล้ว |
| multi-repo QA benchmark | ดี แต่ต้องแข่งวางตำแหน่งกับ CodeScaleBench-Org (220 tasks) ที่ Sourcegraph มีทรัพยากรมากกว่ามาก |
| index staleness | น่าสนใจ แต่ต้องรัน pipeline หลายรอบข้ามช่วงเวลา ต้นทุนเวลาสูงสำหรับ 8 เดือน |
| **entity disambiguation** | **เปิดอยู่จริง** — A-RAG และ DeepRepoQA ระบุตรงกันโดยอิสระว่า entity confusion เป็น failure mode อันดับหนึ่ง **แต่ไม่มีใครแก้** |

**ข้อได้เปรียบเชิงยุทธศาสตร์**: เป็น contribution ระดับ *component* ไม่ใช่ระดับ *ระบบ* → **ไม่ต้องเอาชนะ Cursor หรือ Copilot แบบตัวต่อตัว** ซึ่งเป็นสิ่งที่นักศึกษาคนเดียวทำไม่ได้อยู่แล้ว แค่ต้องแสดงว่า *เพิ่ม* ชั้น disambiguation เข้าไปใน baseline แล้วดีขึ้น

### สมมติฐานที่ falsifiable

> H1: การทำ explicit cross-repo entity disambiguation เพิ่มความถูกต้องของคำตอบ บนคำถามที่ entity ชนกัน เมื่อเทียบกับ hybrid retrieval baseline ที่ไม่จัดการการชน

**ถ้าผลออกมาว่าไม่ช่วย ก็ยังเป็นโครงงานที่สมบูรณ์** — เป็น negative result ที่มีค่า เพราะขัดกับสิ่งที่งานสองชิ้นคาดเดาไว้ นี่คือเหตุผลสำคัญที่เลือกข้อนี้: **นักศึกษาไม่มีทางส่งงานไม่ได้**

### ผลพลอยได้ที่ทำให้โครงงานแข็งขึ้นโดยไม่เพิ่มงาน

การจะวัด H1 ได้ **ต้องสร้างชุดคำถามที่ entity ชนกันข้าม repo อยู่แล้ว** — ซึ่งก็คือ multi-repo QA evaluation set ขนาดเล็กที่ตรงเป้า ได้ contribution ข้อ 2 มาฟรี โดยไม่ต้องประกาศแข่งกับ CodeScaleBench

### Baseline ที่ต้องเทียบ (มีของจริงให้ใช้ ไม่ต้องสร้างเอง)

1. ripgrep + LLM (baseline โง่แต่แข็ง — ตั๋ว 01 ยืนยันว่า agentic grep แข็งจริง)
2. `flupkede/codesearch` MCP — hybrid RRF multi-repo มีอยู่แล้ว รันโลคัล
3. Claude Code + `--add-dir`
4. (ถ้าไหว) Sourcebot

### สิ่งที่ต้องเลิกอ้างทันที

- "Cursor/Copilot are single-repo" — ผิด
- "No lightweight cross-repo solution currently exists" — ผิด
- "The routing/pruning policy is the core contribution" — ถูกตีพิมพ์ไปแล้ว
- "real access is unavailable" — ผิด

### สิ่งที่ต้องอ้างอิงแน่นอน

arXiv 2512.05908, arXiv 2606.22417, arXiv 2605.16352 (LARGER), CoReQA, SWE-QA, StackRepoQA, CodeScaleBench, และเปลี่ยนจาก *Lost in the Middle* ไปอ้าง arXiv 2510.05381
