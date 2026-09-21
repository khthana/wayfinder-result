# Landscape ของ Code Retrieval สำหรับ LLM Coding Agents

**วันที่ทำวิจัย:** 2026-09-01
**ขอบเขต:** งานวิจัย peer-reviewed (arXiv / ACL / ICLR / ICSE-ICSME) + engineering evidence จากเครื่องมือจริง (Anthropic, Cursor, Cline, Cognition/Windsurf, Sourcegraph, NVIDIA)
**หมายเหตุเรื่องความน่าเชื่อถือ:** ทุก URL ในเอกสารนี้มาจากผลการค้นหาจริงหรือถูก fetch จริง รายการที่ fetch ไม่สำเร็จหรือได้มาจาก search snippet เท่านั้น ถูกทำเครื่องหมาย `[ยังไม่ verify โดยตรง]` ไว้

---

## Executive Summary (5 บรรทัด)

1. **ปัญหาหลักของ embedding RAG บนโค้ดคือ "ความแม่นยำ" ไม่ใช่ "ความเร็ว"** — CORE-Bench (arXiv 2606.11864) พบว่า Qwen3-Embedding-8B ทำได้ 71.7 NDCG@10 บน code search แบบดั้งเดิม แต่ตกเหลือ **20.3 NDCG@10** บนงาน issue-to-edit localization ซึ่งเป็นงานจริงของ coding agent
2. **Query latency ของ dense retrieval เป็นหลัก milliseconds และแทบไม่ใช่คอขวด** แต่ latency *เชิงเปรียบเทียบ* แย่กว่า sparse มาก — arXiv 2510.20609 วัดได้ว่า dense encoder ช้ากว่า BM25 ราว **100x** และ config ที่ต่างกันทำให้ retrieval latency ต่างกันได้ถึง **200x**
3. **ต้นทุนจริงของ RAG อยู่ที่ indexing/re-indexing และ staleness** — Cursor รายงานว่า index repo ที่ p99 ใช้เวลา **4.03 ชั่วโมง** ก่อนทำ index reuse และเหลือ **21 วินาที** หลังทำ (blog 2026-01-27)
4. **หลายเครื่องมือเชิงพาณิชย์ย้ายไป agentic search จริง (Claude Code, Cline, Sourcegraph Cody/Amp, Codex CLI) แต่หลักฐานเชิงตัวเลขที่เผยแพร่นั้น "บาง"** — Boris Cherny (ผู้สร้าง Claude Code) พูดเองว่า *"This was just vibes, so internal vibes."*
5. **หลักฐานวิชาการปี 2026 ชี้ว่า "ไม่มีผู้ชนะเดี่ยว"** — agentic explorer ชนะ sparse retrieval ขาดลอย (SWE-Explore: HitRegion 0.51-0.53 vs 0.07-0.12) แต่ embedding ที่ fine-tune เฉพาะงานก็ชนะ agentic baseline ได้ (NV-EmbedCode 71.95% file recall vs Agentless 65.55%) และ **Cursor วัด A/B จริงได้ว่า semantic search ช่วยเพิ่มความแม่นยำเฉลี่ย 12.5%** เหนือ grep-only

> **บทสรุปสำหรับอาจารย์:** คำตอบที่ถูกต้องในปี 2026 ไม่ใช่ "RAG หรือ agentic" แต่คือ **agentic loop เป็น control plane + มี retrieval tool หลายชั้น (grep/regex index → structural/graph → semantic) ให้ agent เรียกใช้เอง**

---

## 1. Embedding-based RAG (chunk + vector search) บนโค้ด

### 1.1 จุดอ่อนเรื่อง recall/accuracy — นี่คือจุดอ่อนที่ใหญ่ที่สุด

**หลักฐานหลัก: CORE-Bench (arXiv:2606.11864, v1 2026-06-10, v3 2026-08-24)**
เป็น benchmark ที่แยกวัด code retrieval 3 ระดับ: (1) code understanding, (2) issue-to-edit localization, (3) broader context retrieval สร้างจาก SWE-bench-series มี 180K+ queries

ตัวเลขจริงของ Qwen3-Embedding-8B:

| ระดับงาน | NDCG@10 | Recall@100 |
|---|---|---|
| Level-1: traditional code search | **71.7** | 96.9 |
| Level-2: issue-to-edit localization | **20.3** | 48.0 |
| Level-3: broader context retrieval | **34.4** | 41.5 |

คำพูดสำคัญจากเปเปอร์:
> "Experiments with representative embedding models show a sharp drop from traditional code search to code retrieval in agentic coding settings."

> "Strong performance on existing benchmarks may therefore overstate a model's usefulness for coding agents."

**นัยสำหรับนักศึกษา:** อย่าเชื่อคะแนน CodeSearchNet / CoIR ของ embedding model โดยตรง มันไม่ทำนายผลบนงานจริงแบบ "แก้ issue นี้ต้องแตะไฟล์ไหน"

**หลักฐานเสริม: SWE-Explore (arXiv:2606.07297, 2026-06-05)** — 848 issues, 10 ภาษา, 203 repos ผลที่ K=5 regions:

| Explorer | HitRegion | Precision | Line Recall | Context Efficiency |
|---|---|---|---|---|
| Oracle | 0.915 | 1.000 | 0.953 | 1.000 |
| General coding agents (Claude Code / Mini-SWE-Agent / OpenHands) | 0.51–0.53 | 0.49–0.60 | 0.15–0.18 | 0.754–0.829 |
| Specialized localizers (LocAgent) | 0.472 | 0.642 | — | — |
| Sparse retrievers (BM25, TF-IDF) | **0.07–0.12** | 0.06–0.12 | — | 0.087–0.190 |

> "Agentic exploration is a clear step above non-agentic retrieval"

*ข้อควรระวังในการอ่าน:* ตารางนี้ไม่ได้แยก dense embedding retriever ออกมาเป็นแถวชัดเจนใน excerpt ที่ดึงมาได้ ตัวเลข 0.07–0.12 เป็นของ sparse (BM25/TF-IDF) — อย่าเหมารวมว่า dense แย่เท่ากัน

### 1.2 Chunk-boundary problem — มีจริง แต่ผลกระทบเล็กกว่าที่คิด

**cAST (arXiv:2506.15655, 2025-06-18; Findings of EMNLP 2025)** เสนอ AST-based chunking (tree-sitter, recursive split-then-merge):
> "Existing line-based chunking heuristics often break semantic structures, splitting functions or merging unrelated code, which can degrade generation quality."

ผลที่ได้: **Recall@5 +4.3 points** บน RepoEval, **Pass@1 +2.67 points** บน SWE-bench

**แต่มีหลักฐานขัดแย้ง: Practical Code RAG at Scale (arXiv:2510.20609, 2025-10-23, JetBrains Research)**
> "(4) Simple line-based chunking matches syntax-aware splitting across budgets."

และ:
> "(3) Optimal chunk size scales with available context: 32-64 line chunks work best at small budgets, and whole-file retrieval becomes competitive at 16000 tokens."

**ข้อสรุปที่ซื่อสัตย์:** chunk boundary เป็นปัญหาจริงในเชิงแนวคิด แต่ **ผลเชิงตัวเลขอยู่ระดับ 2–4 points เท่านั้น และงานสองชิ้นให้ผลขัดกัน** ถ้า context window ใหญ่พอ (16k+ tokens) การ retrieve ทั้งไฟล์ก็แข่งได้ — นี่เป็นข้อค้นพบที่สำคัญมากสำหรับ senior project เพราะแปลว่า **ไม่ควรใช้เวลา 2 เดือนไปกับการ tune chunking**

Cline อธิบายปัญหานี้เชิงคุณภาพ (blog 2025-05-27):
> "When you chunk code for embeddings, you're literally tearing apart its logic."
เปรียบเทียบกับ "listening to random 10-second clips" ของซิมโฟนี — แต่ **ไม่มีตัวเลขใด ๆ ประกอบในบทความ**

### 1.3 Indexing cost / staleness — ต้นทุนจริงอยู่ตรงนี้

**Cursor, "Securely indexing large codebases" (2026-01-27)** — ตัวเลขจากการทำ index reuse (dedup ข้าม repo/ผู้ใช้ผ่าน Merkle tree + embedding cache ตาม hash ของ chunk):

| Percentile | ก่อนทำ reuse | หลังทำ reuse |
|---|---|---|
| Median | 7.87 วินาที | 525 ms |
| p90 | 2.82 นาที | 1.87 วินาที |
| p99 | **4.03 ชั่วโมง** | **21 วินาที** |

สถาปัตยกรรม: Merkle tree ของ hash ทุกไฟล์/โฟลเดอร์ → sync เฉพาะส่วนที่ hash ต่าง → split เป็น syntactic chunks → embed → cache by chunk hash

**นี่คือหลักฐานตัวเลขที่ตรงที่สุดสำหรับคำถาม "RAG ช้าตรงไหน":** ช้าที่ cold-start indexing (ชั่วโมง ๆ ที่ p99) ไม่ใช่ที่ query

**Staleness — Sourcegraph ถอด embeddings ออกจาก Cody**
`[ยังไม่ verify โดยตรง — sourcegraph.com คืน HTTP 403 ให้ WebFetch; ข้อมูลจาก search snippet ของ blog "Toward infinite context for code" (ก.พ. 2024) และ Cody docs]`
เหตุผลที่ปรากฏใน snippet: embeddings ต้องส่ง source code ไป OpenAI API, ต้องเก็บ/ดูแล/อัปเดต vector, และ *"Searching vector databases for codebases with >100,000 repositories is complex and resource-intensive"* ทำให้ทำ multi-repo context ไม่ได้ มี PR `sourcegraph/cody#2879` (ม.ค. 2024) ชื่อ "Removes remote embeddings from consumer and enterprise"

**ต้นทุนเงินของการ re-embed** `[secondary source, ยังไม่ verify]`
ตัวเลขที่พบใน search: re-embed corpus หนึ่งด้วย `text-embedding-3-small` ($0.02/1M tokens) ≈ $166, `text-embedding-3-large` ($0.13/1M tokens) ≈ $1,079 — ตัวเลขนี้ไม่ระบุขนาด corpus ชัดเจนใน snippet ที่ได้ **อย่าใช้อ้างอิงในรายงาน**

### 1.4 Latency ของ dense retrieval

**Practical Code RAG at Scale (arXiv:2510.20609)** — เปเปอร์ที่วัด latency ตรงที่สุด:
> "(1) For PL-PL, sparse BM25 with word-level splitting is the most effective and practical, significantly outperforming dense alternatives while being an order of magnitude faster."

> "(2) For NL-PL, proprietary dense encoders (Voyager-3 family) consistently beat sparse retrievers, however requiring 100x larger latency."

> "(5) Retrieval latency varies by up to 200x across configurations; BPE-based splitting is needlessly slow, and BM25 + word splitting offers the best quality-latency trade-off."

**อ่านตัวเลขนี้ให้ถูก:** "100x ช้ากว่า" ในบริบทที่ baseline เป็นหลัก ms ก็ยังอาจเป็นหลักร้อย ms ซึ่งจิ๊บจ๊อยเมื่อเทียบกับ LLM generation ที่กินเวลาหลายวินาที — ดูหัวข้อ "ตอบสมมติฐาน" ด้านล่าง

---

## 2. Agentic Search (agent ใช้ grep/ripgrep/glob/LSP สำรวจเอง)

### 2.1 ใครย้ายจริง และย้ายเพราะอะไร

| เครื่องมือ | สถานะ | หลักฐาน |
|---|---|---|
| **Claude Code (Anthropic)** | ย้ายจาก RAG+vector DB → agentic search (glob/grep) | Boris Cherny, Latent Space podcast 2025-05-07; Anthropic engineering blog 2025-09-29 |
| **Cline** | ไม่เคย index เลย โดยเจตนา (filesystem traversal + AST) | cline.bot blog 2025-05-27 |
| **Sourcegraph Cody → Amp** | ถอด embeddings ออก, ใช้ keyword/regex/structural search + SCIP | `[ยังไม่ verify โดยตรง]` |
| **OpenAI Codex CLI** | ripgrep เป็น default ใน core prompt | prompt ของ Codex CLI: *"When searching for text or files, prefer `rg` / `rg --files` since `rg` is faster than grep."* |
| **Aider** | ไม่ใช่ agentic search แต่เป็น structural index (tree-sitter + PageRank repomap) | repomap.py |
| **Cursor** | **ไม่ได้ย้าย** — ยังใช้ embeddings + เพิ่ม n-gram regex index + grep | cursor.com/blog/semsearch (2025-11-06), fast-regex-search (2026-03-23) |
| **Windsurf / Cognition** | ไม่ใช่ embedding — ใช้ LLM reranking แบบขนาน (M-Query/Riptide) แล้วต่อยอดเป็น SWE-grep (RL agentic retrieval) | ZenML LLMOps DB (2024); cognition.com/blog/swe-grep (2025-10-16) |

**คำพูดต้นทาง — Boris Cherny (Latent Space, 2025-05-07):**
> "agentic search just outperformed everything. By a lot."

**และคำเตือนสำคัญที่มาในประโยคถัดไป:**
> "This was just vibes, so internal vibes. There's some internal benchmarks also, but mostly vibes."

**Boris Cherny บน X** `[จาก search result, ไม่ได้ fetch หน้าเดิม]`:
> "Early versions of Claude Code used RAG + a local vector db, but we found pretty quickly that agentic search generally works better. It is also simpler and doesn't have the same issues around security, privacy, staleness, and reliability."

**Anthropic engineering blog, "Effective context engineering for AI agents" (2025-09-29):**
> "Rather than pre-processing all relevant data up front, agents built with the 'just-in-time' approach maintain lightweight identifiers (file paths, stored queries, web links, etc.) and use these references to dynamically load data into context at runtime using tools."

> "Claude Code is an agent that employs this hybrid model: CLAUDE.md files are naively dropped into context up front, while primitives like glob and grep allow it to navigate its environment and retrieve files just-in-time, effectively bypassing the issues of stale indexing and complex syntax trees."

**สังเกตคำว่า "hybrid model" — แม้แต่ Anthropic เองก็ไม่ได้เรียกมันว่า pure agentic search**

### 2.2 มีตัวเลขที่เผยแพร่ไหม — คำตอบตรง ๆ คือ "น้อยมาก"

- Anthropic: **ไม่มีตัวเลขเผยแพร่** มีแต่คำว่า "by a lot" + admit ว่าเป็น vibes
- Cline: **ไม่มีตัวเลขเลย** ทั้งบทความ
- Augment (via Jason Liu / Colin Flaherty, 2025-09-11): *"We explored adding various embedding-based retrieval tools, but found that for SweeBench tasks this was not the bottleneck - grep and find were sufficient."* — **ไม่มีตัวเลขประกอบ** และเปเปอร์เองระบุ scope: repo "relatively small", โค้ด "highly structured with distinctive keywords"
- ตัวเลขวิชาการที่มีจริง: SWE-Explore (ดูตารางข้อ 1.1) — agentic explorers 0.51–0.53 HitRegion vs sparse 0.07–0.12

### 2.3 จุดอ่อนของ agentic search ที่มีตัวเลขรองรับ

**(ก) ช้าและกินเวลา turn** — Cognition, "Introducing SWE-grep" (2025-10-16):
> agents spend ">60% of their first turn just retrieving context"
เป้าหมาย latency ที่ Cognition ตั้ง: **5 วินาที ("the flow window")**
Throughput: SWE-grep-mini >2,800 tok/s (20x เร็วกว่า Haiku 4.5 ที่ 140 tok/s), SWE-grep >650 tok/s (4.5x)
Architecture: RL-trained, "up to 8 parallel tool calls per turn in a maximum of 4 turns"
Ablation: "by increasing the amount of parallelism from 4 to 8 searches per turn, we could reduce the number of turns spent searching from 6 to 4"

**(ข) grep เองก็ช้าบน monorepo ใหญ่** — Cursor, Vicent Marti (2026-03-23):
> "We routinely see `rg` invocations that take more than 15 seconds"
Cursor จึงสร้าง **trigram n-gram inverted index** เพื่อให้ regex search เป็นระดับ ms — คือ **Cursor เพิ่ม index เพื่อแก้ปัญหาความช้าของ grep** ซึ่งเป็นทิศตรงข้ามกับ narrative "agentic search แทน index"

**(ค) agent พลาดไฟล์เป้าหมายบ่อยกว่าที่คิด** — Agent Retrieval Bench (arXiv:2607.24882, 2026-07):
trajectory จริงของ agent "never touch any gold file on **35.2%** of OpenAI samples and **27.2–29.3%** of Codex samples" ทั้งที่มี iterative search

| Agent | Events/sample | Any-gold rate | Median first hit |
|---|---|---|---|
| OpenAI GPT-5.4-mini | 3.2 | 64.8% | step 2 |
| Codex CLI GPT-5.4 | 6.2 | 72.8% | step 3 |
| Codex CLI GPT-5.5 | 6.5 | 70.7% | step 3 |

**(ง) LSP ไม่ถูกใช้จริงเท่าที่คิด** — บทวิเคราะห์ yage.ai (2026-03-27): Claude Code v2.0.74 (ธ.ค. 2025) เพิ่ม LSP รองรับ 11 ภาษา แต่ user feedback ว่า "I haven't come across a case where it has used the LSP yet" บทความเสนอ mental model ว่า **grep = hypothesis generation, LSP = hypothesis verification** และ asymmetric failure: grep ให้ false positive (จัดการได้) ส่วน LSP ให้ false negative/crash (อันตรายกว่าสำหรับ agent)

---

## 3. Hybrid Retrieval (BM25 + dense + reranking) สำหรับโค้ด

### 3.1 หลักฐานที่ชัดที่สุด: hybrid ชนะทั้งสองแบบเดี่ยว

**A Deep Dive into RAG for Code Completion: Experience on WeChat (arXiv:2507.18515, 2025-07-24, ICSME 2025 Industry Track)**
Scope: 1,669 internal repos, 26 open-source LLMs (0.5B–671B)
> "the combination of lexical and semantic retrieval techniques yields optimal results, demonstrating complementary strengths."
ผลที่ดีที่สุด: **BM25 + GTE-Qwen** — Qwen2.5-32B ได้ 63.73/72.25 (CB/ES) `[ตัวเลขนี้มาจาก search snippet; abstract ที่ fetch ได้ยืนยันข้อสรุปเชิงคุณภาพแต่ไม่มีตัวเลข]`

**Agent Retrieval Bench (arXiv:2607.24882, 2026-07)** — 427 samples, 25 repos
Seed-intervention pilot (45 stratified samples, ป้อน context เริ่มต้นต่างกันให้ agent):

| Arm | Final File F1 | Any-gold | First hit | Tool calls |
|---|---|---|---|---|
| No seed | 0.3222 | 51.11% | 3.0 | 3.71 |
| Random non-gold | 0.3437 | 73.33% | 3.0 | 8.49 |
| Lexical (BM25) | 0.3981 | 80.00% | 2.0 | 6.24 |
| Qwen3-8B embedding | 0.3726 | 75.56% | 1.0 | 5.51 |
| **RRF hybrid** | 0.3967 | 80.00% | **1.0** | **5.42** |
| Oracle gold | 0.6337 | 100.00% | 0.0 | 4.18 |

**สองข้อสังเกตสำคัญมากสำหรับ senior project:**
1. **ให้ seed ที่ดีลด tool calls จาก 6.24 → 5.42 และทำให้ first hit เร็วขึ้นจาก step 3 → step 1** คือ retrieval ไม่ได้แทน agentic search แต่ **เร่ง** มัน
2. **ช่องว่างถึง oracle ยังกว้างมาก** (0.3967 → 0.6337, +0.3115 F1) — ยังมี headroom เยอะ เป็นพื้นที่วิจัยที่ดี

RRF fusion ของ Qwen3-8B embedding + RepoMap (287 samples): Recall@20 = 0.7331, MRR ขึ้นจาก 0.2296 (best single) → **0.2713**
> "semantic and structural methods are complementary"

Leaderboard เต็ม (345 samples) แสดงว่า **ไม่มีผู้ชนะเดี่ยว**:

| Model | R@20 | MRR | BCY@8k |
|---|---|---|---|
| Qwen3-4B | 0.6306 | **0.2379** | 0.3409 |
| Qwen3-8B | **0.7029** | 0.2336 | 0.3732 |
| RepoMap (structural) | 0.6333 | 0.2158 | **0.3788** |

> "Qwen3-4B has the best sample-weighted MRR, Qwen3-8B has the best weighted Recall@20, and RepoMap has the best weighted BCY@8k."

### 3.2 Cursor: หลักฐาน A/B test ในโปรดักชันจริงว่า semantic search ช่วย

**Cursor, "Improving agent with semantic search" (2025-11-06, Stefan Heule, Emily Jia, Naman Jain)** — นี่คือหลักฐานเชิงตัวเลขที่แข็งที่สุดฝั่ง "embeddings ยังมีประโยชน์":

- **"achieving on average 12.5% higher accuracy in answering questions (6.5%–23.5% depending on the model)"**
- code retention เพิ่ม 0.3% โดยรวม แต่เพิ่ม **2.6% สำหรับ codebase ใหญ่ (1,000+ ไฟล์)**
- **"We observed a 2.2% increase in dissatisfied follow-up user requests when semantic search was not available"**
- วิธี train: ใช้ agent session traces เป็น training data → *"We provide these traces to an LLM, which ranks what content would have been most helpful at each step"* → train embedding model ให้ similarity score ตรงกับ ranking นั้น
- eval set ภายในชื่อ **Cursor Context Bench**

**และประโยคที่ตอบคำถามของอาจารย์ตรงที่สุด:**
> "Our agent makes heavy use of grep as well as semantic search, and the combination of these two leads to the best outcomes."

### 3.3 Reranking

- **Windsurf/Codeium M-Query (Riptide)** — เลือกทาง "brute-force LLM reranking" แทน embeddings: *"parallel LLM calls to reason over each item in the codebase"* บน GPU หลายร้อยตัว เหตุผลที่ปฏิเสธ embeddings: *"you cannot distill all possible English queries and their relationships to code into a fixed-dimensional vector space"* และสังเกตว่า *"embedding performance has plateaued"* โดยโมเดลต่าง ๆ ลู่เข้าหากันใน ±5%
  - claim ว่าเร็วกว่า/แม่นกว่า embedding search 3x/300% `[claim จาก search snippet + secondary sources; ตัวเลขนี้ ZenML page ที่ fetch ได้ ไม่ยืนยัน — ให้ถือเป็น marketing claim ที่ยังไม่ verify]`
  - ตัวเลขที่ ZenML page ยืนยันได้: *"their computation costs are 1/100th of competitors using APIs, enabling them to provide 100x the compute per user"*
- **CodeRAG-Bench** `[จาก search snippet]`: "Reranking within an optimal range of 200-800 tokens greatly degrades results, showing limited utility of current rerankers" — สัญญาณว่า reranker ที่มีอยู่ยังไม่ค่อยช่วยกับโค้ด

---

## 4. Code-graph / Static-analysis Indexes

### 4.1 Aider RepoMap — tree-sitter + PageRank (baseline ที่ทำได้จริงใน senior project)

วิธีทำงาน (จาก `repomap.py` และการวิเคราะห์ของ community):
1. parse ทุกไฟล์ด้วย **tree-sitter** + language-specific tag queries (`.scm`)
2. ดึง tag 2 ชนิด: `def` (function/class/method definitions) และ `ref` (usages)
3. สร้าง **directed multigraph**: node = ไฟล์, edge = "ไฟล์ A อ้างถึง symbol ที่นิยามในไฟล์ B"
4. รัน **PageRank** เพื่อจัดอันดับว่าไฟล์/symbol ไหนสำคัญสุดในบริบทบทสนทนาปัจจุบัน
5. ใส่เฉพาะ ranked skeleton (signature) ลง context ไม่ใช่ทั้งไฟล์

**จุดแข็ง:** ไม่ต้องมี embedding model, ไม่ต้องมี vector DB, สร้างใหม่เร็ว, deterministic
**หลักฐานว่ามันแข่งได้:** Agent Retrieval Bench ให้ RepoMap ชนะ embedding บน metric BCY@8k (0.3788 vs 0.3732) และเป็นครึ่งหนึ่งของ RRF hybrid ที่ดีที่สุด

### 4.2 RepoGraph (ICLR 2025)

**arXiv:2410.14684** (submitted 2024-10-03, revised 2025-03-18), Ouyang et al., ICLR 2025
เป็น **plug-in module** ที่จัดการ repository-level structure ทดสอบโดยเสียบเข้ากับ 4 framework บน SWE-bench + CrossCodeEval
> "substantially boosts the performance of all systems"

ตัวเลข **32.8% relative improvement บน SWE-bench** `[จาก search snippet เท่านั้น — abstract ที่ fetch ได้ไม่มีตัวเลขนี้ ต้องเปิด PDF ยืนยันก่อนอ้างอิงในรายงาน]`

### 4.3 LARGER (arXiv:2605.16352, 2026-05-19) — ทิศทางล่าสุดที่น่าสนใจที่สุด

ไอเดีย: **"Lexically Anchored Structural Localization"** — เริ่มจาก lexical match (grep) → map ไปเป็น graph anchor → ขยายเพื่อนบ้านแบบ confidence-filtered **ภายใน search loop เดิมของ agent** โดยไม่ต้องมี graph database แยก

> "Existing agents navigate repositories primarily through lexical search, often missing structural relations such as imports, call chains, type hierarchies, and code-test links. Graph-based retrieval can recover such dependencies, but existing approaches often require separate graph tools or traversal stages that fragment the agent's interaction loop."

> "LARGER integrates directly into existing CLI coding agents without requiring external graph databases or specialized graph interfaces."

ผล: **file-level Acc@5 บน LocBench +13.9 points** (tuned hyperparameters) / **+11.8 points** (fixed hyperparameters) เหนือ baseline ที่แข็งที่สุด พร้อม gain ต่อเนื่องบน MuLocBench, SWE-Atlas Test Writing, SWE-Atlas Codebase QA

**นี่คือ +13.9 points ซึ่งใหญ่กว่าตัวเลขของ cAST (+4.3) และ Cursor semantic search (+12.5% relative) มาก — และทำได้โดยไม่ต้องมี vector DB เลย** เป็น sweet spot ที่เหมาะกับ senior project มาก

### 4.4 CodexGraph / CGM / อื่น ๆ

- **CodexGraph** (arXiv:2408.03910, NAACL 2025) — expose code graph ให้ LLM ผ่าน graph database interface (Cypher-like) ทดสอบบน CrossCodeEval, SWE-bench, EvoCodeBench `[metadata จาก search result]`
- **GraphCoder** (ASE 2024) — Code Context Graph สำหรับ repo-level code completion `[จาก search result]`
- **Code Graph Model / CGM** (arXiv:2505.16901) — LLM ที่รวม graph เข้าไปในตัวโมเดล `[จาก search result]`
- **RANGER** (arXiv:2509.25257) — Repository-Level Agent for Graph-Enhanced Retrieval `[จาก search result]`

### 4.5 SCIP / LSIF — precise code intelligence

- **SCIP (SCIP Code Intelligence Protocol)** จาก Sourcegraph มาแทน **LSIF (Language Server Index Format)** — เป็น protocol แบบ language-agnostic สำหรับ index source code เพื่อทำ Go-to-definition / Find-references แบบ compiler-accurate
- indexer ที่ผลิต SCIP ได้: `scip-java` (Java/Scala/Kotlin), `scip-typescript`, `rust-analyzer`, `scip-clang` (C/C++), `scip-ruby`, `scip-python`, `scip-dotnet`, `scip-dart`, `scip-php`
- **จุดขาย:** precise ไม่ใช่ heuristic, version-aware, cross-repository
- **จุดอ่อนสำหรับ senior project:** ต้อง build ได้จริง (ต้อง compile), ต้องรัน indexer ต่อภาษา, ไม่ทนต่อโค้ดที่ compile ไม่ผ่าน — ต่างจาก tree-sitter ที่ parse โค้ดพังได้
- **Sourcegraph Amp** วางตัวเป็น agent ทับบน code graph เดิมที่มี keyword/regex/structural search + SCIP `[ยังไม่ verify โดยตรง — sourcegraph.com คืน 403]`

---

## ตอบสมมติฐาน "RAG ช้า"

### คำตัดสิน: **ถูกบางส่วน แต่ระบุมิติผิด และเป็นเหตุผลที่ผิดสำหรับการตัดสินใจออกแบบ**

ต้องแยกเป็น 4 มิติ:

#### มิติที่ 1 — Query latency: **อาจารย์เข้าใจผิด นี่ไม่ใช่ปัญหา**

ANN vector search บน index ที่ warm แล้วเป็นหลัก **milliseconds** ตัวเลขเชิงเปรียบเทียบที่วัดได้จริง (arXiv:2510.20609) คือ dense ช้ากว่า BM25 ~**100x** ซึ่งฟังดูน่ากลัว **แต่ baseline คือหลัก ms** ดังนั้น 100x ก็ยังอยู่ในระดับสิบ-ร้อย ms

เปรียบเทียบให้เห็นภาพ:
- vector search 1 query: หลัก ms ถึงหลักร้อย ms
- `rg` บน monorepo ใหญ่: **>15 วินาที** (Cursor วัดเอง, 2026-03-23)
- 1 LLM turn ของ agentic search: หลายวินาที และ agent ใช้ **>60% ของ first turn** ไปกับการหา context (Cognition, 2025-10-16)
- Codex CLI เจอ gold file ที่ median step 3 จาก 6.2 tool calls (Agent Retrieval Bench, 2026-07)

**ในทาง latency ต่อ query เดียว embedding RAG มัก *เร็วกว่า* agentic search ด้วยซ้ำ** เพราะ agentic search ต้องจ่ายค่า LLM round-trip หลายรอบ นี่คือเหตุผลตรง ๆ ที่ Cognition ต้องสร้าง SWE-grep บน Cerebras (2,800 tok/s) — เพื่อให้ agentic search ทัน "5-second flow window" ซึ่งเป็นปัญหาที่ vector search ไม่มีตั้งแต่แรก

#### มิติที่ 2 — Indexing time: **อาจารย์ถูก และนี่คือมิติที่ถูกต้องที่สุด**

Cursor p99: **4.03 ชั่วโมง** ต่อ repo (ก่อนทำ index reuse) แม้หลัง optimization เหลือ 21 วินาที ก็เป็นเพราะ dedup ข้ามผู้ใช้ทั้ง platform — ซึ่ง **senior project ทำแบบนั้นไม่ได้** นักศึกษาคนเดียวจะเจอ cold-start จริงเต็ม ๆ

#### มิติที่ 3 — Cost: **ถูกบางส่วน แต่ถูกลงมากแล้ว**

Embedding API ราคาถูกลงมาก (text-embedding-3-small $0.02/1M tokens) สำหรับ repo เดียวในระดับ senior project นี่แทบไม่ใช่ต้นทุน **ต้นทุนที่แท้จริงคือเวลาวิศวกรรมในการดูแล pipeline** (re-embed, versioning, drift) ไม่ใช่ค่า API

#### มิติที่ 4 — Accuracy/Recall: **นี่คือปัญหาตัวจริง และอาจารย์ยังไม่ได้พูดถึง**

หลักฐานที่หนักที่สุดทั้งหมดในเอกสารนี้อยู่ตรงนี้:
- CORE-Bench: NDCG@10 ตกจาก **71.7 → 20.3** เมื่อย้ายจาก code search ไป agentic localization
- SWE-Explore: sparse retriever HitRegion **0.07–0.12** vs agentic 0.51–0.53
- Agent Retrieval Bench: embedding seed ทำ File F1 ได้ 0.3726 ขณะที่ oracle ได้ 0.6337

**แต่ต้องซื่อสัตย์กับหลักฐานฝั่งตรงข้ามด้วย — embedding ที่ทำถูกวิธีชนะ:**
- **NV-EmbedCode (NVIDIA Nemotron-CORTEXA, 2025-04-09)**: file recall **71.95%** vs Agentless (GPT-4o) 65.55% vs NV-EmbedQA-v2 62.81% vs **BM25 40.67%** — คือ embedding ที่ fine-tune บน SWE-bench training set ชนะ agentic baseline และชนะ BM25 ขาดลอย พร้อมกับทำ SWE-bench Verified ได้ **68.2%** ที่ **$3.28/problem**
- **Cursor A/B production**: semantic search เพิ่มความแม่นยำเฉลี่ย **12.5%** (6.5–23.5%) เหนือ grep-only และ code retention +2.6% บน codebase 1,000+ ไฟล์

### สรุปเป็นประโยคเดียวสำหรับอาจารย์

> **"RAG ช้า" ผิดถ้าหมายถึง query latency (มันเร็วกว่า agent loop ด้วยซ้ำ), ถูกถ้าหมายถึง cold-start indexing (p99 หลายชั่วโมงในโปรดักชันจริง), แต่ประเด็นที่สำคัญกว่าทั้งสองอย่างคือ off-the-shelf embedding มี recall ต่ำมากบนงาน "issue นี้ต้องแก้ที่ไหน" (NDCG@10 ตกจาก 71.7 เหลือ 20.3) — ปัญหาไม่ใช่ speed แต่เป็น task mismatch และสิ่งที่แก้มันได้คือ fine-tuning หรือ hybrid ไม่ใช่การทิ้ง retrieval ไปใช้ grep เปล่า ๆ**

### ข้อควรระวังเรื่องคุณภาพหลักฐาน (ต้องบอกอาจารย์)

หลักฐานฝั่ง "agentic search ชนะ" ที่ดังที่สุด (Anthropic, Cline, Augment) **ไม่มีตัวเลขเผยแพร่เลยแม้แต่ชิ้นเดียว** — Boris Cherny พูดเองว่า *"mostly vibes"* ส่วนหลักฐานฝั่ง "retrieval ช่วย" (Cursor A/B, NVIDIA CORTEXA, Agent Retrieval Bench, WeChat ICSME) **มีตัวเลขครบ** อาจารย์ควรระวังว่า narrative "ทุกคนทิ้ง RAG แล้ว" ในบล็อกปี 2026 หลายอันเป็น echo chamber ที่อ้างต่อ ๆ กันจากคำพูด "vibes" คำเดียว และ **ไม่ตรงกับสิ่งที่ Cursor (ผู้เล่นรายใหญ่ที่สุดรายหนึ่ง) ทำจริงและวัดจริง**

---

## ข้อเสนอแนะสำหรับ senior project (8 เดือน, นักศึกษาคนเดียว)

จากหลักฐานข้างต้น ลำดับความคุ้มค่าต่อแรงที่ลง:

1. **สร้าง baseline ให้ครบก่อน**: ripgrep + tree-sitter repomap (Aider-style PageRank) — ไม่ต้องมี GPU/vector DB, มีหลักฐานว่าแข่งกับ embedding ได้ (BCY@8k 0.3788 vs 0.3732)
2. **เพิ่ม BM25** — arXiv:2510.20609 พบว่า BM25+word splitting ให้ quality-latency trade-off ที่ดีที่สุดสำหรับ PL-PL และเร็วกว่า dense หนึ่ง order of magnitude
3. **ถ้ามีเวลา ทำ RRF hybrid** (lexical + dense) — Agent Retrieval Bench แสดง MRR ขึ้นจาก 0.2296 → 0.2713
4. **ทิศทางที่ novel ที่สุดและทำได้จริง: LARGER-style lexically-anchored graph expansion** (+13.9 Acc@5, ไม่ต้องมี graph DB)
5. **อย่าเสียเวลากับ chunking strategy** — cAST ให้แค่ +4.3 Recall@5 และ arXiv:2510.20609 พบว่า line-based chunking เท่ากับ syntax-aware
6. **Metric ที่ควรใช้:** file-level Acc@5 / Recall@20 / MRR + context efficiency + tool-call count (ไม่ใช่แค่ SWE-bench resolve rate ซึ่ง noisy และแพง)

---

## Annotated Bibliography

### A. Peer-reviewed / arXiv papers

1. **CORE-Bench: A Comprehensive Benchmark for Code Retrieval in the Era of Agentic Coding** — Zhang, Zhang, Li, Long, Hu, Xie, Zhang, Zhuang — v1 2026-06-10, v3 2026-08-24
   https://arxiv.org/abs/2606.11864 (HTML: https://arxiv.org/html/2606.11864v2)
   *สำคัญเพราะ:* หลักฐานตัวเลขที่แข็งที่สุดว่า embedding model ที่ดังบน code search ตกฮวบบนงาน agentic จริง (NDCG@10 71.7 → 20.3) — เป็นแกนของคำตอบว่า "ปัญหาคือ accuracy ไม่ใช่ speed"

2. **SWE-Explore: Benchmarking How Coding Agents Explore Repositories** — Zhang, Wang, Liang, Shi, Zeng, Wang, He, Xu, Ye, Cai, Gu — 2026-06-05
   https://arxiv.org/abs/2606.07297 (HTML: https://arxiv.org/html/2606.07297v1)
   *สำคัญเพราะ:* head-to-head จริงระหว่าง agentic explorers กับ classical retrieval บน 848 issues / 10 ภาษา / 203 repos พร้อมตาราง HitRegion, precision, context efficiency

3. **Agent Retrieval Bench: Evaluating Repository Context Retrieval for Coding Agents** — Qin (NUS), Xie (PKU) — 2026-07
   https://arxiv.org/html/2607.24882v1
   *สำคัญเพราะ:* seed-intervention ablation ที่แยก lexical / embedding / RRF hybrid / oracle ออกจากกันชัดเจน และวัด tool-call count — พิสูจน์ว่า retrieval ที่ดี **เร่ง** agentic search ไม่ใช่แทนที่

4. **Practical Code RAG at Scale: Task-Aware Retrieval Design Choices under Compute Budgets** — Galimzyanov, Kolomyttseva, Bogomolov (JetBrains Research) — 2025-10-23
   https://arxiv.org/abs/2510.20609
   *สำคัญเพราะ:* เปเปอร์เดียวที่วัด **latency** ของ retrieval config อย่างเป็นระบบ (200x variance, dense ช้ากว่า sparse 100x) และหักล้างความเชื่อเรื่อง syntax-aware chunking

5. **A Deep Dive into Retrieval-Augmented Generation for Code Completion: Experience on WeChat** — Yang, Peng, Gao, Wang, Huang, Deng — 2025-07-24, ICSME 2025 Industry Track
   https://arxiv.org/abs/2507.18515
   *สำคัญเพราะ:* หลักฐานอุตสาหกรรมจริงบน 1,669 internal repos ว่า lexical + semantic เสริมกัน (BM25 + GTE-Qwen ดีที่สุด)

6. **cAST: Enhancing Code RAG with Structural Chunking via Abstract Syntax Tree** — Zhang, Zhao, Wang, Yang, Wei, Wu — 2025-06-18, Findings of EMNLP 2025
   https://arxiv.org/abs/2506.15655 | https://aclanthology.org/2025.findings-emnlp.430/ | code: https://github.com/yilinjz/astchunk
   *สำคัญเพราะ:* กำหนดขนาดผลกระทบจริงของ chunk-boundary problem ไว้ที่ +4.3 Recall@5 / +2.67 Pass@1 — เล็กกว่าที่ blog ทั่วไปทำให้เชื่อ

7. **LARGER: Lexically Anchored Repository Graph Exploration and Retrieval** — Hu, Su, Zhao, Zhu, Haque — 2026-05-19
   https://arxiv.org/abs/2605.16352
   *สำคัญเพราะ:* gain สูงสุดที่พบในงานวิจัยทั้งหมดนี้ (+13.9 Acc@5 บน LocBench) โดยไม่ต้องมี vector DB หรือ graph DB — เป็น template ที่เหมาะกับ senior project ที่สุด

8. **RepoGraph: Enhancing AI Software Engineering with Repository-level Code Graph** — Ouyang, Yu, Ma, Xiao, Zhang, Jia, Han, Zhang, Yu — 2024-10-03 (rev. 2025-03-18), ICLR 2025
   https://arxiv.org/abs/2410.14684 | code: https://github.com/ozyyshr/RepoGraph
   *สำคัญเพราะ:* พิสูจน์ว่า code graph เป็น plug-in module ที่ยกระดับ agent ได้หลายตัวพร้อมกัน (ตัวเลข 32.8% relative ยังต้อง verify จาก PDF)

9. **Retrieval-Augmented Code Generation: A Survey with Focus on Repository-Level Approaches** — Tao, Li, Qin, Liu — v1 2025-10-06, v3 2026-05-20
   https://arxiv.org/abs/2510.04905
   *สำคัญเพราะ:* survey ล่าสุดที่จัดหมวดตาม retrieval substrate / control regime / evaluation setting — ใช้เป็นโครงของ literature review ในเล่มโปรเจกต์ได้เลย

10. **Nemotron-CORTEXA: Enhancing LLM Agents for Software Engineering Tasks via Improved Localization and Solution Diversity** — NVIDIA ADLR — 2025-04-09
    https://research.nvidia.com/labs/adlr/cortexa/ | paper: https://openreview.net/pdf?id=k6p8UKRdH7 | code: https://github.com/NVIDIA/Nemotron-CORTEXA | model: https://huggingface.co/nvidia/NV-EmbedCode-7b-v1
    *สำคัญเพราะ:* หลักฐานที่แข็งที่สุดว่า **embedding ที่ fine-tune ชนะ agentic baseline** — file recall 71.95% (NV-EmbedCode) vs 65.55% (Agentless/GPT-4o) vs 40.67% (BM25); SWE-bench Verified 68.2% ที่ $3.28/problem

11. **CodexGraph: Bridging LLMs and Code Repositories via Code Graph Databases** — NAACL 2025 `[metadata จาก search result เท่านั้น]`
    https://arxiv.org/html/2408.03910v2
    *สำคัญเพราะ:* ตัวแทนแนวทาง "graph DB + Cypher interface" ซึ่งเป็นขั้วตรงข้ามกับ LARGER — ดีสำหรับเปรียบเทียบในบทที่ 2

12. **Exploration Structure in LLM Agents for Multi-File Change Localization** — 2026-06 `[metadata จาก search result เท่านั้น]`
    https://arxiv.org/abs/2606.11976v1
    *สำคัญเพราะ:* พบว่า "forced multi-agent consultation does not measurably help and raises token cost substantially" — เตือนไม่ให้นักศึกษาไล่ตาม multi-agent โดยไม่จำเป็น

### B. Engineering evidence (บล็อกทางการของบริษัท / primary sources)

13. **Anthropic — "Effective context engineering for AI agents"** — 2025-09-29
    https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
    *สำคัญเพราะ:* คำอธิบายทางการของ just-in-time context loading และยอมรับเองว่า Claude Code เป็น **"hybrid model"** ไม่ใช่ pure agentic search

14. **Latent Space — "Claude Code: Anthropic's Agent in Your Terminal"** (สัมภาษณ์ Boris Cherny) — 2025-05-07
    https://www.latent.space/p/claude-code
    *สำคัญเพราะ:* แหล่งต้นทางของประโยค "agentic search just outperformed everything. By a lot." **พร้อมคำสารภาพว่า "This was just vibes, so internal vibes."** — ต้องอ้างทั้งสองประโยคคู่กันเสมอ

15. **Boris Cherny บน X** `[จาก search result, ไม่ได้ fetch หน้าเดิม]`
    https://x.com/bcherny/status/2017824286489383315
    *สำคัญเพราะ:* ยืนยันว่า Claude Code รุ่นแรกใช้ RAG + local vector DB จริง ก่อนเปลี่ยน เหตุผล: "simpler... security, privacy, staleness, and reliability"

16. **Cursor — "Improving agent with semantic search"** — Heule, Jia, Jain — 2025-11-06
    https://cursor.com/blog/semsearch
    *สำคัญเพราะ:* **หลักฐาน A/B production เพียงชิ้นเดียวที่มีตัวเลขจริง** ว่า semantic search ช่วย (+12.5% เฉลี่ย, 6.5–23.5%) และคำยืนยันว่า grep + semantic ร่วมกันดีที่สุด — เป็น counter-evidence ที่ต้องมีในรายงาน

17. **Cursor — "Securely indexing large codebases"** — 2026-01-27
    https://cursor.com/blog/secure-codebase-indexing
    *สำคัญเพราะ:* ตัวเลข indexing cost จริง (p99 4.03 ชม. → 21 วิ) + สถาปัตยกรรม Merkle tree sync + embedding cache — ตอบมิติ "indexing time" ของสมมติฐานอาจารย์โดยตรง

18. **Cursor — "Fast regex search"** — Vicent Marti — 2026-03-23
    https://cursor.com/blog/fast-regex-search
    *สำคัญเพราะ:* หลักฐานว่า **grep เองก็ช้า** (">15 seconds" บน monorepo) และผู้เล่นรายใหญ่กำลังสร้าง trigram index มา *เพิ่ม* ไม่ใช่ *ลด* index

19. **Cognition — "Introducing SWE-grep and SWE-grep-mini: RL for Multi-Turn, Fast Context Retrieval"** — 2025-10-16
    https://cognition.com/blog/swe-grep
    *สำคัญเพราะ:* ตัวเลข latency ของ agentic search ตรง ๆ (agent ใช้ >60% ของ turn แรกไปกับ retrieval; เป้า 5-second flow window; 2,800 tok/s; parallelism 4→8 ลด turn 6→4) — พิสูจน์ว่า **agentic search ต่างหากที่มีปัญหา latency**

20. **Cline — "Why Cline Doesn't Index Your Codebase (And Why That's a Good Thing)"** — 2025-05-27
    https://cline.bot/blog/why-cline-doesnt-index-your-codebase-and-why-thats-a-good-thing
    *สำคัญเพราะ:* ตัวอย่างชัดเจนของข้อโต้แย้งเชิงคุณภาพ 3 ข้อ (fragmentation / index decay / security) **โดยไม่มีตัวเลขสนับสนุนเลย** — ใช้สอนนักศึกษาเรื่องการประเมินคุณภาพหลักฐาน

21. **Jason Liu / Colin Flaherty — "Why Grep Beat Embeddings in Our SWE-Bench Agent (Lessons from Augment)"** — 2025-09-11
    https://jxnl.co/writing/2025/09/11/why-grep-beat-embeddings-in-our-swe-bench-agent-lessons-from-augment/
    *สำคัญเพราะ:* คำพูดตรงจากอดีต founding engineer ของ Augment: *"for SweeBench tasks this was not the bottleneck - grep and find were sufficient"* **แต่ระบุ scope เองว่า repo เล็กและโค้ดมี distinctive keywords** — ไม่ generalize

22. **ZenML LLMOps Database — Codeium/Windsurf: Advanced Context-Aware Code Generation with Parallel LLM Processing (M-Query / Riptide)** — 2024
    https://www.zenml.io/llmops-database/advanced-context-aware-code-generation-with-custom-infrastructure-and-parallel-llm-processing
    *สำคัญเพราะ:* ทางเลือกที่สาม — LLM reranking แบบขนานแทน embedding เหตุผลที่ปฏิเสธ embedding: *"you cannot distill all possible English queries and their relationships to code into a fixed-dimensional vector space"* และ *"embedding performance has plateaued"* (±5%)

23. **yage.ai — "Why Coding Agents Still Use grep as Their Search Backbone"** — 2026-03-27
    https://yage.ai/share/why-coding-agents-still-use-grep-en-20260327.html
    *สำคัญเพราะ:* ตารางเปรียบเทียบ tool stack ของ Claude Code / Codex CLI / OpenCode / Cursor / Aider / Continue ที่ละเอียดที่สุดที่พบ + mental model "grep = hypothesis generation, LSP = hypothesis verification" + LSP failure taxonomy (Princeton Lanser-CLI, arXiv:2510.22907)

24. **Pragmatic Engineer — "Building Claude Code with Boris Cherny"**
    https://newsletter.pragmaticengineer.com/p/building-claude-code-with-boris-cherny
    *สำคัญเพราะ:* แหล่งอ้างอิงของคำพูด "Claude Code's agentic search is really just glob and grep, and it outperformed RAG" `[อ้างผ่าน yage.ai; ยังไม่ fetch โดยตรง]`

25. **Sourcegraph — SCIP Code Intelligence Protocol** `[หน้าหลักคืน 403 ต่อ WebFetch; metadata จาก search result]`
    https://scip-code.org/ | https://sourcegraph.com/blog/announcing-scip | https://docs.sourcegraph.com/code_intelligence/explanations/precise_code_intelligence
    *สำคัญเพราะ:* มาตรฐาน precise code intelligence ที่มาแทน LSIF — เป็นตัวเลือกสำหรับ code-graph index ที่ compiler-accurate แต่ต้อง build ได้

26. **Sourcegraph — "Toward infinite context for code"** (ก.พ. 2024) `[HTTP 403 — verify ไม่ได้โดยตรง; ข้อมูลจาก search snippet เท่านั้น]`
    https://sourcegraph.com/blog/towards-infinite-context-for-code
    *สำคัญเพราะ:* เป็นเหตุการณ์ที่ vendor รายใหญ่ **ถอด embeddings ออกจากโปรดักชัน** อ้างเหตุผลเรื่อง scaling (>100,000 repos) และภาระ ops — **ต้องหาทางเข้าถึงหน้านี้ก่อนอ้างในเล่ม**

27. **Aider repomap (tree-sitter + PageRank)** — source: https://github.com/Aider-AI/aider (`aider/repomap.py`)
    คำอธิบายภายนอก: https://anishgandhi.com/aider-pagerank-codebase-ranking/ `[secondary]`
    *สำคัญเพราะ:* implementation อ้างอิงที่นักศึกษาลอกโครงได้ทันที และเป็น baseline ที่ Agent Retrieval Bench ใช้จริง (แถว "RepoMap")

### C. Secondary / synthesis (ใช้เป็น map ไม่ใช่หลักฐาน)

28. **"Agentic, Semantic, or Both? Notes from the Code Search Debate"** — 2026-05-18
    https://wowelec.wordpress.com/2026/05/18/agentic-semantic-or-both-notes-from-the-code-search-debate/
    *สำคัญเพราะ:* ตารางสรุปจุดยืนของแต่ละเจ้า (Claude Code = navigate, Cursor = index-first, Cody = สลับจาก embeddings ไป BM25, Aider = tree-sitter, Devin = index) — ใช้เป็น starting map แต่ **ต้องตามไป primary source ทุกข้อ**

29. **"AI Agents Don't Need Vector Search Anymore: Inside the Agentic Search Stack Replacing RAG in 2026"**
    https://buzzgrewal.medium.com/ai-agents-dont-need-vector-search-anymore-inside-the-agentic-search-stack-replacing-rag-in-2026-58efcabe4f6f
    *สำคัญเพราะ:* ตัวอย่างของ narrative "ทุกคนทิ้ง RAG แล้ว" ที่ **ขัดกับหลักฐานของ Cursor โดยตรง** — ใช้เป็นกรณีศึกษาว่าทำไมต้องตรวจ primary source

30. **Awesome-Repo-Level-Code-Generation (reading list)**
    https://github.com/YerbaPage/Awesome-Repo-Level-Code-Generation
    *สำคัญเพราะ:* รายการเปเปอร์ repo-level code generation / issue resolution ที่ maintain อยู่ ใช้ขยาย literature review ต่อได้

---

## รายการที่ยัง verify ไม่ได้ (ต้องตามต่อก่อนใส่ในเล่มโปรเจกต์)

| ข้ออ้าง | สถานะ |
|---|---|
| Sourcegraph blog "Toward infinite context for code" — เหตุผลถอด embeddings | HTTP 403, มีแต่ search snippet |
| Sourcegraph Cody docs — deprecation notice ของ embeddings | HTTP 403 |
| RepoGraph "32.8% relative improvement on SWE-bench" | abstract ที่ fetch ได้ไม่มีตัวเลข ต้องเปิด PDF |
| Riptide "300% / 3x better accuracy than embedding search" | marketing claim, ZenML page ที่ fetch ได้ไม่ยืนยัน |
| WeChat paper "BM25+GTE-Qwen → 63.73/72.25 CB/ES" | จาก search snippet, abstract ยืนยันเชิงคุณภาพเท่านั้น |
| ต้นทุน re-embed $166 / $1,079 | secondary blog, ไม่ระบุขนาด corpus |
| Sourcegraph Amp — สถาปัตยกรรม retrieval ปัจจุบัน | หน้า blog คืน 403 |
| CodeRAG-Bench "reranking 200-800 tokens degrades results" | จาก search snippet, ยังไม่ fetch เปเปอร์ |
