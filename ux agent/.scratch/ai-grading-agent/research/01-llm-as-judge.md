# 01 — LLM-as-judge สำหรับการประเมินงานออกแบบตาม rubric

Status: กำลังวิจัย (เขียนแบบสะสม — เนื้อหาเพิ่มขึ้นเรื่อย ๆ)
วันที่: 2026-08-05

> วัตถุดิบสำหรับบท literature review ของ senior project "AI Agent ช่วยตรวจงานวิชา UX/UI"
> ทุกข้ออ้างมี URL ที่ตรวจสอบได้ ถ้าไม่ยืนยันได้จะระบุว่า "ยังยืนยันไม่ได้"

---

## 1. งานวิจัยหลัก: LLM-as-a-judge

### 1.1 Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena
- Zheng, Chiang, Sheng, Zhuang, Wu, Zhuang, Lin, Li, Li, Xing, Zhang, Gonzalez, Stoica
- NeurIPS 2023 Datasets and Benchmarks Track — https://arxiv.org/abs/2306.05685
- **ผลหลัก:** GPT-4 ในฐานะ judge เห็นตรงกับ human preference **มากกว่า 80%** ซึ่งเป็น "ระดับเดียวกับที่มนุษย์สองคนเห็นตรงกันเอง" — เป็นตัวเลขอ้างอิงที่ถูกอ้างมากที่สุดในวรรณกรรมสายนี้
- เปิดเผยอคติ 3 อย่างที่กลายเป็นคำศัพท์มาตรฐาน: **position bias, verbosity bias, self-enhancement bias** + ข้อจำกัดด้าน reasoning (โดยเฉพาะโจทย์คณิตศาสตร์/ตรรกะ)
- เสนอวิธีลด: สลับตำแหน่ง (swap positions), few-shot judge, chain-of-thought, **reference-guided judging**
- ปล่อยข้อมูลสาธารณะ: MT-bench questions, 3K expert votes, 30K conversations

### 1.2 Prometheus / Prometheus 2 — evaluator model ที่รับ rubric ที่ผู้ใช้เขียนเอง
- Kim et al., "Prometheus: Inducing Fine-grained Evaluation Capability in Language Models", ICLR 2024 — https://arxiv.org/abs/2310.08491
- Kim et al., "Prometheus 2: An Open Source Language Model Specialized in Evaluating Other Language Models", EMNLP 2024 — https://arxiv.org/abs/2405.01535
- **ตัวเลขสำคัญ:** เมื่อประเมินด้วย rubric ที่คนเขียนเอง Prometheus ได้ **Pearson r = 0.897** กับผู้ประเมินมนุษย์ เทียบกับ GPT-4-0613 ที่ **0.882** และ ChatGPT ที่ **0.392**
- นัยต่อโครงงาน: โมเดล open-weight ขนาด 7B/8x7B ที่ fine-tune มาเพื่อ "อ่าน rubric แล้วให้คะแนน 1–5 พร้อมเหตุผล" ทำได้ใกล้เคียง GPT-4 → มีทางเลือก self-host บน infra คณะ

### 1.3 Position bias เชิงลึก — Large Language Models are not Fair Evaluators
- Wang, Li, Chen et al. — https://arxiv.org/abs/2305.17926 (ACL 2024)
- **ตัวเลขที่ช็อก:** แค่สลับลำดับคำตอบ ทำให้ Vicuna-13B "ชนะ" ChatGPT ได้ **66 จาก 80 คำถาม** เมื่อใช้ ChatGPT เป็นผู้ตัดสิน
- เสนอ calibration framework 3 ชั้น:
  1. **Multiple Evidence Calibration (MEC)** — บังคับให้ judge เขียนหลักฐาน/เหตุผลหลายชุดก่อนให้คะแนน
  2. **Balanced Position Calibration (BPC)** — รันทุกลำดับที่เป็นไปได้แล้วเฉลี่ย
  3. **Human-in-the-Loop Calibration** — ใช้ balanced position diversity entropy คัดเคสที่ต้องให้คนดู
- โค้ด: https://github.com/i-Eval/FairEval

---

## 2. การประเมินงาน UI/UX ด้วย LLM (ส่วนที่ตรงกับโครงงานที่สุด)

### 2.1 Generating Automatic Feedback on UI Mockups with Large Language Models
- Peitong Duan, Jeremy Warner, Yang Li, Bjoern Hartmann — **CHI 2024** — https://arxiv.org/abs/2403.13139 / https://dl.acm.org/doi/10.1145/3613904.3642782
- **นี่คืองานที่ใกล้โครงงานเรามากที่สุด**: ปลั๊กอิน **Figma** ที่รับ UI design + ชุด heuristic ที่เขียนเป็นข้อความ แล้วสร้าง feedback เป็นข้อเสนอแนะเชิงสร้างสรรค์
- ขนาดการทดลอง: ประเมิน **51 UIs** ด้วย heuristic 3 ชุด (Nielsen's 10 Usability Heuristics, Luther et al. visual design principles, Duan et al. 5 semantic grouping guidelines) + ศึกษากับ **นักออกแบบผู้เชี่ยวชาญ 12 คน**
- **ผล:** GPT-4 feedback มีประโยชน์ในการ (ก) จับ error เล็ก ๆ ที่คนมองข้าม (ข) ปรับปรุงข้อความ (ค) พิจารณา semantics ของ UI — แต่ **utility ลดลงเมื่อวนรอบซ้ำ ๆ** (iteration หลัง ๆ feedback มีประโยชน์น้อยลง)
- ผู้เข้าร่วมยังเห็นประโยชน์แม้ข้อเสนอจะไม่สมบูรณ์ → สนับสนุนโมเดล "AI ช่วยคน" ไม่ใช่ "AI แทนคน"

### 2.2 UIClip: A Data-driven Model for Assessing User Interface Design
- Jason Wu et al. — **UIST 2024** — https://arxiv.org/abs/2404.12500 / https://dl.acm.org/doi/10.1145/3654777.3676408
- โมเดลให้ "คะแนนคุณภาพการออกแบบ" จาก screenshot + คำบรรยาย train ด้วย crawling + synthetic augmentation + human ratings
- เทียบกับ UI ที่ **นักออกแบบ 12 คน** จัดอันดับ → UIClip ได้ agreement กับ ground-truth ranking สูงสุดเมื่อเทียบกับ baseline อื่น (รวม LLM ทั่วไป)
- นัย: มีทางเลือกใช้โมเดลเฉพาะทางเสริม LLM สำหรับ "คุณภาพเชิงภาพ"

### 2.3 Can GPT-4o Evaluate Usability Like Human Experts?
- Gomes et al. (ชื่อผู้แต่งต้องยืนยันเพิ่ม) — https://arxiv.org/abs/2506.16345 — ตีพิมพ์ในเล่ม Springer (HCI series)
- ให้ GPT-4o ทำ heuristic evaluation จาก screenshot ตาม Nielsen's Heuristics เทียบกับผู้เชี่ยวชาญ
- **ผลเชิงคุณภาพ:** GPT-4o เก่งกับ heuristic กลุ่ม "aesthetic and minimalist design" และ "match between system and the real world" แต่ **อ่อนมากกับ "user control and freedom" / flexibility** — และสร้าง **false positive จำนวนมากจาก hallucination**
- เสนอ hybrid: ให้ LLM ทำ pre-analysis → คนกรอง → คนโฟกัส heuristic ที่ LLM ทำไม่ได้

**ตัวเลขที่ยืนยันแล้ว (สำคัญมากสำหรับ ticket 09):**
- ผู้แต่ง: Guilherme Guerino, Luiz Rodrigues, Bruna Capeleti, Rafael Ferreira Mello, André Freire, Luciana Zaina
- Venue: **INTERACT 2025** (20th IFIP TC13 International Conference on Human-Computer Interaction)
- **GPT-4o จับได้เพียง 21.2% ของปัญหาที่ผู้เชี่ยวชาญมนุษย์พบ** (recall ต่ำมาก)
- **GPT-4o สร้างปัญหาเพิ่มอีก 27 ข้อที่ผู้เชี่ยวชาญไม่พบ** ซึ่งหลายข้อมาจาก hallucination / การ "เดา" ปัญหา
- → บทเรียน: **LLM ไม่สามารถแทน heuristic evaluation ของมนุษย์ได้** แต่ใช้เป็นชั้น pre-analysis ได้

---

## 3. อคติที่รู้จักแล้วของ LLM judge (พร้อมตัวเลข)

### 3.1 Position bias
- **หลักฐาน:** Wang et al. — สลับลำดับทำให้ผลพลิก **66/80 คำถาม** (https://arxiv.org/abs/2305.17926)
- **วิธีลด:** Balanced Position Calibration (รันทุก permutation แล้วเฉลี่ย); Zheng et al. เรียกว่า swap positions และนับเฉพาะเคสที่ผลตรงกันทั้งสองลำดับเป็น "consistent"
- **นัยต่อโครงงาน:** ถ้าใช้ **direct scoring ตาม rubric (ให้คะแนนงานทีละชิ้น)** แทน pairwise comparison → position bias แทบไม่มีผล นี่เป็นเหตุผลเชิงออกแบบที่สำคัญ

### 3.2 Verbosity / length bias
- Dubois, Galambosi, Liang, Hashimoto, "Length-Controlled AlpacaEval: A Simple Way to Debias Automatic Evaluators" — https://arxiv.org/abs/2404.04475
- LLM judge ชอบคำตอบยาวกว่าอย่างเป็นระบบ
- **วิธีลด:** fit generalized linear model ทำนาย preference จาก "ผลต่างความยาว" แล้วทำ counterfactual โดยตั้งผลต่าง = 0
- **ผล:** Spearman correlation กับ LMSYS Chatbot Arena เพิ่มจาก **0.94 → 0.98** และทนต่อการ "เขียนยาวเพื่อโกงคะแนน" ได้ดีขึ้น
- **นัยต่อโครงงาน:** นักศึกษาที่เขียนรายงาน/ใส่ annotation เยอะอาจได้คะแนนสูงเกินจริง → ต้องมี guard เช่น บังคับให้ judge อ้างหลักฐานเฉพาะเจาะจง และตรวจสอบว่าคะแนนสัมพันธ์กับความยาวหรือไม่

### 3.3 Self-enhancement / self-preference bias
- Panickssery, Bowman, Feng, "LLM Evaluators Recognize and Favor Their Own Generations", **NeurIPS 2024** — https://arxiv.org/abs/2404.13076
- GPT-3.5 Turbo, GPT-4, Llama 2 ให้คะแนนสรุปที่ตัวเองเขียนสูงกว่าที่โมเดลอื่น/มนุษย์เขียน
- **GPT-4 แยกออกว่าข้อความไหนตัวเองเขียนได้แม่น 73.5%** (เทียบกับอีก 2 LLM + มนุษย์) และพบ **correlation เชิงเส้นระหว่างความสามารถ self-recognition กับความแรงของ self-preference bias**
- **นัยต่อโครงงาน (สำคัญมาก):** ถ้านักศึกษาใช้ AI ช่วยเขียนงาน แล้วเราใช้ LLM ตระกูลเดียวกันตรวจ → คะแนนจะเอนเอียงขึ้น เชื่อมกับประเด็น "ความซื่อสัตย์ทางวิชาการ" ใน map
- ดูเพิ่ม: "Self-Preference Bias in LLM-as-a-Judge" — https://arxiv.org/abs/2410.21819

### 3.4 ความไม่คงเส้นคงวา + score distribution ผิดรูป (งานที่ให้ตัวเลขละเอียดที่สุด)
- Stureborg, Alikaniotis, Suhara (Grammarly & NVIDIA), "Large Language Models are Inconsistent and Biased Evaluators" — https://arxiv.org/abs/2405.01724
- **Inter-sample agreement ของ GPT-4 เอง: Krippendorff's α = 0.587** เทียบกับ **ผู้ประเมินมนุษย์ α = 0.659** → รันซ้ำแล้วคะแนนไม่นิ่ง แย่กว่ามนุษย์
- **Correlation กับมนุษย์บน SummEval:** ดีที่สุดที่ scale 1-10 ได้ Kendall's **τ = 0.428** เฉลี่ยทุกมิติ (มิติ Relevance ดีสุด τ = 0.462)
- **Round-number bias:** เมื่อสั่งให้ให้คะแนน 1-100 โมเดลกระจุกที่ 60, 70, 80, 90, 95 และแทบไม่ใช้ช่วง 1-60 เลย → **สเกลละเอียดไม่ได้ให้ resolution จริง**
- **Anchoring effect (สำคัญกับ rubric หลาย criterion):** เมื่อให้ประเมินหลาย attribute ในการ generate ครั้งเดียว คะแนนของ attribute ที่ตามมาถูก "ยึด" กับ attribute แรกอย่างรุนแรง — **Pearson r ระหว่าง attribute ที่ต่อเนื่องกัน: มนุษย์ = 0.315 แต่ GPT-4 = 0.979**
- **คำแนะนำที่วัดผลแล้ว:** ใช้สเกล **1-10**, **ไม่ใช้ CoT**, **temperature = 0**, **ประเมินทีละ attribute ต่อการ generate หนึ่งครั้ง**, และเก็บ source document ไว้ใน context → ปรับปรุงได้อย่างมีนัยสำคัญบน RoSE (τ = 0.220 บน CNNDM)
- ⚠️ หมายเหตุ: ข้อค้นพบ "ไม่ใช้ CoT" ขัดกับ Zheng et al. และ G-Eval — ดูหัวข้อ "สิ่งที่วรรณกรรมยังไม่ตกลงกัน" ด้านล่าง

---

## 4. Automated Essay Scoring (AES) ด้วย LLM — ตัวเลข QWK จริง

### 4.1 ผลบวก
- Stahl, Biermann, Nehring, Wachsmuth, "Exploring LLM Prompting Strategies for Joint Essay Scoring and Feedback Generation", **BEA 2024 (ACL workshop)** — https://aclanthology.org/2024.bea-1.23/
  - ทำ AES + feedback generation **พร้อมกัน** (แรงบันดาลใจจาก Chain-of-Thought) → **ช่วยให้ AES ดีขึ้น**
  - แต่ผลย้อนกลับ (คะแนนช่วยให้ feedback ดีขึ้น) **ต่ำ**
- มีรายงานว่า GPT-4 ทำ **QWK > 0.6 ใน 44% ของเคส** และเหนือกว่า GPT-3.5 อย่างชัดเจน (ต้องยืนยันเปเปอร์ต้นทางเพิ่ม — ดูหมายเหตุ)

### 4.2 ผลลบที่สำคัญมาก (ต้องอ้างในบทวิจารณ์)
- Gaggioli, Casaburi, Ercolani, Collovà, Torre, Davide (2025), "Assessing the Reliability and Validity of Large Language Models for Automated Assessment of Student Essays in Higher Education" — https://arxiv.org/abs/2508.02442
  - **67 เรียงความภาษาอิตาลี** จากวิชาจิตวิทยาระดับมหาวิทยาลัย
  - โมเดลที่ทดสอบ: Claude 3.5, DeepSeek v2, Gemini 2.5, GPT-4, Mistral 24B
  - **Human-LLM agreement ต่ำและไม่มีนัยสำคัญทางสถิติ (QWK)** — บางรายงาน QWK อยู่ในช่วง -0.04 ถึง 0.02
  - **ความเสถียรภายในโมเดลเอง: median Kendall's W < 0.30** จาก 3 รอบ prompt ซ้ำ
  - Rubric 4 เกณฑ์: Pertinence, Coherence, Originality, Feasibility → โมเดลต่าง ๆ เห็นตรงกันปานกลางเฉพาะ **Coherence** และ **Originality** แต่ **Pertinence** และ **Feasibility** แทบไม่ตรงกันเลย
  - พบ **systematic bias: ให้คะแนน Coherence สูงเกินจริง**
  - สรุปของผู้เขียน: LLM ยังทำงานที่ต้องใช้ **disciplinary insight และ contextual sensitivity** ไม่ได้

**บทเรียนเชิงวิธีวิทยาสำหรับโครงงาน:** ตัวเลข agreement ในวรรณกรรม **กระจายกว้างมาก (QWK ≈ 0 ถึง > 0.9)** ขึ้นกับ (ก) ประเภทงาน (ข) ภาษา (ค) ความชัดของ rubric (ง) ว่ามี anchor/reference หรือไม่ → **ห้ามอ้างตัวเลขจากเปเปอร์อื่นเป็นเป้าหมายของระบบตัวเอง ต้องวัด agreement บน gold set ของวิชาเอง**

---

## 5. เทคนิคที่พิสูจน์แล้วว่าเพิ่มความน่าเชื่อถือ

### 5.1 Chain-of-Thought + form-filling (G-Eval)
- Liu, Iter, Xu, Wang, Xu, Zhu, "G-Eval: NLG Evaluation using GPT-4 with Better Human Alignment", **EMNLP 2023** — https://arxiv.org/abs/2303.16634 / https://aclanthology.org/2023.emnlp-main.153/
- แนวคิด: ให้ LLM **สร้าง evaluation steps จาก criterion เอง (auto-CoT)** แล้วค่อยกรอกฟอร์มให้คะแนน + ใช้ **probability-weighted score** (ถ่วงน้ำหนักคะแนนด้วย token probability) เพื่อให้ได้คะแนนต่อเนื่องแทนจำนวนเต็ม
- **ผล:** Spearman **0.514** กับมนุษย์บนงาน summarization — เหนือ baseline เดิมทั้งหมดอย่างมีนัยสำคัญ
- **ข้อควรระวังจากเปเปอร์เอง:** G-Eval มีแนวโน้มให้คะแนนข้อความที่ LLM สร้างสูงกว่าที่มนุษย์เขียน (self-preference)

### 5.2 แตก rubric เป็น checklist แบบ binary (แนวทางที่หลักฐานหนักที่สุด)
- Lee, Kim et al., "CheckEval: A reliable LLM-as-a-Judge framework for evaluating text generation using checklists", **EMNLP 2025** — https://arxiv.org/abs/2403.18771 / https://aclanthology.org/2025.emnlp-main.796/
- 3 ขั้น: (1) มนุษย์เลือก dimension + sub-dimension (2) generate checklist เป็นคำถาม yes/no (มี question diversification + elaboration) (3) ให้โมเดลตอบ yes/no ทีละข้อ แล้วรวมเป็นคะแนน
- **ผลเชิงตัวเลข:** ทดลองกับ evaluator model **12 ตัว** → **agreement เฉลี่ยระหว่างโมเดลเพิ่มขึ้น 0.45** และ **ลด score variance** ลงชัดเจน
- ประโยชน์เสริม: ตรวจสอบย้อนได้ว่าคะแนนมาจากข้อไหน (traceable binary decisions)
- **นี่คือหลักฐานตรงที่สุดสำหรับการออกแบบ rubric ของโครงงาน: อย่าให้ AI ให้ "คะแนนรวม" ให้แตกเป็นข้อ yes/no ที่ตรวจสอบได้**
- โค้ด: https://github.com/yukyunglee/CheckEval

### 5.3 Rubric ที่เจาะจงต่อโจทย์ ดีกว่า rubric ทั่วไป
- Pathak, Gandhi, Uttam et al., "Rubric Is All You Need: Enhancing LLM-based Code Evaluation With Question-Specific Rubrics", **ICER 2025** — https://arxiv.org/abs/2503.23989 / https://dl.acm.org/doi/10.1145/3702652.3744220
- Dataset: **150 งาน Data Structures & Algorithms** + **80 งาน OOP**
- วัดด้วย Spearman correlation, **Cohen's Kappa** และ metric ใหม่ **"Leniency"** (วัดว่า LLM ใจดี/โหดกว่าผู้เชี่ยวชาญแค่ไหน)
- ผล: **rubric ที่เขียนเจาะจงต่อโจทย์เพิ่มคุณภาพการประเมินเชิงตรรกะอย่างมีนัยสำคัญ** เทียบกับ rubric ทั่วไป
- มีการเปรียบเทียบ **Complete Rubric Evaluation vs Pointwise Rubric Evaluation (PRE)** — PRE ตรวจทีละ criterion ให้ feedback ละเอียดกว่า
- **นัยต่อโครงงาน:** rubric กลาง ๆ ของวิชาไม่พอ ต้องมี rubric ต่อ assignment (สอดคล้องกับ ticket calibration)

### 5.4 Ensemble / Panel of judges แทน judge เดี่ยว
- Verga et al., "Replacing Judges with Juries: Evaluating LLM Generations with a Panel of Diverse Models" — https://arxiv.org/abs/2404.18796
- **PoLL** = ใช้โมเดลเล็กหลายตัวจาก **คนละตระกูล** โหวตกัน
- **ผล:** PoLL ของโมเดลเล็กหลายตัว **ทำได้ดีกว่า judge ใหญ่ตัวเดียว (GPT-4)** ทั้งด้านคุณภาพและ **มี intra-model bias น้อยกว่า** เพราะตระกูลโมเดลไม่ทับกัน + **ถูกกว่ามาก**
- ทดลองบน 3 judge settings และ 6 datasets
- **นัยต่อโครงงาน:** เป็นทางออกเชิงงบประมาณสำหรับ infra คณะ และลด self-preference bias ไปพร้อมกัน

### 5.5 Multiple Evidence Calibration / บังคับให้อ้างหลักฐานก่อนให้คะแนน
- Wang et al. (https://arxiv.org/abs/2305.17926) — MEC: generate เหตุผลหลายชุดก่อนคะแนน แล้วเฉลี่ย → ลด bias ได้จริง
- Wang, Ding, Wu et al., "AutoSCORE: Enhancing Automated Scoring with Multi-Agent Large Language Models via Structured Component Recognition", **AAAI 2026** — https://arxiv.org/abs/2509.21910
  - สถาปัตยกรรม 2 agent: (1) **Scoring Rubric Component Extraction Agent** ดึง "หลักฐาน" ที่ตรงกับแต่ละองค์ประกอบของ rubric ออกจากงานนักศึกษาก่อน แล้วเข้ารหัสเป็นโครงสร้าง (2) **Scoring Agent** ให้คะแนนจากโครงสร้างนั้น
  - โมเดล: GPT-4o, LLaMA-3.1-8B, LLaMA-3.1-70B; ข้อมูล: **4 dataset จาก ASAP benchmark**
  - **ผล:** ดีขึ้นทั้ง accuracy, QWK, correlation และลด MAE/RMSE เทียบกับ single-agent — **ได้ประโยชน์มากที่สุดกับ rubric ที่ซับซ้อนหลายมิติ และกับโมเดลเล็ก**
  - **นี่คือ pattern "extract evidence → then score" ที่ควรลอกมาใช้ตรง ๆ**

### 5.6 การ align เกณฑ์กับอาจารย์ (criteria drift) — EvalGen
- Shankar, Zamfirescu-Pereira, Hartmann, Parameswaran, Arawjo, "Who Validates the Validators? Aligning LLM-Assisted Evaluation of LLM Outputs with Human Preferences", **UIST 2024** — https://arxiv.org/abs/2404.12272 / https://dl.acm.org/doi/10.1145/3654777.3676450
- ชี้ปัญหาแบบ "เต่าซ้อนเต่า": LLM ที่ทำหน้าที่ประเมิน **สืบทอดปัญหาทั้งหมดของ LLM ที่มันประเมิน** → ต้องมีมนุษย์ validate ตัว evaluator เอง
- EvalGen: ให้มนุษย์ให้เกรดตัวอย่างชุดย่อย → ใช้ feedback นั้นเลือก implementation ของ evaluator ที่ตรงกับมนุษย์ที่สุด
- **ข้อค้นพบเชิงคุณภาพที่สำคัญ:** ผู้ใช้ "**ปรับเกณฑ์ไปเรื่อย ๆ ระหว่างที่เห็นผลลัพธ์**" (criteria drift) — เกณฑ์ไม่ได้นิ่งตั้งแต่ต้น
- **นัยต่อโครงงาน:** ยืนยันว่า **calibration loop เป็นฟีเจอร์จำเป็น ไม่ใช่ของแถม** — สอดคล้องกับข้อเท็จจริงที่ปักหมุดไว้ใน map

---

## 6. การออกแบบ prompt / agent สำหรับการตรวจ

### 6.1 Single-agent vs multi-agent — หลักฐานยังผสม
- ฝั่งสนับสนุน multi-agent: AutoSCORE (https://arxiv.org/abs/2509.21910) แยก extract-evidence agent กับ scoring agent → ดีขึ้นชัด โดยเฉพาะ rubric หลายมิติ
- แต่มีหลักฐานว่า **ความซับซ้อนไม่ได้การันตีผลดีเสมอ**: มีรายงานว่า single-agent + few-shot ชนะ multi-agent ในการประเมิน student reflection
  - ดู "Specialists or Generalists? Multi-Agent and Single-Agent LLMs for Essay Grading" — https://arxiv.org/abs/2601.22386 *(เปเปอร์ใหม่ ควรตรวจสอบตัวเลขก่อนอ้างในเล่ม)*
- **ข้อสรุปที่ปลอดภัย:** แยก **stage** (extract evidence → score → aggregate) มีหลักฐานหนุนชัดกว่าแยก **agent ต่อ criterion** เฉย ๆ

### 6.2 หลักการเชิง prompt ที่มีตัวเลขรองรับ (สรุปใช้ได้ทันที)
จาก Stureborg et al. (https://arxiv.org/abs/2405.01724) และ CheckEval:
1. **สเกล 1-10 หรือ binary — ไม่ใช่ 1-100** (round-number bias ทำให้สเกลละเอียดไม่มีความหมาย)
2. **ประเมินทีละ criterion ต่อการเรียกโมเดลหนึ่งครั้ง** เพื่อตัด anchoring effect (GPT-4 r = 0.979 ระหว่าง attribute ที่ต่อกัน vs มนุษย์ 0.315)
3. **temperature = 0**
4. **เก็บชิ้นงานต้นฉบับไว้ใน context ตลอด** อย่าให้ตัดสินจากบทสรุปของตัวเอง
5. **บังคับให้อ้างหลักฐานเฉพาะเจาะจงก่อนให้คะแนน** (MEC / AutoSCORE)
6. ใช้ **direct scoring ตาม rubric** แทน pairwise → หนี position bias

---

## 7. Vision / multimodal judge — สิ่งที่ต้องรู้ก่อนตรวจงานที่เป็นภาพ

### 7.1 MLLM-as-a-Judge (benchmark หลักของสาขา)
- Chen et al., "MLLM-as-a-Judge: Assessing Multimodal LLM-as-a-Judge with Vision-Language Benchmark", **ICML 2024** — https://arxiv.org/abs/2402.04788 / https://mllm-judge.github.io/
- ทดสอบ 3 รูปแบบงาน: **Scoring Evaluation, Pair Comparison, Batch Ranking**
- **ผลสำคัญมากสำหรับโครงงาน:** MLLM ทำ **Pair Comparison** ได้ใกล้เคียงมนุษย์ แต่ **เบี่ยงเบนจาก human preference อย่างมีนัยสำคัญใน Scoring Evaluation และ Batch Ranking**
- พบปัญหาเรื้อรัง: อคติหลากหลาย, **hallucinatory responses**, และความไม่คงเส้นคงวาของคำตัดสิน
- ⚠️ **ความตึงเชิงออกแบบ:** ข้อ 6.2 บอกว่าให้เลี่ยง pairwise (position bias) แต่ MLLM-as-a-Judge บอกว่า vision model ทำ **scoring** ได้แย่กว่า **pairwise** → ทางออกที่วรรณกรรมชี้: ใช้ **anchor/reference exemplar** (ตัวอย่างงานระดับ A/B/C) แล้วให้ AI เทียบชิ้นงานกับ anchor แทนการให้คะแนนลอย ๆ — และยังต้องสลับตำแหน่ง anchor

### 7.2 Heuristic evaluation หลายหน้าจอ
- Lubos, Felfernig, Garber, Leitner, Schwazer, Henrich, "Investigating Multimodal Large Language Models to Support Usability Evaluation" — **IEA/AIE 2026, Springer LNAI** — https://arxiv.org/abs/2508.16165
- กรอบคิดที่น่าสนใจ: มอง MLLM เป็นเครื่องมือ **จัดลำดับความสำคัญ (prioritization)** ของปัญหา ไม่ใช่เครื่องมือ "ตัดสิน" — ให้ระบุ + อธิบายปัญหา + จัดอันดับความรุนแรง แล้วมนุษย์รีวิวผ่าน visualization tool
- สรุปของผู้เขียน: MLLM ให้ **complementary insights** — เสริม ไม่ใช่แทน
- ดูเพิ่ม: "AIHeurEval: Generating Heuristic Evaluations on Multiple UI Screens with Multimodal Large Language Models" — https://link.springer.com/chapter/10.1007/978-3-031-94168-9_22 *(ยังไม่ได้ตรวจตัวเลข)*

### 7.3 Accessibility — สายที่ deterministic tool ยังชนะ
- Apple ML Research, "Towards Automated Accessibility Report Generation for Mobile Apps", **ACM TOCHI 2024** — https://arxiv.org/abs/2310.00091 / https://dl.acm.org/doi/10.1145/3674967
- ระบบผสม: app crawling + manual recording + **accessibility scanner ที่มีอยู่แล้ว** แล้วใช้ ML ทำ de-duplication/summarization
- **ตัวเลข:** screen grouping model **accuracy 96.9% / F1 88.8%**; UI element matching heuristics **accuracy 97% / F1 98.2%**
- **บทเรียน:** งานที่เป็น "กฎเชิงกำหนด" (contrast ratio, touch target size, alt text) ควรใช้ **เครื่องมือตรวจแบบ deterministic** ไม่ใช่ LLM — LLM ควรทำหน้าที่สรุป/จัดกลุ่ม/อธิบายเท่านั้น
- นัยต่อโครงงาน: ถ้า rubric มีข้อเรื่อง contrast/ขนาดตัวอักษร → ควรดึงจาก **Figma API/plugin** มาคำนวณตรง ๆ แทนที่จะให้ vision model "มองเอา" (สอดคล้องกับ Duan et al. ที่ป้อน JSON ของ UI ให้ GPT-4 ด้วย ไม่ใช่แค่ภาพ)

---

## 8. หลักฐานจากการนำไปใช้จริงในห้องเรียน

### 8.1 LLM เป็นผู้ตรวจงานในวิชาที่มีนักศึกษา 1,000+ คน (เคสที่ตรงที่สุด)
- Chiang, Chen, Kuan, Yang, Lee (NTU), "Large Language Model as an Assignment Evaluator: Insights, Feedback, and Challenges in a 1000+ Student Course", **EMNLP 2024 (main)** — https://arxiv.org/abs/2407.05216
- ขนาด: **1,028 นักศึกษา** ใช้ GPT-4 ตรวจงานจริง
- **ผล:** นักศึกษายอมรับได้ระดับหนึ่งเมื่อเข้าถึงผู้ตรวจ AI ได้ฟรี **แต่พบ failure mode ร้ายแรง 2 ข้อ**
  1. **LLM ไม่ทำตามคำสั่งประเมิน (instruction adherence ล้มเหลว)** เป็นครั้งคราว
  2. **นักศึกษา prompt-hack ตัวผู้ตรวจได้** — บังคับให้ output สตริงที่ต้องการ → **ได้คะแนนสูงโดยไม่ทำตาม rubric**
- **นัยต่อโครงงาน (สำคัญมาก):** ต้องมี **prompt-injection defense** ในสถาปัตยกรรม — ข้อความในไฟล์งานนักศึกษา (ทั้งใน PDF, ใน Figma layer name, ใน alt text) ต้องถูกปฏิบัติเป็น **untrusted data** ไม่ใช่คำสั่ง

### 8.2 มุมมองอาจารย์/นักเรียนต่อการให้ AI ตรวจ
- Tian, Liu, Esbenshade, Sarkar, Zhang, He, Sun (2025), "Implementation Considerations for Automated AI Grading of Student Work" — https://arxiv.org/abs/2506.07955
- Co-design pilot กับ **ครู 19 คน** ในบริบท K-12 (usage logs + survey + interview)
- **ผล:** ครูเห็นค่ากับ **feedback เชิงบรรยายที่ได้เร็ว เพื่อใช้เชิง formative** และนักเรียนชอบ feedback ที่นำไปแก้งานต่อได้เร็ว
- **แต่:** ครู **ไม่ไว้ใจการให้คะแนนอัตโนมัติ** และเรียกร้อง human oversight; นักเรียนก็ยังกังขากับ "AI-only grading"
- ข้อเสนอ: เครื่องมือต้อง **teacher-centered** และรักษา **pedagogical agency** ของอาจารย์
- **นัยต่อโครงงาน:** จุดขายของระบบควรเป็น "AI เตรียมหลักฐาน + ร่างคะแนน, อาจารย์ตัดสิน" ไม่ใช่ "AI ให้เกรด" — ตรงกับ ticket 14

---

## 9. เอกสารทางการของผู้ให้บริการโมเดล

### Anthropic — "Demystifying evals for AI agents"
- https://www.anthropic.com/engineering/demystifying-evals-for-ai-agents
- คำแนะนำที่ตรงกับหลักฐานเชิงวิชาการอย่างน่าสนใจ:
  - **"สร้าง rubric ที่ชัดเจนและมีโครงสร้างสำหรับแต่ละมิติ แล้วให้ LLM-as-judge แยกตัวตรวจแต่ละมิติ แทนที่จะใช้ตัวเดียวตรวจทุกมิติ"** → ตรงกับผล anchoring effect ของ Stureborg et al. (r = 0.979)
  - rubric ควรมี **explicit assertions** — ข้อความที่วัดได้เป็นรูปธรรม เช่น "Agent showed empathy for customer's frustration" ไม่ใช่คำตัดสินคุณภาพลอย ๆ
  - **"LLM-based rubrics ควรถูก calibrate กับวิจารณญาณผู้เชี่ยวชาญมนุษย์บ่อย ๆ"** โดยเฉพาะโดเมนที่เป็นอัตวิสัย
  - ให้ทางออกกับโมเดล: ใส่คำสั่งให้ตอบ **"Unknown"** เมื่อข้อมูลไม่พอ → กัน hallucinated justification
  - ออกแบบ **partial credit** แทน pass/fail ล้วน
  - ยังต้องมี human review ก่อน deploy เต็มรูปแบบ

*(หมายเหตุ: ควรตรวจสอบหน้า docs อย่างเป็นทางการของ Anthropic/OpenAI เพิ่มเติมก่อนอ้างในเล่ม — เนื้อหาข้างต้นมาจาก engineering blog ของ Anthropic ซึ่งเป็นแหล่งปฐมภูมิของผู้ให้บริการ)*

---

## 10. Survey หลักของสาขา (ใช้เป็นแหล่งอ้างอิงกรอบคิด)
- Li, Jiang, Huang, Beigi, Zhao, Tan, Bhattacharjee, Jiang, Chen, Wu, Shu, Cheng, Liu, "From Generation to Judgment: Opportunities and Challenges of LLM-as-a-judge", **EMNLP 2025** — https://arxiv.org/abs/2411.16594
- เสนอ taxonomy 3 แกน: **What to judge / How to judge / How to benchmark** — ใช้เป็นโครงบทที่ 2 ของรายงานโครงงานได้เลย

