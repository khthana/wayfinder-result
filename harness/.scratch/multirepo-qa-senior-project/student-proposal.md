# ข้อเสนอเดิมของนักศึกษา (ตามที่อาจารย์ส่งมา)

**Multi-Repo/Microservice Codebase Q&A Harness** (current leading candidate)

- **Field**: Software engineering tooling, agentic RAG
- **Idea**: Agent that answers questions spanning multiple repositories/microservices, with a routing layer that decides which repos/files/abstraction-levels are relevant per query.
- **Selection problem**: Relevance shifts by query and by follow-up turn across many repos; naive context-stuffing degrades as repo count grows (per Lost in the Middle / long-context literature). The routing/pruning policy is the core contribution.
- **Differentiator vs existing tools**: Cursor/Copilot are single-repo; Sourcegraph+Cody need enterprise indexing infra. No lightweight cross-repo solution currently exists.
- **VoC access**: Medium — need 3-5 interviews with intern-track classmates (full-stack/backend/DevOps), not yet conducted.
- **Risk**: Benchmark requires a synthetic microservices corpus (sock-shop/TeaStore) since real access is unavailable — need to confirm this is acceptable evidence quality.
