# 03 — Benchmark และวิธีวัดผลสำหรับ repo-level / cross-repo code QA

Type: research
Status: resolved

## Question

โครงงานนี้จะ **วัดผลอย่างไรให้น่าเชื่อถือพอสำหรับ senior project**?

ต้องสำรวจ:
- benchmark ที่มีอยู่: SWE-bench (+ Verified/Multimodal), RepoBench, CrossCodeEval, LongCodeArena, RepoQA, CoIR, CodeSearchNet, DevQualityEval, Aider polyglot ฯลฯ — อันไหนแตะ repo-level หรือ cross-file บ้าง
- **มี benchmark ที่เป็น multi-repo จริงๆ ไหม** หรือยังเป็นช่องว่าง (ถ้าเป็นช่องว่าง นั่นอาจเป็น contribution ที่ดีกว่าตัว routing เอง)
- วิธีสร้าง ground-truth Q&A จาก codebase แบบกึ่งอัตโนมัติ (ใช้ commit/PR/issue เป็น oracle)
- metric ที่ใช้กัน: recall@k, context precision, answer correctness, token cost, latency
- **ความเสี่ยงที่นักศึกษาระบุเอง**: corpus สังเคราะห์ (sock-shop, TeaStore) ถือเป็นหลักฐานที่ยอมรับได้ไหมในงานตีพิมพ์ — หาตัวอย่างงานที่ใช้ระบบ demo แบบนี้เป็น subject แล้วผ่าน review

## Answer

รายงานเต็มที่ `research/03-benchmarks-evaluation.md` (~45 รายการอ้างอิง)

### สนามนี้เต็มเร็วมากในปี 2024–2026

benchmark ที่แข่งกันตรงๆ และนักศึกษา **ต้องอ้างอิงให้ครบ**: CoReQA (176 repos), CodeRepoQA (585,687 entries/30 repos), SWE-QA (576 pairs/12 repos, ACL 2026 Findings), StackRepoQA (1,318 Q/134 Java projects, มี.ค. 2026), SWE-QA-Pro (มี.ค. 2026)

### ตอบคำถามหลัก: มี multi-repo benchmark ไหม

**ไม่มีในฝั่ง peer-reviewed — แต่ต้องพูดให้ระวัง** เพราะ **CodeScaleBench** ของ Sourcegraph (2026, Apache 2.0, ยังไม่ผ่าน peer review) ระบุชัดว่ามี **136 multi-repo tasks และ 47 cross-repository navigation tasks ข้าม 40+ repo**

benchmark ฝั่งวิชาการทุกตัวเป็น "หลาย repo เป็นแหล่งแยกกัน" หรือ "cross-file ภายใน repo เดียว" — SWE-QA จำกัดข้ออ้าง novelty ของตัวเองไว้ที่ *"within a repository"* **ช่องว่างมีจริง แต่ต้องกรอบให้แม่น ห้ามอ้างเกิน**

**หลักฐานที่แข็งที่สุดว่าช่องว่างมีจริง**: Rafiei Oskooei et al. (LLM4Code @ ICSE 2026) *ต้อง* ใช้ระบบอุตสาหกรรมแบบปิด (DNext) เพราะไม่มี public multi-repo corpus ให้ใช้

### วิธีสร้าง ground truth — มีระเบียบวิธีตีพิมพ์รองรับเพียบ

test-execution oracle (SWE-bench F2P/P2P), จับคู่ issue→PR diff, ใช้คำตอบสุดท้ายของ maintainer เป็น gold (CodeRepoQA), taxonomy-seed + tree-sitter (pipeline ของ SWE-QA: 77,100 issues → 127,415 questions → 1,000 hand-coded → templates), nugget-based relevance (FreshStack)

**ของดีที่สุดสำหรับออกแบบการวัดผล**: Agent Retrieval Bench (arXiv 2607.24882) มี **32 counterfactual wrong-repository controls** และ **50 no-gold examples** เอามาใช้ซ้ำได้ตรงๆ เพื่อทำให้ routing วัดได้ พร้อม base-commit freezing และ query sanitization กัน leakage

### Metric สามชั้น

- routing: Repo-Recall@1, false-routing rate, abstention rate
- retrieval: Recall@k, MRR, nDCG@10
- answer: LLM-as-judge 5 มิติ (ตาม SWE-QA); StackRepoQA รายงาน Cohen's κ 0.78–0.87 เป็นเกณฑ์ความน่าเชื่อถือ
- บวก token/USD/latency พร้อม cost–accuracy Pareto plot (มี precedent จาก DevQualityEval)

### ตอบความเสี่ยงเรื่อง corpus สังเคราะห์ของนักศึกษา

**ยอมรับได้ มีงานตีพิมพ์รองรับชัด**: Train Ticket อยู่ใน IEEE TSE (10.1109/TSE.2018.2887384, 22 replicated faults) และ ICSE 2018; TeaStore ใน IEEE MASCOTS 2018; Sock Shop ใน ECASE@ICSE 2017; Fischer et al. AST 2024 (10.1145/3644032.3644445) เป็น systematic mapping study ยืนยันการใช้ทั้งชุมชน

**แต่ห้ามใช้อย่างเดียว** — ควรเพิ่ม GitHub organization แบบ open-source ที่เป็น polyrepo จริงเป็น corpus ชั้นที่สอง และ **ต้องแก้ถ้อยคำใน proposal**: ประโยค *"real access is unavailable"* **ไม่จริง** — ไม่จริงสำหรับ open-source polyrepo org; ที่เข้าไม่ถึงคือ corpus ระดับองค์กรแบบปิดเท่านั้น ซึ่งไม่จำเป็นต้องใช้

### ตรวจสอบซ้ำแล้ว (2026-09-01)

**CodeScaleBench — มีจริงและใหญ่กว่าที่รายงานไว้** เป็น living benchmark **370 tasks** แบ่งสองส่วน:
- CodeScaleBench-SDLC — 150 tasks, patch-based verifier, มี `ground_truth.json` สำหรับ context-retrieval metrics
- **CodeScaleBench-Org — 220 tasks** ที่ *ต้องใช้ organization-level codebase navigation* ใช้ artifact verifier (agent ผลิต `answer.json` เทียบกับ curator agent)

Sourcegraph รายงานเองว่า **ช่องว่างประสิทธิภาพบน multi-repo tasks กว้างกว่า single-repo** — ยิ่งโค้ดกระจายข้ามขอบเขตองค์กร โอกาสที่ agent จะหาเจอครบยิ่งต่ำ **นี่คือหลักฐานสนับสนุนปัญหาของนักศึกษา แต่ก็เป็นคู่แข่งของ contribution แบบ benchmark ด้วย**

**MULocBench — ตรวจแล้ว ไม่ปิดช่องว่าง** เป็น *multi-type* ไม่ใช่ *multi-repo*: 1,100 issues จาก 46 GitHub Python projects แยกกัน จุดขายคือครอบคลุม non-code files (commits, comments, config, docs) ไม่ใช่คำถามข้าม repo → **ข้อสรุปว่าไม่มี multi-repo QA benchmark ฝั่งวิชาการ ยังยืนอยู่**

**สรุปที่แม่นที่สุดสำหรับเขียนลง proposal**: ไม่มี benchmark **peer-reviewed** ที่คำถาม/งานหนึ่งข้อบังคับให้ใช้ ≥2 repository สำหรับงาน **QA** โดยเฉพาะ — CodeScaleBench-Org แตะพื้นที่นี้แล้วแต่เป็นงาน SDLC/agent-task ของบริษัท ยังไม่ผ่าน peer review และไม่ใช่ QA benchmark
