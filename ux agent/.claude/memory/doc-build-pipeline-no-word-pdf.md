---
name: doc-build-pipeline-no-word-pdf
description: วิธีสร้าง .docx/.pdf บนเครื่องนี้ — ห้ามใช้ Word.ExportAsFixedFormat มันค้างไม่มีวันจบ
metadata: 
  node_type: memory
  type: feedback
  originSessionId: 3c4dc320-7386-4edc-a7f0-147d7484b428
  modified: 2026-08-07T12:48:18.779Z
---

บนเครื่องนี้ **`Word.ExportAsFixedFormat` ค้างค้างไปเรื่อย ๆ ไม่มีวันจบ** ต้องไปฆ่า process ทิ้ง ห้ามใช้เส้นทางนี้สร้าง PDF เด็ดขาด (คำสั่ง Open / อัปเดต TOC / Save ของ Word ยังเร็วปกติ ใช้ได้)

**Why:** เสียเวลาไปหลายรอบกับการรอ Word ที่ไม่มีวันตอบ และเกือบทำให้เข้าใจผิดว่าเอกสารสร้างไม่ได้

**How to apply:** เส้นทางที่ใช้ได้จริงคือ `md → (pandoc) → html → (Chrome headless --print-to-pdf) → pdf` ส่วน `.docx` ใช้ pandoc ตรง ๆ ได้ · ระวังสามข้อที่เคยกัด:
- Markdown ที่มี `---` ติดกับ `#` จะกลายเป็น `<hr>` ลอยอยู่หน้าเปล่า ต้องลบ `<hr>` ที่อยู่ก่อน `<h1>` ด้วย regex `<hr\s*/?>\s*(?=<h1)` ก่อนสั่งพิมพ์
- Chrome เขียน PDF ไม่สำเร็จแบบเงียบ ๆ ได้ ต้องเช็ก timestamp/ขนาดไฟล์ทุกครั้ง อย่าเชื่อว่าคำสั่งผ่านเพราะไม่มี error
- Python บนเครื่องนี้ต้องนำหน้าด้วย `PYTHONIOENCODING=utf-8` ไม่งั้นภาษาไทยพัง (cp874) และ heredoc หลายบรรทัดไม่น่าเชื่อถือ — เขียนสคริปต์ลง scratchpad แล้วค่อยรันดีกว่า

เกี่ยวกับ [[ai-grading-wayfinder-map]]
