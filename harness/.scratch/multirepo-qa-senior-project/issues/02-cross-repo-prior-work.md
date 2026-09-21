# 02 — งานที่มีอยู่แล้วด้าน cross-repo / multi-repo code understanding

Type: research
Status: resolved

## Question

ข้ออ้างของนักศึกษาว่า **"No lightweight cross-repo solution currently exists"** จริงหรือไม่?

นี่คือจุดเปราะที่สุดของ proposal — ถ้ากรรมการยกตัวอย่างเครื่องมือที่มีอยู่แล้วมาโต้ได้ contribution จะพัง ต้องสำรวจให้ครบ:

- Sourcegraph (Zoekt, SCIP, Cody), Glean (Meta), Kythe (Google), OpenGrok
- MCP servers / เครื่องมือ open-source ที่ทำ code search ข้าม repo
- monorepo tooling ที่มี dependency graph ข้าม package (Nx, Bazel, Turborepo)
- งานวิจัยเรื่อง repository-level / cross-file code completion และ code QA
- เครื่องมือที่ index หลาย repo แบบเบาๆ ได้บนเครื่องเดียว

**ผลลัพธ์ที่ต้องการ**: ตารางเปรียบเทียบว่าใครทำอะไรได้แล้ว, ต้องใช้ infra แค่ไหน, และ **ช่องว่างที่เหลืออยู่จริง** คืออะไร (ถ้ายังเหลือ) — เขียนให้ตรงไปตรงมา ถ้าช่องว่างที่นักศึกษาอ้างไม่มีจริง ให้บอกตรงๆ

## Answer

**ข้ออ้าง "No lightweight cross-repo solution currently exists" — ส่วนใหญ่ไม่จริง** รายงานเต็มที่ `research/02-cross-repo-prior-work.md`

- "Cursor เป็น single-repo" ผิดชัดเจน — multi-root workspace ตั้งแต่ ~พ.ค. 2025 และเป็น agent target เต็มตัวใน Cursor 3.2
- Claude Code ทำ multi-repo ได้ผ่าน `--add-dir`
- มีแต่ **Copilot** ที่ข้ออ้างพอฟังขึ้น (semantic index พังใน multi-root workspace)
- "Sourcegraph/Cody ต้องใช้ enterprise infra" — จริง และแรงกว่าที่นักศึกษาเขียนด้วยซ้ำ (Cody Free/Pro ปิด ก.ค. 2025) แต่ไม่ได้แปลว่าเหลือช่องว่าง
- **ตัวอย่างค้านที่อันตรายที่สุด**: `github.com/flupkede/codesearch` — multi-repo semantic code search MCP server มี repo groups + cross-repo RRF fan-out ranking คือ "routing layer" ที่นักศึกษาจะทำพอดี รันออฟไลน์ ไม่ต้องใช้ GPU
- ของเบาอื่นๆ ที่มีอยู่แล้ว: Sourcebot (~3.9k stars, `docker compose up` เดียว), octocode-mcp, repo-lens-mcp, zilliztech/claude-context
- Nx มี MCP server ทางการที่เปิด project graph ให้ LLM แล้ว

**ช่องว่างที่ยังเหลือจริง (เรียงตามความแข็ง)**
1. **ยังไม่มี benchmark สาธารณะที่คำถามหนึ่งข้อ *บังคับ* ให้ต้องใช้ ≥2 repo** — SWE-QA, CoReQA, StackRepoQA ใช้หลาย repo เป็น *subject แยกกัน* ("multi-project" ≠ "cross-repo")
2. ไม่มีใครวัด repo-selection recall/precision เป็น component แยกภายใต้ token budget
3. ไม่มีใครสร้าง *typed* static cross-service edges (HTTP→handler, Kafka topic, protobuf) ข้าม repo แยก
4. cross-repo answer attribution ยังไม่ถูกวัด

**งานที่ใกล้ที่สุดและต้องอ่านก่อนเพื่อน**: arXiv 2512.05908 (ธ.ค. 2025) — two-phase repo routing บน 46 repo / 1.1M LOC ชนะ Copilot และ Cursor RAG แทบเป็นข้อเสนอเดียวกัน แต่ทำ bug localization บน dataset ปิด

**ข้อเสนอปรับกรอบ**: ทิ้งข้ออ้าง "ไม่มีเครื่องมือ" แล้วเปลี่ยนเป็น open cross-repo QA benchmark + typed cross-service edges + routing-strategy study โดยใช้ codesearch/Sourcebot/Claude Code เป็น baseline ที่ต้องเอาชนะ

### ตรวจสอบซ้ำแล้ว (2026-09-01)

**arXiv 2512.05908 — ยืนยันว่ามีจริง** ชื่อเต็ม: *Natural Language Summarization Enables Multi-Repository Bug Localization by LLMs in Microservice Architectures* (Rafiei Oskooei, Yukcu, Bozoglan, Aktas; 5 ธ.ค. 2025)
Abstract ระบุตรงๆ ว่า *"the need to first identify the correct repository"* และใช้ **two-phase search: routing → top-down localization** บน DNext 46 repo / 1.1M LOC ได้ Pass@10 0.82, MRR 0.50 ชนะ Copilot และ Cursor
ประเด็นสำคัญที่แถมมา: วิธีของเขาคือ **NL-to-NL search แทน cross-modal retrieval** — คือเลี่ยง embedding RAG โดยตั้งใจ ซึ่งสอดคล้องกับข้อกังขาของอาจารย์เรื่อง RAG แต่ด้วยเหตุผลเรื่อง *semantic gap* ไม่ใช่เรื่องความเร็ว

**flupkede/codesearch — ยืนยันว่ามีจริง** Rust, 73 stars, multi-repo semantic code search MCP server, repo groups, RRF (vector + BM25), tree-sitter AST chunking 17+ ภาษา, รันโลคัลไม่ต้อง GPU, federation ข้าม instance, SCIP helper สำหรับ C#
