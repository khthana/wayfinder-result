---
name: sample-agent-runnable-core
description: มีตัวอย่างแกนกลางการตรวจที่รันได้จริงอยู่ที่ .scratch/ai-grading-agent/sample-agent — เขียนขึ้นเพราะนักศึกษางงว่าเว็บเชื่อมกับ LLM ยังไง
metadata: 
  node_type: memory
  type: project
  originSessionId: 3c4dc320-7386-4edc-a7f0-147d7484b428
  modified: 2026-08-07T12:48:40.713Z
---

7 ส.ค. 69 อาจารย์ขอ "ตัวอย่างส่วนของ Agent" แล้วชี้เพิ่มว่าจุดที่นักศึกษางงจริง ๆ คือ **"เชื่อมกับ LLM ยังไง"** จึงเขียน `.scratch/ai-grading-agent/sample-agent/` ขึ้นมา — ท่อ 4 ขั้น (ถอดตัวตน → ชั้นกฎ → ชั้น LLM → รวม+สืบย้อน) ตัวต่อ LLM สามตัว (mock / vLLM / Anthropic) และ `demo.ts` ที่พิมพ์ JSON ก้อนจริงที่จะยิงออกไปให้เห็นกับตา เทสต์ 19 ตัวผ่านหมด

**Why:** เป็นของนอก deliverable 3 ข้อที่ตกลงกันไว้ ถ้าไม่รู้ที่มาจะนึกว่าเป็นการเริ่มเขียนระบบจริง ซึ่งขัดกับ destination ของ map ที่บอกว่า **ไม่เขียนระบบจริง**

**How to apply:** ใช้เป็นสื่อประกอบตอนอธิบายให้นักศึกษาฟัง หรือแนบท้ายเอกสาร 01/04 · อย่าขยายมันให้กลายเป็นระบบจริง · ข้อจำกัดที่ต้องพูดทุกครั้ง: **ยังไม่เคยยิงเข้าเครื่อง GPU จริง** เพราะยังไม่มีเครื่อง และตัวเลขจาก MockJudge เป็นของปลอม ห้ามเอาไปเขียนในรายงาน · ข้อควรระวังตอนแก้โค้ด: Node `--experimental-strip-types` **ไม่รองรับ TypeScript parameter property** (`constructor(private readonly x: string)`) ต้องประกาศฟิลด์แยก

เกี่ยวกับ [[ai-grading-wayfinder-map]]
