# 03 — Rubric Calibration: วัดความตรงกับอาจารย์อย่างไร และปรับอย่างไรเมื่อมีตัวอย่างแค่หลักสิบ-หลักร้อย

Status: draft (เขียนแบบสะสม — อัปเดตเป็นรอบ ๆ)
Ticket: `issues/03-rubric-calibration-techniques.md`
วันที่ค้น: 2026-08-05

---

## 1. มาตรวัดความสอดคล้อง (agreement metrics)

### 1.1 Quadratic Weighted Kappa (QWK)

**สูตร**

$$\kappa_w = 1 - \frac{\sum_{i,j} w_{ij} O_{ij}}{\sum_{i,j} w_{ij} E_{ij}}, \qquad w_{ij} = \frac{(i-j)^2}{(N-1)^2}$$

- $O_{ij}$ = confusion matrix ที่สังเกตจริง (อาจารย์ให้ระดับ $i$, AI ให้ระดับ $j$)
- $E_{ij}$ = matrix ที่คาดหวังถ้าสองผู้ตรวจให้คะแนนอิสระกัน (outer product ของ marginal distributions, normalize ให้ผลรวมเท่ากับ $O$)
- $N$ = จำนวนระดับคะแนน (เช่น rubric 4 ระดับ → $N=4$)

น้ำหนัก quadratic ทำให้ "ผิดไป 2 ระดับ" ถูกลงโทษหนักกว่า "ผิดไป 1 ระดับ" ถึง 4 เท่า — เหมาะกับข้อมูล **ordinal** ซึ่งเป็นธรรมชาติของ rubric level (1/2/3/4)

**เกณฑ์ที่ยอมรับในวรรณกรรมการวัดผลการศึกษา**

มาตรฐานที่ถูกอ้างมากที่สุดคือกรอบของ Williamson, Xi & Breyer (2012), *Educational Measurement: Issues and Practice* 31(1), 2–13 — "A Framework for Evaluation and Use of Automated Scoring" ([Wiley](https://onlinelibrary.wiley.com/doi/abs/10.1111/j.1745-3992.2011.00223.x), [ERIC EJ959585](https://eric.ed.gov/?id=EJ959585), [ETS](https://www.ets.org/research/policy_research_reports/publications/article/2012/jely.html)) กำหนดว่า

| เกณฑ์ | ค่าที่ยอมรับ |
|---|---|
| QWK (AI vs human) | ≥ 0.70 |
| Degradation จาก human–human agreement | AI–human ต้องไม่ต่ำกว่า human–human เกิน 0.10 |
| Standardized mean difference (bias) | \|SMD\| < 0.15 |
| Pearson r | ≥ 0.70 |

เกณฑ์ "QWK ≥ 0.70 และห่างจาก human–human ไม่เกิน 0.10" ถูกยืนยันซ้ำใน Ricker-Pedley et al., *On the Limitations of Human-Computer Agreement in Automated Essay Scoring* ([ERIC ED615602](https://files.eric.ed.gov/fulltext/ED615602.pdf)) และใน ACT, *Establishing Standards of Best Practice in Automated Scoring* (2021) ([ACT PDF](https://www.act.org/content/dam/act/unsecured/documents/R2100-auto-scoring-standards-2021-07.pdf))

**ข้อควรระวังที่สำคัญมากสำหรับโครงงานนี้**

Doewes, Kurdhi & Saxena (EDM 2023), *Evaluating Quadratic Weighted Kappa as the Standard Performance Metric for Automated Essay Scoring* ([EDM proceedings](https://educationaldatamining.org/EDM2023/proceedings/2023.EDM-long-papers.9/index.html), [Zenodo](https://zenodo.org/records/8115784), [ERIC ED630859](https://files.eric.ed.gov/fulltext/ED630859.pdf)) ชี้ว่า QWK **ไม่ควรใช้เดี่ยว ๆ** เพราะ

1. **ไวต่อความกว้างของสเกล** — QWK ของ rubric 0–5 กับ 0–10 บนข้อมูลชุดเดียวกันได้ค่าต่างกัน
2. **Kappa paradox** — เมื่อการกระจายคะแนนเบ้ (prevalence สูงในบางระดับ) observed agreement สูงแต่ kappa ต่ำ ซึ่งเป็นสถานการณ์ปกติของงาน นศ. ที่คะแนนกระจุกอยู่ระดับ 3–4
3. **ไวต่อ marginal distribution** — ถ้า AI ให้คะแนนกระจายแคบกว่าอาจารย์ QWK ตกทันทีแม้ MAE จะดี
4. ไม่รองรับผู้ตรวจหลายคนโดยตรง

→ **ข้อเสนอสำหรับโครงงาน:** รายงาน QWK เป็นตัวหลัก แต่ต้องรายงานคู่กับ MAE, exact/adjacent agreement, และ bias (mean signed difference) เสมอ ไม่เช่นนั้นจะตีความผิด

### 1.2 Cohen's kappa (unweighted) และ linear weighted kappa

$$\kappa = \frac{p_o - p_e}{1 - p_e}$$

- $p_o$ = สัดส่วนที่ตรงกันพอดี, $p_e$ = สัดส่วนที่คาดว่าจะตรงกันโดยบังเอิญ
- **ห้ามใช้ unweighted kappa กับ rubric ordinal** เพราะมันมองว่า "ผิด 1 ระดับ" กับ "ผิด 3 ระดับ" แย่เท่ากัน
- ใช้เมื่อผลลัพธ์เป็น nominal จริง ๆ (เช่น "งานนี้เข้าข่ายต้องให้อาจารย์ทวน: ใช่/ไม่ใช่")

เกณฑ์ตีความคลาสสิกของ Landis & Koch (1977): <0 poor, 0.01–0.20 slight, 0.21–0.40 fair, 0.41–0.60 moderate, 0.61–0.80 substantial, 0.81–1.00 almost perfect — แต่ในบริบทการให้คะแนนที่มีผลต่อเกรดนักศึกษา วรรณกรรม automated scoring ใช้ 0.70 เป็นเส้นตัด ไม่ใช่ 0.41

### 1.3 Krippendorff's alpha

$$\alpha = 1 - \frac{D_o}{D_e}$$

โดย $D_o$ = observed disagreement, $D_e$ = expected disagreement คำนวณจาก coincidence matrix

จุดแข็งที่ทำให้เหมาะกับโครงงานนี้มาก (Krippendorff, *Computing Krippendorff's Alpha-Reliability*, [UPenn Annenberg PDF](https://www.asc.upenn.edu/sites/default/files/2021-03/Computing%20Krippendorff's%20Alpha-Reliability.pdf), [หน้าหลัก](https://www.asc.upenn.edu/krippendorffs-alpha-reliability))

- รองรับ **ผู้ตรวจกี่คนก็ได้** (อาจารย์ 1 คน + TA + AI)
- รองรับ **missing data** — สำคัญมาก เพราะจะไม่มีทางที่อาจารย์ตรวจครบทุกงานทุกเกณฑ์
- เลือก difference function ได้ตามระดับการวัด: nominal / ordinal / interval / ratio → ใช้ **ordinal หรือ interval** สำหรับ rubric

**เกณฑ์ที่ Krippendorff เสนอเอง**
- $\alpha \geq 0.800$ → เชื่อถือได้ ใช้สรุปผลได้
- $0.667 \leq \alpha < 0.800$ → ใช้ได้เฉพาะการสรุปเบื้องต้น (tentative conclusions)
- $\alpha < 0.667$ → ไม่ควรใช้

หมายเหตุ: มีงานที่โต้แย้งว่า rule-of-thumb แบบนี้ควรปรับตามบริบทและ simulation ของสถานการณ์จริง — Reliability in evaluator-based tests ([PMC6245899](https://www.ncbi.nlm.nih.gov/pmc/articles/PMC6245899/))

### 1.4 Intraclass Correlation Coefficient (ICC)

ICC มองคะแนนเป็น **continuous** และแยกความแปรปรวนออกเป็นส่วน ๆ

$$ICC(2,1) = \frac{MS_R - MS_E}{MS_R + (k-1)MS_E + \frac{k}{n}(MS_C - MS_E)}$$

ต้องระบุรูปแบบให้ชัดเสมอ (Koo & Li 2016 เป็นคู่มือมาตรฐาน):
- **ICC(2,1) two-way random, absolute agreement, single rater** ← รูปแบบที่ถูกต้องสำหรับ "AI จะแทนอาจารย์ 1 คนได้ไหม" **ใช้ตัวนี้**
- ICC(3,1) two-way mixed, consistency — วัดแค่ว่าเรียงลำดับตรงกันไหม ไม่สนใจ bias คงที่ → **จะปิดบังปัญหา "AI ให้สูงกว่า 0.7 จุดเสมอ"**
- ICC(2,k) — ค่าความเชื่อถือของ "ค่าเฉลี่ยของผู้ตรวจ k คน" จะสูงกว่าเสมอ อย่านำมาอวด

เกณฑ์ตีความของ Koo & Li (2016), *A Guideline of Selecting and Reporting Intraclass Correlation Coefficients for Reliability Research*, J Chiropr Med 15(2):155-163: <0.50 poor, 0.50–0.75 moderate, 0.75–0.90 good, >0.90 excellent — และเน้นย้ำว่า **ต้องรายงาน 95% CI ไม่ใช่ point estimate** ([PMC4913118](https://pmc.ncbi.nlm.nih.gov/articles/PMC4913118/))

### 1.5 MAE / RMSE

$$MAE = \frac{1}{n}\sum |s^{AI}_i - s^{teacher}_i|, \qquad RMSE = \sqrt{\frac{1}{n}\sum (s^{AI}_i - s^{teacher}_i)^2}$$

- **แปลผลง่ายที่สุดสำหรับอาจารย์** — "โดยเฉลี่ยคลาดจากที่อาจารย์ให้ 0.4 ระดับ" สื่อสารได้ทันที
- ต้องคู่กับ **mean signed difference (bias)** $\frac{1}{n}\sum(s^{AI}_i - s^{teacher}_i)$ — MAE ไม่บอกทิศทาง แต่ bias บอก และ bias คือสิ่งที่แก้ได้ด้วย post-calibration (ดูข้อ 6)
- **ไม่ปรับ chance** → MAE ดีอย่างเดียวไม่พอ ถ้า AI ให้ 3 ทุกงานแล้วงาน 70% ได้ 3 จริง MAE จะดูดีมากแต่ระบบไร้ประโยชน์

### 1.6 Exact / Adjacent agreement

- **Exact agreement (%)** = สัดส่วนที่ให้ระดับตรงกันเป๊ะ
- **Adjacent agreement (%)** = สัดส่วนที่ต่างกันไม่เกิน 1 ระดับ (รวม exact)

เกณฑ์จากวรรณกรรม rubric: exact ≥ **70%** และ adjacent ≥ **90%** ถือว่าใช้ได้ (สรุปจาก Jonsson & Svingby 2007 และคู่มือ operational scoring — ดูข้อ 3)

**ข้อดีที่ประเมินค่าไม่ได้สำหรับโครงงานนี้:** ไม่มี chance correction → **ไม่พังเมื่อ N เล็ก** และไม่พังเมื่อการกระจายคะแนนเบ้ ซึ่ง QWK พัง ควรใช้เป็นตัวรายงานหลักคู่กับ QWK

### 1.7 ตารางสรุปการเลือกใช้

| ตัววัด | ระดับการวัด | ปรับ chance | รองรับ >2 ผู้ตรวจ | จับ bias คงที่ | ทนต่อ N เล็ก |
|---|---|---|---|---|---|
| QWK | ordinal | ✅ | ❌ | บางส่วน | ❌ (CI กว้างมาก) |
| Cohen's κ | nominal | ✅ | ❌ | บางส่วน | ❌ |
| Krippendorff's α | ทุกระดับ | ✅ | ✅ | บางส่วน | ปานกลาง (bootstrap ได้) |
| ICC(2,1) | interval | ✅ | ✅ | ✅ | ❌ |
| MAE | interval | ❌ | ❌ | ❌ (ต้องใช้ signed bias) | ✅ |
| Exact/Adjacent % | ordinal | ❌ | ❌ | ❌ | ✅ |

---

## 2. คำตอบเรื่องขนาดตัวอย่าง N — "มี N ตัวอย่าง วัดอะไรได้ ปรับอะไรได้" ⭐

นี่คือส่วนสำคัญที่สุดของเอกสารนี้ และเป็นส่วนที่โครงงานส่วนใหญ่ทำผิด

### 2.1 วรรณกรรมพูดว่าอย่างไร

- Bujang & Baharum (2017), *Guidelines of the minimum sample size requirements for Kappa agreement test*, Epidemiology, Biostatistics and Public Health 14(2) ([UniMi](https://riviste.unimi.it/index.php/ebph/article/view/17614), [ResearchGate](https://www.researchgate.net/publication/320148141_Guidelines_of_the_minimum_sample_size_requirements_for_Cohen's_Kappa)) — ที่ power 80%, α = 0.05 ขนาดตัวอย่างขั้นต่ำสำหรับ kappa agreement test **ตั้งแต่ 2 ถึง 698 ราย** ขึ้นกับ effect size และผู้เขียน**เตือนเองว่าสูตรอาจให้ค่า N ที่เล็กเกินจริงอย่างสุดโต่ง** พร้อมแนะนำว่า **ถ้า marginal distribution ของสองผู้ตรวจไม่เท่ากัน ให้คูณ N ที่คำนวณได้ด้วย 2** — งานตรวจ rubric ที่คะแนนเบ้เข้าข่ายนี้เสมอ
- สำหรับ **weighted kappa** โดยเฉพาะ วรรณกรรมด้าน bootstrap CI ระบุตรง ๆ ว่า *"the minimal sample sizes required for setting confidence limits around a single value of weighted kappa are inordinately large using standard methods"* และวิธี asymptotic **ทำงานได้แย่เมื่อ n ≤ 30** ต้องใช้ bootstrap (BCa) แทน ([exact bootstrap CI for kappa in small samples](https://www.academia.edu/23459545/An_exact_bootstrap_confidence_interval_for_kappa_in_small_samples), [Confidence Intervals for the Kappa Statistic, Stata Journal](https://journals.sagepub.com/doi/pdf/10.1177/1536867X0400400404))
- Koo & Li (2016) ย้ำว่าต้องรายงาน **95% CI ของ ICC เสมอ** เพราะ point estimate ที่ N เล็กไม่มีความหมาย ([PMC4913118](https://pmc.ncbi.nlm.nih.gov/articles/PMC4913118/))
- Baker et al., ICQE 2022, *A Less Overconservative Method for Reliability Estimation for Cohen's Kappa* ([UPenn Learning Analytics](https://learninganalytics.upenn.edu/ryanbaker/ICQE22_pdf_7730.pdf)) — โต้แย้งว่าข้อกำหนด N ที่สูงมากนั้นตั้งอยู่บนสมมติฐานที่อนุรักษ์เกินไป และ kappa ยังใช้ได้ที่ N ระดับที่ใช้กันจริง **หากยกเกณฑ์ kappa ให้สูงขึ้น** เพื่อชดเชย — เป็นหลักฐานสนับสนุนกลยุทธ์ "ตั้งเป้า QWK สูงกว่าเกณฑ์เพื่อชดเชย N น้อย"

### 2.2 การจำลอง (Monte Carlo) เฉพาะบริบทของโครงงานนี้

วรรณกรรมไม่ได้ให้ตัวเลขสำหรับ rubric 4 ระดับที่คะแนนเบ้แบบงาน นศ. จึงจำลองเอง

**ตั้งสมมติฐาน:** rubric 4 ระดับ, การกระจายคะแนนอาจารย์ = [5%, 20%, 45%, 30%] (เบ้ไปทางสูงตามธรรมชาติของงานส่งในชั้นเรียน), ระบบ AI ที่ตรง "ระดับผ่านเกณฑ์" คือ population QWK = 0.710 (exact agreement 67.1%, adjacent 95%, MAE 0.38 ระดับ)
วิธี: สุ่ม N คู่คะแนน 1,500 รอบ, bootstrap percentile 95% CI (B = 800) — สคริปต์เก็บที่ scratchpad `qwk_ci.py`

**ตาราง A — ความกว้างของ 95% CI ของ QWK ตาม N (true QWK = 0.710)**

| N | ค่าเฉลี่ย QWK ที่วัดได้ | SD ของค่าที่วัดได้ | ช่วงที่ค่าวัดได้จะตกอยู่ 95% | ความกว้าง bootstrap 95% CI | P(วัดได้ ≥ 0.70) |
|---|---|---|---|---|---|
| 20 | 0.685 | 0.141 | 0.36 – 0.91 | **0.51** | 0.51 |
| 30 | 0.691 | 0.114 | 0.43 – 0.88 | **0.44** | 0.51 |
| 50 | 0.703 | 0.085 | 0.52 – 0.85 | **0.33** | 0.54 |
| 75 | 0.704 | 0.066 | 0.56 – 0.82 | **0.27** | 0.55 |
| 100 | 0.706 | 0.060 | 0.58 – 0.81 | **0.23** | 0.56 |
| 150 | 0.707 | 0.047 | 0.61 – 0.79 | **0.19** | 0.58 |
| 200 | 0.709 | 0.042 | 0.62 – 0.79 | **0.16** | 0.60 |
| 400 | 0.710 | 0.029 | 0.65 – 0.77 | **0.12** | 0.65 |

**อ่านตารางนี้อย่างไร**

- ที่ **N = 30** (ตัวเลขที่โครงงานนักศึกษามักใช้) ระบบที่ QWK จริง = 0.71 จะรายงานค่าที่ไหนก็ได้ระหว่าง **0.43 ถึง 0.88** — ตัวเลขเดียวที่รายงานออกมาแทบไม่มีข้อมูล ถ้าได้ 0.83 แล้วดีใจ นั่นคือโชค ถ้าได้ 0.47 แล้วท้อ นั่นก็คือโชคเช่นกัน
- **ไม่มี N ในช่วงหลักร้อยที่ทำให้พูดได้ว่า "QWK ≥ 0.70 อย่างมีนัยสำคัญ" ถ้าค่าจริงคือ 0.71** เพราะขอบล่างของ CI จะอยู่ต่ำกว่า 0.70 เสมอ ต้องมี true QWK สูงกว่านั้นมาก (~0.85+) หรือ N หลักพัน
- สิ่งที่พูดได้จริงที่ **N = 100–200** คือ **"QWK อยู่ในช่วง [0.58, 0.81]"** ซึ่งเพียงพอที่จะสรุปว่า *"ระบบอยู่ในระดับ substantial agreement และไม่ได้แย่กว่าเกณฑ์อย่างชัดเจน"* แต่ไม่พอที่จะเคลมว่าผ่านเกณฑ์ 0.70

**ตาราง B — พลังในการเปรียบเทียบสอง prompt/rubric แบบ paired (งานชุดเดียวกัน)**

| N | Δ ใหญ่ (QWK 0.55 → 0.71) — P(bootstrap CI ของผลต่างไม่คร่อม 0) | Δ เล็ก (QWK 0.67 → 0.71) |
|---|---|---|
| 20 | 0.25 | — |
| 30 | 0.31 | — |
| 50 | 0.43 | 0.10 |
| 75 | 0.57 | — |
| 100 | 0.68 | 0.16 |
| 150 | 0.88 | — |
| 200 | 0.94 | 0.28 |
| 400 | 1.00 | 0.51 |
| 800 | — | 0.78 |

**บทเรียนสำคัญที่สุดจากตาราง B:** การเปรียบเทียบแบบ **paired** (prompt เก่า vs prompt ใหม่ บนงานชุดเดียวกัน) มีพลังสูงกว่าการวัดค่าสัมบูรณ์มาก — ที่ N = 150–200 จับการปรับปรุงขนาดใหญ่ (ΔQWK ≈ 0.16) ได้ที่ power ~0.9 แต่การปรับจูนเล็ก ๆ (ΔQWK ≈ 0.04) **ต้องใช้ N เกิน 800 จึงจะแยกออกจากเสียงรบกวน** → **อย่าอ้างชัยชนะจากการขยับ QWK 0.02–0.05 บนชุดทดสอบหลักสิบ นั่นคือ noise ล้วน ๆ**

**ตาราง C — สิ่งที่ราคาถูกกว่ามาก: การวัด bias เชิงระบบ (mean signed difference)**

จำลองระบบที่ให้คะแนนสูงกว่าอาจารย์เฉลี่ย +0.255 ระดับ (SD ของผลต่างรายชิ้น = 0.62)

| N | ความกว้างครึ่งหนึ่งของ 95% CI ของ bias | P(สรุปได้ว่ามี bias จริง) เมื่อ bias = 0.26 | เมื่อ bias = 0.20 |
|---|---|---|---|
| 15 | ±0.31 | 0.39 | 0.28 |
| 20 | ±0.27 | 0.47 | 0.31 |
| 30 | ±0.22 | 0.65 | 0.46 |
| 50 | ±0.17 | **0.81** | 0.62 |
| 75 | ±0.14 | 0.94 | **0.80** |
| 100 | ±0.12 | 0.99 | 0.90 |
| 200 | ±0.09 | 1.00 | 1.00 |

### 2.3 คำตอบสรุป: "มี N เท่านี้ วัดอะไรได้ ปรับอะไรได้"

| ช่วง N (งานที่อาจารย์ตรวจจริง) | **วัดได้จริง** | **ปรับได้จริง** | **ห้ามอ้าง** |
|---|---|---|---|
| **N = 10–30** | ทิศทางของ bias รายเกณฑ์แบบหยาบ; รายการความผิดพลาดเชิงคุณภาพ (error taxonomy); ความชัด/กำกวมของถ้อยคำ rubric | แก้ถ้อยคำ rubric ที่ AI ตีความผิดชัด ๆ; เลือก anchor examples; แก้ระดับที่ AI ไม่เคยใช้เลย | **QWK ทุกชนิด** (CI กว้าง ±0.25); การเปรียบเทียบ prompt สองแบบด้วยตัวเลข |
| **N = 30–60** | bias รายเกณฑ์อย่างมีนัยสำคัญ (ถ้า bias ≥ 0.25 ระดับ); exact/adjacent agreement ±13pp; MAE ±0.16 | linear/offset bias correction รายเกณฑ์; ปรับ threshold การส่งต่อให้มนุษย์ทวน | QWK เป็นตัวเลขเดี่ยว; การเลือก prompt ที่ดีที่สุดจาก 5 ตัวเลือกด้วย QWK |
| **N = 60–120** | QWK พร้อม CI กว้าง ~0.23–0.27 → พูดได้แค่ "ช่วง"; bias ที่เล็กถึง 0.20 ระดับ; ICC(2,1) พร้อม CI กว้าง | prompt optimization แบบมีระเบียบ (train/dev split); isotonic/linear calibration; anchor selection ที่เลือกแล้วเลือกอีก | ว่าผ่านเกณฑ์ 0.70; ว่า prompt ใหม่ดีกว่าเก่าเมื่อ ΔQWK < 0.10 |
| **N = 150–300** | QWK CI กว้าง ~0.16–0.19; ตรวจจับการปรับปรุงขนาดใหญ่ (ΔQWK ≥ 0.15) แบบ paired ได้ที่ power ~0.9 | ทุกอย่างข้างต้น + เลือกระหว่างสถาปัตยกรรมการตรวจที่ต่างกันจริง ๆ | ว่า ΔQWK 0.05 มีความหมาย |

### 2.4 กลยุทธ์ที่ทำให้ N น้อยมีพลังมากขึ้น (เอาไปใช้ใน ticket 15 ได้เลย)

1. **หน่วยวิเคราะห์คือ (งาน × เกณฑ์) ไม่ใช่งาน** — งาน 40 ชิ้น × rubric 6 เกณฑ์ = 240 การตัดสิน ทำให้วัด bias ต่อเกณฑ์ได้ **แต่ต้องยอมรับว่าคะแนนภายในงานเดียวกันมีสหสัมพันธ์กัน** effective N < 240 → ใช้ **cluster bootstrap ที่ resample ที่ระดับงาน** ไม่ใช่ระดับเซลล์ มิฉะนั้น CI จะแคบเกินจริง
2. **วัดผลต่างแบบ paired เสมอ** — เทียบ prompt เก่า/ใหม่บนงานชุดเดียวกัน แล้ว bootstrap ที่ผลต่าง ตัดความแปรปรวนจาก "งานชุดนี้ยากหรือง่าย" ออกไป (ตาราง B)
3. **ตั้งเป้าที่ CI ไม่ใช่ที่ point estimate** — ประกาศ success criterion ล่วงหน้าเป็น *"ขอบล่างของ 95% CI ของ QWK ≥ 0.60 และ |bias| ทุกเกณฑ์ < 0.25 ระดับ และ adjacent agreement ≥ 90%"* ซึ่งบรรลุได้จริงที่ N ~150 และป้องกันตัวได้ในการสอบ
4. **รายงานเทียบกับเพดานมนุษย์เสมอ** — "AI–teacher QWK = 0.68 [0.58, 0.79] เทียบกับ teacher–teacher QWK = 0.71 [0.60, 0.81]" มีน้ำหนักกว่า "QWK = 0.68" มาก และเป็นวิธีเดียวที่ตัวเลข ~0.68 จะเป็นข่าวดี (ตามกรอบ Williamson et al. 2012)
5. **ใช้ metric ที่ไม่ปรับ chance เป็นตัวหลักในการรายงาน** — exact/adjacent agreement และ MAE มี CI แคบกว่า QWK ที่ N เดียวกัน และสื่อสารกับอาจารย์ได้ตรงกว่า
6. **แยก dev set / test set ตั้งแต่วันแรกและอย่าแตะ test set** — เมื่อ N น้อย การปรับ prompt ซ้ำ ๆ บนข้อมูลชุดเดียวคือการ overfit ที่รับประกันได้ ควรแบ่งเช่น 60% dev / 40% held-out และเปิด test set แค่ครั้งเดียวตอนจบ
7. **ให้อาจารย์ตรวจซ้ำ (double-scoring) บางส่วน** ~20–30 ชิ้น เพื่อได้ตัวเลขเพดานมนุษย์ ราคานี้ถูกและให้ผลตอบแทนทางวิชาการสูงสุด

---

## 3. Inter-rater reliability ของมนุษย์ (เพดานบนของ AI)

### 3.1 ตัวเลขหลักจาก meta-review

**Jonsson & Svingby (2007), "The use of scoring rubrics: Reliability, validity and educational consequences", *Educational Research Review* 2(2), 130–144** — review 75 งานวิจัยเรื่อง rubric ใน performance assessment ([ScienceDirect](https://www.sciencedirect.com/science/article/abs/pii/S1747938X07000188), [ERIC EJ796733](https://eric.ed.gov/?id=EJ796733), [Semantic Scholar](https://www.semanticscholar.org/paper/The-use-of-scoring-rubrics:-Reliability,-validity,-Jonsson-Svingby/1d6ddb6e4951eb46e1af75aa9cfab807231d7808))

ข้อค้นพบที่เกี่ยวตรงกับโครงงานนี้
- งาน **open-ended / performance-based ที่ประเมินสมรรถนะซับซ้อน** (ซึ่งงานออกแบบ UX คือกรณีนี้เป๊ะ) มี **exact rater consensus เพียง 55–75%** แม้ใช้ rubric
- เกณฑ์ที่ถือว่าเชื่อถือได้: exact ≥ 70%, adjacent ≥ 90%
- ปัจจัยที่ช่วยเพิ่ม agreement เรียงตามผล:
  1. **Benchmarks / anchor papers ช่วยได้มากที่สุด** ← เป็นหลักฐานตรงว่ากลยุทธ์ few-shot anchor ในข้อ 4 ถูกทาง
  2. Rater training ช่วย แต่ **ไม่มีทางขจัดความต่างได้หมด**
  3. Rubric เฉพาะเจาะจงกับโจทย์ (topic-specific) ให้คะแนนที่ generalizable กว่า generic rubric
  4. สเกลที่มีระดับน้อยกว่าให้ consensus สูงกว่า — ครูให้คะแนนแม่นบนสเกล 2 ระดับมากกว่า 4 ระดับ

### 3.2 ตัวเลขจากงาน portfolio / งานเชิงออกแบบ

- Portfolio assessment ในหลักสูตรแพทย์ (Ewha Womans University): **ICC ของผู้ตรวจทุกคน = 0.382**; หลังคัดผู้ตรวจที่แย่ออกขึ้นเป็น **0.442** — สหสัมพันธ์กับ gold-standard rater แต่ละคนกระจายตั้งแต่ **0.42 ถึง 0.89** ([PMC11717432](https://pmc.ncbi.nlm.nih.gov/articles/PMC11717432/), [JEEHP](https://www.jeehp.org/journal/view.php?number=543))
- Inter-rater reliability ของ portfolio ในสัตวแพทยศาสตร์ ([PubMed 30920333](https://pubmed.ncbi.nlm.nih.gov/30920333/))
- เมื่อผู้ตรวจได้รับการอบรมและใช้เกณฑ์ที่ระบุชัดผ่านกระบวนการเป็นระบบ ICC ขึ้นไปถึง ≥ 0.81 ได้ — แปลว่า **ปัญหาไม่ใช่ที่มนุษย์ แต่อยู่ที่ว่า rubric ชัดพอและมี anchor หรือเปล่า**

### 3.3 ความหมายเชิงออกแบบสำหรับโครงงานนี้

1. **เพดานบนของ AI ≈ ระดับที่มนุษย์สองคนตรงกัน** ถ้าอาจารย์สองคนตรงกัน exact เพียง ~65% การไล่ให้ AI ทำได้ 90% คือการไล่ตามเสียงรบกวน (fitting noise) ไม่ใช่ความแม่นยำ
2. ก่อนวัด AI **ต้องวัด human–human ก่อน** ให้อาจารย์ (หรืออาจารย์ + TA) ตรวจงานชุดเดียวกัน 20–30 ชิ้น แล้วรายงาน QWK/ICC ของคู่มนุษย์ ตัวเลขนี้คือ **denominator ของงานวิจัยทั้งเล่ม**
3. ถ้าหาผู้ตรวจคนที่สองไม่ได้เลย ให้ใช้ **intra-rater reliability**: อาจารย์คนเดิมตรวจงานชุดเดิมซ้ำหลังผ่านไป 2–3 สัปดาห์แบบสุ่มลำดับใหม่ — ตัวเลขนี้ก็เป็นเพดานบนที่ใช้อ้างได้และทำได้จริงด้วยคนคนเดียว
4. เป้าหมายที่ป้องกันตัวได้ในเชิงวิชาการ = **"AI–teacher agreement ไม่ต่ำกว่า teacher–teacher (หรือ teacher–self) agreement เกิน 0.10 QWK"** ไม่ใช่ "QWK ≥ 0.70" ลอย ๆ (ตามกรอบ Williamson et al. 2012)

---

## 4. Few-shot / anchor example selection

### 4.1 ผลที่รายงานไว้ — anchor ให้ผลตอบแทนสูงสุดต่อจำนวนตัวอย่างที่ใช้

| งาน | ผล |
|---|---|
| Few-shot vs zero-shot ใน AES ด้วย LLM | QWK **0.306 (zero-shot) → 0.531 (six-shot)** — เพิ่มขึ้น +0.225 จากตัวอย่างเพียง 6 ชิ้น |
| GPT-3 zero-shot บน TOEFL 12,100 เรียง | QWK **0.388**; เมื่อรวมกับ linguistic features เป็น **0.605** ([arXiv:2502.09497](https://arxiv.org/html/2502.09497v1)) |
| GPT-4 few-shot | QWK เฉลี่ย **0.474** — ต่ำกว่าโมเดลที่ post-train มาเฉพาะทางอย่างชัดเจน |
| Zero-shot AES ที่ปรับ prompt | ดู [arXiv:2404.04941](https://arxiv.org/abs/2404.04941) |

- **"Anchor is the key: Toward accessible automated essay scoring with large language model through prompting"**, *Studies in Educational Evaluation* (2026) ([ScienceDirect](https://www.sciencedirect.com/science/article/pii/S1075293526000413)) — ชื่อบทความคือข้อสรุปเอง: anchor examples คือปัจจัยชี้ขาดของการทำ AES ด้วย LLM แบบไม่ต้องเทรน
- Jonsson & Svingby (2007) พบตรงกันในฝั่งมนุษย์ว่า **benchmarks/anchors เป็นปัจจัยที่เพิ่ม inter-rater agreement ได้มากที่สุด** ในบรรดามาตรการทั้งหมด → กลไกเดียวกันนี้ทำงานทั้งกับมนุษย์และ LLM ซึ่งเป็นเหตุผลเชิงทฤษฎีที่หนักแน่นสำหรับการออกแบบระบบ

### 4.2 เลือก anchor อย่างไร

หลักฐานจากวรรณกรรมชี้ไป 3 กลยุทธ์ ซึ่งควรใช้ผสมกัน

1. **ครอบคลุมทุกระดับคะแนน (score-coverage / verdict-balanced)** — Autorubric ([arXiv:2603.00077](https://arxiv.org/html/2603.00077v2)) ใช้ **verdict-balanced sampling** โดยเจตนา คือสุ่มตัวอย่างให้จำนวนของแต่ละ verdict เท่ากัน **เพื่อไม่ให้ judge เรียนรู้ base-rate prior** จาก prompt แล้วเดาตามความถี่ ← นี่คือประเด็นสำคัญมากสำหรับโครงงานนี้ เพราะงาน นศ. เบ้ไปทางคะแนนสูง ถ้าใส่ anchor ตามสัดส่วนจริง AI จะเรียนรู้ที่จะให้คะแนนสูงไว้ก่อน
2. **anchor ที่ใกล้เคียงกับงานที่กำลังตรวจ (retrieval-based / kNN)** — ดึง anchor ที่คล้ายกับงานปัจจุบันมาใส่ใน prompt แบบ dynamic แทนที่จะใช้ชุดเดิมตายตัว
3. **anchor ที่อยู่ตรงเส้นแบ่ง (boundary anchors)** — ตัวอย่าง "งานที่เกือบได้ 3 แต่ได้ 2" กับ "งานที่เพิ่งจะได้ 3" สอนเส้นแบ่งได้ดีกว่าตัวอย่างกลาง ๆ ของแต่ละระดับ นี่คือหลักการเดียวกับ **anchor papers ในการอบรมผู้ตรวจข้อสอบอัตนัยจริง** (range-finding / benchmark papers)

### 4.3 ใส่กี่ตัวอย่าง

- DSPy ตั้งค่า default ที่ `max_bootstrapped_demos=4` และ `max_labeled_demos=4` → รวมสูงสุด 8 ตัวอย่างต่อ prompt ([DSPy MIPROv2 API](https://dspy.ai/api/optimizers/MIPROv2/)) และมีกรณีที่ **3 ตัวอย่างก็พอ**
- ผลจาก AES ข้างต้นแสดงว่า 6 ตัวอย่างให้ผลก้าวกระโดดแล้ว
- **ข้อเสนอสำหรับโครงงาน:** rubric 4 ระดับ × 6 เกณฑ์ → ใช้ anchor **2 ชิ้นต่อระดับต่อเกณฑ์ที่สำคัญ** (ไม่ใช่ทุกเกณฑ์) หรือ **8–12 anchor ต่อหนึ่ง criterion-level prompt** และแยก prompt ต่อเกณฑ์แทนที่จะยัดทุกอย่างใน prompt เดียว
- **anchor คือส่วนที่ใช้ตัวอย่างของอาจารย์คุ้มค่าที่สุดเมื่อ N น้อย** — 12 ชิ้นที่คัดมาดีในฐานะ anchor ให้ผลมากกว่า 12 ชิ้นเดียวกันที่ใช้เป็น training signal เชิงสถิติ

---

## 5. Automatic prompt/rubric optimization

### 5.1 ตระกูลวิธี

| วิธี | กลไก | ข้อมูลที่ต้องใช้ |
|---|---|---|
| **APE** (Automatic Prompt Engineer) | LLM สร้าง prompt candidates แล้ววนปรับโดย paraphrase ตัวที่ดีที่สุด | ชุด train เล็ก + metric |
| **OPRO** (Optimization by PROmpting) | ให้ LLM เห็นประวัติ prompt + คะแนนที่ได้ แล้วให้เสนอ prompt ใหม่ | ต้องมี metric ที่วัดซ้ำได้ ค่าใช้จ่ายสูงเพราะประเมินหลายรอบ |
| **TextGrad** | มอง prompt optimization เป็น gradient descent — LLM หนึ่งวิจารณ์ output อีกตัวแก้ prompt ตามคำวิจารณ์ ("textual gradient") | mini-batch ของตัวอย่าง |
| **DSPy** (BootstrapFewShot / MIPROv2 / GEPA) | เฟรมเวิร์กที่ compile ทั้ง instruction และ demo ร่วมกันด้วย Bayesian optimization | ดูตารางถัดไป |

อ้างอิงภาพรวม: [Automating Tools for Prompt Engineering, CACM](https://cacm.acm.org/news/automating-tools-for-prompt-engineering/), [Is It Time To Treat Prompts As Code? A Multi-Use Case Study For Prompt Optimization Using DSPy (arXiv:2507.03620)](https://arxiv.org/pdf/2507.03620), [metaTextGrad (arXiv:2505.18524)](https://arxiv.org/html/2505.18524v1)

### 5.2 คำแนะนำอย่างเป็นทางการของ DSPy ว่าต้องมีข้อมูลเท่าไร ⭐

จากเอกสารทางการ ([dspy optimizers.md บน GitHub](https://github.com/stanfordnlp/dspy/blob/main/docs/docs/learn/optimization/optimizers.md))

| ปริมาณข้อมูล | optimizer ที่แนะนำ |
|---|---|
| **ตัวอย่างน้อยมาก (~10)** | `BootstrapFewShot` |
| **~50 ตัวอย่างขึ้นไป** | `BootstrapFewShotWithRandomSearch` |
| **200 ตัวอย่างขึ้นไป** (ระบุเหตุผลว่า *"to prevent overfitting"*) | `MIPROv2` |
| ต้องการปรับเฉพาะ instruction (0-shot) | `MIPROv2` ตั้งค่าแบบ 0-shot |
| มีข้อมูลมากและต้องการโมเดลเล็กที่มีประสิทธิภาพ | `BootstrapFinetune` |

**ตรงกับขนาดข้อมูลของโครงงานนี้พอดี** — ที่หลักสิบชิ้นให้ใช้ `BootstrapFewShot` (คือการเลือก demo ไม่ใช่การเขียน instruction ใหม่); การใช้ MIPROv2 ที่ N < 200 คือการ overfit ตามคำเตือนของ DSPy เอง

### 5.3 ปรับถ้อยคำ rubric อัตโนมัติ — ผลที่รายงานไว้ ⭐

**Automated Refinement of Essay Scoring Rubrics for Language Models via Reflect-and-Revise** ([arXiv:2510.09030](https://arxiv.org/html/2510.09030))

- **วิธี:** ให้ LLM อ่านเหตุผลการให้คะแนนของตัวเอง เทียบกับคะแนนมนุษย์ที่ไม่ตรง แล้ว **เขียน rubric ใหม่เอง** วนซ้ำ
- **ข้อมูลที่ใช้: 200 เรียง ต่อ dataset (train 100 / validation 100)** + test set แยก ← ตัวเลขนี้สำคัญมาก: **นี่คือขนาดที่งานวิจัยระดับ arXiv ใช้จริงสำหรับงานนี้ ไม่ใช่หมื่นชิ้น** โครงงานนี้เอื้อมถึงได้
- **ผล ASAP:** baseline (rubric ที่มนุษย์เขียน) QWK 0.26 → rubric ที่ปรับแล้ว **0.46–0.48** (สูงสุด +0.47)
- **ผล TOEFL11:** baseline 0.56–0.58 → **0.58–0.64** (สูงสุด +0.19)
- โมเดล: GPT-4.1, GPT-5-mini, Gemini-2.5-Flash/Pro, Qwen3-Next-80B
- **ข้อค้นพบที่ท้าทายสัญชาตญาณมาก:** เริ่มจาก rubric ที่แทบว่างเปล่า (*"Based on the response's content, rate the response on a scale of 1 to 6"*) แล้วให้ระบบปรับเอง ได้ผลเทียบเท่าหรือดีกว่า rubric ละเอียดที่มนุษย์เขียน → **rubric ที่มนุษย์เขียนดีสำหรับมนุษย์ ไม่จำเป็นต้องดีสำหรับ LLM** และนี่คือเหตุผลทางวิชาการที่หนักแน่นที่สุดที่โครงงานนี้ควรมีฟีเจอร์ "ปรับถ้อยคำ rubric ให้ AI" แยกจาก "rubric ที่แสดงให้นักศึกษาเห็น"

งานที่เกี่ยวข้องอื่น
- **Learnable Assessment Skills for LLM-based Automated Scoring: Rubric Construction via Iterative Optimization** ([arXiv:2605.29274](https://arxiv.org/html/2605.29274v1))
- **Autorubric: A Unified Framework for Rubric-Based LLM Evaluation** ([arXiv:2603.00077](https://arxiv.org/html/2603.00077v1)) — จัดการ position bias, verbosity bias, criterion conflation ซึ่งเป็นปัญหาที่จะเจอตรง ๆ
- **LLM-based Automated Grading with Human-in-the-Loop** ([arXiv:2504.05239](https://arxiv.org/pdf/2504.05239))
- **LLM Essay Scoring Under Holistic and Analytic Rubrics: Prompt Effects and Bias** ([arXiv:2604.00259](https://arxiv.org/html/2604.00259))
- **The application of GPT-4 in grading design university students' assignment and providing feedback** ([arXiv:2409.17698](https://arxiv.org/pdf/2409.17698)) — ใกล้เคียงโดเมนของโครงงานนี้มากที่สุด (งานออกแบบระดับมหาวิทยาลัย)

### 5.4 คำเตือนเรื่อง overfitting ที่ N น้อย

จากตาราง B ในข้อ 2: ที่ N = 100 ความสามารถในการแยกว่า prompt ใหม่ดีกว่าเก่า **เมื่อ ΔQWK = 0.04 มีเพียง 16%** แต่ prompt optimizer จะ "เลือกตัวที่ดีที่สุด" จากผู้สมัครหลายสิบตัวเสมอ → **ค่าที่ดีที่สุดบน dev set ที่ N = 100 จะสูงกว่าค่าจริงอย่างเป็นระบบ (winner's curse)** ดังนั้น
- ต้องมี **held-out test set ที่ไม่เคยถูกใช้เลือก prompt** และรายงานตัวเลขจากชุดนั้นเท่านั้น
- จำกัดจำนวน candidate ที่ลอง (เช่น ≤ 10) เพราะยิ่งลองมาก winner's curse ยิ่งรุนแรง
- ที่ N หลักสิบ ให้ใช้ **prompt optimization เป็นเครื่องมือสร้างสมมติฐาน** (บอกว่าถ้อยคำไหนน่าจะกำกวม) แล้วให้ **อาจารย์เป็นผู้ตัดสินใจขั้นสุดท้าย** ไม่ใช่ให้ตัวเลขตัดสิน

---

## 6. Score post-calibration (แก้ที่ตัวเลข ไม่ใช่ที่ prompt)

### 6.1 คำถามหลักของ ticket: "ถ้า AI ให้สูงกว่าอาจารย์ 0.7 จุดเสมอในเกณฑ์หนึ่ง จะแก้ที่ prompt หรือ post-processing?"

**คำตอบ: แก้ที่ post-processing ก่อน เสมอ** ด้วยเหตุผลเชิงสถิติ 3 ข้อ

1. **bias คงที่คือสิ่งที่วัดได้ถูกที่สุดเมื่อ N น้อย** — จากตาราง C ในข้อ 2: ที่ N = 50 ตรวจจับ bias ขนาด 0.25 ระดับได้ที่ power 0.81 ขณะที่ ΔQWK ขนาดเดียวกันจากการแก้ prompt ต้องใช้ N เป็นร้อย
2. **การแก้ prompt เป็นการแทรกแซงที่ผลข้างเคียงคาดเดาไม่ได้** — เปลี่ยนถ้อยคำเกณฑ์หนึ่งอาจไปเปลี่ยนคะแนนเกณฑ์อื่นด้วย (criterion conflation) ขณะที่การลบ offset เป็นการแปลงแบบ monotone ที่รู้ผลแน่นอน: MAE ดีขึ้นทันที ลำดับไม่เปลี่ยน
3. **ตรวจสอบย้อนกลับได้และอธิบายให้อาจารย์ฟังได้** — "ระบบหักคะแนนเกณฑ์ 'Visual Hierarchy' ลง 0.7 เพราะจากงาน 60 ชิ้นที่อาจารย์ตรวจ ระบบให้สูงกว่าเฉลี่ย 0.7" เป็นประโยคที่อธิบายและ audit ได้ ต่างจาก "เราแก้คำในเกณฑ์"

**แต่**: ถ้าปัญหาไม่ใช่ offset คงที่ แต่เป็น **AI ตีความเกณฑ์ผิดความหมาย** (เช่นเข้าใจว่า "consistency" คือความสวยงาม) การชดเชยเชิงตัวเลขจะซ่อนปัญหาไว้ ต้องดู **error taxonomy เชิงคุณภาพ** ควบคู่เสมอ กฎง่าย ๆ: **ถ้า bias คงที่ทุกระดับคะแนน → post-calibrate; ถ้า bias ต่างกันตามระดับ (เช่น ให้สูงเกินเฉพาะงานอ่อน) → เป็นปัญหาความเข้าใจ ต้องแก้ prompt/anchor**

### 6.2 วิธีการเรียงตามความต้องการข้อมูล

| วิธี | พารามิเตอร์ที่ต้อง fit | N ขั้นต่ำที่พอใช้ | เหมาะเมื่อ |
|---|---|---|---|
| **Constant offset** $\hat{s} = s - b$ | 1 | **~20–30 ต่อเกณฑ์** | bias คงที่ ← เริ่มที่นี่เสมอ |
| **Linear / Platt-style** $\hat{s} = \alpha s + \beta$ | 2 | ~40–60 | AI ให้คะแนนกระจายแคบ/กว้างกว่าอาจารย์ (regression to the mean) — พบบ่อยมากใน LLM judge |
| **Isotonic regression** | non-parametric (monotone piecewise-constant) | **~100+** | ความสัมพันธ์ไม่เป็นเส้นตรงแต่ยัง monotone; **overfit ง่ายมากที่ N น้อย** |
| **Ordinal threshold / cut-point tuning** | K−1 thresholds | ~50–80 | คุมการกระจายของระดับที่ออกให้ตรงกับ marginal ของอาจารย์โดยตรง — ช่วย QWK ได้มากเพราะ QWK ไวต่อ marginal |

หลักฐาน
- Isotonic regression เป็นวิธีที่ให้ผลลด divergence จาก human distribution ได้มากที่สุดในการ calibrate LLM judge และเหมาะเพราะ **ไม่ตั้งสมมติฐานรูปร่างของ mapping แต่บังคับ monotonicity ซึ่งรักษาโครงสร้าง ordinal ไว้** ([LLMs Capture Emotion Labels, Not Emotion Uncertainty, arXiv:2604.27345](https://arxiv.org/pdf/2604.27345))
- **Per-category bias correction** (ประมาณ bias ต่อหมวดจาก training data แล้วลบออกและ re-normalize) **ได้ผลดีกับโมเดลที่มี systematic bias ชัด (GPT −9.5%, Llama −11.3%) แต่ทำให้แย่ลงกับโมเดลที่ bias สมดุลอยู่แล้ว** ([เดียวกัน](https://arxiv.org/pdf/2604.27345)) → **ต้องทดสอบก่อนใช้ ไม่ใช่เปิดไว้ตลอด**
- **Calibrate, Don't Curate: Label-Efficient Estimation from Noisy LLM Judges** ([arXiv:2605.09702](https://arxiv.org/html/2605.09702v1), Li, พ.ค. 2026) — *"With as few as 30 calibration labels, inclusion dominates selection"* คือ **30 ป้ายกำกับจากมนุษย์ก็เพียงพอที่จะทำให้การ calibrate ผู้ตรวจหลายตัวชนะการคัดผู้ตรวจทิ้ง** ← เป็นหลักฐานตรงว่า **การ calibrate มีประโยชน์จริงที่ N ระดับ 30** ซึ่งตรงกับงบข้อมูลของโครงงานนี้ เปรียบเทียบวิธี Platt / temperature / beta calibration / isotonic / Dawid–Skene
- **A Finite-Calibration Regime Map for LLM Judge Panels** ([arXiv:2606.01034](https://arxiv.org/html/2606.01034v1), Zhu & Rao, พ.ค. 2026) — สรุปสำคัญ: **"small-sample calibration (dozens of labels) favors low-dimensional aggregators"** การ calibrate แบบ joint table (พารามิเตอร์เยอะ) จะดีกว่าก็ต่อเมื่อมีป้ายกำกับหลักร้อย (RewardBench: 800 พอ / 50 ไม่พอ; LLMBar: 300 ดีกว่า / 20 sparse เกินไป) → **ที่ N หลักสิบ ให้ใช้ calibration ที่มีพารามิเตอร์น้อยที่สุดเท่าที่จะทำได้** (offset > linear > isotonic)
- **PRECISE: Reducing the Bias of LLM Evaluations Using Prediction-Powered Ranking Estimation** ([arXiv:2601.18777](https://arxiv.org/html/2601.18777)) — เทคนิค prediction-powered inference: ใช้ป้ายกำกับมนุษย์จำนวนน้อยมาแก้ bias ของการประเมินด้วย LLM บนข้อมูลจำนวนมาก **โดยยังคงความถูกต้องทางสถิติ** ← เป็นกรอบที่ตรงกับสถานการณ์ "อาจารย์ตรวจ 50 ชิ้น AI ตรวจ 500 ชิ้น" อย่างยิ่ง ควรพิจารณาสำหรับการรายงานคะแนนรวมของทั้งชั้น

### 6.3 ข้อควรระวังเรื่องการ calibrate

- **fit calibration บน dev set แล้วประเมินบน test set เท่านั้น** — calibration ที่ fit และวัดบนข้อมูลชุดเดียวกันจะดูดีเสมอ
- **calibrate ต่อเกณฑ์ ไม่ใช่ต่อคะแนนรวม** — bias ของแต่ละเกณฑ์ต่างทิศทางกันได้ และการรวมก่อนจะกลบกันเอง
- **ห้าม calibrate ข้ามงานคนละประเภท** (เช่น wireframe assignment กับ usability report) เพราะ bias คนละแบบ
- ต้อง **บันทึกทั้งคะแนนดิบและคะแนนหลัง calibrate** ในฐานข้อมูล เพื่อให้ audit ได้และเพื่อ re-fit ได้เมื่อข้อมูลเพิ่ม

---

## 7. Active learning / sampling — ให้อาจารย์ตรวจงานชิ้นไหน

### 7.1 ตัวเลือกและข้อดี/ข้อเสีย

| กลยุทธ์ | ได้อะไร | เสียอะไร |
|---|---|---|
| **สุ่มล้วน (random)** | เป็นชุดเดียวที่ใช้**ประมาณค่า agreement ได้อย่างไม่มีอคติ** | ไม่มีประสิทธิภาพในการสอนระบบ |
| **Stratified by AI score** | ครอบคลุมทุกระดับ ป้องกันการที่ระดับหายาก (1, 2) ไม่มีตัวอย่างเลย | ต้องถ่วงน้ำหนักกลับเมื่อประมาณค่า agreement รวม |
| **Uncertainty sampling** (AI ไม่มั่นใจ — self-consistency ต่ำ, logprob ต่ำ, หรือรัน 3 ครั้งได้คนละคำตอบ) | สอนระบบได้เร็วที่สุดต่อชิ้น | **ทำให้ประมาณค่า agreement เอนเอียงต่ำเกินจริง** เพราะเลือกเฉพาะเคสยาก |
| **Boundary sampling** (คะแนนใกล้เส้นแบ่งเกรด) | ลดความเสี่ยงที่มีผลจริงต่อนักศึกษาได้ตรงจุดที่สุด | เอนเอียงเช่นกัน |
| **Disagreement sampling** (AI ต่างจาก heuristic/AI ตัวอื่น) | หา failure mode ได้ดี | เอนเอียง |

### 7.2 ข้อสรุปเชิงออกแบบที่สำคัญที่สุด ⭐

**ต้องแยกงบตรวจของอาจารย์ออกเป็นสองก้อนที่ไม่ปนกัน**

1. **Measurement set (สุ่มล้วน หรือ stratified พร้อมน้ำหนัก)** — ~40–60% ของงบ ใช้ตอบว่า "ระบบตรงแค่ไหน" **ห้ามใช้ปรับ prompt เด็ดขาด** และห้ามเลือกด้วย uncertainty
2. **Improvement set (uncertainty / boundary / disagreement)** — ~40–60% ใช้เป็น anchor, ใช้ fit calibration, ใช้หา failure mode

การใช้ชุดที่เลือกด้วย uncertainty มารายงาน QWK คือความผิดพลาดเชิงระเบียบวิธีที่ทำให้ตัวเลขทั้งหมดไร้ความหมาย และเป็นสิ่งที่กรรมการสอบจะจับได้

### 7.3 อ้างอิง

- **A Survey of LLM-based Active Learning** ([ACL 2025](https://aclanthology.org/2025.acl-long.708.pdf), [arXiv:2502.11767](https://arxiv.org/html/2502.11767v1)) — สรุปว่างานสมัยใหม่ผสม selection metric ดั้งเดิมกับความสามารถของ LLM เช่นรวม kNN + perplexity เพื่อสมดุลระหว่าง **uncertainty และ diversity**; และมีแนวทางใช้ LLM สร้างหลายคำตอบแล้วใช้ **ความแปรปรวนระหว่างคำตอบเป็น proxy ของความไม่มั่นใจ** ← วิธีนี้ทำได้ทันทีกับ API เชิงพาณิชย์ที่ไม่คืน logprob และเป็นข้อเสนอที่ควรใส่ใน ticket 15
- **Active Testing of Large Language Models via Approximate Neyman Allocation** ([arXiv:2605.10075](https://arxiv.org/pdf/2605.10075)) — การออกแบบ **stratified sampling แบบลดความแปรปรวน** เพื่อ *ประเมิน* LLM ด้วยงบป้ายกำกับน้อย ← ตรงกับปัญหา "จะประมาณ agreement ของทั้งชั้นด้วยการตรวจแค่ 50 ชิ้นอย่างไรให้ CI แคบที่สุด" คำตอบคือ **Neyman allocation: จัดสรรตัวอย่างไปยัง stratum ที่มีความแปรปรวนสูงมากกว่า** ไม่ใช่แบ่งเท่ากัน
- Uncertainty Sampling เป็นกลยุทธ์ที่ใช้มากที่สุดเพราะเรียบง่ายและมีประสิทธิภาพ ([LLM on a Budget, arXiv:2511.11574](https://arxiv.org/pdf/2511.11574))
- **Optimal Labeler Assignment and Sampling for Active Learning in the Presence of Imperfect Labels** ([arXiv:2512.12870](https://arxiv.org/html/2512.12870)) — ตรงกับความจริงที่ว่า**อาจารย์เองก็ไม่ใช่ oracle** (ดูข้อ 3)

### 7.4 ลำดับการใช้งบตรวจของอาจารย์ที่แนะนำ (สำหรับ ticket 15)

| รอบ | จำนวน | วิธีเลือก | เอาไปทำอะไร |
|---|---|---|---|
| 0 | 20–30 | สุ่ม + **ตรวจซ้ำโดยอาจารย์คนเดิม/คนที่สอง** | หา human ceiling (ตัวหารของทั้งงานวิจัย) |
| 1 | 20–30 | stratified ตามระดับคะแนนที่คาด | สร้าง anchor set + หา error taxonomy |
| 2 | 30–40 | สุ่มล้วน — **ล็อกเป็น held-out test set** | รายงานผลครั้งเดียวตอนจบ ห้ามแตะ |
| 3 | 30–50 | uncertainty + boundary | fit calibration, ปรับถ้อยคำ rubric |
| 4 (ต่อเนื่อง) | ทุกครั้งที่อาจารย์แก้คะแนน AI ในระบบจริง | ฟรี — มาจากการใช้งานปกติ | สะสม N ให้โตขึ้นเรื่อย ๆ ← **ออกแบบ UI ให้เก็บทุกการแก้คะแนนเป็น labeled data อัตโนมัติ คือฟีเจอร์ที่มีมูลค่าสูงสุดของทั้งระบบ** |

---

## 8. เมื่อ rubric เปลี่ยนเวอร์ชัน — score comparability

วรรณกรรมด้าน educational measurement เรื่อง **equating/linking ข้าม rubric version โดยตรงมีน้อยมาก** (equating ส่วนใหญ่พูดถึง test forms ไม่ใช่ rubric) สิ่งที่พบและใช้ได้

- **การเปลี่ยน rubric เพียงผิวเผินก็เปลี่ยนคะแนนได้จริงและมาก** — งานที่ทดสอบ *rubric perturbations* (เปลี่ยนรูปแบบถ้อยคำหรือลำดับเกณฑ์โดยรักษาความหมายเดิม) พบ residual ของคะแนนดิบตั้งแต่ **−26.8 ถึง +11.6 percentage points** ([From Rubrics to Reliable Scores: Evidence-Grounded Text Evaluation with LLM Judges, arXiv:2601.08654](https://arxiv.org/pdf/2601.08654)) → **ห้ามสมมติว่าคะแนนก่อน/หลังแก้ rubric เทียบกันได้ แม้จะ "แก้แค่ถ้อยคำ"**
- แนวทาง equating สมัยใหม่ที่ยกมาปรับใช้ได้: **kernel equating with nonequivalent groups และ covariates** ([Sequential generalized kernel equating, arXiv:2605.28559](https://arxiv.org/pdf/2605.28559))
- **iRULER: Intelligible Rubric-Based User-Defined LLM Evaluation for Revision** ([arXiv:2602.12779](https://arxiv.org/pdf/2602.12779)) — เครื่องมือช่วยผู้ใช้แก้ rubric อย่างเข้าใจผลกระทบ
- มาตรฐานอ้างอิงสูงสุดคือ *Standards for Educational and Psychological Testing* (AERA/APA/NCME)

### ข้อเสนอเชิงปฏิบัติสำหรับระบบ

1. **rubric เป็น immutable versioned object** — ทุกคะแนนในฐานข้อมูลต้องผูกกับ `rubric_version_id` ห้าม edit in place เด็ดขาด (ออกแบบไว้ใน data model ตั้งแต่ต้น)
2. **anchor set ต้องผูกกับเวอร์ชันด้วย** — anchor ที่ให้คะแนนตาม rubric v1 ใช้กับ v2 ไม่ได้โดยอัตโนมัติ
3. **ใช้ common-item / common-person design แบบง่าย** — เก็บ **"bridge set" ประมาณ 15–25 ชิ้นที่ตรวจด้วยทั้งสองเวอร์ชัน** เมื่อเปลี่ยน rubric ทำให้ประมาณ shift เฉลี่ยและ rescale ได้ (ตาราง C บอกว่า N ~20–30 พอสำหรับประมาณ shift แต่ไม่พอสำหรับประมาณการเปลี่ยนรูปทั้งการแจกแจง)
4. **รีเซ็ต calibration parameters เมื่อ rubric เปลี่ยนแบบมีความหมาย** — ไม่ใช่ carry over
5. **แยก "rubric ที่แสดงต่อนักศึกษา" ออกจาก "rubric prompt ที่ AI ใช้"** — ทั้งสองผูกกับเวอร์ชันเดียวกันได้ แต่ปรับถ้อยคำอิสระต่อกัน (สนับสนุนโดยข้อค้นพบใน 5.3 ว่า rubric ที่ดีสำหรับมนุษย์ไม่จำเป็นต้องดีสำหรับ LLM)
