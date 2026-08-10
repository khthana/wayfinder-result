# ผลิต: แพ็กเกจ CP-C (Design: Functional Decomposition + Architecture, S6–S7) ครบ 4 ชิ้น

Type: task
Status: resolved
Blocked by: 05

## Question

ผลิตแพ็กเกจ 4 ชิ้นของ **CP-C · Design** (S6 Functional Decomposition → `F` · S7 Architecture/module mapping → `C`) mirror gold reference CP-B — เปลี่ยน criteria list ให้ตรงขั้น, คงโครง 4 projection, เดินตัวอย่าง EEG SleepInsight ต่อจาก QR ของ CP-B.

แกน traceability ของขั้นนี้: `QR/TC → F → C` (ไม่มี F ลอย = scope creep; ทุก QR ต้องมี F รับผิดชอบ). ใช้ Pahl&Beitz function tree, Parnas information-hiding, และ FR/NFR (NFR = constraint ไม่ใช่ leaf).

**Output:** ไฟล์แพ็กเกจ CP-C ครบ 4 ชิ้น (linked asset).

## Answer

Asset: [`artifacts/07-package-cp-c-design.md`](../artifacts/07-package-cp-c-design.md) — CP-C ครบ 4 ชิ้น

criteria = **FD1–FD7** (แกน FD2/FD3 = F↔QR ไม่มี orphan; FD5 = F↔C; FD6 = Parnas information-hiding ซ่อนโมเดล AI). ตัวอย่าง EEG: overall function → function tree F-01..F-06 (F-04 จำแนก มี sub + ซ่อนโมเดลใน F-04.2) โยงหา QR/TC, NFR latency = constraint บน F-04.2, จับลง C-01..C-05 พร้อม secret ต่อ module. Rubric 100 คะแนน (traceability กลางเส้น 45% + information-hiding/decomposition 45%). Self-check checklist + LLM prompt. worked example EEG ต่อเนื่อง CR→TC→QR→F→C.
