# 01 — Landscape: embedding RAG vs agentic grep vs hybrid vs code graph สำหรับ LLM coding

Type: research
Status: resolved

## Question

สำหรับการให้ LLM ตอบคำถาม/เขียนโค้ดใน codebase ขนาดใหญ่ ปัจจุบันมีแนวทาง retrieval อะไรบ้าง และ **หลักฐานบอกว่าอันไหนชนะเมื่อไหร่**?

ต้องได้คำตอบเฉพาะเจาะจงเรื่อง:
- embedding-based RAG (chunk + vector search) — จุดอ่อนจริงคืออะไร: latency, indexing cost, recall บนโค้ด, ปัญหา chunk boundary, index staleness
- agentic search (ให้ agent ใช้ grep/ripgrep/glob/LSP เดินสำรวจเอง) — ทำไมเครื่องมือเชิงพาณิชย์หลายตัวถึงย้ายมาทางนี้ มีหลักฐานเชิงตัวเลขไหม
- hybrid (BM25 + dense + rerank)
- code graph / static analysis index (call graph, import graph, SCIP/LSIF, tree-sitter)

**สมมติฐานที่ต้องตรวจ**: อาจารย์เชื่อว่า "RAG น่าจะช้า" — จริงไหม ช้าในมิติไหน มีตัวเลขไหม หรือจริงๆ ปัญหาหลักคือ *ความแม่น* ไม่ใช่ *ความเร็ว*

ต้องการทั้ง peer-reviewed papers และหลักฐานเชิงวิศวกรรมจากของจริง (บล็อกวิศวกรรม, repo)

## Answer

รายงานเต็มที่ `research/01-retrieval-landscape.md`

### ตอบสมมติฐาน "RAG ช้า" — ถูกครึ่งเดียว และชี้ผิดมิติ

- **Query latency ของ dense retrieval อยู่ระดับมิลลิวินาที และ *เร็วกว่า* agentic search** ซึ่งต้องเสีย LLM round-trip หลายรอบต่อการค้นหนึ่งครั้ง
- **ที่ "ช้า" จริงคือ cold-start indexing** — Cursor เปิดตัวเลขจริง: p99 indexing 4.03 ชั่วโมง ก่อนทำ index reuse, เหลือ 21 วินาทีหลังทำ (เป็น dedup ระดับแพลตฟอร์มที่นักศึกษาคนเดียวทำตามไม่ได้)
- **ปัญหาจริงคือ recall ไม่ใช่ speed** — CORE-Bench: Qwen3-Embedding-8B ตกจาก NDCG@10 71.7 บน code search แบบดั้งเดิม เหลือ 20.3 บนงาน issue-to-edit localization

### agentic search ชนะจริงไหม

- ชนะ sparse retrieval ชัดเจน: SWE-Explore (848 issues/203 repos) agentic explorer HitRegion 0.51–0.53 vs BM25/TF-IDF 0.07–0.12
- **แต่หลักฐานฝั่งอุตสาหกรรมที่คนอ้างกันบ่อยไม่มีตัวเลข** — คำพูด "agentic search outperformed everything, by a lot" ตามด้วยประโยคว่า *"This was just vibes, so internal vibes"* ทันที; โพสต์ของ Cline และ Augment ไม่มี metric เลย
- **หลักฐานฝั่งตรงข้ามมีแหล่งแข็งกว่า** — A/B บน production ของ Cursor (พ.ย. 2025): semantic search ให้ accuracy +12.5% เฉลี่ย (ช่วง 6.5–23.5%) เหนือ grep อย่างเดียว และ +2.6% code retention บน repo 1000+ ไฟล์ สรุปเองว่า grep + semantic รวมกันดีที่สุด
- embedding ที่ fine-tune แล้วชนะ agentic baseline: NV-EmbedCode file recall 71.95% vs Agentless/GPT-4o 65.55% vs BM25 40.67%
- **agentic search มีปัญหา latency ของตัวเอง** — agent ใช้เวลา >60% ของเทิร์นแรกไปกับการ retrieve; ripgrep เองใช้เวลาเกิน 15 วินาทีบน monorepo ใหญ่ของ Cursor ซึ่งเป็นเหตุผลที่ Cursor ไปสร้าง trigram index — คือ *เพิ่ม* index ไม่ใช่ *เอาออก*

### สรุปเชิงแนะแนว

**Hybrid คือคำตอบเชิงประจักษ์** ไม่มีวิธีเดียวที่ชนะทุก metric — retrieval seed ลด tool call ของ agent จาก 6.24 เหลือ 5.42 และเลื่อน gold hit แรกจาก step 3 มาเป็น step 1; RRF ของ embedding+RepoMap ดัน MRR 0.2296→0.2713

**ทิศทางที่น่าสนใจที่สุดที่เจอ: LARGER** — lexically-anchored graph expansion ที่ทำ*ภายใน* loop ค้นหาเดิมของ agent ได้ +13.9 Acc@5 บน LocBench **โดยไม่ต้องมี vector DB และไม่ต้องมี graph DB** เหมาะกับข้อจำกัดของนักศึกษาคนเดียวมาก

**สิ่งที่ควรบอกนักศึกษาว่าอย่าไปเสียเวลา**: การจูน chunking — cAST ได้แค่ +4.3 Recall@5 และ JetBrains พบว่า line-based chunking เทียบเท่า syntax-aware
