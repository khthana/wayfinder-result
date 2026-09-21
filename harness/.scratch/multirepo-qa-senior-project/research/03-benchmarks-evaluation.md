# 03 — Benchmarks & Evaluation สำหรับ Multi-Repo Codebase Q&A Agent

**วันที่ค้นคว้า:** 2026-09-01
**ผู้อ่านเป้าหมาย:** อาจารย์ที่ปรึกษา (เพื่อแปลงเป็นคำแนะนำให้นักศึกษา)
**สถานะหลักฐาน:** ทุก URL ในเอกสารนี้มาจากผลการค้นหาจริงหรือหน้าที่ fetch จริง รายการที่ยังยืนยันไม่ครบถูกทำเครื่องหมาย `[ยังไม่ยืนยัน]`

---

## 1. บทสรุปผู้บริหาร (Executive Summary)

**ข้อสรุป 6 ข้อ:**

1. **สนาม repo-level code QA เพิ่งจะแน่นขึ้นมากในช่วง 2024–2026** — มี benchmark ที่ตรงกับงานนักศึกษาแล้วอย่างน้อย 5 ตัว: CoReQA (ม.ค. 2025), SWE-QA (ก.ย. 2025, ACL Findings), CodeRepoQA (ธ.ค. 2024), StackRepoQA (มี.ค. 2026) และ SWE-QA-Pro (มี.ค. 2026) นักศึกษา **ต้องอ้างถึงชุดนี้** ในบทที่ 2 ไม่งั้นจะถูกมองว่าไม่รู้ state of the art

2. **คำตอบตรงต่อคำถามหลัก: benchmark ที่เป็น MULTI-REPOSITORY จริง ๆ "เกือบ" ไม่มีในวงวิชาการ แต่ **มีแล้วในฝั่งอุตสาหกรรม** — `CodeScaleBench` ของ Sourcegraph (2026) มี multi-repository tasks 136 ข้อ และ cross-repository navigation 47 ข้อ ซึ่งเป็นตัวแรกที่พบว่าประกาศตัวเลข cross-repo อย่างชัดเจน แต่ **ไม่ผ่าน peer review** และเป็นงาน SWE tasks ไม่ใช่ Q&A ล้วน → ช่องว่างสำหรับ **multi-repo code *QA* benchmark ที่ผ่าน peer review ยังเปิดอยู่จริง** (รายละเอียดหัวข้อ 3)

3. **มีงานที่ใกล้เคียงโปรเจกต์นักศึกษามากจนต้องอ่านให้ครบ** คือ Rafiei Oskooei et al. (LLM4Code @ ICSE 2026) ทำ "two-phase search: routing bug report ไปยัง repository ที่ถูกต้องก่อน แล้วค่อย localize ภายใน" บนระบบ industrial 46 repos / 1.1M LOC → **นี่คือ "routing algorithm" ที่นักศึกษาเสนอ ในรูปแบบที่ตีพิมพ์แล้ว** ดังนั้น routing เพียงอย่างเดียวไม่ใช่ novelty ที่แข็ง แต่ **benchmark + วิธีวัด** ยังเป็นได้

4. **วิธีสร้าง ground truth แบบกึ่งอัตโนมัติมี published methodology ให้ลอกได้หลายแบบ** — issue→PR pairing (SWE-bench family), maintainer's last reply เป็นเฉลย (CodeRepoQA), seed-question taxonomy + tree-sitter instantiation (SWE-QA), nugget-based (FreshStack), และ workflow-signal mining จาก merged PR / review comment / failing test (Agent Retrieval Bench)

5. **มีเทคนิคประเมิน routing โดยเฉพาะที่ควรลอก** — Agent Retrieval Bench (ก.ค. 2026) ใส่ **"counterfactual wrong-repository controls" 32 ข้อ** และ **"no-gold examples" 50 ข้อ** เพื่อวัดว่า retriever รู้จักปฏิเสธหรือไม่ นี่คือของขวัญสำหรับโปรเจกต์นักศึกษาโดยตรง

6. **เรื่อง synthetic corpus: ยอมรับได้ และมีหลักฐานรองรับหนักแน่น** — TeaStore, Sock Shop, Train Ticket, Online Boutique ถูกใช้เป็น subject system ในงานที่ผ่าน peer review ระดับ IEEE TSE / ICSE / MASCOTS / AST มาต่อเนื่องตั้งแต่ 2017 มี systematic mapping study (AST 2024) ยืนยันการใช้งานอย่างเป็นระบบ **แต่** ต้องเขียน threats to validity ให้ชัด และ **ควรผสม repo จริงเข้าไปด้วย** (รายละเอียดหัวข้อ 6)

---

## 2. ตารางเปรียบเทียบ Benchmark

คอลัมน์ "Multi-repo?" แยก 3 ระดับ:
- **หลาย repo เป็นแหล่งข้อมูล** = benchmark ครอบคลุมหลาย repo แต่แต่ละคำถามอยู่ใน repo เดียว
- **Cross-file** = คำถามข้ามไฟล์ภายใน repo เดียว
- **Cross-repo จริง** = คำตอบเดียวต้องใช้ข้อมูลจาก >1 repository

| Benchmark | ปี | วัดอะไร | ขนาด | ระดับ | Multi-repo? | ผ่าน peer review? |
|---|---|---|---|---|---|---|
| **CodeSearchNet** | 2019 | semantic code search (NL→function) | corpus ~6M functions, 6 ภาษา; eval 99 queries + ~4k expert annotations | function-level | หลาย repo เป็นแหล่งข้อมูล | arXiv + widely cited; ประกาศปิด challenge แล้ว |
| **RepoBench** | 2023 | repo-level auto-completion: R (retrieval), C (completion), P (pipeline) | Python + Java (ตัวเลขแยก task ไม่ปรากฏในหน้า abstract) `[ยังไม่ยืนยันขนาด]` | cross-file | หลาย repo เป็นแหล่งข้อมูล | OpenReview (ICLR track) |
| **CrossCodeEval** | 2023 | code completion ที่ต้องพึ่ง cross-file dependency | ~10K examples, 4 ภาษา (Python, Java, TypeScript, C#) | cross-file | หลาย repo เป็นแหล่งข้อมูล | NeurIPS D&B `[ยังไม่ยืนยัน venue]` |
| **SWE-bench** | 2023 | แก้ issue จริงให้ test ผ่าน | 2,294 instances / 12 Python repos `[ตัวเลข 2,294 มาจากความรู้ทั่วไป — หน้า swebench.com ที่ fetch ไม่ระบุ; ยังไม่ยืนยัน]` | repo-level | หลาย repo เป็นแหล่งข้อมูล | ICLR 2024 |
| **SWE-bench Verified** | 2024-08-13 | subset ที่วิศวกรยืนยันว่าแก้ได้จริง | **500** problems | repo-level | หลาย repo | OpenAI release (ไม่ใช่ paper) |
| **SWE-bench Multimodal** | 2024-10 | แก้บั๊กใน visual JS software (มี screenshot/UI) | **617 instances / 17 JavaScript repos** | repo-level | หลาย repo | arXiv 2410.03859 (ICLR 2025 `[ยังไม่ยืนยัน]`) |
| **SWE-bench-Live** | 2025-05 | เหมือน SWE-bench แต่ auto-curate ต่อเนื่อง กัน contamination | **1,319 tasks / 93 repos** (issue หลังปี 2024) | repo-level | หลาย repo | arXiv 2505.23419 |
| **SWE-bench Pro** | 2025 | tasks ยากกว่า มี private/held-out set | **1,865 tasks / 41 repos** = public 731 + private 276 + held-out 858 | repo-level | หลาย repo | Scale AI (industry) |
| **Long Code Arena** | 2024-06 | 6 งานที่ต้องใช้ project-wide context | lib-based codegen 150/62 libs; CI repair 77; project-level completion 934; commit-msg 163/34 repos; **bug localization 150 (50/ภาษา, Py/Java/Kotlin)**; module summarization 216/43 repos | project-wide | หลาย repo | arXiv 2406.11612 (JetBrains Research + TU Delft) |
| **RepoQA** | 2024-06 | Searching Needle Function — หา function จากคำบรรยาย NL ใน long context | **500 tests / 50 repos / 5 ภาษา** (10 needle/repo) | long-context ใน repo เดียว | หลาย repo เป็นแหล่งข้อมูล | arXiv 2406.06025 (EvalPlus team) |
| **CodeRepoQA** | 2024-12 | multi-turn SE QA จาก issue threads | **585,687 entries**, เฉลี่ย 6.62 turns, 30 repos, 5 ภาษา (Python/Java/TS/JS/Go) | repo-level (ข้อความ ไม่ใช่โค้ดล้วน) | หลาย repo | arXiv 2412.14764 |
| **CoReQA** | 2025-01 | code repository QA จาก GitHub issues+comments | **176 repos / 4 ภาษา** (จำนวนคำถามไม่ระบุใน abstract) | repo-level | หลาย repo | arXiv 2501.03447 |
| **CoIR** | 2024→2025 | code information retrieval 4 paradigm (text↔code, code↔code, hybrid) | **10 datasets / 8 tasks / 7 domains** | snippet + mixed | ไม่ใช่ repo-level | **ACL 2025 Main** |
| **CodeRAG-Bench** | 2024-06 | RAG ช่วย code generation ได้แค่ไหน; วัดทั้ง retrieval และ end-to-end | หลาย source: competition, tutorial, library docs, StackOverflow, GitHub repos | mixed + repo-level | หลาย repo | **NAACL 2025 Findings** |
| **SWE-QA** | 2025-09 | repo-level code QA: cross-file reasoning, multi-hop dependency | **576 QA pairs / 12 Python repos** (48/repo), taxonomy What 13.3% / Why 23.1% / Where 28.4% / How 35.2% | repo-level, cross-file | หลาย repo | **ACL 2026 Findings** |
| **StackRepoQA** | 2026-03 | repo-level QA จากคำถามนักพัฒนาจริง | **1,318 questions / 134 Java projects** | repo-level | หลาย repo (ผู้เขียนระบุว่าเป็น "multi-project") | arXiv 2603.26567 (Virginia Tech) `[venue ยังไม่ยืนยัน]` |
| **SWE-QA-Pro** | 2026-03 | repo-level code understanding บน long-tail repos + executable env | ขนาดไม่ปรากฏใน abstract `[ยังไม่ยืนยัน]`; ใช้ issue-driven clustering + difficulty calibration | repo-level | หลาย repo | arXiv 2603.16124 |
| **Agent Retrieval Bench** | 2026-07 | repository context retrieval สำหรับ coding agent, 4 tasks | **427 samples / 25 repos**: positive 345 (code2test 106, comment2context 80, trace2code 101, edit2ripple 58) + **no-gold 50** + **wrong-repository controls 32**; corpus 308 base-commit snapshots ~392k files, 6 ภาษา | repo-level retrieval | หลาย repo + **มี wrong-repo control** | arXiv 2607.24882 |
| **CodeScaleBench** (Sourcegraph) | 2026 | coding agent บน codebase ใหญ่ + งานข้าม repo | canonical `csb-v2-dual264` = 264 tasks; validated set 275 = **single-repo 186 / dual-repo 28 / multi-repo 61**; ชุด v1 371 tasks; รวม **multi-repository tasks 136 ข้าม 28 repo sets, cross-repository navigation 47**; 40+ repos, 9 ภาษา, repo ตั้งแต่ <100MB ถึง >5GB | organization-level | ✅ **Cross-repo จริง** | ❌ industry blog + GitHub (Apache 2.0) |
| **DevQualityEval** | 2024– | คุณภาพ code generation หลายมิติ (unit test gen, compile, repair) + วัด cost/chattiness/reliability | ชุด task หลายภาษา + private set กัน contamination `[ตัวเลขยังไม่ยืนยัน]` | file/function-level | ไม่ใช่ | ❌ industry (Symflower), open source |
| **Aider Polyglot** | 2024– | แก้โค้ดข้ามภาษาให้ hidden test ผ่าน | **225 Exercism exercises**, 6 ภาษา (C++, Go, Java, JS, Python, Rust) | file-level | ไม่ใช่ | ❌ industry |
| **Terminal-Bench (2.0)** | 2025–2026 | agent ทำงานจริงใน shell (SWE, ML, security, data science) | **89 tasks** ใน pool ของ 2.0 | environment-level | ไม่ใช่ | ❌ community/industry |
| **FreshStack** | 2025 | retrieval บน technical documents จากคำถาม SO จริง + GitHub repos, nugget-based | หลาย niche domains `[ตัวเลขยังไม่ยืนยัน]` | document retrieval | หลาย repo | arXiv 2504.13128 (Thakur, Lin et al.) |
| **RCAEval** | 2024-12 | root cause analysis บน microservice telemetry | **735 failure cases / 3 microservice systems** | ระบบรัน ไม่ใช่โค้ด | หลาย system | arXiv 2412.17015 `[venue ยังไม่ยืนยัน]` |

---

## 3. คำตัดสินตรง: "มี multi-repository code QA benchmark หรือไม่?"

### คำตอบ: **ยังไม่มีในวงวิชาการที่ผ่าน peer review — แต่ต้องระวังว่ามี 2 ตัวที่เข้าใกล้มาก และตัวหนึ่งอาจถือว่ามีแล้วในฝั่งอุตสาหกรรม**

**ต้องแยกความหมายให้ชัดก่อน** เพราะเปเปอร์จำนวนมากใช้คำว่า "multi-repository" หลวม ๆ:

| ความหมาย | มีไหม | ตัวอย่าง |
|---|---|---|
| (A) benchmark ที่ **ดึงข้อมูลจากหลาย repo** | ✅ มีเพียบ | RepoQA (50), CoReQA (176), StackRepoQA (134), SWE-bench-Live (93) |
| (B) คำถามที่ต้อง **reasoning ข้ามไฟล์ภายใน repo เดียว** | ✅ มี | SWE-QA, CrossCodeEval, Long Code Arena |
| (C) คำถามเดียวที่ **คำตอบกระจายอยู่ใน >1 repository** | ⚠️ **หายาก** | CodeScaleBench (industry) เท่านั้นที่ประกาศตัวเลขชัด |
| (D) **task ของการเลือก repo ที่ถูกต้อง (routing) พร้อม negative control** | ⚠️ เพิ่งมี | Agent Retrieval Bench (wrong-repository controls 32 ข้อ) — แต่เป็น side-control ไม่ใช่ task หลัก |

### หลักฐานสนับสนุนว่าช่องว่างมีจริง

- **SWE-QA (ACL 2026 Findings)** ระบุตรง ๆ ว่า *"none of these existing benchmarks require models to understand code modules as interconnected architectural components within a repository"* — สังเกตว่าแม้แต่ตัวที่ใหม่ที่สุดก็ยังพูดถึงขอบเขต **"within a repository"** เท่านั้น
- **StackRepoQA (มี.ค. 2026)** วางตัวเองว่าแก้ปัญหา "single-project evaluation" ไม่ได้อ้างว่าทำ cross-repository reasoning
- **Rafiei Oskooei et al. (LLM4Code @ ICSE 2026)** ที่ทำ multi-repo bug localization จริง ๆ ต้องไปใช้ **ระบบ industrial ปิด (DNext, 46 repos, 1.1M LOC)** เพราะไม่มี public benchmark ให้ใช้ — นี่คือหลักฐานทางอ้อมที่แข็งที่สุดว่า **ช่องว่างมีจริง**
- **CodeScaleBench (Sourcegraph, 2026)** เป็นหลักฐานตรงข้ามเพียงตัวเดียวที่พบ: มี multi-repo 136 tasks และ cross-repo navigation 47 tasks พร้อมหมวดหมู่อย่าง Dependency Tracing, Cross-Org Discovery, Cross-Repo Discovery, Onboarding & Comprehension แต่ (ก) เป็น blog + GitHub ไม่ผ่าน peer review (ข) เป็น **SWE task ที่ verify ด้วย deterministic gated verifier** ไม่ใช่ open-ended Q&A (ค) เพิ่งออกและยังปรับ version อยู่ (v1 371 → v2 264/275)

### สิ่งที่ควรบอกนักศึกษา

> **ข้อเสนอที่แข็งกว่า "อัลกอริทึม routing"** คือ:
> **"MicroQA-Bench: benchmark สำหรับ code QA ที่คำตอบต้องข้าม repository ในสถาปัตยกรรม microservices พร้อม routing-aware metrics"**
>
> เหตุผล: (1) ช่องว่างยืนยันได้จากคำพูดของ SWE-QA เอง (2) งาน ICSE 2026 ที่ทำเรื่องนี้ต้องใช้ข้อมูลปิด ทำให้ reproduce ไม่ได้ — โปรเจกต์นี้ทำเวอร์ชัน public ได้ (3) benchmark เป็น contribution ที่ประเมินง่ายกว่าอัลกอริทึม เพราะ "สร้างเสร็จก็คือเสร็จ" ไม่ต้องชนะ baseline (4) นักศึกษาคนเดียว 2 เทอมทำ benchmark ขนาด 150–300 คำถาม + baseline 3 ตัว ได้จริง ในขณะที่การพิสูจน์ว่า routing algorithm ใหม่ชนะ ต้องมี benchmark อยู่ก่อนแล้ว
>
> **ทางที่ดีที่สุดคือทำทั้งคู่แต่วางน้ำหนักใหม่:** benchmark = contribution หลัก, routing = baseline ตัวหนึ่งที่นักศึกษาเสนอเอง

**คำเตือนสำคัญ:** ต้องให้นักศึกษาค้นซ้ำก่อนส่งข้อเสนอ และต้องอ้าง CodeScaleBench ให้ตรงไปตรงมาว่า "มีงานอุตสาหกรรมที่ครอบคลุม multi-repo แล้ว แต่ยังไม่มีเวอร์ชันวิชาการที่เน้น QA" ถ้าไม่อ้างแล้วกรรมการหาเจอ จะเสียหายมาก

---

## 4. วิธีสร้าง Ground-Truth Q&A แบบกึ่งอัตโนมัติ (มี published methodology)

### 4.1 รูปแบบ oracle ที่ใช้กันจริง — เรียงตามความน่าเชื่อถือ

| ชนิด oracle | หลักการ | ใช้ในงานไหน | ความแข็งของหลักฐาน |
|---|---|---|---|
| **Execution / test-based** | ground truth = test เปลี่ยนสถานะ Fail→Pass และ Pass→Pass ไม่พัง | SWE-bench family (F2P/P2P), SWE-bench-Live, CodeScaleBench (deterministic gated verifier), SWE-QA-Pro (executable environments) | ⭐ แข็งที่สุด — objective ตรวจซ้ำได้ |
| **Commit / PR diff เป็นเฉลย localization** | ไฟล์ที่ถูกแก้ใน PR ที่ปิด issue = gold files | Long Code Arena bug localization, Agent Retrieval Bench (code2test / edit2ripple), Rafiei Oskooei et al. | ⭐ แข็ง — ใช้ได้กับคำถาม "โค้ดส่วนนี้อยู่ที่ไหน" |
| **Issue thread + maintainer reply** | คำตอบสุดท้ายของ MEMBER / AUTHOR / CONTRIBUTOR = ground truth | CodeRepoQA (585,687 entries), CoReQA (176 repos) | ⭐⭐ ปานกลาง — natural แต่ noisy, ต้อง filter |
| **Failing-test trace → source** | เอา stack trace ตอน base commit ไป map กลับหาโค้ดต้นเหตุ | Agent Retrieval Bench (trace2code, 101 samples) | ⭐ แข็ง |
| **Review comment + commit ที่ตอบสนอง** | comment ใน review = query, commit ที่แก้ตาม = gold context | Agent Retrieval Bench (comment2context, 80 samples) | ⭐ แข็ง |
| **Taxonomy-driven seed + template instantiation** | สร้าง taxonomy จาก issue จริง → seed question → instantiate ด้วย tree-sitter → RAG ร่างคำตอบ → มนุษย์ตรวจ | **SWE-QA** — วิธีที่เหมาะกับโปรเจกต์นี้ที่สุด | ⭐⭐ ปานกลาง-แข็ง ถ้ามี human validation |
| **Nugget-based relevance** | แตกคำตอบเป็น "nuggets" แล้ววัดว่าเอกสารที่ retrieve ครอบคลุม nugget กี่ % | **FreshStack** (Thakur, Lin et al.) | ⭐⭐ ปานกลาง — ดีสำหรับ open-ended answer |
| **LLM-generated + human curation** | LLM ร่าง annotation แล้วมนุษย์ตรวจ | RepoQA (GPT-4-Turbo เขียน description โดยห้ามเอ่ยชื่อ function/variable เพื่อกัน keyword matching) | ⭐ ปานกลาง — ต้องระวัง self-preference bias |

### 4.2 pipeline ที่แนะนำสำหรับนักศึกษา (ผสมจาก SWE-QA + Agent Retrieval Bench)

1. **สร้าง taxonomy** — เก็บ issue / discussion จาก repo microservices จริง (เช่น Train Ticket มี issue จริงบน GitHub) แล้วโค้ดด้วยมือ ~100–200 ข้อ เพื่อได้หมวดคำถาม ตามแบบ SWE-QA (What/Why/Where/How × sub-intent)
2. **เพิ่มหมวดที่เป็น multi-repo โดยเฉพาะ** — นี่คือ novelty: เช่น *cross-service call tracing*, *contract/DTO consistency*, *shared config drift*, *service dependency chain* — หมวดเหล่านี้ **ไม่มีใน SWE-QA เพราะเขาทำ repo เดียว**
3. **Instantiate อัตโนมัติ** — ใช้ tree-sitter / static analysis ดึงชื่อ service, endpoint, DTO, env var แล้วเติมลง template
4. **สร้างเฉลยด้วย oracle ที่ตรวจได้** — ผูก gold answer เข้ากับ **ไฟล์/ฟังก์ชันจริง** ไม่ใช่แค่ข้อความ เพื่อให้วัด retrieval ได้ด้วย (ตามแบบ Agent Retrieval Bench)
5. **ใส่ negative controls** — เลียนแบบ Agent Retrieval Bench: **wrong-repository controls** (คำถามที่ repo ที่ดูคล้ายที่สุดไม่ใช่คำตอบ) และ **no-gold questions** (คำตอบไม่มีในโค้ดเลย ต้องตอบว่า "ไม่พบ") → **นี่คือส่วนที่ทำให้ benchmark วัด routing ได้จริง ไม่ใช่แค่ retrieval**
6. **Human validation** — นักศึกษาตรวจเองทั้งหมด + ให้อาจารย์/เพื่อนตรวจ subset (เช่น 15–20%) แล้วรายงาน **Cohen's κ** (StackRepoQA รายงาน κ = 0.78–0.87 เป็นเกณฑ์อ้างอิงที่ดี)
7. **Sanitization** — ลบ patch, commit hash, path ที่ตรงเป๊ะ, ชื่อไฟล์ ออกจากคำถาม (Agent Retrieval Bench ทำแบบนี้) มิฉะนั้นจะกลายเป็น string matching ไม่ใช่ comprehension
8. **Freeze ที่ base commit** — ตรึง corpus ที่ commit ก่อนแก้ เพื่อกัน leakage (Agent Retrieval Bench ใช้ 308 base-commit snapshots)

### 4.3 ขนาดที่สมจริงสำหรับ senior project 1 คน 2 เทอม

- SWE-QA = 576 ข้อ / 12 repos โดยทีมวิจัย → นักศึกษาคนเดียวควรตั้งเป้า **150–250 ข้อ** ครอบคลุม **2–3 ระบบ microservices** (แต่ละระบบมี 6–64 services)
- แนะนำสัดส่วน: single-repo 40% / cross-repo 40% / no-gold + wrong-repo control 20%

---

## 5. Metrics ที่ใช้กันจริง

### 5.1 Retrieval / routing layer

| Metric | ใช้ที่ไหน | หมายเหตุสำหรับโปรเจกต์นี้ |
|---|---|---|
| **Recall@k** (k = 1, 5, 10, 20) | Agent Retrieval Bench, Long Code Arena bug localization, repo-level neural code search | metric หลักของ retrieval |
| **MRR** | Agent Retrieval Bench, Rafiei Oskooei et al. (MRR = 0.50 บน 46 repos) | ให้ตัวเลขอ้างอิงเทียบได้ |
| **Pass@k** | Rafiei Oskooei et al. (Pass@10 = 0.82) | เหมาะกับ localization |
| **Precision@k / MAP / F1** | Long Code Arena bug localization | |
| **nDCG@10** | **CoIR** (metric หลัก) | มาตรฐาน IR |
| **alpha-nDCG + nugget coverage** | FreshStack | เหมาะเมื่อคำตอบต้องการหลายชิ้นหลักฐาน |
| **BCY@B (Budgeted Context Yield)** | Agent Retrieval Bench | วัดว่า gold context ใส่ได้เท่าไรใน token budget ที่กำหนด — **metric นี้ตรงกับ multi-repo มาก** เพราะปัญหาหลักคือ context ไม่พอ |
| **Routing accuracy / wrong-repo rejection rate** | ยังไม่พบชื่อ metric มาตรฐาน `[ยังไม่ยืนยัน]` | **ช่องว่างที่นักศึกษาเสนอ metric เองได้** — เช่น Repo-Recall@1, False-Routing Rate |

### 5.2 Answer quality layer

| Metric | ใช้ที่ไหน |
|---|---|
| **LLM-as-judge, 5 มิติ (correctness, completeness, relevance, clarity, reasoning quality) สเกล 1–5, majority voting 5 ครั้ง/มิติ** | **SWE-QA** (ใช้ GPT-5 เป็น judge) |
| **LLM-as-judge 4 มิติ (correctness, completeness, faithfulness, conciseness) สเกล 1–10** | **StackRepoQA** |
| **LLM-as-judge 5 aspects** | CoReQA |
| **RAGAS: faithfulness, answer relevance, context precision, context recall** | Es et al., arXiv 2309.15217 — reference-free |
| **BLEU-threshold matching** | RepoQA (ต้องได้ BLEU สูงสุด และ > 0.8) — ใช้ได้เพราะคำตอบคือโค้ด |
| **ChrF / ROUGE / BERTScore / CompScore** | Long Code Arena |
| **resolved rate (test pass)** | SWE-bench family, SWE-bench Pro |

### 5.3 ความน่าเชื่อถือของ LLM-as-judge — ต้องรายงานอะไร

จากงานที่ค้นพบ:
- **StackRepoQA** รายงาน **Cohen's κ = 0.78–0.87** ระหว่าง LLM judge กับมนุษย์ → เป็นตัวเลขอ้างอิงที่ดี
- **SWE-QA** ใช้ human evaluation โดยวิศวกร 3 คน (ประสบการณ์ 4+ ปี) ประเมิน 144 คำถาม สเกล 10 ระดับ ในมิติเดียวกับ LLM judge — **แต่ไม่ได้รายงาน κ/Fleiss' kappa** (จุดอ่อนที่นักศึกษาทำให้ดีกว่าได้)
- MT-Bench-style: LLM judge แข็ง ๆ ได้ agreement ~80% กับมนุษย์ ซึ่งใกล้ระดับ human-human agreement `[อ้างจาก secondary source — ยังไม่ยืนยันจากเปเปอร์ต้นทาง]`
- งานสำรวจอื่น ๆ รายงาน κ ราว 0.61–0.64 ในบางงาน → **ช่วง κ 0.6–0.85 คือสิ่งที่ควรคาดหวัง**

**คำแนะนำ:** ให้นักศึกษา **บังคับตัวเองรายงาน κ** ระหว่าง LLM judge กับ human rating บน stratified subset (เช่น 20% ของคำถาม) ถ้า κ < 0.6 ต้องปรับ rubric ก่อนใช้ผล — นี่จะทำให้ committee เชื่อผลมาก และเป็นจุดที่ senior project ส่วนใหญ่ทำหล่น

### 5.4 Cost / latency — มีที่อ้างอิงแล้ว

- **DevQualityEval** วัด "costs, performance, efficiency (chattiness), reliability" เป็นส่วนหนึ่งของ leaderboard อย่างเป็นทางการ → อ้างได้ว่าการวัด cost เป็นแนวปฏิบัติที่ยอมรับแล้ว
- **CodeScaleBench** รายงานว่า MCP-augmented agents ถูกกว่า 30%, เร็วกว่า 38%, retrieval precision ดีขึ้น 2–3 เท่า → เป็นตัวอย่างการรายงาน cost/latency คู่กับ accuracy
- **แนะนำให้รายงาน:** tokens ต่อคำถาม (แยก prompt/completion), USD ต่อ 100 คำถาม, end-to-end latency (p50/p95), จำนวน tool call / retrieval round
- **ทำ cost–accuracy Pareto plot** — routing ที่ดีควรลด token โดยไม่เสีย accuracy นี่คือวิธีทำให้ "routing" มีค่าเชิงวัดได้ ไม่ใช่แค่ความเร็ว

---

## 6. คำตัดสินเรื่องคุณภาพหลักฐาน: ใช้ synthetic microservices corpus ได้หรือไม่?

### คำตอบ: **ได้ และมีหลักฐานรองรับหนักแน่น — แต่ห้ามใช้เดี่ยว ๆ**

### 6.1 หลักฐานว่าระบบเหล่านี้ผ่าน peer review ในฐานะ subject system

| ระบบ | หลักฐาน peer review |
|---|---|
| **TeaStore** | von Kistowski et al., *"TeaStore: A Micro-Service Reference Application for Benchmarking, Modeling and Resource Management Research"*, **IEEE MASCOTS 2018** — ออกแบบมาเพื่อเป็น research artifact โดยเฉพาะ; ยังมีเวอร์ชันต่อยอด *"TeaStore: A Micro-Service Reference Application for Research Use"* ตีพิมพ์ใน Springer LNCS |
| **Train Ticket** | Zhou, Peng, Xie, Sun et al., *"Fault Analysis and Debugging of Microservice Systems: Industrial Survey, Benchmark System, and Empirical Study"*, **IEEE Transactions on Software Engineering** (DOI 10.1109/TSE.2018.2887384) — วารสาร Q1 ของ SE; ก่อนหน้านั้นมี poster ที่ **ICSE 2018** (*"Benchmarking Microservice Systems for Software Engineering Research"*) — Train Ticket มี **64 services** เป็นระบบใหญ่ที่สุดในกลุ่ม demo |
| **Sock Shop** | ถูกประเมินเป็น candidate benchmark ใน Aderaldo, Mendonça, Pahl, Jamshidi, *"Benchmark Requirements for Microservices Architecture Research"*, **ECASE @ ICSE 2017** (19 services, Java/Go/Node.js) |
| **Google Online Boutique / microservices-demo** | ใช้เป็น subject system ในงานที่ผ่าน review เช่น BARO (root cause analysis), RCAEval, และงาน resource prediction บน Kubernetes |
| **ภาพรวมทั้งกลุ่ม** | Fischer, Urbanke, Ramler, Steidl, Felderer, *"An Overview of Microservice-Based Systems Used for Evaluation in Testing and Monitoring: A Systematic Mapping Study"*, **AST 2024** (ACM/IEEE), pp. 182–192, DOI 10.1145/3644032.3644445 — เป็น **systematic mapping study ที่ยืนยันว่าชุมชนวิจัยใช้ระบบเหล่านี้อย่างเป็นระบบ** และคัด 11 ระบบยอดนิยม: TrainTicket, Sock Shop, TeaStore, DeathStarBench (Social Network / Media Service / Hotel Reservation), Hipstershop, PyMicro, PiggyMetrics, Online Boutique, Rideshare |

**สรุปข้อนี้:** นี่คือหลักฐานที่นักศึกษาต้องการพอดี — **มี systematic mapping study ระดับ ACM/IEEE ที่ยืนยันว่าการใช้ demo microservices เป็น subject system เป็นแนวปฏิบัติมาตรฐานของวงการ** ให้อ้าง Fischer et al. (AST 2024) เป็นหลักฐานหลัก และ Zhou et al. (IEEE TSE 2018) เป็นตัวอย่างที่หนักที่สุด

### 6.2 ข้อจำกัดที่ต้องเขียนไว้ใน Threats to Validity

จากงานวิจัย microservices ที่พบ ข้อวิจารณ์ที่เกิดซ้ำ:
- ระบบ open-source microservices ส่วนใหญ่ **สร้างมาเพื่อ demo** ไม่สะท้อน ecosystem ทั้งหมด; scale และ maturity ต่างจากระบบจริงของบริษัทใหญ่มาก
- **workload ที่ใช้ทดลองอาจไม่สะท้อน workload จริง**
- ระบบจริงมี failure pattern ที่ซับซ้อนและหลากหลายกว่า → กระทบ generalizability
- โค้ดของ demo มักสะอาดเกินจริง: ไม่มี legacy, ไม่มี dead code, ไม่มีความไม่สอดคล้องสะสม — **ซึ่งเป็นสิ่งที่ทำให้ multi-repo QA ยากในโลกจริง**

### 6.3 ข้อเสนอที่ทำให้หลักฐาน "ผ่านได้แน่นอน"

ให้นักศึกษาทำ **corpus 2 ชั้น** แทนที่จะพึ่ง demo อย่างเดียว:

**ชั้นที่ 1 — Controlled corpus (demo systems)**
- Train Ticket (64 services, มี fault 22 แบบที่ replicate ไว้แล้วใน TSE paper), TeaStore, Sock Shop, Online Boutique
- ข้อดี: ground truth สร้างได้แม่นยำ, reproducible 100%, มี citation รองรับ, deploy ได้จริงเพื่อทำ execution-based oracle
- บทบาท: **วัด internal validity** — พิสูจน์ว่าระบบทำงานถูกต้อง

**ชั้นที่ 2 — Real multi-repo organization (สำคัญมาก, ทำได้โดยไม่ต้องขออนุญาตใคร)**
- ใช้ **GitHub organization จริงที่เป็น open source และมีหลาย repo ที่คุยกัน** เช่น org ที่มี service หลายตัว + shared client library + shared schema/proto
- ข้อดี: ตอบข้อวิจารณ์ "synthetic เกินไป" ได้ทันที และมี issue/PR จริงให้ใช้เป็น oracle
- **ประเด็นสำคัญที่ต้องบอกนักศึกษา:** ข้อความ risk เดิมที่ว่า "real access is unavailable" **ไม่ถูกต้อง** — โค้ด open-source multi-repo มีให้ใช้ฟรีมหาศาล สิ่งที่ไม่มีคือ *proprietary enterprise* codebase ซึ่งไม่จำเป็นสำหรับ senior project ให้แก้ถ้อยคำในข้อเสนอเป็น *"ไม่มี proprietary enterprise corpus จึงใช้ demo systems ร่วมกับ open-source multi-repo organizations"*

**ชั้นที่ 3 (ถ้ามีเวลา) — cross-corpus generalization test**
- train/tune บนชั้น 1 แล้ว report ผลบนชั้น 2 โดยไม่ tune → เป็นหลักฐาน external validity ที่แข็งมากสำหรับ senior project

### 6.4 คำตัดสินสรุป

> ✅ **synthetic demo corpus เป็นหลักฐานที่ยอมรับได้** สำหรับ senior project และมีงานตีพิมพ์ระดับ IEEE TSE / ICSE / MASCOTS / AST รองรับ
> ⚠️ **แต่จะถูกโจมตีเรื่อง external validity แน่นอน** ถ้าใช้เดี่ยว ๆ
> ✅ **วิธีแก้ที่ต้นทุนต่ำ:** เพิ่ม open-source multi-repo organization อย่างน้อย 1 แห่ง + เขียน threats to validity ให้ตรงไปตรงมา + รายงาน κ ของ LLM judge
> ❌ **สิ่งที่จะทำให้ตก:** ใช้ demo อย่างเดียว + LLM-as-judge อย่างเดียว + ไม่มี human validation + ไม่มี negative control

---

## 7. ข้อเสนอ Evaluation Plan สำหรับโปรเจกต์นี้ (สังเคราะห์)

**เทอม 1 — สร้าง benchmark (contribution หลัก)**
1. เลือกระบบ: Train Ticket + TeaStore + Sock Shop (ชั้น 1) และ open-source org 1 แห่ง (ชั้น 2)
2. สร้าง taxonomy คำถามแบบ SWE-QA แต่เพิ่มหมวด cross-repo (cross-service call, contract consistency, config drift, dependency chain)
3. สร้าง 150–250 คำถาม: single-repo 40% / cross-repo 40% / negative control 20%
4. ผูกเฉลยกับไฟล์/ฟังก์ชันจริงเพื่อให้วัด retrieval ได้
5. Human validation + รายงาน Cohen's κ

**เทอม 2 — ประเมินระบบ**
6. Baseline อย่างน้อย 4 ตัว: (a) BM25 flat over all repos, (b) dense embedding flat, (c) long-context ยัดทั้งหมด, (d) **routing-then-retrieve ของนักศึกษา** — และควรมี (e) NL-summary routing แบบ Rafiei Oskooei et al. เป็น strong baseline
7. Metrics 3 ชั้น: **routing** (Repo-Recall@1, false-routing rate บน wrong-repo controls, abstention rate บน no-gold), **retrieval** (Recall@k, MRR, BCY@B), **answer** (LLM-as-judge 5 มิติ + κ กับมนุษย์)
8. Cost/latency: tokens ต่อคำถาม, USD/100 คำถาม, latency p50/p95 → **plot Pareto**
9. Ablation: ปิด routing ออก แล้วดูว่า accuracy/cost เปลี่ยนอย่างไร — นี่คือหลักฐานว่า routing มีค่าจริง

---

## 8. บรรณานุกรมพร้อมคำอธิบาย (Annotated Bibliography)

### 8.1 Repo-level / Multi-repo QA benchmarks (สำคัญที่สุด)

1. **SWE-QA: Can Language Models Answer Repository-level Code Questions?** (2025-09-17) — https://arxiv.org/abs/2509.14635 | PDF: https://arxiv.org/pdf/2509.14635 | HTML: https://arxiv.org/html/2509.14635v1 | ACL: https://aclanthology.org/2026.findings-acl.402/ | HF: https://huggingface.co/papers/2509.14635
   *576 QA pairs / 12 Python repos จาก SWE-bench (astropy, django, flask, matplotlib, pylint, pytest, requests, scikit-learn, sphinx, sqlfluff, sympy, xarray), 48 คำถาม/repo. Taxonomy 2 ระดับ: What 13.3% / Why 23.1% / Where 28.4% / How 35.2% × 12 fine-grained intentions. Pipeline: crawl 77,100 issues → extract 127,415 questions → manual code 1,000 → tree-sitter instantiation → RAG-drafted answers → expert review. Judge = GPT-5, 5 มิติ (correctness, completeness, relevance, clarity, reasoning quality) สเกล 1–5 majority-vote 5 ครั้ง. Human eval: 3 วิศวกร ประเมิน 144 คำถาม สเกล 10. ระบุชัดว่า benchmark เดิม "ไม่บังคับให้เข้าใจ code module ในฐานะองค์ประกอบสถาปัตยกรรมที่เชื่อมกัน" — **นี่คือ blueprint หลักสำหรับ methodology ของนักศึกษา***

2. **Beyond Code Snippets: Benchmarking LLMs on Repository-Level Question Answering (StackRepoQA)** — Alebachew, Leary, Vaishampayan, Brown (Virginia Tech), 2026-03-27 — https://arxiv.org/html/2603.26567v1 | PDF: https://arxiv.org/pdf/2603.26567
   *1,318 คำถามจริงจาก Stack Overflow map ไป 134 Java projects. คัด repo ด้วย 20+ contributors, 10+ stars + manual verification. LLM-as-judge 4 มิติ (correctness, completeness, faithfulness, conciseness) สเกล 1–10, **Cohen's κ = 0.78–0.87**. เรียกตัวเองว่า "multi-project" แต่คำถาม scope อยู่ใน project เดียว — **ตัวอย่างสำคัญของการใช้คำว่า multi-repo ในความหมาย (A) ไม่ใช่ (C)***

3. **CoReQA: Uncovering Potentials of Language Models in Code Repository Question Answering** (2025-01-07) — https://arxiv.org/abs/2501.03447 | PDF: https://arxiv.org/pdf/2501.03447
   *สร้างจาก GitHub issues + comments ของ 176 repos ยอดนิยม 4 ภาษา. LLM-as-judge 5 aspects. baseline 3 ตัว (short-context + generic retrieval ×2, long-context ทั้ง repo ×1). พบว่าโมเดล proprietary/long-context ยังทำ repo-level question ได้ไม่ดี*

4. **CodeRepoQA: A Large-scale Benchmark for Software Engineering Question Answering** (2024-12-19) — https://arxiv.org/abs/2412.14764 | PDF: https://arxiv.org/pdf/2412.14764
   ***585,687 entries**, เฉลี่ย 6.62 dialogue turns, crawl จาก 30 repos, 5 ภาษา (Python, Java, TypeScript, JavaScript, Go). **Ground truth = คำตอบสุดท้ายของ maintainer (MEMBER/AUTHOR/CONTRIBUTOR)** — เป็น oracle strategy ที่ scale ได้ที่สุดและอ้างได้. พบว่า medium-length context ให้ผลดีที่สุด*

5. **SWE-QA-Pro: A Representative Benchmark and Scalable Training Recipe for Repository-Level Code Understanding** (2026-03-17) — https://arxiv.org/abs/2603.16124 | PDF: https://arxiv.org/pdf/2603.16124
   *ต่อยอด SWE-QA ใช้ long-tail repos ที่มี executable environment เพื่อกัน memorization; issue-driven clustering + difficulty calibration (ตัดคำถามที่ direct-answer baseline ตอบได้ทิ้ง). agentic workflow ชนะ direct answering ~13 จุดใน Claude Sonnet 4.5. `[ขนาด dataset ยังไม่ยืนยัน — abstract ไม่ระบุ]`*

6. **RepoQA: Evaluating Long Context Code Understanding** (2024-06-10) — https://arxiv.org/html/2406.06025 | PDF: https://arxiv.org/pdf/2406.06025
   *Task "Searching Needle Function" 500 tests / 50 repos / 5 ภาษา (Python, C++, Java, TypeScript, Rust) 10 needle/repo. Ground truth กึ่งอัตโนมัติ: GPT-4-Turbo เขียน description 4 ส่วน (purpose/input/output/procedure) **โดยห้ามเอ่ยชื่อ function หรือ variable** เพื่อกัน keyword matching. Metric = BLEU สูงสุดในบริบท และ > 0.8. ประเมิน 33 LLMs*

7. **DeepRepoQA (OpenReview submission)** — https://openreview.net/pdf/7ad2c0eaa143bfa3eed9179418773d8d727a2dde.pdf
   *`[ยังไม่ได้ fetch เนื้อหา — พบเฉพาะในผลค้นหา; ต้องตรวจสอบก่อนอ้าง]`*

### 8.2 Multi-repo / cross-repo โดยตรง — หัวใจของคำถามหลัก

8. **CodeScaleBench: Testing coding agents on large codebases and multi-repo software engineering tasks** — Sourcegraph, 2026 — Blog: https://sourcegraph.com/blog/codescalebench-testing-coding-agents-on-large-codebases-and-multi-repo-software-engineering-tasks | GitHub: https://github.com/sourcegraph/CodeScaleBench | Report: https://sourcegraph.com/resources/ebooks/code-scale-bench-report
   ***หลักฐานตรงข้ามที่สำคัญที่สุดต่อข้ออ้าง "ไม่มี multi-repo benchmark"*** *— `csb-v2-full-validated` 275 tasks = single-repo 186 / dual-repo 28 / multi-repo 61; รวม multi-repository tasks 136 ข้าม 28 repo sets และ cross-repository navigation 47 (dependency tracing, org-wide discovery). แบ่ง CodeScaleBench-SDLC (150 tasks, patch-based verifier) และ -Org (220 tasks, org-level navigation). 40+ repos (Kubernetes, Django, Linux, VSCode), 9 ภาษา, repo <100MB ถึง >5GB. Verifier แบบ "deterministic gated" มี calibration gate: empty answer ≤0.1, canonical ≥0.9, adversarial ≤0.5. Apache 2.0. รายงาน MCP-augmented agent ถูกกว่า 30% เร็วกว่า 38% retrieval precision ดีขึ้น 2–3×. **ไม่ผ่าน peer review** และเป็น SWE task ไม่ใช่ open-ended QA*

9. **Natural Language Summarization Enables Multi-Repository Bug Localization by LLMs in Microservice Architectures** — Rafiei Oskooei, Yukcu, Bozoglan, Aktas — **LLM4Code Workshop @ ICSE 2026** (submitted 2025-12-05) — https://arxiv.org/abs/2512.05908 | HTML: https://arxiv.org/html/2512.05908 | PDF: https://arxiv.org/pdf/2512.05908
   ***งานที่ใกล้โปรเจกต์นักศึกษาที่สุด*** *— แปลง codebase เป็น hierarchical NL summary ระดับ file / directory / repository แล้วทำ NL-to-NL search แทน cross-modal retrieval. **Two-phase: routing bug report → repository ที่เกี่ยวข้อง ก่อน แล้วจึง top-down localization ภายใน** ประเมินบน DNext ระบบ industrial **46 repositories / 1.1M LOC** ได้ **Pass@10 = 0.82, MRR = 0.50** ชนะ retrieval baselines และ agentic RAG (GitHub Copilot, Cursor). ข้อเท็จจริงที่เขาต้องใช้ระบบ industrial ปิด = หลักฐานว่าไม่มี public multi-repo benchmark. `[วิธีสร้าง ground truth ไม่ปรากฏใน abstract — ต้องอ่าน full paper]`*

10. **Agent Retrieval Bench: Evaluating Repository Context Retrieval for Coding Agents** — Qin (NUS), Xie (PKU), 2026-07 — https://arxiv.org/html/2607.24882v1
    ***แหล่ง methodology สำหรับ negative control*** *— 427 samples / 25 repos: positive 345 (code2test 106, comment2context 80, trace2code 101, edit2ripple 58) + **natural no-gold 50** + **counterfactual wrong-repository controls 32**. Mining จาก merged PR (impl+test), inline review comment + response commit, reproduced failing test ที่ base commit, anchored edit + changed-file evidence. Corpus แช่แข็งที่ 308 base-commit snapshots ~392k ไฟล์ 6 ภาษา. Sanitize ลบ patch/commit hash/exact path/autogenerated artifacts. Metrics: Recall@5/10/20, MRR, **BCY@B (Budgeted Context Yield)**. รายงานผลแบบ sample-/task-/repository-weighted*

### 8.3 SWE-bench family

11. **SWE-bench (Overview)** — https://www.swebench.com/SWE-bench/
    *ระบุ SWE-bench Verified ปล่อย 2024-08-13 มี **500 engineer-confirmed solvable problems**; SWE-bench Multimodal ปล่อย 2025-01-13 พร้อม private test split*

12. **SWE-bench Multimodal: Do AI Systems Generalize to Visual Software Domains?** (2024-10-04) — https://arxiv.org/abs/2410.03859 | PDF: https://arxiv.org/pdf/2410.03859
    ***617 task instances / 17 JavaScript libraries*** *(web interface design, diagramming, data visualization, syntax highlighting, interactive mapping)*

13. **SWE-bench Goes Live!** (SWE-bench-Live, 2025-05) — https://arxiv.org/abs/2505.23419 | Leaderboard: https://swe-bench-live.github.io/ | HF: https://huggingface.co/papers/2505.23419
    ***1,319 tasks / 93 repos** จาก issue ที่สร้างหลังปี 2024, มี Docker image ต่อ task, automated curation pipeline เพื่อกัน contamination*

14. **SWE-Bench Pro Leaderboard (Public Dataset)** — Scale AI — https://labs.scale.com/leaderboard/swe_bench_pro_public
    ***1,865 tasks / 41 repos**: Public 731 (GPL repos), Private 276 (proprietary startup codebases), Held-out 858. Top model ~23% resolution (เทียบกับ 70%+ บน SWE-bench Verified)*

15. **SWE-rebench: An Automated Pipeline for Task Collection and Decontaminated Evaluation** (2025-05) — https://arxiv.org/abs/2505.20411 | PDF: https://arxiv.org/pdf/2505.20411
    *`[อ้างจากผลค้นหา ยังไม่ได้ fetch — เป็นอีกตัวอย่างของ automated task collection pipeline ที่นักศึกษาอ่านเพิ่มได้]`*

### 8.4 Repo-level code completion / retrieval benchmarks

16. **RepoBench: Benchmarking Repository-Level Code Auto-Completion Systems** (2023-06-05, v2 2023-10-04) — https://arxiv.org/abs/2306.03091 | OpenReview: https://openreview.net/forum?id=pPjZIOuQuF
    *Python + Java; 3 tasks: RepoBench-R (retrieval snippet จากไฟล์อื่น), RepoBench-C (ทำนายบรรทัดถัดไปจาก in-file + cross-file context), RepoBench-P (pipeline รวม). **ไม่มี unit test** จึงคำนวณ pass@k ไม่ได้ ใช้ similarity metric แทน. `[ขนาด dataset แต่ละ task ยังไม่ยืนยัน]`*

17. **CrossCodeEval** — https://www.emergentmind.com/topics/crosscodeeval-benchmark
    *~10K examples, 4 ภาษา (Python, Java, TypeScript, C#), สร้างด้วย static analysis + retrieval เพื่อบังคับให้ต้องใช้ cross-file context. เป็น single-line generation + similarity metric*

18. **Long Code Arena: a Set of Benchmarks for Long-Context Code Models** — JetBrains Research + TU Delft (2024-06-17) — https://arxiv.org/abs/2406.11612 | HTML: https://arxiv.org/html/2406.11612v1 | Space: https://huggingface.co/spaces/JetBrains-Research/long-code-arena
    *6 tasks: library-based code generation (150 datapoints / 62 libraries, ChrF + API Recall), CI builds repair (77, % fixed builds), project-level code completion (934 = 144+224+270+296, exact match), commit message generation (163 / 34 repos, BLEU/ROUGE/ChrF/BERTScore), **bug localization (150 = 50/ภาษา, Python/Java/Kotlin, Recall@k/Precision@k/F1/MAP)**, module summarization (216 / 43 repos, ChrF + CompScore). Dataset ผ่าน manual verification ทุกตัว*

19. **CoIR: A Comprehensive Benchmark for Code Information Retrieval Models** — **ACL 2025 Main** — https://aclanthology.org/2025.acl-long.1072/ | arXiv: https://arxiv.org/pdf/2407.02883 | HTML: https://arxiv.org/html/2407.02883v3 | GitHub: https://github.com/coir-team/coir
    *10 datasets / 8 retrieval tasks / 7 domains (GitHub Functions, Web Query, Database, Contest, Deep Learning, StackOverflow, Code Instruction). 4 paradigm: text→code, code→text, code→code, hybrid. **Metric หลัก NDCG@10** + precision/recall. วิจารณ์ CodeSearchNet/CosQA/XcodeEval ว่าโดน overfit*

20. **CodeSearchNet Challenge: Evaluating the State of Semantic Code Search** (2019) — https://arxiv.org/pdf/1909.09436 | GitHub: https://github.com/github/CodeSearchNet | MSR: https://www.microsoft.com/en-us/research/publication/codesearchnet-challenge-evaluating-the-state-of-semantic-code-search-2/
    *Corpus ~6M functions, 6 ภาษา (Go, Java, JavaScript, PHP, Python, Ruby) + auto-generated query-like NL สำหรับ 2M functions. Eval set = **99 natural language queries + ~4k expert relevance annotations**. **Challenge ปิดแล้ว** (ระบุบนหน้า GitHub) — อ้างเป็น historical baseline ได้ แต่อย่าใช้เป็น benchmark หลัก*

21. **CodeRAG-Bench: Can Retrieval Augment Code Generation?** — **NAACL 2025 Findings** — https://aclanthology.org/2025.findings-naacl.176/ | arXiv: https://arxiv.org/abs/2406.14497
    *ประเมินทั้ง retrieval และ end-to-end code generation แยกกัน ครอบคลุม basic programming / open-domain / repository-level. Document source: competition solutions, tutorials, library docs, StackOverflow, GitHub repos. **ข้อค้นพบสำคัญ: retriever ยังหา context ที่มีประโยชน์ไม่ได้เมื่อ lexical overlap ต่ำ** — เป็นหลักฐานสนับสนุนว่าปัญหา retrieval ยังไม่ถูกแก้*

22. **FreshStack: Building Realistic Benchmarks for Evaluating Retrieval on Technical Documents** — Thakur, Lin et al. (2025) — https://arxiv.org/pdf/2504.13128 | Site: https://fresh-stack.github.io/
    *Pipeline อัตโนมัติ: คำถามจริงจาก Stack Overflow + GitHub repos → LLM แตก **nuggets** → หา gold document ที่มี nugget. Metrics: **alpha-nDCG, coverage, Recall@k**. `[ขนาด dataset ยังไม่ยืนยัน]`*

### 8.5 Industry / agent benchmarks (สำหรับ cost & latency)

23. **DevQualityEval** — Symflower — Docs: https://docs.symflower.com/docs/devqualityeval/ | GitHub: https://github.com/symflower/eval-dev-quality | Leaderboard: https://symflower.com/en/products/devqualityeval-leaderboard/
    *วัดหลายมิติรวมถึง **costs, performance, efficiency (chattiness), reliability** และมี private dataset กัน contamination — **อ้างเป็นหลักฐานว่าการรายงาน cost เป็นแนวปฏิบัติที่ยอมรับ***

24. **Aider Polyglot Leaderboard** — https://llm-stats.com/benchmarks/aider-polyglot
    ***225 Exercism exercises**, 6 ภาษา (C++, Go, Java, JavaScript, Python, Rust), hidden test suite. ไม่ใช่ repo-level — อ้างเพื่อ contrast เท่านั้น*

25. **Terminal-Bench 2.0** — Epoch AI — https://epoch.ai/benchmarks/terminal-bench
    ***89 tasks** ใน task pool ของ 2.0 ครอบคลุม SE, ML, security, data science ใน shell environment จริง. ไม่ใช่ repo-QA — อ้างเพื่อ contrast*

26. **Benchmarks evaluating LLM agents for software development** — Symflower blog — https://symflower.com/en/company/blog/2025/benchmarks-llm-agents/ | เพิ่มเติม: https://symflower.com/en/company/blog/2024/comparing-llm-benchmarks/
    *ภาพรวมเปรียบเทียบ benchmark — ใช้เป็น secondary source สำหรับ related work*

27. **15 LLM coding benchmarks** — Evidently AI — https://www.evidentlyai.com/blog/llm-coding-benchmarks
    *สรุปรวม benchmark ยอดนิยม — secondary source*

### 8.6 LLM-as-judge & RAG metrics

28. **Ragas: Automated Evaluation of Retrieval Augmented Generation** — Es et al. (2023-09-26) — https://arxiv.org/abs/2309.15217 | Docs: https://docs.ragas.io/en/stable/concepts/metrics/available_metrics/
    *Framework reference-free: faithfulness, answer relevance, context relevance/precision/recall — **metric ชุดมาตรฐานสำหรับส่วน RAG ของโปรเจกต์***

29. **Judge's Verdict: A Comprehensive Analysis of LLM Judge Capability Through Human Agreement** (2025-10) — https://arxiv.org/pdf/2510.09738
    *`[อ้างจากผลค้นหา ยังไม่ได้ fetch — น่าจะเป็นแหล่งอ้างอิงที่ดีที่สุดสำหรับหัวข้อ LLM-judge reliability ควรให้นักศึกษาอ่านเต็ม]`*

30. **From Code to Courtroom: LLMs as the New Software Judges** (2025-03) — https://arxiv.org/pdf/2503.02246
    *`[อ้างจากผลค้นหา ยังไม่ได้ fetch — survey ของ LLM-as-judge ในบริบท SE โดยเฉพาะ ตรงกับโปรเจกต์นี้มาก ควรอ่าน]`*

31. **How to Calibrate LLM-as-Judge with Human Corrections** — LangChain — https://www.langchain.com/resources/llm-as-a-judge
    *แนวปฏิบัติ calibration — secondary source แต่ practical*

### 8.7 Microservices demo systems — หลักฐานคุณภาพ corpus

32. **An Overview of Microservice-Based Systems Used for Evaluation in Testing and Monitoring: A Systematic Mapping Study** — Fischer, Urbanke, Ramler, Steidl, Felderer — **AST 2024 (ACM/IEEE), pp. 182–192**, DOI 10.1145/3644032.3644445 — https://dl.acm.org/doi/10.1145/3644032.3644445 | PDF: https://elib.dlr.de/211381/1/2024_AST_MicroService_BMK.pdf | IEEE: https://ieeexplore.ieee.org/iel8/10555578/10556387/10556489.pdf | Replication data: https://github.com/software-competence-center-hagenberg/2024-AST-Microservices-QA
    ***หลักฐานชิ้นสำคัญที่สุดสำหรับคำถามเรื่อง evidence quality*** *— systematic mapping study ที่คัด 11 ระบบยอดนิยม: TrainTicket, Sock Shop, TeaStore, DeathStarBench (Social Network / Media Service / Hotel Reservation), Hipstershop, PyMicro, PiggyMetrics, Online Boutique, Rideshare พร้อมคุณลักษณะ (ขนาด, test ที่มี, เทคโนโลยี). มี replication package บน GitHub. **อ้างประโยคนี้ในบท methodology ได้เลย** `[ยังไม่ได้อ่าน full text — PDF เป็น binary ที่ fetch ไม่สำเร็จ ควรให้นักศึกษาโหลดอ่านเอง]`*

33. **Fault Analysis and Debugging of Microservice Systems: Industrial Survey, Benchmark System, and Empirical Study** — Zhou, Peng, Xie, Sun, Ji, Li, Ding — **IEEE Transactions on Software Engineering**, DOI 10.1109/TSE.2018.2887384 — https://dl.acm.org/doi/10.1109/tse.2018.2887384
    ***หลักฐานว่า Train Ticket ผ่าน peer review ระดับวารสาร Q1*** *— industrial survey + สร้าง benchmark microservice system ขนาดกลาง + **replicate 22 faults** — ชุด fault นี้ใช้เป็น ground truth ได้โดยตรง*

34. **Benchmarking Microservice Systems for Software Engineering Research** (poster) — Zhou, Peng, Xie, Sun, Xu, Ji, Zhao — **ICSE 2018 Companion** — https://dl.acm.org/doi/10.1145/3183440.3194991 | PDF: https://cspengxin.github.io/publications/icse18poster-microservices.pdf
    *ต้นทางของ Train Ticket ที่ ICSE*

35. **Train Ticket — A Benchmark Microservice System** (source) — https://github.com/FudanSELab/train-ticket
    ***64 services** มีทั้ง sync และ async communication, call chain ซับซ้อนกว่า Sock Shop และ Online Boutique — **เป็นระบบใหญ่ที่สุดในกลุ่ม demo และเหมาะที่สุดกับ multi-repo QA***

36. **Benchmark Requirements for Microservices Architecture Research** — Aderaldo, Mendonça, Pahl, Jamshidi — **ECASE @ ICSE 2017** — https://dl.acm.org/doi/10.5555/3101282.3101285 | Semantic Scholar: https://www.semanticscholar.org/paper/6e642e64872236a7c8a18da4ea8e35bed1671e4f | DBLP: https://dblp.org/db/conf/icse/ecase2017.html
    *เสนอเกณฑ์เลือก community benchmark และประเมิน candidate: Acme Air (4 services, Java+Node.js), Spring Cloud demo (8), movie recommendation (11 Java), **Socks Shop (19 services, Java/Go/Node.js)**, MusicStore (8 .NET). **อ้างเป็นหลักฐานว่า Sock Shop ถูกพิจารณาเป็น research benchmark ตั้งแต่ 2017***

37. **TeaStore: A Micro-Service Reference Application for Benchmarking, Modeling and Resource Management Research** — von Kistowski et al. — **IEEE MASCOTS 2018** — https://www.researchgate.net/publication/328638010_TeaStore_A_Micro-Service_Reference_Application_for_Benchmarking_Modeling_and_Resource_Management_Research | Springer (research-use version): https://link.springer.com/chapter/10.1007/978-3-030-41705-5_14 | scite: https://scite.ai/reports/teastore-a-micro-service-reference-application-QepjA66
    ***TeaStore ถูกออกแบบมาเพื่อการวิจัยโดยตรง*** *— demonstrate ใน 3 บริบท: performance modeling, cloud resource management, energy efficiency analysis*

38. **Sock Shop — Deployment scripts & config** — https://github.com/microservices-demo/microservices-demo
    *Spring Boot + Go kit + Node.js, Docker; ตัวเลขจำนวน service แตกต่างกันในแต่ละแหล่ง (15 vs 19) — **นักศึกษาต้องนับเองจาก repo จริงแล้วรายงานตัวเลขของตัวเอง***

39. **Google microservices-demo (Online Boutique)** — https://github.com/criblio/google-microservices-demo (mirror)
    *10–12 services, gRPC, Kubernetes/Istio — จำนวน service ที่รายงานต่างกันตามเวอร์ชัน (10, 11, 12) `[ต้องนับเองจาก upstream repo]`*

40. **BARO: Robust Root Cause Analysis for Microservices via Multivariate Bayesian Online Change Point Detection** — https://arxiv.org/pdf/2405.09330
    *ตัวอย่างงานที่ deploy **Online Boutique + Sock Shop + Train Ticket** พร้อมกันบน Kubernetes cluster — **อ้างเป็น precedent ว่าใช้หลายระบบพร้อมกันเป็นเรื่องปกติ***

41. **RCAEval: A Benchmark for Root Cause Analysis of Microservice Systems with Telemetry Data** (2024-12-22) — https://arxiv.org/abs/2412.17015 | PDF: https://arxiv.org/pdf/2412.17015
    ***3 datasets / 735 failure cases / 3 microservice systems*** *— ตัวอย่าง benchmark ที่สร้างจาก demo systems ล้วน ๆ และได้รับการยอมรับ `[venue ยังไม่ยืนยัน]`*

42. **Benchmarks for End-to-End Microservices Testing** — https://arxiv.org/pdf/2306.05895
    *`[อ้างจากผลค้นหา ยังไม่ได้ fetch]`*

### 8.8 แหล่งรวบรวม (ใช้ค้นต่อ)

43. **Awesome-Repo-Level-Code-Generation** — https://github.com/YerbaPage/Awesome-Repo-Level-Code-Generation
44. **repo-level-codegen-papers** — https://github.com/allanj/repo-level-codegen-papers
    *ทั้งสองเป็น curated list ที่อัปเดตอยู่ — **ให้นักศึกษาเช็คซ้ำก่อนส่งข้อเสนอ** เพื่อยืนยันว่าไม่มี multi-repo QA benchmark ตัวใหม่โผล่มา*
45. **Microservices Reference and Benchmark Applications** (แคตตาล็อก) — https://notes.davidkopp.de/30-knowledge/microservices-reference-and-benchmark-applications/
    *รายการระบบ microservices สำหรับงานวิจัย — secondary source ที่ใช้หา candidate corpus ได้เร็ว*

---

## 9. ข้อควรระวังสำหรับอาจารย์

1. **ตัวเลขที่ทำเครื่องหมาย `[ยังไม่ยืนยัน]`** ต้องให้นักศึกษาเปิดเปเปอร์ตรวจเองก่อนใส่ในรายงาน — โดยเฉพาะขนาด RepoBench, SWE-bench full (2,294), SWE-QA-Pro, FreshStack, DevQualityEval
2. **arXiv preprint ≠ peer-reviewed** — งานปี 2026 หลายชิ้น (StackRepoQA, SWE-QA-Pro, Agent Retrieval Bench) ยังเป็น preprint ให้ระบุสถานะให้ตรงในบรรณานุกรม
3. **CodeScaleBench เป็นความเสี่ยงต่อ novelty claim ของนักศึกษา** — ต้องอ่านและอ้างอิงตรงไปตรงมา และปรับ framing เป็น "peer-reviewed academic multi-repo *QA* benchmark ยังไม่มี" ไม่ใช่ "multi-repo benchmark ไม่มี"
4. **สนามนี้เคลื่อนเร็วมาก** — benchmark ใหม่โผล่แทบทุกเดือน ให้นักศึกษาค้นซ้ำก่อนสอบหัวข้อ และอีกครั้งก่อนสอบจบ
5. **ข้อความ risk เดิมของนักศึกษาผิดบางส่วน** — "real access is unavailable" ไม่จริงสำหรับ open-source multi-repo organizations ควรแก้ถ้อยคำ
