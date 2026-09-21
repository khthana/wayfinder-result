# 04 — วรรณกรรมด้าน routing, context pruning และ memory สำหรับ LLM agents

Type: research
Status: resolved

## Question

"Routing/pruning policy" ที่นักศึกษาอ้างเป็น core contribution — **มีงานอะไรรองรับหรือกินพื้นที่ไปแล้วบ้าง**?

ต้องสำรวจ:
- Lost in the Middle และงานต่อยอด — long-context degradation ยังจริงกับโมเดลรุ่นใหม่ไหม หรือถูกแก้ไปแล้ว (สำคัญมาก เพราะ premise ทั้งหมดของนักศึกษาวางบนงานนี้)
- context compression / pruning: LLMLingua, selective context, provence, context distillation
- retrieval routing และ adaptive retrieval: Self-RAG, Adaptive-RAG, CRAG, router models
- agent memory systems: MemGPT/Letta, agentic memory, hierarchical summarization ของ codebase
- tool/resource selection เมื่อมี source จำนวนมาก

**ผลลัพธ์ที่ต้องการ**: routing policy แบบไหนที่ยังเป็นโจทย์เปิดจริง และแบบไหนที่ถูกทำไปแล้วจนไม่เหลือ novelty สำหรับ ป.ตรี

## Answer

รายงานเต็มที่ `research/04-routing-context-pruning.md`

### สถานะ premise "Lost in the Middle" — ถูกครึ่งเดียว และอ้างผิดเปเปอร์

- long-context degradation **ยังจริงในปี 2026** — LongBench Pro (ม.ค. 2026, 46 โมเดล) ยืนยันว่า effective context < claimed context; MRCR v2 8-needle @1M อยู่ที่ 24.5%–76% แล้วแต่โมเดล; ความจุใช้งานจริง ~60-70% ของที่ประกาศ
- **แต่กลไกเปลี่ยนไปแล้ว** — Context Rot ของ Chroma (18 โมเดล) ชี้ว่า semantic similarity และ distractor สำคัญกว่าตำแหน่ง; arXiv:2510.10276 เสนอว่า lost-in-the-middle เป็น emergent property ของ training distribution ไม่ใช่ของสถาปัตยกรรม คือ *เทรนให้หายได้*
- **นักศึกษาควรเปลี่ยนไปอ้าง** arXiv:2510.05381 (*Context Length Alone Hurts LLM Performance Despite Perfect Retrieval*) บวกเหตุผลเรื่อง cost/latency และ physical scale ซึ่งไม่หมดอายุเมื่อโมเดลเก่งขึ้น

### สิ่งที่ถูกทำไปแล้ว (ไม่เหลือ novelty)

- **arXiv:2512.05908** — hierarchical NL summaries + two-phase repo routing บน 46 repo/1.1M LOC ชนะ Copilot (0.61) และ Cursor (0.57) ที่ Pass@10 0.82 **ต้องอ้างอิง มิฉะนั้นเป็นปัญหา academic integrity**
- **"Code Isn't Memory" (arXiv:2606.22417, มิ.ย. 2026)** — ablation แบบมี p-value แสดงว่า structural index ดัน localization acc@5 จาก 44.3%→84.5% เหนือ agentic grep คือคำถาม *"routing ช่วยไหม"* ถูกตอบไปแล้ว
- abstraction-level selection — ปิดโดย RAPTOR และ A-RAG (arXiv:2602.03442)
- query-complexity routing — ปิดโดย Adaptive-RAG, RAGRouter-Bench (ก.พ. 2026) และ lightweight-router baseline study (เม.ย. 2026) ซึ่งตัวมันเองก็ขนาดเท่า senior project
- token-level compression (LLMLingua-2, Provence), tool/source selection (RAG-MCP, ToolRet, Toolshed), OS-style agent memory (MemGPT/Letta — และ Letta แสดงว่า filesystem เปล่าๆ ชนะ Mem0 บน LoCoMo 74.0 vs 68.5)

### สิ่งที่ยังเปิดอยู่จริง

1. **ไม่มี public cross-repo QA benchmark** — SWE-QA และ DeepRepoQA เป็น per-repo ทั้งคู่ *(ตรงกับข้อสรุปตั๋ว 02 โดยอิสระ)*
2. **index staleness vs routing accuracy** — 2512.05908 ระบุเองว่ายังไม่ได้ทำ
3. **cross-repo name collision / entity disambiguation** — ทั้ง A-RAG และ DeepRepoQA ระบุว่า entity confusion เป็น failure mode อันดับหนึ่ง
4. adaptive compression budgets
5. cost/accuracy Pareto ใน multi-repo

### ข้อเสนอปรับกรอบ

เปลี่ยนจาก "novel routing policy" เป็นหนึ่งใน: (ก) public multi-repo QA benchmark + paradigm comparison, (ข) cross-repo name-collision disambiguation, (ค) staleness/incremental-update study — **ทั้งสามใช้ระบบเดียวกับที่นักศึกษาจะสร้างอยู่แล้ว เปลี่ยนแค่ข้ออ้างในบทที่ 1 และ 5**

### ตรวจสอบซ้ำแล้ว (2026-09-01)

**arXiv:2606.22417 — มีจริง** *Code Isn't Memory: A Structural Codebase Index Inside a Coding Agent* (Bhola, Krishnan, Kurmala, NS; 21 มิ.ย. 2026)
ยืนยันว่าเปรียบเทียบ structural index กับ agentic-grep จริง บน SWE-PolyBench Verified + SWE-bench Pro, สามอาร์ม, สาม seed, sandbox แบบ leak-audited
⚠️ **แต่ตัวเลข acc@5 44.3%→84.5% ไม่ปรากฏใน abstract** — abstract บอกแค่ *"a large localization gain and a statistically separated resolve gain"* ตัวเลขอาจอยู่ในเนื้อเปเปอร์ **ห้ามอ้างตัวเลขนี้จนกว่าจะเปิดอ่านฉบับเต็ม**
ประเด็นที่ abstract เน้นและมีประโยชน์กว่าตัวเลข: index **ไม่แพงขึ้น** — no cost penalty per cell, cost ต่อ solve *ต่ำลง*

**arXiv:2605.16352 — มีจริง** *LARGER: Lexically Anchored Repository Graph Exploration and Retrieval* (Hu, Su, Zhao, Zhu, Haque; 8 พ.ค. 2026)
ยืนยัน +13.9 Acc@5 บน LocBench (tuned) และ +11.8 (fixed hyperparameters) โดย **ไม่ต้องมี external graph database** — integrate เข้า CLI coding agent ที่มีอยู่ได้เลย
**เบาะแสเพิ่ม**: เปเปอร์นี้ประเมินบน **MuLocBench** และ **SWE-Atlas Codebase QA** ด้วย — ทั้งสองตัวยังไม่ถูกสำรวจในตั๋ว 03 ควรตามต่อ อาจกระทบข้อสรุปว่า "ไม่มี multi-repo benchmark"
