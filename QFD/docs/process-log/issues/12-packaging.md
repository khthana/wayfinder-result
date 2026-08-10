# Packaging: รวมเล่มเป็น Handbook เดียว + export Word

Type: task
Status: resolved
Blocked by: (none)

## Question

รวม deliverable ทั้งหมด (stage model + CP-A..F + Y4 addendum) เป็น **handbook เดียว** ที่แจกนักศึกษา/อาจารย์ได้จริง แล้ว export เป็น Word (.docx) ด้วย pandoc.

ลำดับเนื้อหา: ภาพรวม/stage model ก่อน → CP-A..F ตามลำดับ 3 เทอม → Y4 addendum ปิดท้าย. Research summary (asset 01) เป็นภาคผนวกแยก ไม่รวมเข้าเล่มหลัก (นักศึกษา/อาจารย์ไม่ต้องอ่านทุกวัน).

**Output:** ไฟล์ `handbook.md` รวม + `handbook.docx` (แปลงด้วย pandoc) — ปัจจุบันเก็บเป็น `docs/process-log/superseded/handbook-v1-combined.md/.docx`.

## Answer

**v1 (superseded):** [`handbook.md`](../superseded/handbook-v1-combined.md) + [`handbook.docx`](../superseded/handbook-v1-combined.docx) — รวม 9 บทเป็นเล่มเดียว (นักศึกษา+อาจารย์ปนกัน). Export ด้วย `pandoc handbook.md -o handbook.docx --toc --toc-depth=2`, ใช้ raw-openxml page break block (`\newpage` ไม่ทำงานกับ docx writer ของ pandoc — ทดสอบแล้วพบว่า render เป็น literal text; ใช้ ` ```{=openxml}<w:p><w:r><w:br w:type="page"/></w:r></w:p>``` ` แทน ซึ่งได้ผล).

**v2 (current, หลัง feedback อาจารย์ว่า v1 อ่านยากเหมือนสรุปดื้อ ๆ):** ดู ticket "ปรับปรุงคู่มือให้อ่านง่ายขึ้น + แยกเป็น 2 เล่ม" — แยกเป็น `handbook-student.md/.docx` และ `handbook-advisor.md/.docx`, เขียนใหม่เป็นเรื่องเล่าขยายความ, ลดการยึดติดเทอม 1/2/3.

หมายเหตุเทคนิคที่ยังใช้ได้กับทุก build: pandoc page-break trick (raw openxml block) และ `--toc --toc-depth=2` ยังคงเป็นวิธี export มาตรฐานของ effort นี้.
