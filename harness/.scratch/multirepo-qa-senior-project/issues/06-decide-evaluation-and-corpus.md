# 06 — ตัดสิน: corpus และวิธีวัดผล

Type: grilling
Status: resolved
Blocked by: 03, 05

## Question

จะวัดว่า contribution ที่เลือกไว้ในตั๋ว 05 **ได้ผลจริง** ด้วยอะไร?

ต้องตัดสิน:
- corpus: sock-shop/TeaStore ตามที่นักศึกษาเสนอ หรือใช้ repo จริงบน GitHub ที่มีหลาย service (หลายองค์กร open-source เป็น multi-repo จริง — อาจดีกว่า synthetic)
- ground truth มาจากไหน ใครเขียนคำถาม กี่ข้อถึงจะพอ
- baseline ที่จะเทียบด้วยคืออะไร (ต้องมีอย่างน้อยหนึ่ง baseline ที่โง่ๆ แต่แข็ง เช่น ripgrep + LLM)
- metric หลักหนึ่งตัว metric รองสองสามตัว
- **ตอบความเสี่ยงที่นักศึกษาระบุเอง**: หลักฐานจาก corpus สังเคราะห์คุณภาพพอไหม ถ้าไม่พอจะเสริมอย่างไร

## Answer

### Corpus สองชั้น

**ชั้นที่ 1 — ควบคุมได้ ใช้ตอนพัฒนา**
**Sock Shop เป็นตัวเลือกหลัก** เพราะเป็น **polyrepo จริง** — แต่ละ service แยก repo (`front-end`, `catalogue`, `carts`, `orders`, `payment`, `shipping`, `user`, `queue-master`) ต่างจาก Train Ticket และ Google Online Boutique ที่เป็น monorepo หลาย service ซึ่ง **ใช้ไม่ได้กับโจทย์ cross-repo**
> ประเด็นนี้สำคัญและถูกมองข้ามง่าย: "microservice demo" ส่วนใหญ่อยู่ใน repo เดียว ต้องเช็คก่อนเลือก

**ชั้นที่ 2 — ของจริง ใช้ตอนวัดผลจริง**
GitHub organization ที่เป็น polyrepo แท้และมี type ชื่อซ้ำข้าม repo สูง: `kubernetes` (kubernetes, api, apimachinery, client-go, kubectl — มี type ชื่อซ้ำเยอะมาก), HashiCorp, Grafana, Netflix OSS
**แก้ถ้อยคำใน proposal**: "real access is unavailable" ไม่จริง — org เหล่านี้เปิดหมด

### ผลเชิงประจักษ์ชิ้นแรกที่ต้องทำก่อนอย่างอื่น

**วัด collision rate**: ใช้ tree-sitter ดึงชื่อ top-level symbol ต่อ repo แล้วหาชื่อที่ปรากฏใน ≥2 repo
> "จากการสำรวจ N polyrepo organization พบว่า X% ของชื่อสัญลักษณ์ระดับบนสุดชนกันข้าม repo ตั้งแต่ 2 repo ขึ้นไป"

นี่เป็น **ผลลัพธ์ที่ตีพิมพ์ได้ในตัวเอง** และเป็นจุด go/no-go แรก — ถ้าการชนเกิดน้อย ต้องเปลี่ยนหัวข้อ **ตั้งแต่เดือนที่ 2 ไม่ใช่เดือนที่ 6**

### สร้าง ground truth

1. ตรวจจับการชนอัตโนมัติด้วย tree-sitter → ได้รายการ entity ที่ชน
2. สร้างคำถามจาก template ที่พุ่งเป้า entity ที่ชน (ใช้ pipeline ของ SWE-QA เป็นแบบ: taxonomy-seed → template instantiation)
3. **เป้าหมายที่คนเดียวทำไหว: ~200 คำถาม โดย ~100 ข้อตรวจด้วยมือ** (SWE-QA ทั้งทีมทำ hand-coded 1,000 ข้อ — อย่าตั้งเป้าเท่านั้น)
4. เสริมด้วยคำถามธรรมชาติจาก issue/PR history
5. **Control ที่ต้องมี** (ยืมจาก Agent Retrieval Bench): counterfactual wrong-repository controls และ no-gold examples สำหรับวัด abstention — ตัวนี้ทำให้ผลน่าเชื่อถือขึ้นมาก โดยเพิ่มงานน้อย

### Metric สามชั้น

| ชั้น | metric |
|---|---|
| **หลัก** | answer accuracy บนคำถามที่ entity ชน (LLM-as-judge) |
| component | entity resolution precision/recall, Repo-Recall@1, false-routing rate, abstention rate |
| ต้นทุน | tokens, USD, latency + cost–accuracy Pareto plot |

### ใช้พื้นสถิติของนักศึกษาให้เต็ม — นี่คือจุดแข็งที่หาได้ยากใน senior project

- รัน **≥3 seeds** รายงาน **confidence interval** ไม่ใช่เลขเดี่ยว (GPU server ทำให้ทำได้ฟรี)
- **paired test** ระหว่าง baseline กับ method บนคำถามชุดเดียวกัน
- **Cohen's κ** ระหว่าง LLM-as-judge กับ label ที่นักศึกษาตรวจเอง บน sample — StackRepoQA ตั้งเกณฑ์ไว้ที่ 0.78–0.87 ใช้เป็นเป้า
> ถ้า κ ต่ำ ต้องเลิกใช้ LLM-as-judge เป็น metric หลัก **ต้องเช็คก่อนใช้ ไม่ใช่หลัง**

### Baseline (ของจริงทั้งหมด ไม่ต้องสร้างเอง)

1. ripgrep + LLM — baseline โง่แต่แข็ง
2. `flupkede/codesearch` MCP — hybrid RRF multi-repo
3. Claude Code + `--add-dir`
4. (ถ้าเวลาเหลือ) Sourcebot
