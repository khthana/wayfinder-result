# 02 — Figma REST API ดึงอะไรมาตรวจได้บ้าง และเข้าถึงไฟล์ของนักศึกษาได้อย่างไร

Type: research
Status: resolved
Blocked by: —

## Question

ถ้าให้นักศึกษาส่ง "ลิงก์ Figma" ระบบจะดึงอะไรมาตรวจได้จริงบ้าง และต้องจัดการเรื่องสิทธิ์อย่างไร?

ต้องการคำตอบที่ครอบคลุม

1. **โครงสร้างข้อมูลที่ดึงได้จาก `GET /v1/files/:key`** — node tree, ประเภท node, ชื่อ layer, absoluteBoundingBox, typography (font family/size/weight/line-height), fills/strokes พร้อมค่าสี, auto-layout, constraints, component และ component instance, styles ที่ตั้งชื่อไว้ — สรุปว่าอะไรวัดเชิงปริมาณได้บ้าง (เช่น จำนวนขนาดตัวอักษรที่ใช้ทั้งไฟล์ → วัดความสม่ำเสมอของ type scale; contrast ระหว่าง fill กับ text → ตรวจ WCAG ได้เองโดยไม่ต้องพึ่ง LLM)
2. **การ render เป็นภาพ** — `GET /v1/images/:key` ใช้อย่างไร, กำหนด scale/format ได้แค่ไหน, render เฉพาะบาง node ได้ไหม, ข้อจำกัดขนาด/จำนวน, URL ที่ได้หมดอายุเมื่อไร
3. **Prototype/flow** — ดึง interaction, transition, flow starting point ได้ไหม (สำคัญมากถ้า rubric มีเกณฑ์เรื่อง user flow)
4. **Comments API** — เขียน comment กลับเข้าไฟล์ นศ. ได้ไหม (ทางเลือกหนึ่งของการส่ง feedback)
5. **Authentication** — personal access token vs OAuth 2.0; ถ้าไฟล์เป็นของ นศ. เอง ระบบของอาจารย์จะเข้าถึงได้ทางไหนบ้าง (นศ. แชร์แบบ view link สาธารณะ? เชิญบัญชีกลางเข้าไฟล์? นศ. ทำ OAuth ให้ระบบ?) — ข้อดีข้อเสียของแต่ละทาง โดยเฉพาะกับบัญชี Figma ฟรี/education
6. **Rate limit และโควตา** — เรียกได้กี่ครั้งต่อนาที ราคา (ถ้ามี) ข้อจำกัดของ education plan
7. **ทางถอย** — ถ้าเข้าถึง API ไม่ได้ ทางเลือกคืออะไร (ให้ นศ. export PDF/PNG แนบมาด้วย? ใช้ `.fig` file? plugin ฝั่ง Figma?)

เน้นเอกสารทางการของ Figma (developers.figma.com) เป็นหลัก

## หมายเหตุบริบท

ผลของ ticket นี้กำหนดว่า AI จะได้เห็นชิ้นงานในรูปแบบไหน (ภาพอย่างเดียว vs ภาพ+โครงสร้าง) ซึ่งเปลี่ยนทั้งความสามารถของ agent (ticket 09) และสถาปัตยกรรม (ticket 17)

## Answer

ผลการค้นอยู่ที่ [`research/02-figma-api.md`](../research/02-figma-api.md) — 825 บรรทัด 16 หัวข้อ ละเอียดที่สุดในชุด อ้างอิงจาก `developers.figma.com/docs/rest-api/` (URL เดิม `figma.com/developers/api` ถูก redirect แล้ว)

หัวข้อสำคัญ: โครงสร้าง `GET /v1/files/:key` · node properties ที่วัดเชิงปริมาณได้ · `GET /v1/images/:key` สำหรับ render ภาพ · authentication (PAT vs OAuth) · **rate limits — จุดที่ต้องระวังที่สุด** · type definitions จาก official OpenAPI spec · prototype/flow ดึงได้จริง · **Comments API ส่ง feedback กลับเข้าไฟล์นักศึกษาได้** · ข้อจำกัดตาม plan/seat/education · **ตัวเลือกการเข้าถึงไฟล์ของนักศึกษาพร้อมข้อดีข้อเสีย** · ⭐ **สัญญาณคุณภาพ UX/UI ที่คำนวณได้แบบ deterministic โดยไม่ต้องใช้ LLM** (หัวข้อ 14 — เป็นวัตถุดิบตรงของ ticket 09 ชั้นที่ 1) · ทางถอยถ้าเข้าถึง REST API ไม่ได้ · **รายการสิ่งที่ยัง UNVERIFIED ที่ต้อง spike ก่อน commit สถาปัตยกรรม** (หัวข้อ 16 — อ่านก่อนทำ ticket 17)

ปลดล็อก ticket 09 และ 17
