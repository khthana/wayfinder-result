# แผนที่: ให้คำแนะนำหัวข้อ Senior Project — Multi-Repo Codebase Q&A

Labels: wayfinder:map

## Destination

เอกสารสองชิ้นที่อาจารย์ใช้ได้จริง: (1) **เอกสารแนะแนว** — บทวิเคราะห์ว่าข้อเสนอ agentic-RAG ของนักศึกษาตรงไหนแข็ง ตรงไหนเปราะ ทางเลือกอื่นที่ควรพิจารณา พร้อมแหล่งอ้างอิงจริง (paper + GitHub) และคำถามที่ควรถามนักศึกษา และ (2) **โครงร่างโครงงาน** ที่นักศึกษาเอาไปเขียน proposal ต่อได้ — ขอบเขต วิธีการ วิธีวัดผล และ timeline 2 ภาคเรียน

ถึงปลายทางเมื่อ: ตัดสินใจครบว่า contribution คืออะไร วัดผลอย่างไร ขอบเขตแค่ไหน แล้วเขียนสองเอกสารนี้เสร็จ

## Notes

- **โดเมน**: LLM coding agents, code retrieval, context engineering, software engineering research
- **ผู้ทำ**: นักศึกษา ป.ตรี วิศวกรรมคอมพิวเตอร์ **คนเดียว**, 2 ภาคเรียน (~8 เดือน)
- **ข้อเสนอเดิมของนักศึกษา** (ดู `student-proposal.md`): Multi-Repo/Microservice Codebase Q&A Harness, agentic RAG, routing layer เป็น core contribution
- **หัวข้อเปิดกว้าง** — ไม่ล็อกที่ RAG อาจารย์ยินดีให้เปลี่ยนแนวทาง
- **แผนที่นี้เป็นการ "ตัดสินใจ" ไม่ใช่ "เขียนโปรแกรม"** — อาจารย์ระบุชัดว่าไม่ต้องการตั๋วสำหรับ implement โค้ด ตั๋วทุกใบต้องจบที่คำตัดสิน ไม่ใช่ deliverable ที่เป็นซอฟต์แวร์
- **สมมติฐานที่อาจารย์ตั้งไว้และต้องพิสูจน์**: "RAG น่าจะช้า" — ต้องหาหลักฐานว่าจริงแค่ไหน ช้าในมิติไหน (latency / indexing cost / recall)
- **จุดเปราะที่สุดของ proposal ที่เห็นตั้งแต่แรก**: ข้ออ้าง "No lightweight cross-repo solution currently exists"
- **ทรัพยากร**: ภาควิชามี **GPU server** ขอใช้ได้ → รัน embedding model และ LLM-as-judge โลคัลได้ ต้นทุนต่อรอบทดลองต่ำมาก
- **พื้นความรู้นักศึกษา**: เคยเรียน**สถิติเบื้องต้น** แต่**ไม่มีพื้น IR** → ต้องกันเวลาเรียน IR ~1 เดือน, ห้ามมอบงานเขียน static analyzer/code graph DB เอง, แต่ใช้พื้นสถิติเป็นจุดแข็ง (paired test, Cohen κ, CI)
- **เกณฑ์ภาควิชา**: รับ**ทั้ง**งานเชิง novelty และระบบที่ใช้งานได้ → ออกแบบให้ส่งทั้งเครื่องมือและผลการทดลอง
- skills ที่ควรใช้: `/grilling`, `/domain-modeling`, `/research`

## Decisions so far

<!-- index — หนึ่งบรรทัดต่อหนึ่งตั๋วที่ปิดแล้ว -->

- [01 — Landscape retrieval](issues/01-retrieval-landscape-rag-vs-agentic-search.md) — **"RAG ช้า" ถูกครึ่งเดียว**: query latency เร็วกว่า agentic search ด้วยซ้ำ ที่ช้าจริงคือ cold-start indexing (Cursor p99 4.03 ชม.) และปัญหาแท้จริงคือ **recall ไม่ใช่ speed**; hybrid ชนะเชิงประจักษ์; ทิศทางที่เหมาะกับคนเดียวคือ LARGER — graph expansion ใน loop เดิม ไม่ต้องมี vector/graph DB
- [02 — งานที่มีอยู่แล้วด้าน cross-repo](issues/02-cross-repo-prior-work.md) — ข้ออ้าง "ไม่มี lightweight cross-repo solution" **ส่วนใหญ่ไม่จริง**: Cursor/Claude Code ทำ multi-repo ได้แล้ว, มี codesearch MCP + Sourcebot ที่เบาและทำ cross-repo routing อยู่แล้ว; ช่องว่างที่เหลือจริงคือ **ไม่มี benchmark ที่คำถามบังคับให้ใช้ ≥2 repo** และ typed cross-service edges; งานใกล้ที่สุดคือ arXiv 2512.05908
- [04 — routing / context pruning](issues/04-routing-and-context-pruning.md) — premise *Lost in the Middle* **ยังจริงแต่กลไกเปลี่ยน** ควรเปลี่ยนไปอ้าง arXiv:2510.05381 แทน; routing/abstraction-selection/query-routing/compression/memory **ถูกทำไปหมดแล้ว**; ที่เหลือเปิดจริงคือ cross-repo benchmark, index staleness, **cross-repo name collision** (failure mode อันดับ 1 ของงานที่มีอยู่)
- [03 — benchmark และการวัดผล](issues/03-benchmarks-and-evaluation.md) — สนาม repo-level QA เต็มแล้ว (CoReQA/CodeRepoQA/SWE-QA/StackRepoQA/SWE-QA-Pro); **ไม่มี multi-repo QA benchmark ฝั่ง peer-reviewed จริง แต่ Sourcegraph CodeScaleBench มี 136 multi-repo + 47 cross-repo tasks แล้ว** ต้องกรอบข้ออ้างให้แม่น; corpus สังเคราะห์ (Train Ticket/TeaStore/Sock Shop) **ยอมรับได้ มี citation รองรับ** แต่ต้องเสริม open-source polyrepo org จริง — และประโยค "real access is unavailable" ใน proposal ไม่จริง
- [05 — ตัดสิน core contribution](issues/05-decide-core-contribution.md) — **เลือก cross-repo entity disambiguation** (ชื่อชนกันข้าม repo) เป็น contribution ระดับ *component* จึงไม่ต้องเอาชนะ Cursor/Copilot ตัวต่อตัว; ได้ multi-repo QA evaluation set ที่ตรงเป้าเป็นผลพลอยได้; ถ้าผลออกมาว่าไม่ช่วยก็ยังเป็น negative result ที่ส่งได้ — นักศึกษาไม่มีทางส่งงานไม่ได้
- [06 — corpus และการวัดผล](issues/06-decide-evaluation-and-corpus.md) — **Sock Shop เป็น polyrepo จริง** (Train Ticket/Online Boutique เป็น monorepo ใช้ไม่ได้) + org จริงอย่าง kubernetes เป็นชั้นสอง; วัด collision rate ด้วย tree-sitter เป็นผลชิ้นแรก; ~200 คำถาม ~100 ตรวจมือ; ใช้พื้นสถิติทำ paired test + CI + Cohen κ
- [07 — ขอบเขตและ timeline](issues/07-decide-scope-and-timeline.md) — GPU ทำ embedding + judge, API ทำเฉพาะตัว agent; ห้ามเขียน static analyzer/จูน chunking; timeline 8 เดือนพร้อม **go/no-go เดือน 2, 4, 6**; ขอบเขตขั้นต่ำที่ยังจบได้ = collision study + dataset + baseline comparison
- [08 — VoC interviews](issues/08-decide-voc-interviews.md) — **ลดบทบาทเหลือ 1 สัปดาห์ในเดือน 2** ใช้เป็น motivating evidence เท่านั้น เพราะข้ออ้างหลักพิสูจน์จากโค้ดได้โดยตรง; ถามความถี่ของปัญหา ไม่ใช่ถามฟีเจอร์
- [09 — เอกสารแนะแนว + โครงร่าง](issues/09-write-advising-brief-and-outline.md) — เผยแพร่แล้วที่ https://claude.ai/code/artifact/3a9ea5eb-0854-4e70-b618-2f90f9c1e55c — ต้นฉบับคือ brief.html ในโฟลเดอร์นี้

## แผนที่ปิดแล้ว

ถึงปลายทางเมื่อ 1 ก.ย. 2026 — ตั๋วครบ 9 ใบ ทางไปสู่การแนะนำนักศึกษาชัดแล้ว ไม่เหลืออะไรต้องตัดสินก่อนลงมือ

## Not yet specified

- **จะเขียน paper ตีพิมพ์ด้วยไหม** — ถ้าใช่ เกณฑ์ novelty และ evaluation ต้องเข้มขึ้น

## Out of scope

- การเขียนโค้ดจริงของโครงงาน — อาจารย์ระบุชัดว่าไม่ต้องการตั๋วสำหรับ implement
- การเขียน proposal ฉบับสมบูรณ์แทนนักศึกษา — เราให้โครงร่าง นักศึกษาเขียนเอง
