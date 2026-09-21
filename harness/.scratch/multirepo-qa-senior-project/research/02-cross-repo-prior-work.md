# 02 — Cross-Repo Code QA: Prior Work และการตรวจสอบข้ออ้างของนักศึกษา

วันที่สำรวจ: 2026-09-01
ข้ออ้างที่ตรวจสอบ: *"Cursor/Copilot are single-repo; Sourcegraph+Cody need enterprise indexing infra. No lightweight cross-repo solution currently exists."*

---

## VERDICT (สรุปตรงประเด็น)

**ข้ออ้างนี้ผิดเป็นส่วนใหญ่ (mostly FALSE) และถ้านักศึกษาเสนอต่อกรรมการในรูปแบบนี้ จะถูกตีตกได้ภายใน 5 นาที** เพราะกรรมการสามารถชี้ tool ที่มีอยู่จริงได้ทันที

แยกเป็นข้อย่อย:

| ข้อย่อยของนักศึกษา | คำตัดสิน | เหตุผลสั้น |
|---|---|---|
| "Cursor เป็น single-repo" | **ผิดชัดเจน** | Cursor รองรับ multi-root workspace มาตั้งแต่ พ.ค. 2025 และยกระดับเป็น first-class agent target ใน Cursor 3.2 (24 เม.ย. 2026) — index แยกต่อ root, `@Codebase` ค้นข้าม root ทั้งหมด |
| "Copilot เป็น single-repo" | **จริงบางส่วน** | นี่คือส่วนเดียวที่ยังพอยืนได้: VS Code Copilot semantic index **พังจริง** ใน multi-root workspace (issue #313181, #271044), `copilot-instructions.md` ไม่ถูกอ่านใน multi-root (#264837), Copilot cloud/coding agent ยัง sandbox อยู่ที่ 1 repo |
| "Claude Code เป็น single-repo" | **ผิด** (นักศึกษาไม่ได้อ้าง แต่กรรมการจะถาม) | มี `--add-dir` / `/add-dir` / `additionalDirectories` ใน settings.json ให้ทำงานหลาย repo ใน session เดียว |
| "Sourcegraph+Cody ต้องใช้ enterprise infra" | **จริง และจริงหนักกว่าที่นักศึกษาคิด** | Cody Free/Pro ถูกปิด 23 ก.ค. 2025, source code ย้ายเป็น private ตั้งแต่ ส.ค. 2024, Deep Search + MCP server เป็น Enterprise-only |
| **"No lightweight cross-repo solution currently exists"** | **ผิดชัดเจน — นี่คือจุดตายของ proposal** | มีอย่างน้อย 4–5 ตัวที่ทำสิ่งนี้อยู่แล้วบน laptop เครื่องเดียว โดยเฉพาะ `flupkede/codesearch` ซึ่งเป็น multi-repo semantic code search MCP server เขียนด้วย Rust, ไม่ต้องใช้ GPU / Docker / vector DB ภายนอก, ทำงาน offline เต็มรูปแบบ และมี **cross-repo RRF fan-out ranking + repository groups** ซึ่งก็คือ "routing layer" ที่นักศึกษาบอกว่าจะสร้างพอดี |

### สรุปประโยคเดียวสำหรับอาจารย์

> ระบบ "agent ที่ตอบคำถามข้ามหลาย repo โดยมี routing layer เลือก repo/file" **มีคนทำแล้ว หลายเจ้า และเบาพอที่จะรันบน laptop** — ดังนั้น novelty เชิง "สร้างของที่ยังไม่มี" **หายไปทั้งหมด** แต่ยังมีช่องว่างที่แท้จริงเหลืออยู่ 4 ข้อ (ดูหัวข้อ "ช่องว่างที่เหลืออยู่จริง") ซึ่งเป็นช่องว่างเชิง **evaluation และ semantic linkage** ไม่ใช่เชิง engineering artifact นักศึกษาต้องย้ายจุดยืนของ contribution ไปตรงนั้น มิฉะนั้นโครงงานจะกลายเป็น re-implementation

---

## ตารางเปรียบเทียบ (Comparison Table)

### กลุ่ม A — AI coding assistants เชิงพาณิชย์

| Tool | ทำอะไร | Infra ที่ต้องใช้ | Cross-repo? | LLM-facing? | Link |
|---|---|---|---|---|---|
| **Cursor** (3.2, 24 เม.ย. 2026) | IDE agent; multi-root workspace, per-root indexing, unified `@Codebase` ข้ามทุก root, `@Files` มี root prefix | Cloud index (ของ Cursor เอง) + subscription | **ใช่ เต็มรูปแบบ** | ใช่ | https://cursor.com/changelog/04-24-26 |
| **GitHub Copilot (VS Code chat)** | `@workspace` / workspace context | VS Code + Copilot subscription | **บกพร่อง** — semantic index สร้างไม่ผ่านใน multi-root; agent เห็นแค่ primary cwd/git-root | ใช่ | https://github.com/microsoft/vscode/issues/313181 |
| **Copilot cloud / coding agent** | issue → PR แบบ async | GitHub org + subscription | **ไม่** — sandbox ต่อ 1 repo; เลือก repo ตอน assign ได้เท่านั้น | ใช่ | https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent |
| **Claude Code** | terminal agent, agentic grep/glob ไม่ใช้ pre-built index | ไม่ต้องมี index infra เลย | **ใช่** ผ่าน `--add-dir` / `additionalDirectories` (แต่เป็น file access ไม่ใช่ index-based routing) | ใช่ | https://claudelog.com/faqs/--add-dir/ |
| **Sourcegraph Cody** | AI assistant บน Sourcegraph index | **Enterprise เท่านั้น** (Free/Pro ปิด 23 ก.ค. 2025) | ใช่ | ใช่ | https://sourcegraph.com/blog/changes-to-cody-free-pro-and-enterprise-starter-plans |
| **Sourcegraph Deep Search + MCP server** | cross-repo code intelligence ป้อนให้ agent ภายนอกผ่าน MCP | **Enterprise / Enterprise Starter**; self-hosted ราคาสูง | ใช่ | ใช่ (MCP) | https://sourcegraph.com/mcp · https://sourcegraph.com/docs/deep-search |

### กลุ่ม B — Code search / indexing infrastructure (คลาสสิก)

| Tool | ทำอะไร | Infra ที่ต้องใช้ | Cross-repo? | LLM-facing? | Link |
|---|---|---|---|---|---|
| **Zoekt** | trigram-based code search index (เอนจินเบื้องหลัง Sourcegraph และ Sourcebot) | Go binary + disk index | ใช่ | **ไม่** (ต้องห่อเอง) | อ้างอิงผ่าน Sourcebot ด้านล่าง |
| **Sourcebot** | self-hosted code search หลาย repo/หลาย code host, code navigation, **"Ask Sourcebot"** ตอบคำถามพร้อม inline citations | **Docker Compose ตัวเดียว, localhost:3000** — เบามาก | **ใช่** | **ใช่** (Ask + มี MCP server ให้ agent ภายนอก query) | https://github.com/sourcebot-dev/sourcebot (~3.9k stars) |
| **OpenGrok** | full-text + symbol + history search, Java | Tomcat/servlet container + ctags | ใช่ (หลาย project) | ไม่ | https://en.wikipedia.org/wiki/Google_Kythe (บริบทเปรียบเทียบ) |
| **Hound** | Go backend + React frontend, index ต่อ repo | Go binary + config | ใช่ | ไม่ | https://www.libhunt.com/r/hound |
| **livegrep** | interactive regex search ระดับ GB, in-memory index | stateless server + codesearch backend | ใช่ | ไม่ | https://sourceforge.net/software/product/livegrep/alternatives |
| **Google Kythe** | semantic cross-reference index หลายภาษา (SCIP/kzip) | หนัก — ต้อง build integration | ใช่ (โดยออกแบบ) | ไม่ | https://kythe.io/docs/kythe-overview.html — **หมายเหตุ: ทีม Kythe ที่ US ถูก layoff เม.ย. 2024, maintenance ซบเซา** |
| **Meta Glean** | fact database ของโครงสร้างโค้ด, query ได้ | หนัก (internal-scale) | ใช่ | ไม่ | https://en.wikipedia.org/wiki/Google_Kythe (เปรียบเทียบ) |

### กลุ่ม C — MCP servers / open-source tools ที่ทำ cross-repo code context ให้ LLM (**กลุ่มที่อันตรายที่สุดต่อ proposal**)

| Tool | ทำอะไร | Infra ที่ต้องใช้ | Cross-repo? | LLM-facing? | Link |
|---|---|---|---|---|---|
| **flupkede/codesearch** ⚠️ | **Multi-repo semantic code search MCP (Rust)**: hybrid vector + BM25, tree-sitter AST chunking, **multi-repo serve mode**, **fan-out queries ข้าม repository groups + cross-repo RRF ranking**, virtual `all` group, federation ไปยัง peer instances | **ไม่ต้องมี GPU / Docker / vector DB ภายนอก**; embedded embeddings (fastembed) + LMDB; ใช้ดิสก์หลักร้อย MB; **offline 100%** | **ใช่ + มี routing จริง** | **ใช่** (6 MCP tools: `search`, `find`, `explore`, `get_chunk`, `find_impact`, `status`) | https://github.com/flupkede/codesearch (Apache-2.0, ~73 stars) |
| **octocode / octocode-mcp** | code research platform สำหรับ agent: local + GitHub search, ripgrep + AST structural search, LSP semantics, ตาม import chain เชื่อม repo ที่เกี่ยวข้อง, Rust engine ย่อ/skeletonize โค้ดให้ประหยัด token | Node/CLI + GitHub token | **ใช่ โดยตรง** ("best tool for cross-repository research") | ใช่ (MCP + CLI) | https://github.com/bgauryy/octocode · https://news.ycombinator.com/item?id=45796836 |
| **Zilliz claude-context** | semantic code search MCP, hybrid BM25 + dense, ลด token ~40% | Node 20–22 + **Milvus/Zilliz Cloud** + embedding API key | ใช่ (index หลาย codebase) | ใช่ | https://github.com/zilliztech/claude-context (~11.9k stars) |
| **Context-Engine (m1rl0k)** | MCP retrieval stack: dense + lexical + reranker, ReFRAG micro-chunking, มีเอกสาร **MULTI_REPO_COLLECTIONS** | หนักกว่า (vector store) | ใช่ | ใช่ | https://github.com/m1rl0k/Context-Engine/blob/test/docs/MULTI_REPO_COLLECTIONS.md |
| **repo-lens-mcp (YohannHommet)** | ลงทะเบียนหลาย local repo ใน `repolens.yaml` แล้ว AST structural search ข้ามทุก repo (JS/TS/PHP) | `npx` + ast-grep เท่านั้น — เบามาก | **ใช่** | ใช่ | https://github.com/YohannHommet/repo-lens-mcp (AGPL-3.0, **0 stars — เป็นของเล่น**) |
| **code-index-mcp** | index/search/analyze codebase, minimal setup | Python | บางส่วน (หลาย project) | ใช่ | https://github.com/johnhuang316/code-index-mcp |
| **code-search-mcp (LLMTooling)** | codebase search tools, AST 15 ภาษา | Node | บางส่วน | ใช่ | https://github.com/LLMTooling/code-search-mcp |
| **ast-grep-mcp** | structural search ผ่าน MCP | Rust binary | ตามที่ชี้ path | ใช่ | https://github.com/ast-grep/ast-grep-mcp |

### กลุ่ม D — Monorepo/polyrepo tooling ที่เปิด dependency graph ให้ LLM

| Tool | ทำอะไร | Infra ที่ต้องใช้ | Cross-repo? | LLM-facing? | Link |
|---|---|---|---|---|---|
| **Nx MCP server (`nx-mcp`)** | เปิด project graph ทั้งชุดให้ LLM: project relationships, file mappings, runnable tasks, ownership, tech stack, generators, docs; มี minimal mode ลด token | `npx nx-mcp` — เบา | **ใช่ ภายใน workspace** (Nx รองรับหลาย project; ไม่ใช่หลาย git repo แยกกัน) | **ใช่ — official** | https://github.com/nrwl/nx-console/tree/master/apps/nx-mcp · https://www.npmjs.com/package/nx-mcp |
| **Bazel MCP servers** (nacgarg, aaomidi) | build/test/**query dependency graph**/list targets ผ่าน MCP | Bazel workspace | ภายใน workspace | ใช่ (community, ไม่ official) | https://github.com/aaomidi/mcp-bazel · https://mcpservers.org/servers/nacgarg/bazel-mcp-server |
| **Turborepo agent skill** | official skill: task pipeline, caching, filter/affected | Turborepo | ภายใน workspace | ใช่ (skill ไม่ใช่ graph API) | https://mcp.directory/skills/turborepo |
| **Moon** | monorepo task runner | — | — | **ไม่พบหลักฐาน MCP server** | — |
| **monorepo.tools/ai** | หน้าเปรียบเทียบ monorepo tools ในมุม AI | — | — | — | https://monorepo.tools/ai |

**ข้อสังเกตสำคัญของกลุ่ม D:** เครื่องมือเหล่านี้เปิด graph ที่ **แม่นยำระดับ build-system** ให้ LLM ได้แล้ว แต่ทั้งหมดอยู่ภายใน **workspace เดียว** — ไม่มีตัวไหนสร้าง graph ข้าม git repo ที่แยกกันจริง ๆ นี่คือรอยแยกที่ยังเปิดอยู่

### กลุ่ม E — งานวิจัย

| งาน | ทำอะไร | ระดับ | Cross-repo จริงหรือไม่ | Link |
|---|---|---|---|---|
| **RepoCoder** (EMNLP 2023) | iterative retrieval-generation สำหรับ repo-level completion; ดีกว่า in-file baseline >10% | repo-level | ไม่ (ภายใน repo) | https://aclanthology.org/2023.emnlp-main.151/ · https://arxiv.org/abs/2303.12570 |
| **CoCoMIC** | เรียน in-file + cross-file context ร่วมกัน; มี CCFinder ดึง cross-file context ด้วย static analysis | cross-file | ไม่ | https://www.semanticscholar.org/paper/7b00c4deb35471194bbbbd338191165d53167fe5 |
| **RepoHyper** | Search-Expand-Refine บน semantic graph | repo-level | ไม่ | https://arxiv.org/abs/2403.06095 |
| **CodeRAG** (2509.16112) | หา knowledge ที่ relevant *และจำเป็น* สำหรับ repo-level completion | repo-level | ไม่ | https://arxiv.org/abs/2509.16112 |
| **SWE-QA** (2509.14635, ก.ย. 2025 / rev. เม.ย. 2026) | benchmark repo-level QA 576 Q&A จาก 77,100 issues ใน 11 repo; มี **SWE-QA-Agent** เป็น agentic baseline | repo-level QA | **ไม่** — คำถามอยู่ภายใน repo เดียว, หลาย repo = หลาย subject | https://arxiv.org/abs/2509.14635 |
| **CoReQA** (2501.03447) | repo QA benchmark จาก GitHub issues/discussions หลายภาษา | repo-level QA | ไม่ | https://arxiv.org/abs/2501.03447 |
| **StackRepoQA / Beyond Code Snippets** (2603.26567, มี.ค. 2026) | "first multi-project, repository-level QA dataset" 1,318 คำถามจาก 134 Java projects | repo-level QA | **ไม่** — "multi-project" = หลาย subject ไม่ใช่คำถามข้าม repo | https://arxiv.org/abs/2603.26567 |
| **RepoNavigator** (2512.20957) | RL agent ที่มี tool เดียว: jump-to-definition ตาม execution flow | repo-level navigation | ไม่ | https://arxiv.org/abs/2512.20957 |
| **RepoQA-Agent + MCTS** (2510.26287) | RL ขับด้วย Monte-Carlo Tree Search สำหรับ repo QA | repo-level QA | ไม่ | https://arxiv.org/abs/2510.26287 |
| **RepoTransAgent** (2508.17720) | multi-agent (RAG / Context / Refine) สำหรับ repo-aware code translation | repo-level | ไม่ | https://arxiv.org/abs/2508.17720 |
| ⚠️ **NL Summarization for Multi-Repository Bug Localization in Microservices** (2512.05908, 5 ธ.ค. 2025) | **แปลง codebase เป็น hierarchical NL summaries (file → directory → repo) แล้วทำ NL-to-NL search; สองเฟส: (1) route bug report ไปยัง repo ที่เกี่ยวข้อง (2) localize ภายใน repo นั้น** — ทดสอบบนระบบอุตสาหกรรม **46 repos, 1.1M LOC**, Pass@10 = 0.82, MRR = 0.50, ชนะ RAG ของ Copilot และ Cursor | **multi-repo microservice, มี routing layer** | **ใช่ — ตรงกับ proposal ของนักศึกษาที่สุด** | https://arxiv.org/abs/2512.05908 |
| **PRAXIS** (2512.22113, DSN 2026) | LLM traversal บน **Service Dependency Graph (SDG) ระดับ microservice + Program Dependence Graph ระดับโค้ดต่อ service** เพื่อทำ root-cause analysis; แม่นขึ้นถึง 6.3× | cross-service | **ใช่** (แต่เป้าหมายคือ RCA ไม่ใช่ QA และต้องมี observability data) | https://arxiv.org/abs/2512.22113 |
| **Repository Intelligence Graph (RIG)** (2601.10112, 15 ม.ค. 2026) | สร้าง deterministic architectural map จาก build/test artifacts (CMake File API, CTest) แล้วส่งเป็น JSON ให้ agent; ความแม่น +12.2%, เวลา -53.9% บน 8 repos | repo/build-level | ไม่ (แต่ deterministic map เป็นแนวคิดที่นำมาต่อยอดข้าม repo ได้) | https://arxiv.org/abs/2601.10112 |
| **MIRAGE** (2604.04806, ICSE 2026) | online LLM simulation สำหรับ microservice dependency testing | cross-service | ใช่ (testing) | https://arxiv.org/abs/2604.04806 |
| **OOPS** (JSS 2026) | สร้าง OpenAPI spec จาก source ด้วย LLM agent workflow; ใช้ **API dependency graph** แก้ปัญหา context length ใน cross-file analysis | cross-file → API contract | บางส่วน | https://www.sciencedirect.com/science/article/abs/pii/S0164121226001470 |
| **SWE-Router** (2607.00053) / **Agent-as-a-Router** (2606.22902) | routing ในงาน agentic SE — แต่เป็น **model routing** (เลือกโมเดล/ต้นทุน) ไม่ใช่ **repo routing** | — | ไม่ | https://arxiv.org/abs/2607.00053 · https://arxiv.org/abs/2606.22902 |

---

## ช่องว่างที่เหลืออยู่จริง

หลังจากตัดสิ่งที่มีคนทำแล้วออกทั้งหมด สิ่งที่ **ยังไม่มีใครทำจริง ๆ** เหลืออยู่ 4 ข้อ เรียงตามความเหมาะสมกับโครงงานปริญญาตรีคนเดียว:

### 1. ไม่มี public benchmark ที่คำถามหนึ่งข้อ *ต้อง* ใช้ ≥2 repositories จึงจะตอบได้ ⭐ (ช่องว่างที่แข็งแรงที่สุด)

นี่คือข้อค้นพบที่สำคัญที่สุดของการสำรวจครั้งนี้ Benchmark repo-QA ทุกตัวที่มีอยู่ — SWE-QA (11 repos), CoReQA, StackRepoQA (134 Java projects), CrossCodeEval — ล้วนใช้หลาย repository เป็น **subject หลายตัว** ไม่ใช่คำถามที่ต้องข้าม repo คำว่า "cross-file" และ "multi-project" ในเปเปอร์เหล่านี้ **ไม่เท่ากับ "cross-repo"** ซึ่งเป็นความกำกวมที่นักศึกษาสามารถชี้ให้เห็นได้อย่างมีน้ำหนัก

สิ่งที่ยังไม่มี: dataset ของคำถามแบบ *"เมื่อ order-service ยิง POST /v1/payments แล้ว field `idempotency_key` ถูกตรวจสอบที่ไหน และถ้า schema เปลี่ยนจะพังตรงไหนบ้าง"* — คำถามที่ ground truth กระจายอยู่ใน ≥2 repo และประเมินได้ว่า agent หา repo/file ถูกหรือไม่

งานเดียวที่ใกล้ที่สุดคือ arXiv 2512.05908 แต่ (ก) เป็น **bug localization** ไม่ใช่ QA (ข) ทดสอบบน **ระบบอุตสาหกรรมปิดที่เดียว 46 repos** ไม่ reproducible (ค) ไม่ปล่อย dataset สาธารณะ → **มีที่ว่างชัดเจนสำหรับ open, reproducible cross-repo QA benchmark**

### 2. ไม่มีใครวัด "routing layer" แยกเป็น component เอกเทศ

ทุก tool ในกลุ่ม C มี routing แบบใดแบบหนึ่ง (RRF fan-out, repo groups, import-chain following) แต่**ไม่มีใครรายงานตัวเลข**ว่า:
- repo-selection recall@k เป็นเท่าไร เมื่อมี 5 / 20 / 50 repos
- เมื่อจำนวน repo เพิ่มขึ้น precision ตกเร็วแค่ไหน (scaling curve)
- ภายใต้ token budget คงที่ กลยุทธ์ routing แบบไหนคุ้มที่สุด — BM25 fan-out vs. dense vs. LLM-as-router vs. NL-summary routing (แบบ 2512.05908)

**นี่คือคำถามวิจัยที่วัดได้ ทำคนเดียวได้ และไม่มีคนตอบ** และมันสอดคล้องกับสิ่งที่นักศึกษาสนใจอยู่แล้ว (routing layer)

### 3. ไม่มีใครสร้าง *typed cross-service edges* ระหว่าง repo ที่แยกกัน

ทุก tool ที่สำรวจใช้ความคล้าย (lexical/vector) ในการเชื่อม repo — **ไม่มีตัวไหน**สร้างเส้นเชื่อมเชิงความหมายที่มีชนิด เช่น:
- HTTP client call ใน repo A → route handler ใน repo B (ผ่าน OpenAPI/route table)
- Kafka producer ใน A → consumer ใน B (ผ่านชื่อ topic)
- gRPC/protobuf service definition ที่ใช้ร่วมกัน
- shared DB table / migration ที่ผูกสอง service เข้าด้วยกัน

หลักฐานยืนยัน: GitHub community discussion #189213 (11 มี.ค. 2026) ขอฟีเจอร์นี้ตรง ๆ — "detect API contract mismatches between services" — และ **ยังคงสถานะ Unanswered ไม่มีคำตอบจาก GitHub** ส่วน PRAXIS สร้าง SDG ได้จริงแต่ได้มาจาก **observability traces ของระบบที่รันอยู่** ไม่ใช่จาก static analysis ของ source เพียงอย่างเดียว และ Nx/Bazel ทำ graph แม่นแต่ **ถูกจำกัดอยู่ใน workspace เดียว**

ช่องว่างจริง = **static, build-system-agnostic, cross-repo service graph ที่ derive จาก source อย่างเดียว แล้วเปิดให้ LLM ใช้ผ่าน MCP**

### 4. Answer attribution ข้าม repo

Sourcebot ทำ inline citation ได้ แต่ภายใน index เดียว ยังไม่มีงานที่ประเมินว่า เมื่อคำตอบประกอบจาก 3 repo แล้ว agent ระบุแหล่งที่มาถูกต้องกี่เปอร์เซ็นต์ และตรวจจับ "hallucinated cross-repo link" ได้หรือไม่ — เป็นช่องว่างรอง แต่เสริมข้อ 1 ได้ดี

### สิ่งที่ **ไม่** ใช่ช่องว่าง (ห้ามอ้างเด็ดขาด)

- ❌ "ยังไม่มี lightweight multi-repo indexer" → `flupkede/codesearch`, Sourcebot, repo-lens-mcp
- ❌ "ยังไม่มี MCP server ที่ค้นข้าม repo" → มีอย่างน้อย 8 ตัว
- ❌ "ยังไม่มี routing/fan-out ข้าม repo" → codesearch มี repo groups + cross-repo RRF
- ❌ "Cursor ทำ multi-repo ไม่ได้" → ทำได้ตั้งแต่ พ.ค. 2025
- ❌ "ต้องใช้ enterprise infra เท่านั้นถึงจะ index หลาย repo ได้" → Sourcebot = `docker compose up` ตัวเดียว

### ข้อเสนอการรีเฟรม (สำหรับอาจารย์ใช้แนะนำนักศึกษา)

เปลี่ยนจาก **"สร้าง harness ที่ยังไม่มีใครมี"** เป็น:

> **"Cross-repo routing สำหรับ code QA: benchmark และการศึกษาเชิงประจักษ์"** — สร้าง open benchmark ของคำถามที่ต้องข้าม repo จริง (ข้อ 1), เสริมด้วย typed cross-service edges จาก static analysis (ข้อ 3), แล้ววัดว่า routing strategy แบบต่าง ๆ ทำได้ดีแค่ไหนภายใต้ token budget (ข้อ 2) โดยใช้ `flupkede/codesearch` / Sourcebot / Claude Code `--add-dir` เป็น **baseline ที่ต้องเอาชนะ ไม่ใช่ช่องว่างที่ต้องเติม**

โครงสร้างนี้ปลอดภัยจากกรรมการ เพราะแม้จะมีคนชี้ tool ที่มีอยู่ ก็ยิ่งเสริม motivation ของ benchmark แทนที่จะทำลายมัน

---

## บรรณานุกรมพร้อมคำอธิบาย (Annotated Bibliography)

### เครื่องมือเชิงพาณิชย์ / สถานะปัจจุบัน

1. **Cursor Changelog 3.2 — "Multitask, Worktrees, and Multi-root Workspaces"** (24 เม.ย. 2026)
   https://cursor.com/changelog/04-24-26
   หลักฐานปฐมภูมิที่หักล้างข้ออ้าง "Cursor is single-repo" โดยตรง: *"A single agent session can now target a reusable workspace made of multiple folders... cross-repo changes spanning frontend, backend, and shared libraries."*

2. **eric zakariasson (Cursor) — ประกาศ multi-root workspace** (พ.ค. 2025)
   https://x.com/ericzakariasson/status/1923763194268664315
   *"multi-root workspace support is live in cursor for developers that work across multiple repositories... cursor will index them all"* — ยืนยันว่าฟีเจอร์นี้มีมานานกว่า 1 ปีก่อนวันที่นักศึกษาเขียน proposal

3. **microsoft/vscode Issue #313181 — "Codebase Semantic Index does no longer work on multi-root workspaces"**
   https://github.com/microsoft/vscode/issues/313181
   หลักฐานว่าข้ออ้างเรื่อง Copilot **ยังพอยืนได้บางส่วน**: index แต่ละ repo แยกกันสำเร็จ แต่รวมกันไม่สำเร็จ

4. **microsoft/vscode Issue #271044** — Copilot local indexing ทำ workspaceStorage เสียหายใน multi-root ที่มี >3000 ไฟล์
   https://github.com/microsoft/vscode/issues/271044

5. **microsoft/vscode Issue #264837** — `copilot-instructions.md` ไม่ถูกใช้ใน multi-root workspace
   https://github.com/microsoft/vscode/issues/264837

6. **github/copilot-cli Issue #1826** — ขอรองรับ `.code-workspace` เพื่อให้เห็น folder เพิ่ม; ระบุว่า agent เห็นแค่ primary cwd/git-root
   https://github.com/github/copilot-cli/issues/1826

7. **GitHub community Discussion #189213 — "Feature Request: Cross-Repository Context for Copilot (Web ↔ Microservice)"** (โพสต์ 11 มี.ค. 2026; คอมเมนต์ 9 พ.ค. 2026; แก้ไข มิ.ย. 2026)
   https://github.com/orgs/community/discussions/189213
   **สำคัญมาก** — ยืนยันว่าความต้องการ cross-repo semantic understanding (API contract mismatch) ยังไม่ถูกตอบสนอง สถานะ **Unanswered** และมีคอมเมนต์ว่า *"No mainstream agent (Copilot, Cursor, Claude Code, Codex) handles cross-repo context out of the box"* — ประโยคนี้เป็นความเห็นของสมาชิกชุมชน ไม่ใช่คำแถลงทางการ และขัดกับ changelog ของ Cursor ในข้อ 1 ควรอ้างอย่างระมัดระวัง

8. **GitHub community Discussion #190627 — Copilot Coding Agents กับ poly-repo**
   https://github.com/orgs/community/discussions/190627
   Coding agent ถูก sandbox ที่ 1 repo; แนวปฏิบัติคือแตกงานเป็น 1 repo = 1 task แล้วประสาน PR ตามลำดับ dependency

9. **GitHub community Discussion #175497 — Optimizing GitHub Copilot for Multi-Repository Teams in VS Code**
   https://github.com/orgs/community/discussions/175497

10. **GitHub Docs — About GitHub Copilot cloud agent**
    https://docs.github.com/copilot/concepts/agents/coding-agent/about-coding-agent

11. **GitHub Changelog — "Manage Copilot coding agent repository access via the API"** (24 มี.ค. 2026)
    https://github.blog/changelog/2026-03-24-manage-copilot-coding-agent-repository-access-via-the-api/
    เป็นเรื่อง *สิทธิ์เข้าถึง* repo ไม่ใช่ *บริบทข้าม* repo — อย่าสับสน

12. **VS Code Docs — How Copilot understands your workspace**
    https://code.visualstudio.com/docs/agents/reference/workspace-context

13. **ClaudeLog — Claude Code `--add-dir` Additional Working Directories Guide**
    https://claudelog.com/faqs/--add-dir/
    ยืนยันว่า Claude Code ทำงานหลาย directory/repo ใน session เดียวได้ ทั้งผ่าน flag, slash command, และ `additionalDirectories` ใน settings.json

14. **Sourcegraph Blog — "Changes to Cody Free, Pro, and Enterprise Starter plans"**
    https://sourcegraph.com/blog/changes-to-cody-free-pro-and-enterprise-starter-plans
    Cody Free/Pro ปิดรับสมัคร 25 มิ.ย. 2025 และหยุดให้บริการ 23 ก.ค. 2025; เหลือแต่ Cody Enterprise

15. **Wikipedia — Sourcegraph**
    https://en.wikipedia.org/wiki/Sourcegraph
    source code ย้ายไป private repo ส.ค. 2024 (ไม่ source-available อีกต่อไป); Amc/Amp แยกบริษัท 2 ธ.ค. 2025

16. **Sourcegraph MCP Server**
    https://sourcegraph.com/mcp — cross-repository code intelligence ให้ agent ผ่าน MCP, **Enterprise plans เท่านั้น, สถานะ experimental**

17. **Sourcegraph Deep Search docs**
    https://sourcegraph.com/docs/deep-search — รองรับบน Enterprise Starter และ Enterprise

### เครื่องมือ open-source ที่ทำ cross-repo แบบเบา (หลักฐานหลักที่หักล้างข้ออ้าง)

18. ⚠️ **flupkede/codesearch** — *Multi-repo semantic code search MCP server in Rust*
    https://github.com/flupkede/codesearch
    **หลักฐานชิ้นที่ทำลายข้ออ้าง "no lightweight cross-repo solution exists" มากที่สุด** hybrid vector + BM25, tree-sitter AST chunking, multi-repo serve mode, fan-out ข้าม repository groups พร้อม cross-repo RRF ranking, virtual `all` group, federation ไป peer instances; **ไม่ต้องมี GPU, Docker, หรือ external vector DB**; embedded fastembed + LMDB; offline สมบูรณ์; Apache-2.0; ~73 stars; 6 MCP tools

19. **sourcebot-dev/sourcebot**
    https://github.com/sourcebot-dev/sourcebot
    self-hosted code search + navigation ข้ามหลาย repo/หลาย code host (GitHub, GitLab, Bitbucket, Gitea, Gerrit) บน Zoekt; **"Ask Sourcebot"** ตอบคำถามพร้อม inline citations; deploy ด้วย **Docker Compose ตัวเดียว**; ~3.9k stars — หักล้าง "ต้องมี enterprise infra"

20. **bgauryy/octocode** และ **bgauryy/octocode-mcp**
    https://github.com/bgauryy/octocode · https://github.com/bgauryy/octocode-mcp/blob/main/packages/octocode-mcp/README.md
    ประกาศตัวเป็น *"best tool for cross-repository research"*; ตาม import chain เชื่อม repo; ripgrep + AST + LSP; Rust engine minify/skeletonize เพื่อประหยัด token
    อภิปรายบน Hacker News: https://news.ycombinator.com/item?id=45796836

21. **zilliztech/claude-context**
    https://github.com/zilliztech/claude-context
    semantic code search MCP, hybrid BM25 + dense, ลด token ~40%; **ต้องใช้ Milvus/Zilliz Cloud + embedding API key** (จุดนี้ *ไม่* lightweight — ใช้เป็นตัวอย่างเปรียบเทียบว่า codesearch เบากว่า); ~11.9k stars (ณ 22 มิ.ย. 2026)

22. **YohannHommet/repo-lens-mcp**
    https://github.com/YohannHommet/repo-lens-mcp
    ลงทะเบียนหลาย local repo ผ่าน `repolens.yaml` แล้ว AST structural search ข้ามทุก repo; รัน `npx` + ast-grep; AGPL-3.0; **0 stars** — พิสูจน์ว่าไอเดียนี้ "ใครก็ทำได้" ซึ่งยิ่งอันตรายต่อ novelty claim แต่ก็แสดงว่าคุณภาพ/ความสมบูรณ์ยังต่ำ

23. **m1rl0k/Context-Engine — MULTI_REPO_COLLECTIONS.md**
    https://github.com/m1rl0k/Context-Engine/blob/test/docs/MULTI_REPO_COLLECTIONS.md
    MCP retrieval stack: dense + lexical + reranker, ReFRAG micro-chunking, มี multi-repo collections โดยเฉพาะ

24. **johnhuang316/code-index-mcp**
    https://github.com/johnhuang316/code-index-mcp — index/search/analyze codebase ผ่าน MCP, minimal setup

25. **LLMTooling/code-search-mcp**
    https://github.com/LLMTooling/code-search-mcp — codebase search tools พร้อม AST 15 ภาษา

26. **ast-grep/ast-grep-mcp**
    https://github.com/ast-grep/ast-grep-mcp — structural search ผ่าน MCP

27. **Awesome MCP Servers — MCP Repo Search Server**
    https://mcpservers.org/servers/yohannhommet/mcp-repo-search-server

### Monorepo tooling ที่เปิด graph ให้ LLM

28. **Nx MCP Server (nrwl/nx-console)**
    https://github.com/nrwl/nx-console/tree/master/apps/nx-mcp · https://www.npmjs.com/package/nx-mcp
    **official** MCP server ที่เปิด project graph, file mappings, tasks, ownership, tech stack, generators, docs ให้ LLM; มี minimal mode ลด token — เป็นหลักฐานว่า "expose dependency graph to LLM" มีคนทำแล้วในระดับ production

29. **aaomidi/mcp-bazel** และ **nacgarg/bazel-mcp-server**
    https://github.com/aaomidi/mcp-bazel · https://mcpservers.org/servers/nacgarg/bazel-mcp-server
    community MCP servers ที่เปิด `bazel query` dependency graph ให้ agent (ไม่มี official จากทีม Bazel)

30. **Turborepo agent skill (Vercel)**
    https://mcp.directory/skills/turborepo — official skill ครอบคลุม pipeline, caching, filter/affected

31. **monorepo.tools — Monorepos & AI**
    https://monorepo.tools/ai

### โครงสร้างพื้นฐาน code indexing แบบคลาสสิก

32. **Kythe Overview**
    https://kythe.io/docs/kythe-overview.html — semantic cross-reference index หลายภาษาที่ Google สร้าง

33. **Wikipedia — Google Kythe**
    https://en.wikipedia.org/wiki/Google_Kythe — **สำคัญ:** ทีม Kythe ที่สหรัฐฯ ถูกเลิกจ้างทั้งทีม เม.ย. 2024 การบำรุงรักษาซบเซาลง; หน้านี้ยังกล่าวถึง Glean ของ Meta ในเชิงเปรียบเทียบ

34. **libhunt — Hound**
    https://www.libhunt.com/r/hound — Go backend + React frontend, index ต่อ repo

35. **SourceForge — livegrep alternatives**
    https://sourceforge.net/software/product/livegrep/alternatives — livegrep stateless, in-memory index ระดับ GB

36. **SourceForge / Slashdot — OpenGrok alternatives (2026)**
    https://sourceforge.net/software/product/OpenGrok/alternatives · https://slashdot.org/software/p/OpenGrok/alternatives
    จัดอันดับ Zoekt เป็นทางเลือก free ที่แข็งแรงที่สุด

### งานวิจัย

37. **RepoCoder: Repository-Level Code Completion Through Iterative Retrieval and Generation** (EMNLP 2023)
    https://aclanthology.org/2023.emnlp-main.151/ · https://arxiv.org/abs/2303.12570
    งานอ้างอิงมาตรฐานของ repo-level retrieval; iterative retrieve-generate; ดีกว่า in-file baseline >10% — **ภายใน repo เดียว**

38. **CoCoMIC: Code Completion by Jointly Modeling In-file and Cross-file Context**
    https://www.semanticscholar.org/paper/7b00c4deb35471194bbbbd338191165d53167fe5
    มี CCFinder ใช้ static analysis ดึง cross-file context — เป็นต้นแบบของ "static analysis เป็นตัวช่วย retrieval" ที่นักศึกษาควรยกไปใช้ระดับ cross-repo

39. **RepoHyper: Search-Expand-Refine on Semantic Graphs for Repository-Level Code Completion**
    https://arxiv.org/abs/2403.06095

40. **CodeRAG: Finding Relevant and Necessary Knowledge for Retrieval-Augmented Repository-Level Code Completion** (2509.16112)
    https://arxiv.org/abs/2509.16112

41. **SWE-QA: Can Language Models Answer Repository-level Code Questions?** (ส่ง 18 ก.ย. 2025, แก้ไข 26 เม.ย. 2026)
    https://arxiv.org/abs/2509.14635
    Peng, Shi, Wang, Zhang, Shen, Gu — 576 Q&A จาก 77,100 GitHub issues ใน 11 repo, ครอบคลุม intention understanding, cross-file reasoning, multi-hop dependency; เสนอ **SWE-QA-Agent** เป็น agentic baseline
    **ประเด็นสำคัญ:** เป็น repo-level ไม่ใช่ cross-repo — เป็นทั้ง baseline ที่นักศึกษาต้องเทียบ และเป็นหลักฐานว่า benchmark cross-repo ยังว่าง

42. **CoReQA: Uncovering Potentials of Language Models in Code Repository Question Answering** (2501.03447)
    https://arxiv.org/abs/2501.03447

43. **Beyond Code Snippets: Benchmarking LLMs on Repository-Level Question Answering** (ส่ง 27 มี.ค. 2026, แก้ไข 4 เม.ย. 2026)
    https://arxiv.org/abs/2603.26567
    Alebachew, Leary, Vaishampayan, Brown — เสนอ **StackRepoQA**, *"the first multi-project, repository-level question answering dataset"*, 1,318 คำถามจาก 134 Java projects
    **สำคัญมากสำหรับการวางตำแหน่ง:** เขาเคลม "first multi-project" ในความหมาย *หลาย subject* ไม่ใช่ *คำถามข้าม repo* — นักศึกษาต้องแยกความแตกต่างนี้ให้ชัดในบทที่ 2 มิฉะนั้นกรรมการจะคิดว่าถูกทำไปแล้ว

44. ⚠️ **Natural Language Summarization Enables Multi-Repository Bug Localization by LLMs in Microservice Architectures** (ส่ง 5 ธ.ค. 2025)
    https://arxiv.org/abs/2512.05908
    Amirkia Rafiei Oskooei, S. Selcan Yukcu, Mehmet Cevheri Bozoglan, Mehmet S. Aktas
    **เปเปอร์ที่ใกล้เคียง proposal ของนักศึกษาที่สุด และต้องอ่านให้จบก่อนเขียนบทที่ 1** สถาปัตยกรรมสองเฟส: routing bug report → repo ที่เกี่ยวข้อง แล้วจึง localize ภายใน; แปลง codebase เป็น hierarchical NL summary (file → directory → repo) แล้วทำ NL-to-NL search แทน cross-modal retrieval; ประเมินบนระบบอุตสาหกรรม **46 repos / 1.1M LOC** ได้ Pass@10 = 0.82, MRR = 0.50, ชนะ RAG ของ GitHub Copilot และ Cursor
    **ข้อจำกัดที่เปิดช่องให้นักศึกษา:** เป็น bug localization ไม่ใช่ open-ended QA; dataset เป็นระบบปิดของบริษัท ไม่ reproducible; ไม่มี typed cross-service edges (พึ่ง NL similarity ล้วน)

45. **PRAXIS: Integrating Program Analysis with Observability for Root-Cause Analysis** (ธ.ค. 2025, รับตีพิมพ์ DSN 2026)
    https://arxiv.org/abs/2512.22113
    Shengkun Cui, Rahul Krishna, Saurabh Jha, Ravishankar K. Iyer — LLM traversal บน Service Dependency Graph (ระดับ microservice) + Program Dependence Graph (ระดับโค้ดต่อ service); แม่นขึ้นสูงสุด 6.3× และลด token
    ต่างจากโครงงานตรงที่ต้องมี **observability data จากระบบที่รันจริง** — จุดนี้คือช่องว่างของนักศึกษา: สร้าง SDG จาก **static source เท่านั้น**

46. **Repository Intelligence Graph: Deterministic Architectural Map for LLM Code Assistants** (15 ม.ค. 2026)
    https://arxiv.org/abs/2601.10112
    Tsvi Cherny-Shahar, Amiram Yehudai — SPADE extractor ดึง buildable components/aggregators/runners/tests/external packages จาก CMake File API + CTest แล้วส่ง JSON ให้ agent; ทดสอบ 8 repos: accuracy +12.2%, completion time -53.9%
    หลักฐานเชิงบวกว่า **deterministic structural context ช่วย agent ได้จริง** — ใช้อ้างสนับสนุน routing layer ของนักศึกษาได้

47. **RepoNavigator — One Tool Is Enough: RL of LLM Agents for Repository-Level Code Navigation** (2512.20957)
    https://arxiv.org/abs/2512.20957

48. **Empowering RepoQA-Agent based on RL Driven by Monte-Carlo Tree Search** (2510.26287)
    https://arxiv.org/abs/2510.26287

49. **RepoTransAgent: Multi-Agent LLM Framework for Repository-Aware Code Translation** (2508.17720)
    https://arxiv.org/abs/2508.17720

50. **Repository-level Code Search with Neural Retrieval Methods** (2502.07067)
    https://arxiv.org/abs/2502.07067

51. **A Large-scale Benchmark for Software Engineering Question Answering** (2412.14764)
    https://arxiv.org/abs/2412.14764 — 5 ภาษา, 30 repo ชื่อดัง; ยังเป็น repo-level

52. **MIRAGE: Online LLM Simulation for Microservice Dependency Testing** (ICSE 2026)
    https://arxiv.org/abs/2604.04806

53. **OOPS: Automated generation of REST API specification via LLMs** (Journal of Systems and Software, 2026)
    https://www.sciencedirect.com/science/article/abs/pii/S0164121226001470
    ใช้ **API dependency graph** แก้ปัญหา context length ใน cross-file analysis — เทคนิคที่ยกมาใช้สร้าง cross-repo API edges ได้

54. **SWE-Router: Routing in Multi-turn Agentic Software Engineering Tasks** (2607.00053)
    https://arxiv.org/abs/2607.00053
55. **Agent-as-a-Router: Agentic Model Routing for Coding Tasks** (2606.22902)
    https://arxiv.org/abs/2606.22902
    **หมายเหตุ:** สองงานนี้เป็น *model routing* (เลือกโมเดล/ควบคุมต้นทุน) **ไม่ใช่ repo routing** — อย่าอ้างผิด แต่ควรอ้างเพื่อแยกให้ชัดว่าคำว่า "routing" ในวรรณกรรมหมายถึงคนละเรื่อง ซึ่งช่วยตอกย้ำว่า *repo routing* ยังไม่มีชื่อเรียกและยังไม่มีคนวัด

56. **Retrieval-Conditioned Topology Selection for Multi-Agent Code Generation** (2605.05657)
    https://arxiv.org/abs/2605.05657
    จำแนก query เป็น 5 ประเภท (identifier, exact, conceptual, dependency, structural) แล้ว route ไปยังกลยุทธ์เฉพาะทาง รวมผลด้วย RRF — เป็น query-type routing ที่นักศึกษายืมโครงมาใช้กับ repo routing ได้

---

## หมายเหตุด้านความน่าเชื่อถือของแหล่งข้อมูล

- URL ทุกรายการข้างบนมาจากผลค้นหาจริงหรือหน้าที่เปิดอ่านจริงในการสำรวจครั้งนี้ ไม่มีการแต่งขึ้น
- ข้อมูลที่ **เปิดอ่านหน้าเว็บโดยตรง** (ความน่าเชื่อถือสูงสุด): Cursor changelog 3.2, GitHub discussion #189213, `flupkede/codesearch`, `sourcebot-dev/sourcebot`, `YohannHommet/repo-lens-mcp`, arXiv 2512.05908, 2509.14635, 2603.26567, 2512.22113, 2601.10112
- ข้อมูลที่มาจาก **สรุปผลการค้นหา** (ควรตรวจซ้ำก่อนใส่ในเล่ม): ตัวเลข stars ของ zilliztech/claude-context, รายละเอียด pricing ของ Sourcegraph, สถานะ MCP ของ Sourcebot (หน้า README หลักไม่ได้ระบุ MCP ชัด — พบใน third-party directory เท่านั้น จึงควรยืนยันจาก docs ของ Sourcebot ก่อนอ้าง)
- มีบล็อก SEO คุณภาพต่ำหลายแห่งอ้างว่า "GitHub ประกาศ Copilot Workspace multi-repository เมื่อ 5 พ.ค. 2026 รองรับ 10 repo" — **ไม่พบการยืนยันใน github.blog changelog หรือ docs.github.com จึงไม่นำมาใช้ และนักศึกษาไม่ควรอ้าง**
