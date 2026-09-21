# 04 — Routing, Context Pruning และ Agent Memory: งานที่มีอยู่แล้ว และช่องว่างที่เหลือจริง

วันที่ค้นคว้า: 2026-09-01
ผู้อ่านเป้าหมาย: อาจารย์ที่ปรึกษา (เพื่อแปลงเป็นคำแนะนำให้นักศึกษา)
ขอบเขต: ตรวจสอบ premise ของนักศึกษา ("naive context-stuffing degrades as repo count grows, per Lost in the Middle") และสำรวจ prior art ด้าน routing / context pruning / agent memory / tool selection

---

## Executive Summary (สรุปสำหรับผู้บริหาร)

**1. Premise ของนักศึกษา "ยังจริง" แต่ "อ้างผิดเปเปอร์"**
Long-context degradation ยังไม่ถูกแก้ในปี 2026 — หลักฐานล่าสุด (LongBench Pro ม.ค. 2026, benchmark roundup มี.ค. 2026, Chroma Context Rot) ยืนยันตรงกันว่า effective context length ยังสั้นกว่า claimed context length อย่างมีนัยสำคัญ และ MRCR v2 8-needle ที่ 1M token ยังมีโมเดลระดับ frontier ทำได้เพียง 24.5%–76% แล้วแต่ผู้ผลิต แต่ **กลไกที่อธิบายเรื่องนี้ในปี 2026 ไม่ใช่ "U-shape / lost in the middle" แบบปี 2023 อีกแล้ว** — งานปี 2025–2026 ชี้ว่าตัวการหลักคือ (ก) ความยาว context ต่อให้ retrieval สมบูรณ์แบบ, (ข) distractor ที่คล้ายคำตอบ, (ค) semantic distance ระหว่างคำถามกับคำตอบ ไม่ใช่ "ตำแหน่งกลาง" เพียงอย่างเดียว **ถ้านักศึกษาอ้าง Liu et al. 2023 เป็นเสาหลักของ motivation จะโดนกรรมการซัก และควรเปลี่ยนไปอ้างงานปี 2025–2026 แทน**

**2. ไอเดีย "routing policy เลือก repo/file/abstraction-level" มี prior art ตรงตัวแล้ว ระดับที่แทบเป็นเปเปอร์เดียวกัน**
`arXiv:2512.05908` (ธ.ค. 2025) ทำ two-phase hierarchical routing: สรุป repo/directory/file เป็น natural language แล้ว route ข้าม 46 repositories (1.1M LOC) ด้วย LLM ranking — ชนะ GitHub Copilot และ Cursor ชัดเจน (Pass@10 = 0.82 vs 0.61/0.57) นี่คือ "core contribution" ที่นักศึกษาอ้างเกือบตรงตัว **นักศึกษาต้องอ่านเปเปอร์นี้ก่อนเขียนบทที่ 2 ไม่งั้นโครงงานจะไม่มี novelty เลย**

**3. ส่วนที่ "ปิดไปแล้ว" ชัดเจน:** hierarchical summarization ของ codebase, query-complexity classifier routing (Adaptive-RAG ตระกูล + RAGRouter-Bench), token-level context compression (LLMLingua-2 / Provence), tool retrieval เมื่อมี tool เยอะ (RAG-MCP / ToolRet / Toolshed), OS-style agent memory paging (MemGPT/Letta) — 4 กลุ่มนี้ undergrad ไม่ควรอ้างว่าเป็นของใหม่

**4. ส่วนที่ "ยังเปิดอยู่จริง":** (ก) การประเมิน routing แบบ multi-repo ยังไม่มี public benchmark — SWE-QA / DeepRepoQA เป็น per-repo ทั้งคู่, (ข) ต้นทุนและการ maintain summary index เมื่อโค้ดเปลี่ยน (staleness / incremental update), (ค) cross-repo dependency reasoning (ตอบคำถามที่ต้องเชื่อม 2+ repo), (ง) การเลือก abstraction level แบบมีเหตุผลควบคุมได้ ไม่ใช่ปล่อยให้ agent เดา

**5. คำแนะนำเชิงกลยุทธ์:** ให้เปลี่ยน framing จาก "เสนอ routing policy ใหม่" → เป็น "**empirical study + benchmark** ของ routing policies บน multi-repo QA ในบริบทที่ยังไม่มีใครวัด" ซึ่งเป็นงานที่ทำได้จริงในระดับ senior project และมี novelty ที่ป้องกันได้

---

## 1. "Lost in the Middle" และงานต่อยอด — สถานะปัจจุบัน

### 1.1 เปเปอร์ต้นทาง
Liu et al., *Lost in the Middle: How Language Models Use Long Contexts* (arXiv:2307.03172, ส่ง 6 ก.ค. 2023, ปรับปรุง 20 พ.ย. 2023, ตีพิมพ์ TACL 2023). ค้นพบ U-shaped performance curve: โมเดลตอบได้ดีเมื่อข้อมูลอยู่ต้นหรือท้าย context แต่แย่ลงมากเมื่ออยู่กลาง ทดสอบบน multi-document QA และ key-value retrieval

**ข้อควรระวัง:** เปเปอร์นี้ทดสอบบนโมเดลยุค 2023 (GPT-3.5, Claude 1.3, MPT-30B, LongChat) กราฟ U-shape ที่คนชอบอ้างมาจาก setting ที่ context ยาวเพียง ~2K–4K token การเอามาอ้างเพื่อ justify ระบบปี 2026 ที่ทำงานกับโมเดล 200K–1M context เป็นการ extrapolate ที่กรรมการมีสิทธิ์ตั้งคำถาม

### 1.2 งานต่อยอดที่เปลี่ยนคำอธิบายเชิงกลไก
- **Found in the Middle** (arXiv:2406.16008) — เชื่อมโยง lost-in-the-middle กับ positional attention bias และเสนอ attention calibration แก้ที่ตัวกลไก attention โดยตรง
- **Lost in the Middle: An Emergent Property from Information Retrieval Demands in LLMs** (arXiv:2510.10276, ต.ค. 2025) — เสนอว่า lost-in-the-middle **ไม่ใช่** การสูญเสียข้อมูลจริง แต่เป็น emergent property จากลักษณะ retrieval demand ในข้อมูล training: long-term retrieval demand ทำให้เกิด primacy, end-weighted short-term demand ทำให้เกิด recency, การ train ร่วมกันทั้งสองทำให้เกิดพฤติกรรม lost-in-the-middle นี่เป็นการ **ลดทอน** สถานะของปรากฏการณ์จาก "ข้อจำกัดเชิงสถาปัตยกรรม" เป็น "artifact ของ training distribution" ซึ่งแปลว่าแก้ได้ด้วยการ train
- **Context Length Alone Hurts LLM Performance Despite Perfect Retrieval** (Du et al., arXiv:2510.05381, ต.ค. 2025) — **สำคัญที่สุดสำหรับนักศึกษา** แสดงว่าแม้ retrieval จะสมบูรณ์แบบ (ป้อนเฉพาะข้อมูลที่ถูกต้อง) ประสิทธิภาพก็ยังลดลงเมื่อ context ยาวขึ้น นี่เป็น argument ที่ **แข็งกว่า** Lost in the Middle มากสำหรับ justify การ pruning เพราะมันบอกว่า "แค่ retrieve ให้ถูกไม่พอ ต้องตัดให้สั้นด้วย"
- **Pause-Tuning for Long-Context Comprehension** (arXiv:2502.20405), **Mitigate Position Bias via Scaling a Single Dimension** (arXiv:2406.02536) — mitigation ฝั่งโมเดล ไม่ใช่ฝั่งระบบ

### 1.3 Benchmark รุ่นใหม่ที่ประเมิน long-context อย่างเข้มงวด

**NoLiMa** (Modarressi et al., ICML 2025, arXiv:2502.05167) — ออกแบบ needle ที่มี lexical overlap กับคำถามน้อยที่สุด บังคับให้โมเดลต้อง infer latent association ไม่ใช่ match คำ ผลลัพธ์รุนแรง:

| Model | Effective context (85% of base) | @32K | @128K |
|---|---|---|---|
| GPT-4.1 | 16K | — | 64.7% |
| GPT-4o | 8K | 81.6% | 56.0% |
| Claude 3.5 Sonnet | 4K | 45.7% | — |
| Llama 3.3 70B | 2K | 59.5% | — |
| Gemini 1.5 Pro | 2K | — | — |
| Gemini 2.0 Flash | — | — | 16.4% |

จาก 13 โมเดลที่อ้าง 128K+ มี 11 โมเดลตกต่ำกว่า 50% ของ baseline ที่ 32K
**หมายเหตุสำคัญ:** repo ของ NoLiMa อัปเดตล่าสุดกลางปี 2025 (GPT-4.1 series, Gemini 2.5 Flash, Llama 4) **ยังไม่มีผลของโมเดล 2026** จึงใช้เป็นหลักฐาน "สถานะปัจจุบัน" ไม่ได้โดยตรง

**Chroma — Context Rot** (14 ก.ค. 2025) — ทดสอบ 18 โมเดล (Claude Opus 4 / Sonnet 4 / 3.7 / 3.5 / Haiku 3.5, o3, GPT-4.1 ตระกูล, GPT-4o, Gemini 2.5 Pro/Flash, Gemini 2.0 Flash, Qwen3 235B/32B/8B) ข้อค้นพบสำคัญ:
- ประสิทธิภาพลดลงเมื่อ input ยาวขึ้น **ในทุกโมเดล** แม้บนงานง่าย ๆ เช่น text replication
- ยิ่ง semantic similarity ระหว่างคำถามกับคำตอบต่ำ ยิ่งลดเร็ว
- distractor แต่ละตัวส่งผลไม่เท่ากัน
- **โมเดลทำได้ดีกว่าบน haystack ที่สลับลำดับ มากกว่าบน haystack ที่เรียงอย่างมีตรรกะ** (ข้อนี้ขัดสัญชาตญาณและมีนัยต่อการออกแบบ context ของนักศึกษา)
- สรุปของ Chroma: benchmark ปัจจุบันประเมิน long-context ในโลกจริงได้ไม่ดีพอ สิ่งที่สำคัญกว่าคือ **ข้อมูลถูกนำเสนออย่างไร** ไม่ใช่แค่ retrieve อะไรมา

**LongBench Pro** (arXiv:2601.02872, ม.ค. 2026) — bilingual, 1,500 ตัวอย่างจริง, 11 primary + 25 secondary tasks, ความยาว 8K–256K, ประเมิน **46 โมเดล** ข้อสรุป: "effective context length is typically shorter than the claimed context length, with pronounced cross-lingual misalignment" และ long-context training technique สำคัญกว่าการเพิ่มพารามิเตอร์

**Benchmark roundup มี.ค. 2026** (yage.ai, 15 มี.ค. 2026 — แหล่งรอง ไม่ peer-reviewed แต่รวบรวมตัวเลขที่ผู้ผลิตรายงาน) MRCR v2 8-needle @1M:

| Model | MRCR v2 8-needle @1M |
|---|---|
| Claude Opus 4.6 | 76.0% |
| Claude Sonnet 4.6 | 65.8% |
| GPT-5.4 | 36.6% |
| Gemini 3 Pro | 24.5% |
| Gemini 2.5 Pro | 16.4% |

ที่ 128K มี convergence (Gemini 3.1 Pro และ Claude Sonnet 4.6 ประมาณ 85%) และบทความสรุปว่า effective context ใช้ได้จริงราว 60–70% ของค่าที่ประกาศ Graphwalks BFS ที่ 1M ยังอยู่แถว ~40% ทั้ง Claude Opus 4.6 และ GPT-5.2

**LongBench v2** — Gemini 2.5 Pro นำที่ 63.3% (เหนือ human baseline 53.7%), GPT-4o 46.0%, Claude 3.5 Sonnet 41.0%
**RULER @128K** — มีเพียง Gemini 1.5 Pro (94.4%) และ Jamba-1.5-large (95.1%) ที่รักษาระดับ >90%

---

## 2. Context Compression / Pruning

### 2.1 สายหลักที่ถือว่า "ปิดแล้ว"
- **Selective Context** — ใช้ self-information ของโมเดลฐานตัด phrase/sentence ที่ซ้ำซ้อน ทำให้ประมวลผลเนื้อหาได้ ~2 เท่า ประหยัด compute ~40%
- **LLMLingua / LongLLMLingua** (Microsoft) — perplexity-based token removal, coarse-to-fine
- **LLMLingua-2** — เปลี่ยน paradigm เป็น **token classification**: fine-tune XLM-RoBERTa บนข้อมูลที่ GPT-4 สร้าง ทำ query-agnostic compression เร็วกว่า LLMLingua เดิม 3–6 เท่า คงความแม่นยำ 95–98%
- **RECOMP, Gisting, 500xCompressor** — สาย soft/abstractive compression
- **Provence** (Chirkova et al., ICLR 2025, arXiv:2501.16214) — context pruner สำหรับ RAG โดยเฉพาะ รวม pruning เข้ากับ reranking เป็นโมเดลเดียว **ทำได้ดีที่สุดในบรรดา pruning methods ที่ compression ratio เท่ากัน ชนะ LLMLingua ทั้งที่ใช้ compute น้อยกว่า และเป็นวิธีเดียวที่กด compression สูงได้โดยแทบไม่เสีย performance ในทุก dataset**

### 2.2 งาน 2025–2026 ที่ต่อยอด
- **MOOSComp** (arXiv:2504.16786) — แก้ over-smoothing ของ token-classification compressor แบบ LLMLingua-2 และเพิ่ม outlier score
- **AttentionRAG** (arXiv:2503.10720) — ใช้ attention ของโมเดลนำทางการ prune context ใน RAG
- **Characterizing Prompt Compression Methods for Long Context Inference** (arXiv:2407.08892) — **survey เชิงเปรียบเทียบที่นักศึกษาควรอ่าน** ข้อสรุปสำคัญ: extractive compression ด้วย **reranker** ทำได้ดีที่สุดสำหรับ multi-document QA/RAG มักเพิ่ม accuracy จากการกรอง noise พร้อมได้ compression 2–10 เท่า — กล่าวคือ **baseline ที่ง่ายที่สุดมักชนะวิธีซับซ้อน** ข้อนี้อันตรายต่อ novelty ของนักศึกษามาก
- **Prompt Compression for Large Language Models: A Survey** (arXiv:2410.12388)
- **From Retrieved Context to Runtime Control: Adaptive Compression for Edge-based RAG** (arXiv:2608.19535, ส.ค. 2026) — ชี้ว่า SOTA ปัจจุบันใช้ **fixed compression budget** ที่เลือกแบบ offline แล้วใช้ตอน inference และเสนอให้ปรับ budget แบบ runtime
- **Fixed RAG Compression Collapses Measured Reader Scaling** (arXiv:2606.21807, มิ.ย. 2026)
- **Density-aware Soft Context Compression with Semi-Dynamic Compression Ratio** (arXiv:2603.25926)

**ข้อสังเกตสำคัญ:** ช่องว่างที่งาน 2026 เองระบุคือ **adaptive/dynamic compression budget** (ตัดมากน้อยแค่ไหน ขึ้นกับ query และ context) ไม่ใช่ "จะตัดอะไร" ซึ่งถูกแก้ไปเยอะแล้ว

---

## 3. Adaptive และ Routed Retrieval

### 3.1 งานคลาสสิกที่ปิดแล้ว
- **Self-RAG** (Asai et al., arXiv:2310.11511, ICLR 2024) — ฝัง reflection token ให้โมเดลตัดสินใจเองว่าจะ retrieve ไหม และวิจารณ์ผลลัพธ์ตัวเอง
- **CRAG — Corrective RAG** (arXiv:2401.15884) — lightweight retrieval evaluator ประเมินคุณภาพเอกสารที่ดึงมา ถ้าแย่ก็ trigger web search แก้
- **Adaptive-RAG** (Jeong et al., arXiv:2403.14403, NAACL 2024) — **ตรงกับไอเดียของนักศึกษาที่สุดในกลุ่มนี้** train T5-Large classifier ทำนาย query complexity แล้ว route ไป 3 กลยุทธ์: no-retrieval / single-step / iterative
- **RAPTOR** (arXiv:2401.18059, ICLR 2024) — recursive summarization สร้าง tree แล้ว retrieve ได้หลายระดับ abstraction — **นี่คือ prior art โดยตรงของ "abstraction-level selection" ที่นักศึกษาอ้าง**
- **GraphRAG** (Microsoft, arXiv:2404.16130) — สร้าง entity-relation graph + community summary hierarchy สำหรับ query-focused summarization

### 3.2 งานปี 2025–2026
- **Self-Routing RAG** (arXiv:2504.01018) — ผูก selective retrieval เข้ากับ knowledge verbalization
- **RAGRouter-Bench** (arXiv:2602.00296, ก.พ. 2026) — **benchmark แรกที่ออกแบบมาสำหรับงาน RAG routing โดยเฉพาะ** 7,727 queries, 4 domain, ติดป้าย query type 3 แบบ (factual / reasoning / summarization)
- **Lightweight Query Routing for Adaptive RAG: A Baseline Study on RAGRouter-Bench** (arXiv:2604.03455, เม.ย. 2026) — ขยาย Adaptive-RAG ด้วย classifier ที่เบากว่า เปรียบเทียบ feature type ต่าง ๆ **นี่คือ "senior-project-shaped paper" ที่มีคนทำไปแล้วเมื่อ 5 เดือนก่อน**
- **A-RAG: Scaling Agentic RAG via Hierarchical Retrieval Interfaces** (arXiv:2602.03442, ก.พ. 2026) — **สำคัญมากต่อนักศึกษา** เปิด tool 3 ระดับ granularity ให้ agent เลือกเอง: keyword search (lexical exact) / semantic search (dense vector) / chunk read (full text) แล้วให้ agent ตัดสินใจเองว่าจะเรียกอันไหน พร้อมจำ chunk ที่อ่านแล้วเพื่อไม่ retrieve ซ้ำ ผลชนะ baseline ทั้งหมดบน HotpotQA, MuSiQue, 2WikiMultiHopQA, GraphRAG-Bench ด้วย token consumption เท่ากันหรือน้อยกว่า failure mode หลักที่เหลือคือ **entity confusion** — **กล่าวคือ "ให้ LLM เลือก abstraction level เอง" ถูกทำและ validate ไปแล้ว**
- **RouteRAG** (arXiv:2512.09487) — ใช้ RL route ระหว่าง text และ graph retrieval
- **Do We Still Need GraphRAG? Benchmarking RAG and GraphRAG for Agentic Search Systems** (arXiv:2604.09666, เม.ย. 2026)
- **A Systematic Review of Key RAG Systems: Progress, Gaps, and Future Directions** (arXiv:2507.18910)
- **AgenticRAG-Survey** (GitHub, asinghcsu) — living survey ของ agentic RAG pattern

---

## 4. Agent Memory Systems

### 4.1 ระบบหลัก
- **MemGPT** (Packer et al., arXiv:2310.08560) — virtual context management ยืมแนวคิด OS virtual memory: LLM เป็น memory manager เอง เรียก tool paging ข้อมูลระหว่าง main context (เล็ก) กับ archival store (ไม่จำกัด) ปัจจุบันพัฒนาต่อเป็นบริษัท **Letta**
- **A-MEM: Agentic Memory for LLM Agents** (arXiv:2502.12110, 2025) — ให้ agent ตัดสินใจเองว่าจะเก็บ / ดึง / ลืมอะไร
- **Mem0, Zep** — memory layer แบบ bolt-on ที่ใช้กับ agent framework ใดก็ได้

### 4.2 หลักฐาน 2026 ที่ควรทำให้ระวัง
- **Letta — Benchmarking AI Agent Memory: Is a Filesystem All You Need?** (12 ส.ค. 2025) — **ผลลัพธ์นี้สำคัญมาก** Letta agent ที่ใช้ **แค่ filesystem tool ธรรมดา** ได้ 74.0% บน LoCoMo ชนะ Mem0 ที่ใช้ memory tool เฉพาะทาง (68.5%) ข้อสรุป: "agents today are extremely effective at using filesystem tools, largely due to post-training optimization for agentic coding tasks" และ benchmark memory ปัจจุบันแยกแยะวิธีการต่าง ๆ ได้ไม่ดีพอ — **แปลว่า baseline "ปล่อยให้ agent ใช้ ls/grep/cat เอง" แข็งกว่าที่คนคิด และนักศึกษาต้องเอาชนะ baseline นี้ให้ได้**
- **State of AI Agent Memory 2026** (Mem0, 1 ก.ย. 2026 — แหล่งจากผู้ขาย ควรอ่านอย่างระวังเรื่อง bias) benchmark หลักคือ LoCoMo (1,540 คำถาม), LongMemEval (500 คำถาม), BEAM (1M/10M token) ตัวเลขที่รายงาน: Mem0 LoCoMo 92.5 / LongMemEval 94.4 / BEAM-1M 64.1 / BEAM-10M 48.6; Zep LoCoMo 80.32; Letta LoCoMo 74.0; OpenAI Memory LoCoMo 52.9 **open problem ที่ระบุเอง**: temporal abstraction (ตกลง 25% จาก 1M ไป 10M), cross-session structure, application-level evaluation, privacy, cross-session identity, **memory staleness**
- งาน 2026 ที่วิจารณ์วิธีประเมิน: **MemDelta: Controlled Baselines and Hidden Confounds in Agent Memory Evaluation** (arXiv:2606.29914), **MemFail: Stress-Testing Failure Modes of LLM Memory Systems** (arXiv:2605.26667), **Are We Ready For An Agent-Native Memory System?** (arXiv:2606.24775), **MemPro: Agentic Memory Systems as Evolvable Programs** (arXiv:2606.00619), **Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers** (arXiv:2603.07670)

### 4.3 Context engineering จาก frontier labs
- **Anthropic — Effective context engineering for AI agents** (29 ก.ย. 2025) มองว่า context engineering คือวิวัฒนาการต่อจาก prompt engineering; นิยาม **"context rot"** ว่าเป็นปรากฏการณ์ที่ความสามารถในการ recall ลดลงเมื่อ token เพิ่ม เพราะ transformer สร้างความสัมพันธ์ n² ระหว่าง token ทำให้ attention ถูกยืดจนบาง — **สังเกตว่า Anthropic อธิบายว่าเป็น "performance gradient" ไม่ใช่ "cliff"** เทคนิคที่แนะนำ: **compaction** (สรุป history เมื่อใกล้เต็ม), **structured note-taking** (memory file นอก context window), **sub-agent architecture** (agent ย่อยมี context สะอาดของตัวเอง คืนแค่สรุป), และ **just-in-time retrieval** — เก็บแค่ lightweight identifier (file path, query, link) แล้วโหลดตอน runtime แทนการ pre-process ทั้งหมด พร้อมยอมรับ trade-off ว่า runtime exploration ช้ากว่าแต่ลด context pollution และแนะนำ hybrid
  **นี่เป็นทั้งการสนับสนุนและการคุกคามต่อโครงงานนักศึกษา: สนับสนุนว่าปัญหาจริง แต่คุกคามเพราะวิธีแก้ที่แนะนำถูกเขียนเป็น best practice สาธารณะไปแล้ว**
- **Manus — Context Engineering for AI Agents: Lessons from Building Manus** (Yichao 'Peak' Ji) บทเรียนจาก production: รักษา prompt prefix ให้เสถียรเพื่อ KV-cache, ใช้ append-only context, เขียน `todo.md` ซ้ำ ๆ เพื่อดัน global plan เข้ามาใน attention span ล่าสุด, **เก็บ failure ไว้ใน context ไม่ลบ** เพื่อให้โมเดลเรียนรู้ไม่ทำซ้ำ, เป้าหมายหลักของ sub-agent คือ **context isolation** ไม่ใช่ specialization

---

## 5. Tool / Resource Selection เมื่อมีแหล่งข้อมูลจำนวนมาก

โครงสร้างปัญหา "เลือก repo ไหน" เหมือนกับ "เลือก tool ไหน" ทุกประการ และฝั่ง tool selection มีงานเยอะกว่ามาก

- **RAG-MCP: Mitigating Prompt Bloat in LLM Tool Selection via RAG** (arXiv:2505.03275, พ.ค. 2025) — ย้าย tool discovery ออกจาก prompt ไปเป็น semantic retrieval แล้วส่งเฉพาะ tool description ที่เลือกแล้วเข้าโมเดล **ลด prompt token >50% และเพิ่ม tool selection accuracy กว่า 3 เท่า (43.13% vs 13.62%)** — นี่คือ "prune before you prompt" ในรูปแบบที่ตรงกับ multi-repo routing มาก
- **ToolRet** — tool-retrieval benchmark แรก 7.6k retrieval task บน corpus 43k tools **retriever ที่ดีที่สุดได้แค่ nDCG@10 = 33.83** แสดงว่างานนี้ยังไม่ solved
- **Toolshed: Scale Tool-Equipped Agents with Advanced RAG-Tool Fusion and Tool Knowledge Bases** (arXiv:2410.14594)
- **LiveMCPBench: Can Agents Navigate an Ocean of MCP Tools?** (arXiv:2508.01780) — 95 task, 527 tools ข้อสรุป: **tool retrieval คือคอขวดหลัก** แต่ tool composition สำคัญไม่แพ้กัน
- **MCPVerse** (arXiv:2508.16260), **MCP-Bench** (ICLR 2026), **OSWorld-MCP** (arXiv:2510.24563)
- **ACE-Router: Generalizing History-Aware Routing from MCP Tools to the Agent Web** (arXiv:2601.08276, ม.ค. 2026) — routing ที่ใช้ประวัติการใช้งาน
- Stein MCP census: 177,436 distinct tools จาก 19,388 public repo ที่ verified — สเกลของปัญหาจริง

---

## 6. Code-Specific: Repository-Level Retrieval, Routing และ Benchmark

**ส่วนนี้คือ prior art ที่ตรงกับโครงงานนักศึกษาที่สุด อาจารย์ควรให้เวลานักศึกษาอ่านส่วนนี้มากที่สุด**

### 6.1 Multi-repo routing — มีคนทำแล้ว
**Natural Language Summarization Enables Multi-Repository Bug Localization by LLMs in Microservice Architectures** (Oskooei et al., arXiv:2512.05908, 5 ธ.ค. 2025)
- แปลง multi-repo codebase เป็น **hierarchical NL summary ระดับ file / directory / repository** แล้วทำ NL-to-NL search แทน cross-modal retrieval
- **Phase 1 = repository routing**: ให้ LLM จัดอันดับ top-k repo ที่เกี่ยวข้องจาก repo-level summary
- **Phase 2 = localization**: กรอง directory แล้ว rank file
- ประเมินบน DNext: **46 repos, 1.1M LOC**
- ผล: Pass@10 = 0.82 (Copilot v1.104 = 0.61, Cursor v1.5 = 0.57), MRR = 0.50 (Copilot 0.25, Cursor 0.27)
- Ablation: ตัด directory filtering ออก MRR ตก 54%
- Limitation ที่ระบุเอง: ต้นทุนสร้าง knowledge base ~$80 ครั้งแรก, ประเมินแค่โปรเจกต์ Java เดียว, **ยังไม่มีวิธีจัดตาราง maintain summary เมื่อระบบโต**

**นี่คือ core contribution ที่นักศึกษาอ้างเกือบ 1:1** ถ้าไม่อ้างอิงเปเปอร์นี้ในบทที่ 2 โครงงานจะมีปัญหาเชิงวิชาการทันที

### 6.2 Structural index vs agentic grep — มีการทดลอง controlled แล้ว
**Code Isn't Memory: A Structural Codebase Index Inside a Coding Agent** (Bhola et al., SuperAGI Research, arXiv:2606.22417, 21 มิ.ย. 2026)
- controlled ablation 3 arm: SC-ON (structural index เปิด) / SC-OFF (ปิด) / OpenCode (ripgrep ล้วน) รัน Claude Opus 4.7 บน 91 instance (Go, Java, Python) จาก SWE-PolyBench และ SWE-bench Pro
- ผล within-harness: resolve 41.9% → 50.4% (p=0.003), **localization acc@5 44.3% → 84.5% (p<0.0001)**, ต้นทุนต่อ cell ไม่ต่างกันอย่างมีนัยสำคัญ (p=0.73)
- vs OpenCode: resolve 50.4% vs 45.3% (p=0.087, marginal), cost-per-solve $2.30 vs $2.92, token 10.1k vs 14.0k (p<0.0001)
- index ได้ประโยชน์มากที่สุดกับการแก้ที่ข้ามหลายไฟล์
- Limitation: n เล็ก (75–80 paired), ไม่ครอบคลุม JS/TS, ปัญหา leakage residual

**แปลว่า "index ช่วยหรือไม่" ถูกตอบไปแล้วในระดับที่มี p-value นักศึกษาจะทำซ้ำก็ได้ แต่ไม่ควรอ้างเป็น novelty**

### 6.3 Hierarchical codebase summarization — ปิดแล้ว
- **Hierarchical Repository-Level Code Summarization for Business Applications Using Local LLMs** (arXiv:2501.07857)
- **Repository-Level Code Understanding by LLMs via Hierarchical Summarization: Improving Code Search and Bug Localization** (ICCSA 2025 Workshops, Springer, DOI 10.1007/978-3-031-97576-9_6) — สร้าง abstract repository tree + summary ระดับ project/directory/file
- **Agent4cs: A Multi-agent System for Code Summarization in Large Hierarchical Codebases** (arXiv:2607.01425, ก.ค. 2026) — สรุปสองระดับ function-level + folder-level พร้อม keyword extraction agent และ QA agent
- **HCAG: Hierarchical Abstraction and Retrieval-Augmented Generation on Theoretical Repositories** (arXiv:2603.20299)

### 6.4 Graph-based repo retrieval
**RepoGraph** (arXiv:2410.14684), **LocAgent** (แปลง codebase เป็น directed graph เปิด tool SearchEntity / TraverseGraph), **CodexGraph** (arXiv:2408.03910), **CodeRAG / GraphCodeAgent** (arXiv:2504.10046), **RANGER: Repository-level Agent for Graph-Enhanced Retrieval** (arXiv:2509.25257), **Codebase-Memory: Tree-Sitter-Based Knowledge Graphs for LLM Code Exploration via MCP** (arXiv:2603.27277), **CodeScout: An Effective Recipe for RL of Code Search Agents** (arXiv:2603.17829, มี.ค. 2026 — train policy ด้วย RL, ปล่อย code + weights)
ข้อจำกัดที่วรรณกรรมระบุตรงกัน: graph-based ต้องใช้แรงคนเยอะ, portability ข้ามภาษาต่ำ, computational overhead สูง

### 6.5 Benchmark สำหรับ repo-level QA
- **SWE-QA: Can Language Models Answer Repository-level Code Questions?** (arXiv:2509.14635, ACL Findings 2026) — 720 QA pair, 15 repos, 3.4M LOC, taxonomy สองระดับสร้างจากการวิเคราะห์ 77,100 GitHub issues จาก 12 repo ครอบคลุม intention understanding, cross-file reasoning, multi-hop dependency **แต่คำถามแต่ละข้ออยู่ภายใน repo เดียว**
- **DeepRepoQA: Code Repository Question Answering with Deep Agent Exploration** (arXiv:2608.24221, 25 ส.ค. 2026) — ใช้ MCTS ให้ agent search/navigate/inspect code baseline ที่เทียบ: direct prompting, Sliding Window RAG, Function Chunking RAG, SWE-Agent, OpenHands, Cursor, Tongyi Lingma ผล: 70.06 overall ด้วย GPT-5.1 ดีที่สุดในกลุ่ม open-source open problem ที่ระบุ: semantic ambiguity ทำให้ทุก branch retrieve ผิดเหมือนกัน, ไม่มี fallback เมื่อ semantic search ตัน, คำถามเชิง design ปลายเปิดยังตอบได้แย่
- **Beyond Code Snippets: Benchmarking LLMs on Repository-Level Question Answering** (arXiv:2603.26567)
- **CrossCodeEval**, **M2rc-Eval** (arXiv:2410.21157) — cross-file completion ไม่ใช่ QA

### 6.6 สิ่งที่ frontier coding tool ทำจริงในปี 2026
- **Claude Code ไม่ index codebase เลย** — ใช้ agentic search ด้วย grep/glob on-demand ผู้สร้าง (Boris Cherny) เล่าว่าเวอร์ชันแรกใช้ RAG + local vector DB แต่พบว่า agentic search ชนะอย่างสม่ำเสมอ เพราะ precision (grep ตรงเป๊ะ, embedding ให้ fuzzy positive), simplicity (ไม่ต้อง build/maintain index), freshness
- **Cursor** ใช้ cloud embedding index: คำนวณ Merkle tree ของ hash ไฟล์ ส่ง delta diff ไป embed แล้ว query Turbopuffer ตอน inference
- Trade-off ที่ชุมชนสรุป: Cursor ชนะเรื่อง semantic/conceptual search, Claude Code ชนะเรื่อง precision, freshness, zero setup
- **จุดที่ทั้งคู่อ่อน: multi-repository** — Claude Code ทำงานใน current directory เป็นหลัก และตรงนี้เองที่เปเปอร์ 2512.05908 เอาชนะทั้ง Copilot และ Cursor ได้

---

## สถานะของ premise: Lost in the Middle ยังจริงไหม

### คำตัดสินโดยตรง

**ปรากฏการณ์ยังจริง แต่เปเปอร์ที่นักศึกษาอ้าง "หมดอายุ" แล้ว และคำอธิบายเชิงกลไกเปลี่ยนไปแล้ว**

แยกเป็น 3 ข้อ:

**(1) "Long-context degradation ยังเป็นปัญหาจริงในปี 2026" — ✅ จริง มีหลักฐานหนักแน่น**
LongBench Pro (ม.ค. 2026, 46 โมเดล) สรุปตรง ๆ ว่า effective context length ยังสั้นกว่า claimed context length; ตัวเลข MRCR v2 8-needle @1M ยังกระจายตั้งแต่ 24.5% ถึง 76%; Graphwalks BFS @1M ยังอยู่ราว 40% แม้ในโมเดลที่ดีที่สุด; effective context ใช้ได้จริงราว 60–70% ของค่าประกาศ **นักศึกษายังมีเหตุผลที่จะไม่ยัด context**

**(2) "แต่กลไกไม่ใช่ U-shape / positional bias แบบปี 2023 อีกแล้ว" — ⚠️ นี่คือจุดที่นักศึกษาจะโดนซัก**
- Chroma (2025) พบว่าตัวแปรที่สำคัญกว่าตำแหน่งคือ **semantic similarity ระหว่างคำถามกับคำตอบ** และ **distractor** — และพบผลลัพธ์ที่ขัดกับ intuition ว่าโมเดลทำได้ดีกว่าบน haystack ที่สลับแบบสุ่ม มากกว่าบน haystack ที่เรียงอย่างมีตรรกะ
- arXiv:2510.10276 (2025) เสนอว่า lost-in-the-middle เป็น **emergent property จาก training distribution** ไม่ใช่ข้อจำกัดเชิงสถาปัตยกรรม — คือเป็นสิ่งที่ "train ให้หายได้" และผู้ผลิตก็กำลัง train ให้หายอยู่ (ตัวเลข 128K ที่ ~85% converge กันแล้วในโมเดล 2026)
- Anthropic เองอธิบายว่าเป็น **"performance gradient" ไม่ใช่ "cliff"** — ซึ่งอ่อนกว่าการอ้างว่า "ระบบพังเมื่อ repo เยอะ"

**(3) "ข้อโต้แย้งที่แข็งกว่าและควรใช้แทน" — 👍 นักศึกษาควรเปลี่ยนไปอ้างชุดนี้**
- **arXiv:2510.05381 (Context Length Alone Hurts LLM Performance Despite Perfect Retrieval)** — ตรงประเด็นที่สุด: แม้ retrieve ถูกทั้งหมด ความยาวเพียงอย่างเดียวก็ทำให้แย่ลง นี่ justify "pruning" ได้โดยตรงโดยไม่ต้องพึ่ง positional argument
- **ข้อโต้แย้งเชิงเศรษฐศาสตร์** — ต้นทุน token และ latency เมื่อ repo เพิ่มขึ้นเชิงเส้น; RAG-MCP ลด prompt token >50% พร้อมเพิ่ม accuracy 3 เท่า; Code Isn't Memory วัดได้ว่า index ลด token จาก 14.0k → 10.1k และ cost-per-solve จาก $2.92 → $2.30 **argument นี้ไม่มีวันหมดอายุแม้โมเดลจะเก่งขึ้น** เพราะยิ่ง context window ใหญ่ ต้นทุนการยัดยิ่งแพง
- **ข้อจำกัดเชิงกายภาพ** — 46 repos × 1.1M LOC ไม่มีทางใส่ 1M token window ได้อยู่ดี ต่อให้ไม่มี degradation เลย นี่เป็น argument ที่แข็งที่สุดและง่ายที่สุดที่จะป้องกัน

### สรุปคำแนะนำต่ออาจารย์
premise ไม่ผิด แต่ **การอ้างอิงผิดยุค** ให้นักศึกษา:
1. ลด Lost in the Middle จาก "เสาหลัก" เหลือ "งานบุกเบิกเชิงประวัติศาสตร์" ในบทที่ 2
2. ย้ายน้ำหนักไปที่ arXiv:2510.05381 + Chroma Context Rot + LongBench Pro
3. **ทำให้ argument หลักเป็นเรื่อง scale ทางกายภาพและต้นทุน ไม่ใช่ positional degradation** เพราะข้อแรกป้องกันได้ 100% ส่วนข้อหลังกำลังจะถูกแก้
4. เตรียมตอบคำถามกรรมการ: "ถ้าปีหน้าโมเดลมี 10M context ที่เชื่อถือได้ งานคุณยังมีความหมายไหม" — คำตอบที่ถูกคือ "มี เพราะต้นทุนและ latency" ไม่ใช่ "มี เพราะ lost in the middle"

---

## อะไรยังเปิดอยู่ / อะไรปิดไปแล้ว

### 🔴 ปิดไปแล้ว — undergrad อ้าง novelty ไม่ได้

1. **Hierarchical NL summarization ของ codebase (project → directory → file)**
   ปิดโดย arXiv:2501.07857, ICCSA 2025 (DOI 10.1007/978-3-031-97576-9_6), Agent4cs (2607.01425), HCAG (2603.20299) และที่ตรงที่สุดคือ 2512.05908

2. **Two-phase repository routing ด้วย LLM ranking บน repo-level summary**
   ปิดโดย arXiv:2512.05908 แบบตรงตัว พร้อมตัวเลข ablation และเปรียบเทียบกับ Copilot/Cursor แล้ว

3. **Query-complexity classifier ที่ route ระหว่างกลยุทธ์ retrieval**
   ปิดโดย Adaptive-RAG (2403.14403), Self-RAG (2310.11511), CRAG (2401.15884), Self-Routing RAG (2504.01018) และมี benchmark เฉพาะแล้ว (RAGRouter-Bench 2602.00296) พร้อม baseline study (2604.03455)

4. **การเลือก abstraction level / granularity ระหว่าง retrieval**
   ปิดโดย RAPTOR (2401.18059) ฝั่ง tree summarization และ A-RAG (2602.03442) ฝั่ง agentic granularity selection (keyword / semantic / chunk-read)

5. **Token-level context compression / pruning**
   ปิดโดย LLMLingua-2, Provence (2501.16214), Selective Context, RECOMP, MOOSComp และ survey ยืนยันว่า **reranker-based extractive compression เป็น baseline ที่แข็งจนวิธีซับซ้อนมักไม่ชนะ** (2407.08892)

6. **"เลือก source ไหนจากหลาย source" ในฐานะปัญหา tool retrieval**
   ปิดโดย RAG-MCP (2505.03275), Toolshed (2410.14594), ToolRet, LiveMCPBench (2508.01780), ACE-Router (2601.08276)

7. **OS-style memory paging / external memory file สำหรับ agent**
   ปิดโดย MemGPT (2310.08560), A-MEM (2502.12110), Mem0, Zep — และ Letta แสดงว่า **filesystem เปล่า ๆ ก็ได้ 74% บน LoCoMo** แล้ว

8. **"Structural index ช่วยกว่า agentic grep ไหม" ในบริบท coding agent**
   ปิดโดย Code Isn't Memory (2606.22417) ซึ่งมี controlled ablation, p-value, และการวัดต้นทุนครบ

9. **Context engineering practice ทั่วไป (compaction, note-taking, sub-agent isolation, just-in-time retrieval)**
   ตีพิมพ์เป็น public best practice โดย Anthropic (ก.ย. 2025) และ Manus แล้ว

### 🟢 ยังเปิดอยู่ — มีที่ให้ senior project ยืนได้

1. **ไม่มี public benchmark สำหรับ multi-repo QA ที่คำถามต้องข้าม repo จริง ๆ**
   SWE-QA (15 repos) และ DeepRepoQA เป็น per-repo ทั้งคู่ — คำถามอยู่ในขอบเขต repo เดียว 2512.05908 ทำ multi-repo แต่เป็น **bug localization** ไม่ใช่ QA และใช้ dataset ภายในบริษัท (DNext, Java เดียว) ที่ไม่เปิดสาธารณะ
   **→ การสร้าง public multi-repo QA benchmark ขนาดเล็กแต่สะอาด พร้อม taxonomy ของคำถามข้าม repo เป็น contribution ที่ป้องกันได้และเหมาะกับ workload คนเดียว**

2. **Index staleness และ incremental maintenance เมื่อโค้ดเปลี่ยน**
   2512.05908 ระบุเองว่า "summary maintenance scheduling needs development"; Mem0 ระบุ memory staleness เป็น open problem; Cursor ใช้ Merkle tree ทำ delta แต่ไม่มีใครวัดว่า summary ที่ล้าไป N commit ทำให้ routing accuracy ตกเท่าไร
   **→ "routing accuracy vs index staleness curve" เป็นการทดลองที่ยังไม่มีใครทำ วัดได้ และเล่าเป็นกราฟได้สวย**

3. **Cross-repo dependency reasoning**
   คำถามที่ต้องเชื่อม 2+ repo (เช่น "service A เรียก endpoint นี้ของ service B ตรงไหน และ contract ตรงกันไหม") ยังไม่มี benchmark และ DeepRepoQA ระบุว่าคำถามปลายเปิด/เชิง design ยังตอบได้แย่
   **→ routing ที่ต้องคืนหลาย repo พร้อมกัน (multi-label routing) แทน top-k ranking เป็นช่องว่างเชิงเทคนิคจริง**

4. **Adaptive / dynamic compression budget**
   งาน 2026 เองระบุว่า SOTA ยังใช้ fixed budget ที่เลือก offline (2608.19535, 2606.21807)
   **→ "ตัดมากน้อยแค่ไหนขึ้นกับอะไร" ยังเปิด แต่เตือนว่าเป็นโจทย์ยากกว่าที่ดู**

5. **Cost/latency-aware routing แบบมี explicit budget**
   วรรณกรรมส่วนใหญ่วัด accuracy เป็นหลัก มีน้อยรายที่วัด cost-per-solve อย่างเป็นระบบ (Code Isn't Memory เป็นข้อยกเว้น)
   **→ Pareto frontier ระหว่าง accuracy กับ token/dollar บน multi-repo setting เป็นมุมที่ยังว่างและวัดง่าย**

6. **Failure mode ที่งานล่าสุดยังแก้ไม่ได้ และเป็นโจทย์ขนาดพอดี**
   - A-RAG: **entity confusion** เป็น dominant failure mode
   - DeepRepoQA: **semantic ambiguity ทำให้ทุก search branch ผิดเหมือนกัน** และ **ไม่มี fallback เมื่อ semantic search ตัน**
   - ในบริบท multi-repo ปัญหานี้รุนแรงกว่ามาก เพราะชื่อคลาส/ฟังก์ชันซ้ำกันข้าม repo (เช่น `UserService` มีทุก microservice)
   **→ "name collision across repositories" เป็นปัญหาที่เฉพาะเจาะจงกับ multi-repo, ยังไม่มีใครตั้งชื่อและวัด, และ undergrad คนเดียวทำได้จริง — นี่คือช่องที่ผมแนะนำมากที่สุด**

7. **การเปรียบเทียบ paradigm ที่ยังไม่มีใครทำใน multi-repo setting**
   Claude Code (grep ล้วน) vs Cursor (embedding index) vs summary routing (2512.05908) vs graph (LocAgent/RANGER) — มีการเทียบใน single-repo แล้ว แต่ **ยังไม่มีการเทียบแบบ controlled ใน multi-repo**
   **→ replication study ที่ขยายไปสู่ multi-repo เป็นงานที่มีคุณค่าและไม่ต้องอ้าง novelty เชิงวิธีการ**

### ⚖️ คำตัดสินรวมสำหรับอาจารย์

**"Routing/pruning policy" ในฐานะ core contribution — ไม่ผ่าน** มีเปเปอร์ธันวาคม 2025 ที่ทำสิ่งเดียวกันบน 46 repos พร้อม ablation และเอาชนะเครื่องมือเชิงพาณิชย์แล้ว นักศึกษาจะไม่สามารถอ้าง novelty เชิงวิธีการได้ และถ้ากรรมการคนใดค้นเจอเปเปอร์นี้จะเป็นปัญหาใหญ่

**สิ่งที่ควรทำแทน — เลือกอย่างใดอย่างหนึ่ง:**
- **(A) Benchmark contribution** — สร้าง public multi-repo QA benchmark + taxonomy คำถามข้าม repo แล้ว evaluate 4–5 paradigm ที่มีอยู่แล้ว (grep-agentic / embedding / summary-routing / graph) นี่คือเส้นทางที่ปลอดภัยที่สุดและมีประโยชน์ต่อชุมชนจริง
- **(B) Failure-mode contribution** — โฟกัสที่ **cross-repo name collision / entity disambiguation** ซึ่งเป็น open problem ที่ทั้ง A-RAG และ DeepRepoQA ระบุเอง และรุนแรงเป็นพิเศษใน multi-repo
- **(C) Staleness contribution** — วัดความสัมพันธ์ระหว่างความล้าของ index กับ routing accuracy พร้อมเสนอ incremental update policy ซึ่ง 2512.05908 ระบุเองว่ายังไม่ได้ทำ

ทั้งสามทางเลือกยังใช้ระบบที่นักศึกษาสร้างอยู่แล้วได้ทั้งหมด เปลี่ยนแค่ **การอ้างสิทธิ์ในบทที่ 1 และ 5** ซึ่งเป็นการแก้ที่ถูกที่สุดและได้ผลที่สุด

---

## บรรณานุกรมพร้อมคำอธิบาย

### Lost in the Middle และ long-context evaluation

- **Liu et al., "Lost in the Middle: How Language Models Use Long Contexts"** — TACL 2023, ส่ง 6 ก.ค. 2023, แก้ไข 20 พ.ย. 2023 — https://arxiv.org/abs/2307.03172
  *เปเปอร์ต้นทางของ premise นักศึกษา ทดสอบบนโมเดลยุค 2023 ที่ context สั้น ควรลดสถานะเป็นงานเชิงประวัติศาสตร์*

- **Modarressi et al., "NoLiMa: Long-Context Evaluation Beyond Literal Matching"** — ICML 2025, ก.พ. 2025 — https://arxiv.org/pdf/2502.05167 | repo: https://github.com/adobe-research/NoLiMa | OpenReview: https://openreview.net/forum?id=0OshX1hiSa | PMLR: https://proceedings.mlr.press/v267/modarressi25a.html
  *ตัดสัญญาณ lexical ออก ทำให้ effective context ตกเหลือ 2K–16K ในโมเดลที่อ้าง 128K+ repo อัปเดตล่าสุดกลางปี 2025 ยังไม่มีโมเดล 2026*

- **Chroma Research, "Context Rot: How Increasing Input Tokens Impacts LLM Performance"** — 14 ก.ค. 2025 — https://www.trychroma.com/research/context-rot
  *18 โมเดล 6 การทดลอง ข้อค้นพบที่สำคัญที่สุดคือ semantic similarity และ distractor สำคัญกว่าตำแหน่ง และ haystack ที่สลับสุ่มให้ผลดีกว่าที่เรียงมีตรรกะ*

- **Du et al., "Context Length Alone Hurts LLM Performance Despite Perfect Retrieval"** — 8 ต.ค. 2025 — https://arxiv.org/pdf/2510.05381
  ***เปเปอร์ที่นักศึกษาควรใช้แทน Lost in the Middle*** *— พิสูจน์ว่าแม้ retrieval สมบูรณ์แบบ ความยาวเพียงอย่างเดียวก็ทำให้แย่ลง*

- **"Lost in the Middle: An Emergent Property from Information Retrieval Demands in LLMs"** — ต.ค. 2025 — https://arxiv.org/html/2510.10276v1 | https://openreview.net/pdf/c25408a58e3ac4cfd9f4d8f42820e1f1a710768f.pdf
  *เสนอว่าปรากฏการณ์เกิดจาก training distribution ไม่ใช่สถาปัตยกรรม → แก้ได้ด้วยการ train*

- **"Found in the Middle: Calibrating Positional Attention Bias Improves Long Context Utilization"** — 2024 — https://arxiv.org/html/2406.16008
  *mitigation ฝั่งกลไก attention*

- **"LongBench Pro: A More Realistic and Comprehensive Bilingual Long-Context Evaluation Benchmark"** — 6 ม.ค. 2026 — https://arxiv.org/abs/2601.02872 | https://arxiv.org/pdf/2601.02872
  ***หลักฐาน 2026 ที่ครอบคลุมที่สุด*** *— 46 โมเดล, 8K–256K, สรุปว่า effective < claimed ยังจริง*

- **"LongBench v2: Towards Deeper Understanding and Reasoning on Realistic Long-context Multitasks"** — https://arxiv.org/pdf/2412.15204

- **yage.ai, "Long Context Benchmarks: All Three Hit 1M — Now What?"** — 15 มี.ค. 2026 — https://yage.ai/share/long-context-benchmark-en-20260315.html
  ***แหล่งรอง ไม่ peer-reviewed*** *— รวบรวมตัวเลข MRCR v2 8-needle, Graphwalks, RULER ของโมเดล 2026 ควรตรวจสอบตัวเลขกับ model card ของผู้ผลิตก่อนอ้างในรายงาน*

- **"Mitigate Position Bias in Large Language Models via Scaling a Single Dimension"** — https://arxiv.org/pdf/2406.02536
- **"Pause-Tuning for Long-Context Comprehension"** — ก.พ. 2025 — https://arxiv.org/pdf/2502.20405

### Context compression / pruning

- **Chirkova et al., "Provence: Efficient and Robust Context Pruning for Retrieval-Augmented Generation"** — ICLR 2025, ม.ค. 2025 — https://arxiv.org/pdf/2501.16214 | https://proceedings.iclr.cc/paper_files/paper/2025/file/5e956fef0946dc1e39760f94b78045fe-Paper-Conference.pdf
  ***SOTA ของ context pruning*** *— รวม pruning + reranking เป็นโมเดลเดียว ชนะ LLMLingua ด้วย compute น้อยกว่า*

- **Microsoft LLMLingua (1/2, LongLLMLingua)** — https://github.com/microsoft/LLMLingua/blob/main/DOCUMENT.md
  *LLMLingua-2 = token classification ด้วย XLM-RoBERTa เร็วกว่ารุ่นแรก 3–6 เท่า คงความแม่นยำ 95–98%*

- **"Characterizing Prompt Compression Methods for Long Context Inference"** — 2024 — https://arxiv.org/pdf/2407.08892 | https://arxiv.org/html/2407.08892v1
  ***ควรอ่านก่อนออกแบบ pruning*** *— สรุปว่า reranker-based extractive compression เป็น baseline ที่แข็งมาก*

- **"Prompt Compression for Large Language Models: A Survey"** — https://arxiv.org/pdf/2410.12388
- **"AttentionRAG: Attention-Guided Context Pruning in RAG"** — มี.ค. 2025 — https://arxiv.org/pdf/2503.10720
- **"MOOSComp: Improving Lightweight Long-Context Compressor..."** — เม.ย. 2025 — https://arxiv.org/pdf/2504.16786
- **"From Retrieved Context to Runtime Control: Adaptive Compression for Edge-based RAG"** — ส.ค. 2026 — https://arxiv.org/abs/2608.19535 | https://arxiv.org/html/2608.19535
  *ระบุเองว่า fixed offline budget คือช่องว่างปัจจุบัน*
- **"Fixed RAG Compression Collapses Measured Reader Scaling"** — มิ.ย. 2026 — https://arxiv.org/pdf/2606.21807
- **"Density-aware Soft Context Compression with Semi-Dynamic Compression Ratio"** — https://arxiv.org/pdf/2603.25926

### Adaptive / routed retrieval

- **Asai et al., "Self-RAG: Learning to Retrieve, Generate, and Critique through Self-Reflection"** — ICLR 2024 — https://arxiv.org/abs/2310.11511 | https://selfrag.github.io/
- **Jeong et al., "Adaptive-RAG: Learning to Adapt RAG through Question Complexity"** — NAACL 2024 — https://arxiv.org/abs/2403.14403
  ***prior art ที่ใกล้ที่สุดของ "routing policy"*** *— T5-Large classifier route 3 กลยุทธ์*
- **"Corrective Retrieval Augmented Generation (CRAG)"** — https://arxiv.org/pdf/2401.15884
- **RAPTOR** — arXiv:2401.18059 — *recursive tree summarization, prior art ของ multi-abstraction retrieval*
- **GraphRAG (Microsoft), "From Local to Global: A Graph RAG Approach to Query-Focused Summarization"** — arXiv:2404.16130
- **"Self-Routing RAG: Binding Selective Retrieval with Knowledge Verbalization"** — เม.ย. 2025 — https://arxiv.org/pdf/2504.01018
- **"RAGRouter-Bench: A Dataset and Benchmark for Adaptive RAG Routing"** — ก.พ. 2026 — https://arxiv.org/pdf/2602.00296
  *benchmark เฉพาะสำหรับ RAG routing 7,727 queries*
- **"Lightweight Query Routing for Adaptive RAG: A Baseline Study on RAGRouter-Bench"** — เม.ย. 2026 — https://arxiv.org/pdf/2604.03455 | https://arxiv.org/html/2604.03455v1
  ***งานที่มีขนาดเท่า senior project และทำไปแล้ว*** *— ควรให้นักศึกษาอ่านเพื่อเทียบระดับความคาดหวัง*
- **"A-RAG: Scaling Agentic RAG via Hierarchical Retrieval Interfaces"** — 3 ก.พ. 2026 — https://arxiv.org/html/2602.03442v1
  ***prior art ของ "abstraction-level selection"*** *— agent เลือกเองระหว่าง keyword / semantic / chunk-read; failure mode ที่เหลือคือ entity confusion*
- **"RouteRAG: Efficient RAG from Text and Graph via Reinforcement Learning"** — https://arxiv.org/pdf/2512.09487
- **"Do We Still Need GraphRAG? Benchmarking RAG and GraphRAG for Agentic Search Systems"** — เม.ย. 2026 — https://arxiv.org/html/2604.09666v1
- **"A Systematic Review of Key RAG Systems: Progress, Gaps, and Future Directions"** — ก.ค. 2025 — https://arxiv.org/pdf/2507.18910
- **AgenticRAG-Survey (living repo)** — https://github.com/asinghcsu/AgenticRAG-Survey

### Agent memory และ context engineering

- **Packer et al., "MemGPT: Towards LLMs as Operating Systems"** — https://arxiv.org/abs/2310.08560 | https://ar5iv.labs.arxiv.org/html/2310.08560
- **"A-MEM: Agentic Memory for LLM Agents"** — arXiv:2502.12110 (2025) — *ระบุ ID จากผลการค้นหา ยังไม่ได้เปิดอ่านตัวเปเปอร์เต็ม ควรตรวจสอบก่อนอ้าง*
- **Letta, "Benchmarking AI Agent Memory: Is a Filesystem All You Need?"** — 12 ส.ค. 2025 — https://www.letta.com/blog/benchmarking-ai-agent-memory/
  ***สำคัญมาก*** *— filesystem เปล่า ๆ ได้ 74.0% บน LoCoMo ชนะ Mem0 (68.5%) เป็น baseline ที่นักศึกษาต้องเอาชนะ*
- **Mem0, "State of AI Agent Memory 2026: Benchmarks & Trends Report"** — 1 ก.ย. 2026 — https://mem0.ai/blog/state-of-ai-agent-memory-2026
  ***แหล่งจากผู้ขาย มี bias*** *— แต่รายการ open problem 6 ข้อ (โดยเฉพาะ memory staleness) มีประโยชน์*
- **Anthropic, "Effective context engineering for AI agents"** — 29 ก.ย. 2025 — https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
  ***เอกสารจาก frontier lab ที่ตรงประเด็นที่สุด*** *— compaction / structured note-taking / sub-agent / just-in-time retrieval*
- **Yichao 'Peak' Ji (Manus), "Context Engineering for AI Agents: Lessons from Building Manus"** — https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus | https://medium.com/@peakji/context-engineering-for-ai-agents-lessons-from-building-manus-71883f0a67f2
  *KV-cache stability, append-only context, todo.md recitation, เก็บ failure ไว้, sub-agent เพื่อ context isolation*
- **"Memory for Autonomous LLM Agents: Mechanisms, Evaluation, and Emerging Frontiers"** — https://arxiv.org/html/2603.07670v1
- **"MemDelta: Controlled Baselines and Hidden Confounds in Agent Memory Evaluation"** — มิ.ย. 2026 — https://arxiv.org/pdf/2606.29914
- **"MemFail: Stress-Testing Failure Modes of LLM Memory Systems"** — https://arxiv.org/pdf/2605.26667
- **"Are We Ready For An Agent-Native Memory System?"** — https://arxiv.org/pdf/2606.24775
- **"MemPro: Agentic Memory Systems as Evolvable Programs"** — https://arxiv.org/pdf/2606.00619

### Tool / resource selection

- **"RAG-MCP: Mitigating Prompt Bloat in LLM Tool Selection via Retrieval-Augmented Generation"** — พ.ค. 2025 — https://arxiv.org/abs/2505.03275
  ***analogy ที่ตรงที่สุดกับ "เลือก repo ไหน"*** *— ลด prompt token >50%, accuracy 13.62% → 43.13%*
- **"Toolshed: Scale Tool-Equipped Agents with Advanced RAG-Tool Fusion and Tool Knowledge Bases"** — https://arxiv.org/pdf/2410.14594
- **"LiveMCPBench: Can Agents Navigate an Ocean of MCP Tools?"** — ส.ค. 2025 — https://arxiv.org/pdf/2508.01780
  *tool retrieval คือคอขวดหลัก*
- **"MCPVerse: An Expansive, Real-World Benchmark for Agentic Tool Use"** — https://arxiv.org/pdf/2508.16260
- **MCP-Bench** — ICLR 2026 — https://proceedings.iclr.cc/paper_files/paper/2026/file/9e4b14eb6f16fe7b5818a8d633a0606a-Paper-Conference.pdf
- **"OSWorld-MCP: Benchmarking MCP Tool Invocation In Computer-Use Agents"** — https://arxiv.org/pdf/2510.24563
- **"ACE-Router: Generalizing History-Aware Routing from MCP Tools to the Agent Web"** — ม.ค. 2026 — https://arxiv.org/pdf/2601.08276
- *ToolRet (7.6k tasks / 43k tools, best nDCG@10 = 33.83) — ตัวเลขจากผลการค้นหา ยังไม่ได้เปิดเปเปอร์ต้นทาง ควรหา URL ก่อนอ้าง*

### Code-specific: repository retrieval, routing, benchmark

- **Oskooei et al., "Natural Language Summarization Enables Multi-Repository Bug Localization by LLMs in Microservice Architectures"** — 5 ธ.ค. 2025 — https://arxiv.org/html/2512.05908
  ***prior art ที่ตรงกับโครงงานนักศึกษาที่สุด — ต้องอ่านและอ้างอิง*** *— two-phase repo routing บน 46 repos / 1.1M LOC, Pass@10 0.82 vs Copilot 0.61 / Cursor 0.57*

- **Bhola et al., "Code Isn't Memory: A Structural Codebase Index Inside a Coding Agent"** — SuperAGI Research, 21 มิ.ย. 2026 — https://arxiv.org/html/2606.22417
  ***controlled ablation ที่นักศึกษาควรใช้เป็นแม่แบบวิธีวิจัย*** *— localization acc@5 44.3% → 84.5%, cost-per-solve $2.30 vs $2.92*

- **"SWE-QA: Can Language Models Answer Repository-level Code Questions?"** — ก.ย. 2025, ACL Findings 2026 — https://arxiv.org/pdf/2509.14635 | https://huggingface.co/papers/2509.14635 | https://aclanthology.org/2026.findings-acl.402/
  *720 QA pairs, 15 repos, 3.4M LOC — **per-repo ไม่ใช่ cross-repo***

- **"DeepRepoQA: Code Repository Question Answering with Deep Agent Exploration"** — 25 ส.ค. 2026 — https://arxiv.org/html/2608.24221v1 | https://openreview.net/pdf/7ad2c0eaa143bfa3eed9179418773d8d727a2dde.pdf
  ***SOTA ล่าสุดของ repo QA*** *— MCTS-based exploration, 70.06 ด้วย GPT-5.1; open problem: semantic ambiguity, ไม่มี fallback*

- **"Beyond Code Snippets: Benchmarking LLMs on Repository-Level Question Answering"** — https://arxiv.org/html/2603.26567v2
- **"Hierarchical Repository-Level Code Summarization for Business Applications Using Local LLMs"** — ม.ค. 2025 — https://arxiv.org/pdf/2501.07857
- **"Repository-Level Code Understanding by LLMs via Hierarchical Summarization"** — ICCSA 2025 Workshops — https://link.springer.com/chapter/10.1007/978-3-031-97576-9_6 | https://dl.acm.org/doi/10.1007/978-3-031-97576-9_6
- **"Agent4cs: A Multi-agent System for Code Summarization in Large Hierarchical Codebases"** — ก.ค. 2026 — https://arxiv.org/html/2607.01425 | https://arxiv.org/pdf/2607.01425
- **"HCAG: Hierarchical Abstraction and RAG on Theoretical Repositories with LLMs"** — https://arxiv.org/pdf/2603.20299
- **"RepoGraph: Enhancing AI Software Engineering with Repository-level Code Graph"** — https://arxiv.org/pdf/2410.14684
- **"CodexGraph: Bridging LLMs and Code Repositories via Code Graph Databases"** — https://arxiv.org/pdf/2408.03910
- **"RANGER: Repository-level Agent for Graph-Enhanced Retrieval"** — ก.ย. 2025 — https://arxiv.org/html/2509.25257 | https://arxiv.org/pdf/2509.25257
- **"GraphCodeAgent: Dual Graph-Guided LLM Agent for Retrieval-Augmented Repo-Level Code Generation"** — https://arxiv.org/pdf/2504.10046
- **"Codebase-Memory: Tree-Sitter-Based Knowledge Graphs for LLM Code Exploration via MCP"** — https://arxiv.org/html/2603.27277v1
- **"CodeScout: An Effective Recipe for Reinforcement Learning of Code Search Agents"** — มี.ค. 2026 — https://arxiv.org/html/2603.17829 | https://arxiv.org/pdf/2603.17829
- **"M2rc-Eval: Massively Multilingual Repository-level Code Completion Evaluation"** — https://arxiv.org/pdf/2410.21157
- **Awesome-Repo-Level-Code-Generation (living list)** — https://github.com/YerbaPage/Awesome-Repo-Level-Code-Generation
- **repo-level-codegen-papers (living list)** — https://github.com/allanj/repo-level-codegen-papers

### เครื่องมือจริงในอุตสาหกรรม

- **"Claude Code Doesn't Index Your Codebase. Here's What It Does Instead."** — https://vadim.blog/claude-code-no-indexing/
  *แหล่งรอง — อธิบายว่า Claude Code ใช้ agentic grep/glob ไม่ index และเคยลอง RAG + vector DB แล้วพบว่าแพ้*
- **"Why code search at scale is essential when you grow beyond one repository" (Sourcegraph)** — https://sourcegraph.com/blog/why-code-search-at-scale-is-essential-when-you-grow-beyond-one-repository
  *มุมมองอุตสาหกรรมต่อ multi-repo search*
- **"Indexing Code like Cursor"** — https://htoopyaelwin.medium.com/indexing-code-like-cursor-50e2f484ce55
  *อธิบาย Merkle tree + delta embedding + Turbopuffer ของ Cursor (แหล่งรอง)*
- **"Why I'm Against Claude Code's Grep-Only Retrieval? It Just Burns Too Many Tokens" (Milvus)** — https://milvus.io/blog/why-im-against-claude-codes-grep-only-retrieval-it-just-burns-too-many-tokens.md
  *มุมมองตรงข้าม จากผู้ขาย vector DB — มี bias แต่ให้ argument ฝั่ง cost ที่ใช้ได้*

---

## หมายเหตุเรื่องความน่าเชื่อถือของแหล่งข้อมูล

1. **ตัวเลขโมเดลปี 2026** (Claude Opus 4.6, GPT-5.4, Gemini 3 Pro บน MRCR v2) มาจากบทความสรุปที่ไม่ผ่าน peer review **ก่อนใส่ในรายงานฉบับจริง ควรให้นักศึกษาตรวจสอบกับ model card / system card ของผู้ผลิตโดยตรง**
2. **arXiv preprint ปี 2026 หลายรายการ** ยังไม่ผ่าน peer review — ใช้เพื่อสำรวจ landscape ได้ แต่ควรระบุสถานะ preprint ในบรรณานุกรม
3. **แหล่งจากผู้ขาย** (Mem0, Milvus, Letta) มี bias เชิงพาณิชย์ชัดเจน — ใช้ตัวเลขได้แต่ควรอ่าน conclusion อย่างระวัง
4. รายการที่ผมทำเครื่องหมายว่า "ยังไม่ได้เปิดอ่านเต็ม" (A-MEM, ToolRet) ควรตรวจสอบก่อนอ้างในเอกสารทางการ
