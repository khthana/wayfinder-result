# 04 — RL / Preference Learning บนข้อมูลระดับหลักร้อย: ทำอะไรได้จริง

Status: **complete** — ครบทั้ง 7 หัวข้อ (เขียนแบบ incremental กันงานหาย)
วันที่: 2026-08-05 · ปิดไฟล์ 2026-08-06

---

## บทสรุปผู้บริหาร (verdict ก่อน)

**คำตอบสั้นที่สุด: ทำ RL ได้ และควรทำ แต่ไม่ใช่ RL แบบที่คนส่วนใหญ่นึกถึง — และห้ามให้มันเป็นทางวิกฤตของโครงงาน**

**1. RLHF แบบเต็มรูป (reward model + PPO) และ GRPO เป็นไปไม่ได้ — ปิดประตูตั้งแต่วันนี้**
ไม่ใช่เพราะทีมไม่เก่ง แต่เพราะขนาดข้อมูลห่างกันสองอันดับ: dataset มาตรฐานของ reward model อยู่ที่ **หลักหมื่นถึงหกหมื่นคู่เปรียบเทียบ** ส่วนงานวิจัยที่ใกล้โจทย์เราที่สุด (RLAES 2026) ใช้ **essay 12,978 ชิ้น + GPU A100/H20 แปดใบ** ขณะที่โครงงานนี้มีคู่คะแนนที่ **สะอาดพอจะใช้เป็น ground truth ได้จริงราว 330–1,000 คู่ต่อเทอม** (§2, §6, §7.2)

**2. ตัวเลขที่หลอกตาที่สุดคือจำนวนแถวข้อมูล — ต้องนับให้ถูกตั้งแต่วันแรก**
นับดิบได้ 1,600–3,360 คู่คะแนนระดับเกณฑ์ต่อเทอม ฟังดูเหมือน "หลักพัน" แต่หน่วยที่**อิสระทางสถิติจริง**คือ **นักศึกษา 40–60 คน** เพราะเกณฑ์ 5–7 ข้อของชิ้นงานเดียวถูกตัดสินจากไฟล์ Figma ไฟล์เดียวกัน ผิดทีก็ผิดพร้อมกันทั้งแถว กำลังทางสถิติของโครงงานนี้จึงถูกกำหนดโดยเลข 40–60 ไม่ใช่ 3,360 (§7.2.1)

**3. RL ที่ทำได้จริงและป้องกันตัวได้ในห้องสอบคือ contextual bandit เหนือกลยุทธ์การตรวจ**
มี state/action/reward/policy ครบตามนิยาม MDP (§5.4) ใช้ข้อมูลระดับ **30 observation ต่อ arm** เท่านั้น และ **ไม่ต้องส่ง gradient เข้า LLM เลย** — นี่คือ RL ตัวจริงตัวแรกและตัวเดียวที่แผนนี้รับประกันว่าจะได้ทำ (§3.2, §7.1 ระยะ P2)

**4. เส้นทาง "ไม่เทรนโมเดล" ไม่ใช่เส้นทางด้อยกว่า — วรรณกรรมล่าสุดบอกเอง**
GEPA ชนะ GRPO เฉลี่ย 6% สูงสุด 20% โดยใช้ rollout **น้อยกว่า 35 เท่า** และ OPRO รายงานว่า prompt ที่ optimize แล้วชนะ prompt ที่มนุษย์เขียนได้ถึง 50% บนบางงาน ดังนั้น prompt/rubric optimization + best-of-n จึงเป็นทั้ง **ของที่ต้อง ship** และ **baseline ที่ RL ต้องเอาชนะให้ได้** (§5.1, §7.1 ระยะ P1)

**5. เรียกว่า RL ในวิทยานิพนธ์ได้ไหม — ได้ ถ้าเขียนให้ตรง**
"loop ที่ปรับ prompt จาก feedback" **ไม่ใช่ RL** วรรณกรรมเรียกมันว่า self-refine / reflection / textual gradient การเรียกว่า RL คือการใช้คำเกินจริงที่กรรมการจับได้ทันที ส่วน bandit **เป็น RL จริง** และต้องเขียน MDP ให้ครบทุกองค์ประกอบตาม §5.4 จึงจะปกป้องได้ (§5.2, §5.3)

**6. อันตรายที่สุดไม่ใช่ RL ล้มเหลว แต่คือ RL "ดูเหมือนสำเร็จ" ทั้งที่ข้อมูลปนเปื้อน**
ถ้าอาจารย์เห็นคะแนน AI ก่อนตรวจ **ช่องที่ไม่ถูกแก้ไม่ได้แปลว่าเห็นด้วย** — ข้อมูลทั้งชุดจะเอนเข้าหาคำตอบของ AI เอง แล้วทุกตัวเลขจะสวยขึ้นโดยที่ระบบไม่ได้เก่งขึ้นเลย ดังนั้น **blind mode 20–30% ของทุกชิ้นงาน ไม่ใช่ฟีเจอร์เสริม แต่เป็นเงื่อนไขที่ทำให้ผลการทดลองทั้งโครงงานมีความหมาย** และต้องมีตั้งแต่ระยะ P0 ก่อนแตะ prompt ใด ๆ (§7.7, §7.2.2)

**7. แผนหนึ่งปีที่แนะนำ — ทุกระยะส่งของใช้ได้ก่อน แล้วค่อยต่อยอดเป็นงานวิจัย**
P0 instrumentation + แช่แข็ง test set (ไม่เทรน) → P1 prompt optimization + best-of-n (ไม่เทรน, **นี่คือ calibration loop ที่ต้อง ship**) → P2 contextual bandit (RL ตัวจริง, ไม่มี gradient) → P3 KTO/ORPO + QLoRA **timebox แข็ง 4 สัปดาห์ ยอมล้มได้** → P4 เขียนเล่ม
**ถ้าตัด P3 ทิ้งทั้งหมด โครงงานยังมีระบบครบ ยังมีองค์ประกอบ RL และยังสอบผ่าน** (§7.1)

**8. ผลลบก็เขียนลงเล่มได้ ถ้าเตรียมล่วงหน้า**
เขียน kill criteria K1–K9 ให้วัดได้ **ก่อน**เริ่มทดลอง แล้ว "ข้อมูลระดับหลักร้อยไม่พอสำหรับ preference learning" จะกลายเป็น **negative result ที่มีหลักฐานและมีเปเปอร์รองรับ** ไม่ใช่ "ทำไม่ทัน" (§7.4, §7.6)

> **สิ่งที่ยังต้องยืนยันด้วยของจริง** — ตัวเลข human agreement ของ ASAP (QWK ≈ 0.72–0.85) ยังไม่ได้ตรวจกับเอกสารต้นทาง · เวลาต่อรอบทดลองใน §7.5 เป็นประมาณการที่ต้องแทนที่ด้วยผล smoke test · ต้นทุนต่อการเรียก API จงใจปล่อยว่าง ต้องไปดูหน้า pricing จริง ห้ามเดา · และเพดานความเห็นตรงกันระหว่างมนุษย์ **ต้องวัดเองกับอาจารย์จริง** ไม่ใช่ยืมตัวเลขจากเปเปอร์

---

## 1. ภูมิทัศน์ของวิธีการ (methods landscape)

### DPO — Direct Preference Optimization

- Paper: Rafailov et al., *Direct Preference Optimization: Your Language Model is Secretly a Reward Model*, NeurIPS 2023 — https://arxiv.org/abs/2305.18290
- ตัดขั้นตอน reward model + PPO ออก เหลือ classification loss ธรรมดา แต่ยังต้องโหลด **reference model** ควบคู่ policy model

### ORPO — Odds Ratio Preference Optimization

- Paper: Hong, Lee, Thorne, *ORPO: Monolithic Preference Optimization without Reference Model*, arXiv:2403.07691 — https://arxiv.org/abs/2403.07691
- รวม SFT + preference alignment เป็น stage เดียว **ไม่ต้องมี reference model** → ประหยัด VRAM ที่สุดในตระกูล preference optimization

### KTO — Kahneman-Tversky Optimization

- Paper: Ethayarajh et al., *KTO: Model Alignment as Prospect Theoretic Optimization*, arXiv:2402.01306 — https://arxiv.org/abs/2402.01306
- ใช้สัญญาณ **binary desirable/undesirable** ต่อ 1 ตัวอย่าง ไม่ต้องมีคู่เปรียบเทียบ → เหมาะกับข้อมูลที่มีแค่ "คะแนน" ของอาจารย์

### GRPO — Group Relative Policy Optimization

- Paper: Shao et al., *DeepSeekMath*, arXiv:2402.03300 — https://arxiv.org/abs/2402.03300
- ตัด critic/value model ออก ใช้ค่าเฉลี่ยรางวัลของกลุ่ม sample เป็น baseline

### RLHF/PPO — เส้นฐานดั้งเดิม

- PPO algorithm: Schulman et al., arXiv:1707.06347 — https://arxiv.org/abs/1707.06347
- TRL PPOTrainer doc: https://huggingface.co/docs/trl/en/ppo_trainer

### RLAIF

- Paper: Lee et al., *RLAIF vs. RLHF: Scaling Reinforcement Learning from Human Feedback with AI Feedback*, arXiv:2309.00267 — https://arxiv.org/abs/2309.00267

### RAFT / best-of-n / rejection sampling

- Paper: Dong et al., arXiv:2304.06767 — https://arxiv.org/abs/2304.06767v4

---

### ตารางสรุป: ข้อมูล / คอมพิวต์ / ได้อะไร

| วิธี | ต้องโหลดกี่โมเดลพร้อมกันตอนเทรน | ข้อมูลขั้นต่ำที่สมเหตุสมผล | คอมพิวต์ | ได้อะไร | ทำได้บน Colab/Kaggle ฟรีไหม |
|---|---|---|---|---|---|
| **RLHF / PPO** | **4**: policy + reference + reward model + value model (TRL `PPOTrainer` รับ `model`, `ref_model`, `reward_model`, `value_model` — https://huggingface.co/docs/trl/en/ppo_trainer) | reward model ต้องมี preference pairs ระดับหมื่น+ แล้วยังต้อง rollout อีกมหาศาล (benchmark ของ TRL เองใช้ `--total_episodes 1000000` บน Pythia-1B) | benchmark ทางการของ TRL รันบน DeepSpeed ZeRO-2 หลาย GPU กับโมเดล **1B** เท่านั้น | ผล: PPO checkpoint ได้ preferred rate **64.7%** เทียบ SFT **33.0%** (ตัดสินโดย GPT-4o mini as judge) | **ไม่ได้** — ต่อให้โมเดล 1B ก็ยังเป็นงานหลาย GPU-day |
| **DPO** | **2**: policy + reference (ลดเหลือ ~1 ได้ด้วย `precompute_ref_log_probs=True` หรือใช้ PEFT adapter สลับ) | dataset ตัวอย่างในเอกสาร TRL คือ UltraFeedback (~60k pairs); การศึกษาบน low-resource พบว่าได้ผลน้อยมาก (ดู §1.5) | รองรับ QLoRA ผ่าน `peft_config` + `quantization_config` — บนโมเดล 3–8B ทำบน T4 ได้ | ปรับ "สไตล์/ความชอบ" ได้ ไม่ใช่ปรับ "ความแม่นของคะแนน" | **ได้ทางเทคนิค** บนโมเดล ≤8B ด้วย QLoRA แต่ผลลัพธ์ที่ได้จากข้อมูลหลักร้อยแทบเป็น noise |
| **ORPO** | **1**: policy อย่างเดียว (ไม่มี reference model) | preference pairs เหมือน DPO | **ประหยัด VRAM ที่สุดในตระกูล** เพราะรวม SFT+alignment เป็น stage เดียว | paper รายงานว่า Phi-2 (2.7B), Llama-2 (7B), Mistral (7B) + ORPO บน UltraFeedback แซงโมเดล >7B บางตัว | **ได้ดีที่สุดในกลุ่ม preference optimization** สำหรับ free tier |
| **KTO** | 2 (policy + reference) | **ไม่ต้องมีคู่เปรียบเทียบ** ใช้แค่ label binary desirable/undesirable ต่อ 1 ตัวอย่าง | เทียบเท่า DPO | paper: "matches or exceeds the performance of preference-based methods at scales from 1B to 30B" | ได้ — และ **format ข้อมูลตรงกับสิ่งที่อาจารย์ให้ได้ง่ายที่สุด** (ตรวจแล้ว "ใช้ได้/ใช้ไม่ได้") |
| **GRPO** | 1–2 (`beta=0.0` เป็นค่า default ใน TRL แปลว่า **ไม่โหลด reference model**) แต่ต้อง **generate ตอนเทรน** | ต้องมี **verifiable reward** ไม่ใช่แค่ข้อมูล — ข้อมูลเยอะไม่ช่วยถ้าไม่มี reward function ที่คำนวณได้ | ตัวอย่างในเอกสาร TRL: **Qwen2.5 0.5B บน DeepMath-103K ใช้เวลา ~1 วัน บน 8 GPUs** (https://huggingface.co/docs/trl/en/grpo_trainer); `num_generations=8` เป็น default → generation คือคอขวด | เป็น RL ออนไลน์จริง เหมาะกับงานที่ reward ตรวจได้เป็นโปรแกรม | **ตามหลักการทำได้บนโมเดลจิ๋ว** (Unsloth มี GRPO notebook) แต่รอบการทดลองที่ให้ผลมีความหมายกินเวลาระดับหลายวัน-GPU |
| **RLAIF** | เท่ากับ RLHF/PPO บวกค่าเรียก LLM labeler | ไม่ต้องใช้ human labels — LLM ติดฉลากแทน | เท่า RLHF | "human evaluators prefer generations from both RLAIF and RLHF over a baseline SFT model in ~**70%** of cases"; RLAIF ทำได้ดีแม้ AI labeler ขนาดเท่า policy; d-RLAIF ตัดขั้น RM ออกได้ | ตัวที่ทีมทำได้จริงคือ **d-RLAIF-flavored best-of-n** ไม่ใช่ RLAIF เต็มรูป |
| **Best-of-n / RAFT (rejection sampling FT)** | 1 (หรือ 0 ถ้าทำแค่ inference-time) | **หลักสิบ–ร้อยก็เริ่มได้** เพราะใช้ข้อมูลอาจารย์เป็น *evaluation set* ไม่ใช่ training set | inference-time best-of-n = **ไม่ต้องมี GPU เลย** จ่ายแค่ค่า API × n | RAFT paper เสนอเป็น "simple and stable alternative to PPO" | **ได้เต็ม 100%** |
| **LoRA + SFT** | 1 | Unsloth: "bare minimum of at least **100 rows**"; ">**1,000 rows** is preferable" | 8B QLoRA ต้องการ ~6 GB VRAM (ตาราง §4) | สอน "รูปแบบผลลัพธ์/format" ได้ดี — LIMA แสดงว่า 1,000 ตัวอย่างเปลี่ยนพฤติกรรมได้ | **ได้** |
| **Contextual bandit / MAB** | 0 (ไม่มีการเทรน LLM) | **10–100 observation ต่อ arm** | รันบน CPU ได้ ใช้ scikit-learn/numpy | เลือก prompt/rubric variant ที่ดีที่สุดโดยอัตโนมัติ + มีทฤษฎี regret รองรับ | **ได้ และเป็นตัวเลือกที่ "ฟรี" ที่สุด** |

---

### หลักฐานเรื่อง "ข้อมูลน้อยแค่ไหนถึงพอ"

- **LIMA: Less Is More for Alignment** (Zhou et al., arXiv:2305.11206 — https://arxiv.org/abs/2305.11206)
  - LLaMa **65B** + SFT บน **1,000 curated prompts/responses** เท่านั้น ไม่มี RL ไม่มี preference model
  - ผล human study: LIMA ถูก preferred เท่ากันหรือมากกว่า GPT-4 ใน **43%** ของกรณี, เหนือ Bard **58%**, เหนือ DaVinci003 **65%**
  - ข้อสรุปของ paper: ความรู้เกือบทั้งหมดอยู่ใน pretraining แล้ว — instruction tuning ต้องการข้อมูลจำกัดมากเพื่อสอน "รูปแบบผลลัพธ์"
  - **แต่ต้องอ่านให้ครบ**: 1,000 ตัวอย่างนั้นถูก curate อย่างหนัก และโมเดลฐานคือ 65B — ไม่ใช่ใบอนุญาตให้คิดว่า 60 ตัวอย่างก็พอ

- **An Empirical Study of SFT–DPO Interaction and Parameterization in Small Language Models** (arXiv:2603.20100 — https://arxiv.org/html/2603.20100v1)
  - ทดลองบน GPT-2 (124M) กับข้อมูล paraphrase 2.83k/28.3k/283k และ sonnet generation ที่มีแค่ **131 training sonnets**
  - ผลบน paraphrase: SFT ได้ 89.87% → SFT→DPO ได้ **90.05%** (ต่างกัน < 0.6 จุดในทุกกลยุทธ์)
  - ผลบน sonnet (131 ตัวอย่าง): "DPO provides only minor gains over the SFT baseline on this task"
  - ข้อสรุปที่ตรงกับโจทย์เราเป๊ะ: **"preference optimization is limited in extremely low-resource settings"** — ทั้งขนาดโมเดลเล็กและข้อมูลน้อย ทำให้ DPO แทบไม่มีที่ว่างจะเปลี่ยนพฤติกรรมโมเดลเกินกว่าที่ SFT ทำได้แล้ว
  - และ **"parameterization dominates: FFT consistently outperforms LoRA at matched training depth"** — ผลของการเลือก parameterization ใหญ่กว่าผลของการเลือก objective

---

## 2. Reward model จากคะแนนอาจารย์ — ทำได้ไหม และพังยังไง

### 2.1 กลไกมาตรฐาน: Bradley-Terry บนคู่เปรียบเทียบ

TRL `RewardTrainer` (https://huggingface.co/docs/trl/en/reward_trainer) เทรน reward model ด้วย Bradley-Terry model (Bradley & Terry, 1952):

> p(y⁺ ≻ y⁻ | x) = σ(r(x, y⁺) − r(x, y⁻))
> L(θ) = −E[(x,y⁺,y⁻)~D] [ log σ(r_θ(x, y⁺) − r_θ(x, y⁻)) ]

- dataset ตัวอย่างในเอกสาร: **UltraFeedback** (~60k pairs), **arena-human-preference-55k** (55k pairs) — สเกลที่วงการถือว่า "ปกติ"
- รองรับ LoRA ผ่าน `peft_config=LoraConfig(modules_to_save=["score"])` — **ต้องใส่ `modules_to_save=["score"]`** ไม่งั้น reward head ไม่ถูกเทรน (จุดพลาดที่นักศึกษาจะเจอแน่ ๆ)
- TRL แนะนำ `center_rewards_coefficient` (~0.01) เพราะ Bradley-Terry เป็น **underdetermined** — บวกค่าคงที่เข้าไปในทุก reward ไม่เปลี่ยน preference probability

### 2.2 ทำไม reward model บนข้อมูลน้อยถึงอันตรายเป็นพิเศษ

- Paper: Gao, Schulman, Hilton, *Scaling Laws for Reward Model Overoptimization*, arXiv:2210.10760 — https://arxiv.org/abs/2210.10760
- ประโยคสำคัญจาก abstract: *"Because the reward model is an imperfect proxy, **optimizing its value too much can hinder ground truth performance, in accordance with Goodhart's law**."*
- ผลเชิงปริมาณ: gold reward score เทียบกับระยะทาง KL (d) มี functional form ต่างกันตามวิธี optimize
  - **best-of-n**: R(d) = d(α − βd)
  - **RL**: R(d) = d(α − β log d)
  - สัมประสิทธิ์ scale อย่างราบรื่นตาม **จำนวนพารามิเตอร์ของ reward model**
- และ paper ศึกษาผลของ *"the size of the reward model dataset"* โดยตรง
- **แปลเป็นภาษาโครงงาน**: reward model ที่เทรนจากคู่เปรียบเทียบระดับร้อยจะเป็น proxy ที่ห่วยมาก และ **ยิ่ง optimize มันหนักเท่าไร ผลจริงยิ่งแย่** — นี่ไม่ใช่ปัญหาที่แก้ได้ด้วยการ tune hyperparameter แต่เป็นคุณสมบัติเชิงโครงสร้าง

### 2.3 ทางออกที่เป็นไปได้จริงกว่า: ไม่ต้องเทรน reward model เลย

- **ใช้ rubric-as-reward + LLM-as-judge**: RLAES ใช้ **166 binary rubric items + LLM-as-judge** เป็น reward function โดยตรง (arXiv:2607.19219) — ไม่ต้องเทรน neural reward model จากศูนย์
- **ใช้เมตริกที่คำนวณได้ตรง ๆ เป็น reward**: |score_AI − score_teacher|, QWK, exact-match ของ rubric criterion — เป็น **verifiable reward** ที่ไม่ overfit เพราะไม่มีพารามิเตอร์ให้ overfit
- รากฐานของ LLM-as-judge: Zheng et al., *Judging LLM-as-a-Judge with MT-Bench and Chatbot Arena*, arXiv:2306.05685 — https://arxiv.org/abs/2306.05685
  - ผลหลัก: strong LLM judges อย่าง GPT-4 match human preference ได้ **มากกว่า 80% agreement** ซึ่งเป็นระดับเดียวกับ agreement ระหว่างมนุษย์ด้วยกันเอง
  - **แต่ paper เตือนเรื่อง bias ที่ต้องเขียนลงเอกสารด้วย**: position bias, verbosity bias, self-enhancement bias, และ limited reasoning ability
- **RLAIF** (arXiv:2309.00267) ให้ความชอบธรรมทางวิชาการกับการใช้ AI แทน human labeler: human evaluators prefer generations จากทั้ง RLAIF และ RLHF เหนือ SFT baseline ประมาณ **~70%** ของกรณี และเมื่อให้เทียบ RLAIF vs RLHF โดยตรง มนุษย์ชอบทั้งสองในอัตราเท่ากัน; paper ยังเสนอ **d-RLAIF** ที่ดึง reward จาก off-the-shelf LLM ตรง ๆ ตอน RL โดยไม่ต้องเทรน RM เลย

## 3. ทางเลือกที่เบากว่า RL เต็มรูปแบบ

### 3.1 Best-of-n / Rejection sampling fine-tuning (RAFT)

- Paper: Dong et al., *RAFT: Reward rAnked FineTuning for Generative Foundation Model Alignment*, arXiv:2304.06767 — https://arxiv.org/abs/2304.06767v4
- โค้ดทางการ: https://github.com/RLHFlow/RAFT ซึ่งระบุตรง ๆ ว่า RAFT "also known as **iterative best-of-n fine-tuning or rejection sampling fine-tuning**"
- กลไก: sample หลายคำตอบ → จัดอันดับด้วย reward model → **SFT บนเฉพาะตัวที่ได้ reward สูง** → เป็น alignment ผ่าน supervised learning ล้วน ๆ
- เหตุผลที่ paper เสนอ: RL algorithms อย่าง PPO "present substantial obstacles due to inefficiencies and instabilities"
- **นี่คือจุดที่ทีมนักศึกษาควรลง**: ถ้าไม่ fine-tune เลย ก็เหลือแค่ **best-of-n at inference time** — sample คำตอบการตรวจ n ชุด แล้วเลือกชุดที่ scorer/reward function ให้คะแนนสูงสุด ต้นทุน = ค่า API × n ไม่ต้องมี GPU เลย และวัดผลได้ตรง ๆ ว่า QWK ขยับไหม

### 3.2 Bandit สำหรับเลือก prompt / rubric variant

- Paper: *Bandit-Based Prompt Design Strategy Selection Improves Prompt Optimizers* (OPTS), arXiv:2503.01163 — https://arxiv.org/abs/2503.01163
  - ใส่กลไก **explicit selection** ของ prompt design strategy เข้าไปใน EvoPrompt โดยเสนอ 3 กลไก หนึ่งในนั้นคือ **Thompson sampling**
  - ทดลองบน Llama-3-8B-Instruct และ GPT-4o mini ด้วย BIG-Bench Hard
  - ผล: "the selection of prompt design strategies improves the performance of EvoPrompt, and the **Thompson sampling-based mechanism achieves the best overall results**"
  - **นี่คือหลักฐานว่า bandit-over-prompts เป็นงานวิจัยที่ตีพิมพ์ได้จริง** ไม่ใช่ของปลอม และใช้ข้อมูลระดับหลักร้อย evaluation ก็พอ
- อ้างอิงเสริม (survey): *Multi-Armed Bandits Meet Large Language Models*, arXiv:2505.13355 — https://arxiv.org/pdf/2505.13355
- อ้างอิงเสริม (LLM routing จาก bandit feedback): arXiv:2510.07429 — https://arxiv.org/pdf/2510.07429

**ทำไม bandit ถึงเหมาะกับโจทย์นี้เป็นพิเศษ**
- action space เล็กและกำหนดเอง (prompt variant 3–8 แบบ, rubric phrasing 3–5 แบบ, ตรวจแบบ single-pass vs multi-pass)
- reward วัดได้จากสัญญาณที่มีอยู่แล้ว: อาจารย์แก้คะแนน AI มากแค่ไหน (|score_AI − score_teacher|) หรือปุ่ม accept/edit/reject
- **ต้องการข้อมูลระดับ 10–100 ครั้งต่อ arm** ไม่ใช่หลักหมื่น — และ regret bound เป็นทฤษฎีที่มีอยู่แล้ว เขียนลงบทวิเคราะห์ได้
- เป็น RL **จริง** ในความหมายที่ถูกต้อง: มี action, reward, exploration/exploitation trade-off, policy ที่อัปเดตจากประสบการณ์

---

## 4. LoRA/PEFT + free compute

### ตาราง VRAM ขั้นต่ำจาก Unsloth (official docs)

ที่มา: https://unsloth.ai/docs/get-started/fine-tuning-for-beginners/unsloth-requirements
(เอกสารระบุว่าเป็นค่า "absolute minimum" อาจต่างกันตามโมเดล)

| Model parameters | QLoRA (4-bit) VRAM | LoRA (16-bit) VRAM |
| --- | --- | --- |
| 3B | 3.5 GB | 8 GB |
| 7B | 5 GB | 19 GB |
| 8B | 6 GB | 22 GB |
| 9B | 6.5 GB | 24 GB |
| 11B | 7.5 GB | 29 GB |
| 14B | 8.5 GB | 33 GB |
| 27B | 22 GB | 64 GB |
| 32B | 26 GB | 76 GB |
| 70B | 41 GB | 164 GB |
| 405B | 237 GB | 950 GB |

GPU ที่รองรับ: NVIDIA CUDA Capability 7.0+ (V100, **T4**, RTX 20/30/40/50, A100, H100, L40) — T4 คือการ์ดที่ Colab free tier และ Kaggle แจก

Unsloth ระบุในคู่มือหลักว่า "you can fine-tune or do RL **for free on Colab, Kaggle**, or locally with just 3GB VRAM"
(https://unsloth.ai/docs/get-started/fine-tuning-llms-guide) และแนะนำ **1–3 epochs** เพื่อเลี่ยง overfitting

### คำแนะนำเรื่องขนาดข้อมูลจาก Unsloth (official docs)

จาก https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/datasets-guide :

> *"We generally recommend using a bare minimum of at least **100 rows** of data for fine-tuning to achieve reasonable results."*
> *"a dataset with over **1,000 rows** is preferable, and in this case, more data usually leads to better outcomes."*
> *"If your dataset is too small you can also add **synthetic data** or add a dataset from Hugging Face to diversify it."*

และจาก https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/what-model-should-i-use :

- **1,000+ rows** → fine-tune the base model
- **300–1,000 rows** คุณภาพสูง → base หรือ instruct ก็ได้
- **ต่ำกว่า 300 rows** → *"the instruct model is typically the better choice"* เพราะรักษาความสามารถเดิมไว้

> **ตรงกับสถานการณ์เราเป๊ะ**: ข้อมูลหลักสิบถึงหลักร้อย = โซน "ต่ำกว่า 300 rows" → เอกสารทางการของเครื่องมือเองบอกให้ใช้ **instruct model** ไม่ใช่เทรนใหม่

### Notebook ที่รันได้ฟรีจริงบน Colab / Kaggle วันนี้

จาก https://unsloth.ai/docs/get-started/unsloth-notebooks (official notebook index) — โน้ตบุ๊กที่ระบุว่าใช้ได้บน **free Colab**:

- Gemma 3 (4B), Gemma 3 (270M), **Llama 3.1 (8B)**, Llama 3.2 (1B + 3B), Qwen3 (14B), Phi-4 (14B)

บน **Kaggle (free)**: Gemma-4-31B (ระบุว่า FREE), gpt-oss (20B), Llama 3.1 (8B), Llama 3.2 (1B + 3B)

และมี notebook เฉพาะวิธี **ครบทั้งตระกูล**:
- **DPO** — "DPO Zephyr" (มีทั้ง Colab และ Kaggle)
- **ORPO** — "Llama3 (8B)-ORPO" (มีทั้ง Colab และ Kaggle)
- **KTO** — มี (user-contributed)
- **GRPO** — มีชุดใหญ่ เช่น Qwen3.5 (4B), Llama-3.2-1B FP8

> **ข้อสรุปเชิงกลไก**: การ "รัน DPO/ORPO/GRPO บน Colab ฟรีให้จบโดยไม่ OOM" **เป็นไปได้จริงและมี notebook สำเร็จรูปให้** — ความยากไม่ได้อยู่ที่เครื่อง แต่อยู่ที่ **จะเอาข้อมูลจากไหนให้ผลลัพธ์มีความหมาย**

### ข้อจำกัดของ free tier ที่ต้องวางแผนล่วงหน้า

**Google Colab** (official FAQ: https://research.google.com/colaboratory/faq.html):
- *"In the version of Colab that is free of charge notebooks can run for **at most 12 hours**"*
- *"**Colab resources are not guaranteed and not unlimited**, and usage limits sometimes fluctuate"*
- *"Colab does not publish these limits, in part because they can vary over time"*
- *"The types of GPUs and TPUs that are available in Colab **vary over time**"* → **อย่าออกแบบแผนการทดลองที่พึ่ง GPU รุ่นใดรุ่นหนึ่ง**
- runtime จะถูกตัดถ้า idle

**Kaggle Notebooks**: ให้ GPU ฟรีต่อสัปดาห์แบบมีโควตา (ตัวเลขที่ปรากฏบน Kaggle เองคือ **30 ชั่วโมง/สัปดาห์** และเคยมีการทดลอง "floating quota" ที่ให้เกิน 30 ชั่วโมงตามอุปสงค์ — ประกาศทางการ: https://www.kaggle.com/product-feedback/173129 ; เอกสาร: https://www.kaggle.com/docs/efficient-gpu-usage) ตัวเลือก accelerator ที่มีคือ **T4 ×2 (16 GB ต่อใบ)** และ **P100 (16 GB)**

> **นัยเชิงวางแผน**: T4 16 GB + ตาราง VRAM ข้างบน ⇒ QLoRA บนโมเดลถึง **14B** อยู่ในวิสัย (8.5 GB) ส่วน LoRA 16-bit ได้แค่ระดับ **3B** (8 GB) — และ **PPO เต็มรูป (4 โมเดลพร้อมกัน) ไม่มีทางลง 16 GB สำหรับโมเดลที่ใหญ่พอจะตรวจงานได้**

### คุณภาพเทียบกับ frontier model ที่ prompt ดี

- งานที่ควรอ้างในเอกสาร: *Bridging the LLM Accessibility Divide? Performance, Fairness, and Cost of Closed versus Open LLMs for Automated Essay Scoring*, arXiv:2503.11827 — https://arxiv.org/pdf/2503.11827 (เปรียบเทียบโมเดลปิด vs เปิด บนงาน AES โดยตรง ทั้งด้าน performance, fairness และ cost)
- หลักฐานฝั่ง "prompt ดี ๆ พอไหม": OPRO แสดงว่า prompt ที่ optimize แล้วชนะ prompt ที่มนุษย์ออกแบบได้ *"by up to 8% on GSM8K, and by up to 50% on Big-Bench Hard tasks"* (arXiv:2309.03409) — คือ **การลงแรงกับ prompt ให้ผลตอบแทนสูงมากเมื่อเทียบกับต้นทุน**

## 5. การกำหนดปัญหาให้เป็น RL อย่างซื่อสัตย์ (ส่วนที่สำคัญที่สุดของ ticket นี้)

### 5.1 "loop ที่ปรับ prompt จาก feedback" — วรรณกรรมเรียกมันว่าอะไร

มีชื่อเรียกที่ถูกต้อง มีเปเปอร์รองรับ และ **ไม่ใช่ RL** ในความหมายคลาสสิก:

| สิ่งที่ระบบทำ | ชื่อที่ถูกต้องในวรรณกรรม | Primary source |
|---|---|---|
| LLM สร้างผลลัพธ์ → วิจารณ์ผลลัพธ์ตัวเอง → แก้ ซ้ำ ๆ | **Self-Refine** (iterative refinement with self-feedback) | Madaan et al., arXiv:2303.17651 — https://arxiv.org/abs/2303.17651 |
| Agent สะท้อนความล้มเหลวเป็นข้อความ เก็บใน episodic memory ใช้ในรอบถัดไป | **Reflexion / verbal reinforcement learning** | Shinn et al., NeurIPS 2023, arXiv:2303.11366 — https://arxiv.org/abs/2303.11366 |
| ส่ง feedback เชิงข้อความ "ย้อนกลับ" ผ่านกราฟการคำนวณเพื่อแก้ prompt/ตัวแปร | **Textual gradients / TextGrad** | Yuksekgonul et al., arXiv:2406.07496 — https://arxiv.org/abs/2406.07496 |
| ให้ LLM เสนอ prompt ใหม่จากคะแนนของ prompt เก่า ๆ วนไปเรื่อย ๆ | **OPRO — Optimization by PROmpting** | Yang et al., arXiv:2309.03409 — https://arxiv.org/abs/2309.03409 |
| วิวัฒนาการ prompt ด้วยการสะท้อนภาษาธรรมชาติ + Pareto frontier | **GEPA — reflective prompt evolution** | Agrawal et al., ICLR 2026 (Oral), arXiv:2507.19457 — https://arxiv.org/abs/2507.19457 |
| เลือก prompt/strategy variant ด้วย exploration–exploitation | **Bandit-based prompt strategy selection** | arXiv:2503.01163 — https://arxiv.org/abs/2503.01163 |

**ข้อสังเกตสำคัญ**: Self-Refine paper บอกเองอย่างชัดเจนว่า *"Self-Refine does not require any supervised training data, additional training, or **reinforcement learning**"* — คือผู้เขียนเองแยกวิธีของตัวเองออกจาก RL

Reflexion เรียกตัวเองว่า **"verbal reinforcement learning"** — คือใช้คำว่า reinforcement learning **แบบมี qualifier** และอธิบายว่า self-reflective feedback ทำหน้าที่เป็น *"'semantic' gradient signal"* — เครื่องหมายคำพูดรอบคำว่า semantic เป็นของผู้เขียนเอง แปลว่ารู้ตัวว่าเป็นการใช้เชิงอุปมา

### 5.2 คำตอบตรง ๆ: เรียกว่า RL ได้ไหมในวิทยานิพนธ์

**เรียกว่า "RL" เฉย ๆ ไม่ได้ ถ้าไม่มี policy parameter ที่ถูกอัปเดตด้วย gradient จาก reward** — จะโดนกรรมการสอบซัก และซักถูก

**แต่มีสองทางที่ป้องกันตัวได้เต็มร้อย:**

1. **ทำ bandit จริง** — ถ้าเลือก prompt variant ด้วย Thompson sampling / UCB / ε-greedy จากสัญญาณ reward จริง นั่นคือ **reinforcement learning ของจริง** (contextual bandit เป็น subclass ของ RL ที่มี horizon = 1) มี state (ลักษณะชิ้นงาน), action (เลือก prompt/rubric variant), reward (ความใกล้เคียงคะแนนอาจารย์), มี exploration–exploitation, มี regret bound เขียนวิเคราะห์ได้ **ไม่ต้องขอโทษใคร**

2. **ใช้ชื่อที่ตรงและอ้าง GEPA** — ถ้าทำ loop ปรับ prompt ให้เรียกว่า **reflective prompt optimization / textual-gradient optimization** แล้วอ้าง GEPA (ICLR 2026 Oral) ซึ่งพิสูจน์ว่าวิธีนี้ **ไม่ใช่ของด้อยกว่า RL แต่ชนะ RL ในหลายงาน**:
   > *"Across six tasks, **GEPA outperforms GRPO by 6% on average and by up to 20%, while using up to 35x fewer rollouts**. GEPA also outperforms the leading prompt optimizer, MIPROv2, by over 10% (e.g., +12% accuracy on AIME-2025)"*

   นี่คือ **การ์ดใบที่แข็งที่สุด** ที่ทีมจะถือเข้าห้องสอบ: "เราไม่ได้เลี่ยง RL เพราะทำไม่เป็น เราเลือกวิธีที่วรรณกรรมล่าสุดแสดงว่าดีกว่าและใช้ rollout น้อยกว่า 35 เท่า ภายใต้ข้อจำกัดของเรา"

### 5.3 ข้อควรระวังทางวิชาการที่ต้องเขียนลงเอกสาร

- **Self-refinement ไม่ใช่ยาวิเศษ**: Huang et al., *Large Language Models Cannot Self-Correct Reasoning Yet*, ICLR 2024, arXiv:2310.01798 — https://arxiv.org/abs/2310.01798
  - ผลหลัก: *"LLMs struggle to self-correct their responses **without external feedback**, and at times, their performance even **degrades** after self-correction"*
  - และวิจารณ์งานก่อนหน้าตรง ๆ ว่า *"improvements in prior studies result from using **oracle labels** to guide the self-correction process, and the improvements **vanish when oracle labels are not available**"*
  - **สิ่งที่โครงงานนี้ต้องทำ**: loop การปรับต้องใช้ **external feedback = คะแนนจริงของอาจารย์** เป็น signal ไม่ใช่ให้ AI วิจารณ์ตัวเองลอย ๆ — ถ้าทำแบบหลัง ผลจะไม่ดีขึ้นและมีเปเปอร์ ICLR รอ refute อยู่

### 5.4 องค์ประกอบ MDP ที่ต้องเขียนให้ครบถ้าจะอ้างว่าเป็น RL

| องค์ประกอบ | เวอร์ชันที่ทำได้จริงในโครงงานนี้ |
|---|---|
| **State** (s) | feature ของชิ้นงาน: ประเภทงาน (Figma/PDF), จำนวนหน้าจอ, ความยาวเอกสาร, รหัสวิชา/หัวข้อ rubric |
| **Action** (a) | เลือก 1 จาก K grading strategy: prompt template variant × rubric phrasing × single-pass vs multi-pass vs self-consistency |
| **Reward** (r) | ฟังก์ชันของความไม่ตรงกับอาจารย์ เช่น r = −\|score_AI − score_teacher\| / score_max หรือ r = 1 ถ้าอาจารย์กด accept โดยไม่แก้ |
| **Policy** (π) | contextual bandit (LinUCB / Thompson sampling) เหนือ K arms |
| **Horizon** | 1 (bandit) — ตรงกับความจริงว่าการตรวจแต่ละชิ้นไม่ส่งผลต่อชิ้นถัดไป |
| **สิ่งที่ *ไม่* มี** | ไม่มี state transition ที่ agent ควบคุมได้, ไม่มี credit assignment ข้ามหลาย step, ไม่มี gradient เข้า LLM weights — **ต้องเขียนข้อจำกัดนี้ตรง ๆ ในบท limitations** |

## 6. RL กับ automated scoring / essay grading — มีจริง แต่สเกลคนละโลก

### RLAES (2026) — งานที่ใกล้โจทย์เราที่สุด

- Paper: *Beyond Score Prediction: LLM-Based Essay Scoring and Feedback Generation via Reinforcement Learning with Rubric Rewards*, arXiv:2607.19219 — https://arxiv.org/abs/2607.19219
- ทำ **essay scoring + feedback generation พร้อมกัน** ด้วย RL (GRPO) โดยมี Rubric-based Feedback Evaluation (RFE) = **166 fine-grained binary rubric items** + LLM-as-judge เป็น reward
- ผล: บน ASAP benchmark ได้ **QWK = 0.803** ซึ่ง abstract อ้างว่าดีที่สุดในกลุ่ม LLM-based methods และคุณภาพ feedback เทียบเท่า GPT-5.5
- **ต้นทุนที่ใช้จริง** (จาก implementation details, https://arxiv.org/html/2607.19219v1):
  - base model: **Qwen3.5-9B** + LoRA
  - hardware: **eight NVIDIA H20/A100 GPUs with DeepSpeed ZeRO-3**
  - ข้อมูล: ASAP ทั้งชุด **12,978 essays** (8 prompts) — เทรนรวมทุก prompt
  - 25 epochs, rollout 8 responses/query ที่ temperature 0.9, lr 2.0e-5, KL coefficient = 0

> **นี่คือตัวเลขที่ต้องเอาไปเทียบให้ตรง ๆ ในเอกสารโครงงาน**: งาน SOTA ด้าน RL-for-AES ใช้ข้อมูล ~13,000 ชิ้น และ GPU ระดับ A100 แปดใบ ทีมที่มี 100 ชิ้นและไม่มี GPU อยู่ห่างจากจุดนั้นประมาณ **สองอันดับของขนาด (2 orders of magnitude) ในทั้งสองแกน**

ขนาด ASAP dataset ยืนยันจากหลายแหล่ง: 12,976–12,980 essays จาก 8 prompts (เช่น https://arxiv.org/pdf/2307.05553)

### SaMRL (2024) — RL กับ multi-trait AES

- Paper: *Autoregressive Multi-trait Essay Scoring via Reinforcement Learning with Scoring-aware Multiple Rewards*, arXiv:2409.17472 — https://arxiv.org/abs/2409.17472
- ประเด็นสำคัญเชิงวิธี: **QWK เป็น non-differentiable** จึงใช้เป็น loss ตรง ๆ ไม่ได้ → ต้องใช้ RL ถึงจะ optimize เมตริกที่เราสนใจจริงได้ (นี่คือ *เหตุผลทางวิชาการที่ถูกต้อง* ว่าทำไม AES ถึงต้องใช้ RL ไม่ใช่แค่ SFT)
- ออกแบบ reward = QWK-based reward + mean-squared error penalty, ใช้ autoregressive score generation เพื่อให้มี token probability ให้ RL ใช้ได้
- ผล: abstract รายงานเชิงคุณภาพว่า "notably enhancing scoring of previously inferior prompts" (ไม่ระบุตัวเลข QWK ใน abstract)

## 7. คำแนะนำเชิงปฏิบัติ

### 7.1 แผนแบ่งระยะที่ทำได้จริงใน 1 ปีการศึกษา

**หลักการวางแผนที่ต้องยึดตลอด**: calibration loop = ระบบที่ต้อง ship, RL = experiment ที่ยอมให้ล้มได้
ดังนั้นแผนนี้ถูกออกแบบให้ **ทุกระยะส่งมอบของที่ใช้ได้จริงก่อน แล้วค่อยต่อยอดเป็นงานวิจัย** ถ้าตัดระยะท้ายทิ้งทั้งหมด โครงงานยังมีระบบครบและยังสอบผ่าน

| ระยะ | ช่วงเวลา | ทำอะไร | เทรนโมเดลไหม | ส่งมอบอะไร | ประตูที่ต้องผ่านก่อนไประยะถัดไป |
|---|---|---|---|---|---|
| **P0 — Instrumentation** | ส.ค.–ต.ค. 2026 (ภาค 1 ครึ่งแรก) | วาง schema เก็บข้อมูล (`ScoreEdit`, blind mode), เขียน **eval harness** ที่รัน metric ได้ด้วยคำสั่งเดียว, **แช่แข็ง test set** ก่อนแตะ prompt ใด ๆ | **ไม่** | ระบบตรวจชั้น 1+2 ทำงาน + ตัวเลข baseline ชุดแรก | มี test set ที่แช่แข็งแล้ว ≥ 100 คู่คะแนน และรัน eval ซ้ำได้ผลเดิม |
| **P1 — Prompt/rubric optimization + best-of-n** | ต.ค.–ธ.ค. 2026 | ปรับ prompt ด้วยวิธีที่มีเปเปอร์รองรับ (OPRO / GEPA-style reflective evolution, §5.1) + best-of-n at inference (§3.1) | **ไม่** | **นี่คือ calibration loop ที่ต้อง ship** | QWK บน held-out ขยับเหนือ baseline อย่างมีนัย (§7.3) และมีคู่คะแนนสะสม ≥ 300 |
| **P2 — Contextual bandit (RL ตัวจริงตัวแรก)** | ม.ค.–มี.ค. 2027 (ภาค 2) | เลือก 1 จาก K grading strategy ด้วย Thompson sampling / LinUCB ตาม MDP ใน §5.4 | **ไม่** (ไม่มี gradient เข้า LLM) | ส่วนประกอบ RL ที่ป้องกันตัวได้ในห้องสอบ + กราฟ regret | มี ≥ 30 observation ต่อ arm สำหรับ K ≤ 6 arms (§7.2) |
| **P3 — Offline preference learning (stretch, ยอมล้มได้)** | มี.ค.–เม.ย. 2027 | KTO หรือ ORPO + QLoRA บนโมเดล ≤ 8B จากข้อมูล `ScoreEdit` ที่สะสมได้ **timebox แข็ง 4 สัปดาห์** | **ใช่** | ผลการทดลอง — บวกหรือลบก็เขียนลงเล่มได้ทั้งคู่ (§7.6) | ผ่าน data gate §7.2 เท่านั้น ถ้าไม่ผ่าน **ข้ามไป P4 เลย ไม่ต้องเสียใจ** |
| **P4 — Write-up** | เม.ย.–พ.ค. 2027 | เขียนเล่ม + เตรียมสอบ | — | เล่ม + demo | — |

**เหตุผลที่ P0 และ P1 ต้องไม่มีการเทรนโมเดลเลย**

1. เอกสารทางการของ Unsloth เองระบุว่าข้อมูล **ต่ำกว่า 300 rows ควรใช้ instruct model ไม่ใช่เทรนใหม่** (§4) — ในเดือนแรก ๆ ข้อมูลจริงยังไม่ถึงเกณฑ์นั้นด้วยซ้ำ
2. OPRO รายงานว่า prompt ที่ optimize แล้วชนะ prompt ที่มนุษย์เขียน *"by up to 8% on GSM8K, and by up to 50% on Big-Bench Hard tasks"* (arXiv:2309.03409) — ผลตอบแทนต่อชั่วโมงแรงงานสูงกว่าการเทรนหลายเท่า
3. GEPA ชนะ GRPO เฉลี่ย 6% และสูงสุด 20% โดยใช้ rollout **น้อยกว่า 35 เท่า** (arXiv:2507.19457, §5.2) — คือวรรณกรรมล่าสุดเองบอกว่าเส้นทางไม่เทรนไม่ใช่เส้นทางด้อยกว่า
4. **เหตุผลเชิงปฏิบัติที่หนักที่สุด**: ถ้าไม่มี eval harness + test set ที่แช่แข็งไว้ก่อน การเทรนใด ๆ ก็วัดผลไม่ได้ว่าดีขึ้นจริงไหม → P0 ไม่ใช่ "งานเตรียม" แต่เป็นเงื่อนไขจำเป็นของทุกระยะหลังจากนั้น

**จุดที่ต้องระวังเรื่องปฏิทิน**: ข้อมูลจากภาค 1 จะทยอยเข้ามาตามกำหนดส่งงาน (ราว 8 ชิ้น/เทอม) แปลว่า **P2 เริ่มต้นภาค 2 ได้พอดี** เพราะเพิ่งมีข้อมูลพอ และ P3 ต้องรอถึง มี.ค. 2027 ถึงจะมีข้อมูลสองเทอมรวมกัน — ห้ามวางแผนทำ P3 ในภาค 1 เพราะข้อมูลยังไม่มีทางพอ

### 7.2 ประตูข้อมูล (data gates) — ตัวเลขที่ใช้ตัดสินใจว่าขยับระยะได้หรือยัง

#### 7.2.1 ก่อนอื่น: นับหน่วยข้อมูลให้ถูก

ตัวเลขที่ดูเยอะแต่หลอกตา ต้องแยกสามชั้นให้ชัดตั้งแต่วันแรก:

| หน่วย | สูตร | ปริมาณต่อ 1 เทอม | ใช้ทำอะไรได้ |
|---|---|---|---|
| **คู่คะแนนระดับเกณฑ์** (criterion-level pair) | นศ. × ชิ้นงาน × เกณฑ์ | 40×8×5 = **1,600** ถึง 60×8×7 = **3,360** | นับเป็น row สำหรับ SFT/preference dataset ได้ |
| **ชิ้นงานที่ถูกตรวจ** (submission) | นศ. × ชิ้นงาน | **320–480** | หน่วยของ bandit 1 ครั้ง = 1 การตัดสินใจเลือก strategy |
| **นักศึกษา** (คลัสเตอร์อิสระ) | — | **40–60** | หน่วยที่กำหนด **กำลังทางสถิติจริง** และเป็นหน่วยที่ต้องใช้แบ่ง train/test (§7.3) |

> **กับดักตัวเลขที่ต้องพูดตรง ๆ ในเล่ม**: 3,360 ฟังดูเหมือน "หลักพัน" แต่ criterion-level pairs จากนักศึกษาคนเดียวกัน/ชิ้นงานเดียวกัน **สหสัมพันธ์กันสูงมาก** — เกณฑ์ทั้ง 5–7 ข้อของงานชิ้นเดียวถูกตัดสินจากไฟล์ Figma ไฟล์เดียวกัน ถ้าโมเดลเดาผิดเพราะ "อ่านงานชิ้นนี้ผิด" มันจะผิดพร้อมกันทั้ง 7 แถว จำนวนตัวอย่าง **อิสระ** ที่แท้จริงใกล้ 320–480 มากกว่า 3,360 และถ้ามองที่ระดับนักศึกษาก็เหลือ 40–60

#### 7.2.2 ข้อมูล "สะอาด" มีน้อยกว่าข้อมูลทั้งหมดมาก

ระบบเก็บสองแบบ และคุณภาพต่างกันคนละชั้น:

- **ข้อมูล blind mode** (อาจารย์ตรวจโดยไม่เห็นคะแนน AI) = ข้อมูลที่ใช้เป็น **ground truth และ test set ได้จริง** เพราะไม่มี anchoring
- **ข้อมูล `ScoreEdit` ปกติ** (อาจารย์เห็นคะแนน AI ก่อน) = ปนเปื้อน anchoring โดยโครงสร้าง (§7.7) ใช้เป็น training signal ได้แบบระวัง ๆ แต่ **ห้ามใช้เป็น test set เด็ดขาด**

**ข้อเสนอเชิงปฏิบัติ**: ขอให้อาจารย์ตรวจ blind แบบสุ่ม **20–30% ของชิ้นงานทุกชิ้นงาน** (ไม่ใช่ blind ทั้งชิ้นงานที่ 1 แล้วเลิก) → ได้ blind submissions ราว **65–145 ชิ้น/เทอม** → คู่คะแนนระดับเกณฑ์แบบสะอาดราว **330–1,000 คู่/เทอม**
ตัวเลขนี้คือ **ตัวจำกัดจริงของโครงงาน** ไม่ใช่ 3,360

#### 7.2.3 ตารางประตู

| ประตู | ต้องมีเท่าไร (ระบุหน่วยชัด) | ถึงจะทำอะไรได้ | ที่มาของเกณฑ์ |
|---|---|---|---|
| **G0 — วัดผลได้** | ≥ **100 คู่คะแนนระดับเกณฑ์จาก blind mode** ครอบคลุม ≥ 2 ชิ้นงาน และ ≥ 20 นักศึกษา | คำนวณ QWK/MAE เป็น baseline ได้ (แต่ CI ยังกว้าง — ต้องรายงาน CI เสมอ) | เกณฑ์ขั้นต่ำเพื่อให้ bootstrap CI มีความหมาย |
| **G1 — ปรับ prompt ได้** | ≥ **150 คู่ dev + 100 คู่ test (แช่แข็ง)** แยกกันที่ระดับนักศึกษา | OPRO/GEPA-style prompt optimization, best-of-n | §3.1 ระบุว่า best-of-n "หลักสิบ–ร้อยก็เริ่มได้" เพราะใช้ข้อมูลอาจารย์เป็น *evaluation set* |
| **G2 — bandit ได้** | ≥ **30 การตรวจต่อ arm** และ K ≤ 6 arms → ≥ **180 submissions** ที่ผ่าน bandit | contextual bandit จริง (P2) | ตาราง §1 ระบุ contextual bandit ต้องการ "10–100 observation ต่อ arm"; เลือก 30 เป็นค่ากลางที่ปลอดภัย |
| **G3 — LoRA SFT ได้ (เริ่มมีความหมาย)** | ≥ **1,000 rows** ระดับเกณฑ์ ที่ผ่านการ curate แล้ว | SFT เพื่อสอน **รูปแบบผลลัพธ์** เท่านั้น ไม่ใช่สอนความแม่นของคะแนน | Unsloth: *">1,000 rows is preferable"*; LIMA ใช้ 1,000 curated examples (§1) |
| **G3′ — ต่ำกว่า 300 rows** | < 300 rows | **อย่าเทรน** — เอกสารทางการบอกให้ใช้ instruct model | §4 อ้าง https://unsloth.ai/docs/get-started/fine-tuning-llms-guide/what-model-should-i-use |
| **G4 — KTO / ORPO ได้ (เชิงทดลอง)** | ≥ **1,000 labeled examples** โดยอย่างน้อย **300 มาจาก blind mode** และมีทั้ง desirable/undesirable อย่างน้อยฝั่งละ 30% | รัน P3 ได้ **ในฐานะ experiment เท่านั้น ไม่ใช่ระบบที่ ship** | KTO ใช้ binary label ต่อ 1 ตัวอย่าง ไม่ต้องมีคู่ (§1) — ตรงกับข้อมูลที่อาจารย์ให้ได้ที่สุด |
| **G5 — เทรน reward model + PPO** | ≥ **10,000+ preference pairs** | **ประตูนี้ปิดตายสำหรับโครงงานนี้** | dataset มาตรฐานคือ UltraFeedback ~60k pairs, arena-human-preference-55k (§2.1); TRL benchmark ใช้ `--total_episodes 1000000` (§1) |
| **G6 — GRPO เต็มรูป** | ต้องมี **verifiable reward** ไม่ใช่แค่ข้อมูล + เวลา GPU ระดับหลายวัน | ไม่แนะนำ | RLAES ใช้ ASAP 12,978 essays + **A100/H20 แปดใบ** (§6) — ห่างจากทีมสองอันดับของขนาดในทั้งสองแกน |

> **อ่านตารางนี้ให้ถูก**: ประตูที่ทีมจะผ่านได้จริงในปีเดียวคือ **G0–G2 แน่นอน, G3/G4 ถ้าเก็บข้อมูลขยัน, G5–G6 ไม่มีทาง** และนั่นไม่ใช่ความล้มเหลว — เป็นข้อค้นพบที่ต้องเขียนลงเล่ม (§7.6)

### 7.3 วัดผลอย่างไรว่าดีขึ้นจริง

#### 7.3.1 ชุดเมตริกที่ต้องรายงานพร้อมกัน (ห้ามรายงานตัวเดียว)

| เมตริก | ระดับที่รายงาน | ทำไมต้องมี |
|---|---|---|
| **QWK** (quadratic weighted kappa) | รวม + **แยกรายเกณฑ์** | เมตริกมาตรฐานของวงการ AES ใช้เทียบกับวรรณกรรมได้ตรง ๆ (RLAES รายงาน QWK = 0.803 บน ASAP, §6) |
| **MAE / RMSE** บนสเกลคะแนนดิบ | รวม + รายเกณฑ์ | ตีความง่ายสำหรับอาจารย์ ("เฉลี่ยพลาดไป 0.6 ระดับ") และไม่มีปัญหา kappa paradox |
| **Exact agreement %** และ **adjacent agreement %** (±1 ระดับ) | รายเกณฑ์ | บอกว่าโมเดล "ผิดแบบใกล้เคียง" หรือ "ผิดคนละเรื่อง" ซึ่งมีนัยต่อผู้ใช้ต่างกันมาก |
| **Bias เชิงทิศทาง** (mean signed error) | รายเกณฑ์ | จับอาการ "AI ใจดีเกินไป/โหดเกินไป" ซึ่ง MAE ซ่อนไว้ |
| **อัตราที่อาจารย์กด accept โดยไม่แก้** | รวม | เมตริกเชิงระบบ ตรงกับ reward ของ bandit (§5.4) |
| **95% bootstrap CI** ของทุกตัวข้างบน | ทุกตัว | ที่ n ระดับร้อย **ตัวเลขจุดเดียวไม่มีความหมาย** — ต้องมี CI ทุกครั้ง |

**ข้อควรระวังเรื่อง QWK ที่ต้องเขียนลงเล่ม** — Doewes, Kurdhi & Saxena, *Evaluating Quadratic Weighted Kappa as the Standard Performance Metric for Automated Essay Scoring*, EDM 2023 (https://educationaldatamining.org/EDM2023/proceedings/2023.EDM-long-papers.9/index.html) ระบุข้อจำกัดของ QWK ไว้หลายข้อ: **ความไวต่อ rating scale, kappa paradox, ผลของ prevalence, ผลของตำแหน่งการเห็นตรงกันบนเส้นทแยงมุม และข้อจำกัดเมื่อมีผู้ให้คะแนนจำนวนมาก** — ข้อสรุปของ paper คือการพึ่ง QWK ตัวเดียว *"may not be sufficient"*
สำหรับโครงงานนี้ข้อนี้ **ไม่ใช่รายละเอียดปลีกย่อย**: rubric ของเรามีแค่ 3–5 ระดับ และการกระจายคะแนนจริงมักเบ้ (นักศึกษาส่วนใหญ่ได้ระดับกลาง-สูง) ซึ่งคือเงื่อนไขที่ทำให้ kappa paradox โผล่ชัดที่สุด

#### 7.3.2 การแบ่ง train/test ที่ไม่โกง

**แหล่ง leakage ที่จะเกิดแน่ถ้าไม่ระวัง 4 แหล่ง:**

1. **นักศึกษาคนเดียวกัน** — นักศึกษาคนหนึ่งส่งงาน 8 ชิ้นตลอดเทอม สไตล์งาน/ระดับฝีมือคงที่ ถ้าชิ้นที่ 1–5 อยู่ใน train และชิ้นที่ 6 อยู่ใน test โมเดลไม่ได้ทำนาย มันจำได้
2. **งานกลุ่มเดียวกัน** — สมาชิกกลุ่มเดียวกันส่ง **ไฟล์ Figma เดียวกันหรือใกล้เคียงมาก** ถ้ากระจายคนละฝั่ง คือการเอาคำตอบไปวางใน test ตรง ๆ
3. **เกณฑ์ของชิ้นงานเดียวกัน** — 5–7 criterion rows ของ submission เดียวกันต้องอยู่ฝั่งเดียวกันเสมอ
4. **Prompt/rubric เดียวกันข้ามเทอม** — ถ้าโจทย์ชิ้นงานภาค 1 กับภาค 2 เป็นโจทย์เดียวกัน การเทรนข้ามเทอมคือ leakage เชิงเนื้อหา

**กฎการแบ่งที่ใช้ได้จริง**: split ที่ **หน่วยกลุ่ม (group) = กลุ่มงาน ถ้ามีงานกลุ่ม, ไม่มีก็เป็นนักศึกษา** ด้วย `GroupKFold` / `GroupShuffleSplit` (https://scikit-learn.org/stable/modules/cross_validation.html#group-k-fold) แล้วบังคับ 3 ข้อ:

- ทุก criterion row ของ submission เดียวกัน → fold เดียวกัน (โดยนัยจากข้อบน)
- **test set แช่แข็งตั้งแต่ P0** เขียน hash ของรายชื่อ id ลงไฟล์ commit ไว้ ห้ามแตะอีกจนจบโครงงาน
- ที่ n ระดับร้อย ให้ใช้ **grouped k-fold CV (k=5) รายงานค่าเฉลี่ย ± SD** เป็นตัวเลขหลัก แล้วใช้ frozen test set เป็นการตรวจครั้งเดียวตอนท้าย — ไม่ใช่ single split เพราะ variance สูงเกินไป

> **ข้อควรระวังเรื่องการนับซ้ำ**: เมื่อ split ตามนักศึกษา จำนวน "ตัวอย่างอิสระ" ในแต่ละ fold คือ 8–12 คน ไม่ใช่ 600 แถว — ต้องคำนวณ CI ด้วย **cluster bootstrap (resample ที่ระดับนักศึกษา ไม่ใช่ระดับแถว)** ไม่งั้น CI จะแคบเกินจริงหลายเท่าและจะสรุปว่า "ดีขึ้นอย่างมีนัย" ทั้งที่ไม่ใช่

#### 7.3.3 Baseline ที่ต้องเทียบ (เรียงจากง่ายไปยาก)

| # | Baseline | ทำไมต้องมี |
|---|---|---|
| B0 | **ทำนายระดับที่พบบ่อยที่สุด (majority class) ต่อเกณฑ์** | จับกรณีที่ accuracy สูงเพราะการกระจายเบ้ ไม่ใช่เพราะโมเดลเก่ง — **QWK ของตัวทำนายค่าคงที่ = 0 ตามนิยาม** ซึ่งเป็นเหตุผลที่ต้องรายงาน accuracy คู่ QWK เสมอ |
| B1 | **สุ่มตามการกระจายของคะแนนจริง** | เส้นศูนย์ทางสถิติ |
| B2 | **ชั้นที่ 1 อย่างเดียว** (สัญญาณ deterministic → กฎ threshold) | ตอบคำถามกรรมการว่า "LLM จำเป็นจริงไหม" — ถ้า B2 ใกล้เคียงระบบเต็ม นั่นคือข้อค้นพบสำคัญ |
| B3 | **LLM zero-shot prompt v0** (prompt แรกที่ยังไม่ optimize) | เส้นฐานที่ใช้วัดว่า P1 (prompt optimization) ได้ผลจริงไหม |
| B4 | **LLM + prompt ที่ optimize แล้ว (ผลของ P1)** | เส้นฐานที่ P2/P3 ต้องเอาชนะ — **นี่คือคู่เทียบตัวจริงของสาย RL ไม่ใช่ B3** |
| B5 | **เพดานมนุษย์** (ดู 7.3.4) | เป้าจริง ไม่ใช่ 1.0 |

> **จุดที่ทีมจะโกงตัวเองโดยไม่ตั้งใจ**: เอาผล P3 ไปเทียบ B3 แล้วบอกว่า "RL ทำให้ดีขึ้น 15%" ทั้งที่ 14% มาจาก prompt optimization ใน P1 — **ทุกการเปรียบเทียบของสาย RL ต้องเทียบกับ B4 เท่านั้น**

#### 7.3.4 เพดานมนุษย์ (inter-rater reliability) — เป้าจริงของโครงงาน

ระบบไม่มีทางเก่งเกินความสอดคล้องของมนุษย์ด้วยกันเอง เพราะ "คำตอบที่ถูก" ก็ยังเป็นความเห็นของคนอยู่ดี ดังนั้น **ต้องวัดเพดานนี้เอง ไม่ใช่อ้างจากเปเปอร์**

**วิธีวัดที่ทำได้ในโครงงานนี้:**
- ให้อาจารย์ผู้สอน + ผู้ช่วยสอน (หรืออาจารย์ท่านที่สอง) ตรวจ **ชิ้นงานชุดเดียวกัน 30–50 ชิ้นแบบ blind** → คำนวณ QWK ระหว่างมนุษย์สองคน
- ถ้าหาคนที่สองไม่ได้จริง ๆ ให้ใช้ **intra-rater**: อาจารย์คนเดิมตรวจงานชุดเดิมซ้ำหลังเว้นระยะ ≥ 2 สัปดาห์ โดยสลับลำดับและไม่เห็นคะแนนเดิม → ได้ **เพดานบน** ที่หลวมกว่าแต่ยังใช้อ้างอิงได้ (ต้องระบุในเล่มว่าเป็น intra ไม่ใช่ inter)

**ตัวเลขอ้างอิงจากวรรณกรรม (ใช้เป็นบริบท ไม่ใช่เป้าโดยตรง):**
- ASAP benchmark รายงาน human–human agreement อยู่ในช่วง **QWK ≈ 0.72–0.85** ข้ามชุดเรียงความ 8 ชุด (grade 7–10) — _หมายเหตุ: ตัวเลขช่วงนี้ได้จากผลค้นทุติยภูมิ **ยังไม่ได้ตรวจสอบ** กับเอกสารต้นทางของ ASAP โดยตรง ก่อนใส่ลงเล่มต้องไล่หาต้นทางก่อน_
- เกณฑ์ตีความ kappa ที่อ้างกันทั่วไปคือ Landis & Koch (1977), *Biometrics* 33(1):159–174: 0.41–0.60 = moderate, **0.61–0.80 = substantial**, 0.81–1.00 = almost perfect
- MT-Bench (Zheng et al., arXiv:2306.05685, §2.3) รายงานว่า GPT-4 judge match human preference **> 80% agreement** ซึ่ง *"เป็นระดับเดียวกับ agreement ระหว่างมนุษย์ด้วยกันเอง"* — เป็นหลักฐานว่าเพดานมนุษย์เองก็ไม่ได้สูงลิ่ว

> **เป้าที่ควรตั้งจริงในเล่ม**: ไม่ใช่ "QWK 0.80" แต่คือ **"QWK ของระบบ ≥ 80% ของ QWK ระหว่างมนุษย์สองคนที่วัดได้เองในบริบทเดียวกัน"** — เป้าแบบนี้ป้องกันตัวได้ในห้องสอบ และไม่ผูกโครงงานไว้กับตัวเลขของ dataset ต่างประเทศที่คนละงานคนละ rubric
> **และถ้าวัดแล้วพบว่าอาจารย์สองคนเห็นตรงกันแค่ QWK ~0.5** นั่นคือข้อค้นพบที่มีค่ามาก: มันแปลว่า rubric เองกำกวม และงานที่ควรทำต่อคือ **แก้ rubric ไม่ใช่แก้โมเดล** — เขียนลงเล่มได้เต็ม ๆ

### 7.4 เกณฑ์ยกเลิก (kill criteria) — เขียนไว้ล่วงหน้าก่อนเริ่มทดลอง

**ทำไมต้องเขียนล่วงหน้า**: ถ้าไม่กำหนดไว้ก่อน ทีมจะเถียงกันตอนเดือน เม.ย. ว่า "ลองอีกนิดเดียว" แล้วเสียเวลาที่ควรใช้เขียนเล่ม
เกณฑ์ทั้งหมดข้างล่างต้อง **เขียนลงเอกสารโครงงานตั้งแต่ตอนเสนอหัวข้อ** และให้อาจารย์ที่ปรึกษาเซ็นรับทราบ — การมี pre-registered kill criteria เป็นจุดแข็งทางวิชาการ ไม่ใช่การยอมแพ้

| รหัส | เงื่อนไข (วัดได้ทั้งหมด) | ตรวจเมื่อไร | ทำอะไรเมื่อเข้าเงื่อนไข |
|---|---|---|---|
| **K1 — งบเวลาของระบบหลัก** | งานของ calibration loop เลื่อน milestone > **2 สัปดาห์** จากแผน | ทุกสัปดาห์ | **หยุดสาย RL ทันที** ทุกคนกลับไปทำระบบหลัก ปลดล็อกใหม่ได้เมื่อกลับเข้าแผน |
| **K2 — สัดส่วนแรงงาน** | ชั่วโมงสะสมที่ลงกับสาย RL เกิน **25% ของชั่วโมงทีมทั้งหมด** | ทุกเดือน (ต้อง log ชั่วโมงจริง) | freeze สาย RL จนกว่าสัดส่วนจะกลับมาต่ำกว่าเกณฑ์ |
| **K3 — ประตูข้อมูล** | ถึง **1 มี.ค. 2027** แล้วยังไม่ผ่าน **G4** (§7.2: 1,000 rows + ≥300 จาก blind mode) | 1 มี.ค. 2027 (วันเดียว ไม่มีต่อเวลา) | **ข้าม P3 ทั้งหมด** เขียนบทวิเคราะห์ "ทำไมข้อมูลไม่พอ" แทน |
| **K4 — ปฏิทิน** | ถึง **1 มี.ค. 2027** แล้ว P2 (bandit) ยังรัน end-to-end ไม่ได้ | 1 มี.ค. 2027 | ตัด P3 ทิ้ง แล้วเอาเวลาที่เหลือไปทำ P2 ให้จบให้ได้ |
| **K5 — ไม่มี effect ให้จับ (bandit)** | หลังเก็บครบ **≥30 ครั้ง/arm** แล้ว 95% CI ของ reward เฉลี่ยของ arm ที่ดีที่สุด **ยังคาบเกี่ยว** กับ arm ที่แย่ที่สุด | เมื่อผ่าน G2 + 4 สัปดาห์ | ประกาศผลว่า **"arms แยกไม่ออกที่ขนาดข้อมูลนี้"** — เป็นผลลัพธ์ที่ตีพิมพ์ได้ ไม่ต้องรีดต่อ |
| **K6 — ถดถอย (P3)** | โมเดลที่ผ่าน KTO/ORPO ทำ QWK บน held-out **ต่ำกว่า B4** (prompt ที่ optimize แล้ว) ใน **2 ครั้งติดต่อกัน** ที่ hyperparameter ต่างกัน | ทุกรอบทดลอง | หยุด P3 เขียนรายงานเชิงลบ (§7.6) |
| **K7 — ค่าใช้จ่ายรอบทดลอง** | รอบทดลอง 1 รอบใช้เวลา > **12 ชั่วโมง** (เพดาน Colab free, §4) หรือต้องรันข้ามหลาย session จน reproduce ไม่ได้ | ครั้งแรกที่เกิด | ลดขนาดโมเดล/ข้อมูลลง 1 ขั้น ถ้ายังเกินอีก → หยุด |
| **K8 — reward hacking** | reward เฉลี่ยเพิ่มขึ้น แต่ **คะแนนคุณภาพหลักฐาน/เหตุผลที่มนุษย์ประเมิน (spot-check 20 ชิ้น) ลดลง** | ทุกรอบทดลอง | หยุดทันทีและวิเคราะห์ว่า reward รั่วตรงไหน (§7.7) |
| **K9 — ระบบหลักแย่ลงเพราะ RL** | ฟีเจอร์จาก P2/P3 ทำให้อัตราที่อาจารย์ accept ลดลง หรือ latency เกินเกณฑ์ที่ตั้งไว้ | ทุกครั้งที่ deploy | roll back ทันที (ต้องมี feature flag ตั้งแต่ P0) |

**กติกาเสริมที่ควรตั้ง**
- P3 มี **timebox แข็ง 4 สัปดาห์** ตามตาราง §7.1 นับจากวันเริ่ม ไม่ว่าจะได้ผลหรือไม่ก็ตาม เมื่อครบให้หยุดและเขียน
- ทุกครั้งที่เข้าเงื่อนไข kill ให้บันทึก **วันที่ + ตัวเลขที่วัดได้ + การตัดสินใจ** ลง log เดียว → log นี้กลายเป็น **ภาคผนวกของเล่มโดยตรง** และเป็นหลักฐานว่าทีมทำงานอย่างมีวินัย ไม่ใช่ทำไม่ทัน

### 7.5 งบคอมพิวต์และเวลาจริง

#### 7.5.1 สิ่งที่ free tier ให้ได้จริง (สรุปจาก §4)

| ทรัพยากร | ที่ได้ | ข้อจำกัดที่ต้องออกแบบรอบตัวมัน |
|---|---|---|
| **Colab free** | GPU ที่ **ไม่รับประกันรุ่น** (มัก T4 16 GB) | รันได้ **≤ 12 ชม./notebook**, ตัดเมื่อ idle, *"resources are not guaranteed and not unlimited"*, *"types of GPUs available vary over time"* (https://research.google.com/colaboratory/faq.html) |
| **Kaggle Notebooks** | **T4 ×2 (16 GB/ใบ)** หรือ **P100 (16 GB)**, โควตา **~30 ชม./สัปดาห์** | ต่อ session มีเพดานเวลาเช่นกัน; โควตารายสัปดาห์คือ **งบจริงที่ต้องจัดสรร** (https://www.kaggle.com/docs/efficient-gpu-usage) |
| **เพดานโมเดลที่ลงได้บน 16 GB** | QLoRA ถึง ~**14B** (8.5 GB), LoRA 16-bit ได้แค่ ~**3B** (8 GB) | ตาราง Unsloth §4 |
| **สิ่งที่ลงไม่ได้เด็ดขาด** | PPO เต็มรูป (policy + reference + reward + value = 4 โมเดลพร้อมกัน) | §1 และ §4 |

#### 7.5.2 งบเวลาต่อรอบทดลอง — ประมาณการที่ต้องวัดจริงก่อนเชื่อ

> ตัวเลขในตารางนี้เป็น **ประมาณการเชิงวางแผน ยังไม่ได้ตรวจสอบด้วยการรันจริง** ให้ทีมรัน smoke test 50 rows ในสัปดาห์แรกของแต่ละระยะเพื่อแทนที่ตัวเลขเหล่านี้ด้วยของจริง

| งาน | ฮาร์ดแวร์ | ประมาณเวลา 1 รอบ | จำนวนรอบที่พอสรุปผลได้ | งบรวม |
|---|---|---|---|---|
| **P1 — prompt optimization (OPRO/GEPA-style)** | **ไม่ใช้ GPU** ใช้ API เท่านั้น | นาที–ชั่วโมง ต่อ 1 iteration (ขึ้นกับขนาด dev set) | 20–50 iterations | จำกัดด้วย **ค่า API ไม่ใช่ GPU** |
| **P1 — best-of-n at inference** | ไม่ใช้ GPU | ต้นทุน = ค่า API × n ต่อการตรวจ 1 ชิ้น | — | **ทดสอบ n ∈ {1,3,5} แล้วรายงาน QWK-vs-cost curve** เป็นผลงานหนึ่งชิ้น |
| **P2 — contextual bandit** | **CPU ล้วน** (scikit-learn/numpy, §1) | วินาที | รันซ้ำ 100+ seeds ได้สบาย | ~ศูนย์ |
| **P3 — QLoRA SFT บน ~1,000 rows, โมเดล 7–8B** | T4 16 GB | **ประมาณ 1–3 ชม.** ที่ 1–3 epochs (Unsloth แนะนำ 1–3 epochs เพื่อเลี่ยง overfitting, §4) | ≥ 6 รอบ (ต้องกวาด hyperparameter) | **~10–20 ชม. GPU** → พอดีกับโควตา Kaggle ~1 สัปดาห์ |
| **P3 — KTO/ORPO บนข้อมูลเท่ากัน** | T4 16 GB | ใกล้เคียงกัน; **ORPO ถูกที่สุดเพราะไม่มี reference model** (§1) | ≥ 6 รอบ | ~10–25 ชม. GPU |
| **GRPO เต็มรูป** | — | เอกสาร TRL: Qwen2.5-0.5B บน DeepMath-103K = **~1 วัน บน 8 GPUs** (§1) | — | **เกินงบอย่างไม่มีทางเลี่ยง** |

#### 7.5.3 กฎการใช้งบที่ควรบังคับตัวเอง

1. **ทุกรอบทดลองต้องจบใน 1 session** — ถ้ารอบหนึ่งต้องรันข้ามหลาย session ให้ถือว่าเข้าเงื่อนไข **K7** และลดขนาดลง เหตุผล: Colab ตัด session ได้ทุกเมื่อและไม่มีการรับประกัน checkpoint
2. **เขียน checkpoint ลง Google Drive ทุก N step ตั้งแต่รอบแรก** — ไม่ใช่หลังเจอปัญหา
3. **อย่าออกแบบการทดลองที่พึ่ง GPU รุ่นใดรุ่นหนึ่ง** — FAQ ของ Colab บอกตรง ๆ ว่ารุ่นเปลี่ยนตามเวลา ให้ใช้ **Kaggle เป็นเครื่องหลักของการทดลองที่ต้อง reproduce** เพราะ accelerator เลือกได้ชัดกว่าและมีโควตาที่คาดเดาได้ (30 ชม./สัปดาห์) แล้วใช้ Colab เป็นเครื่องสำรอง
4. **งบ API ต้องประมาณล่วงหน้าและตั้งเพดาน** — best-of-n ที่ n=5 คือค่าใช้จ่ายต่อการตรวจ **5 เท่า** ของ n=1 คูณกับ 320–480 ชิ้น/เทอม สูตรงบคือ
   `ต้นทุนต่อเทอม ≈ (จำนวน submission) × (จำนวนเกณฑ์) × n × (ต้นทุนต่อการเรียก 1 ครั้ง)`
   _ต้นทุนต่อการเรียกต้องไปดูหน้า pricing จริงของผู้ให้บริการตอนวางแผน — **ห้ามเดา** และตัวเลขนี้ยังไม่ได้กรอกในเอกสารนี้_
5. **แยกงบ "รันระบบจริง" ออกจากงบ "ทดลอง" คนละบัญชี** — ไม่งั้นการทดลองที่คุมไม่อยู่จะกินโควตาจนระบบที่อาจารย์ใช้จริงล่ม ซึ่งเป็นความล้มเหลวที่แพงที่สุดของโครงงานนี้

### 7.6 สิ่งที่ต้องเขียนลงเล่มไม่ว่าผลจะออกมาทางไหน

**หลักการ**: โครงสร้างเล่มต้องถูกออกแบบให้ **บทที่ 1–4 ไม่ขึ้นกับผลของสาย RL เลย** และให้ผลของสาย RL อยู่ในบทเดียวที่เขียนได้สองเวอร์ชัน โครงงานจึงสอบผ่านได้ทั้งสองทาง

#### 7.6.1 ส่วนที่เขียนเหมือนกันทั้งสองทาง (เขียนได้ตั้งแต่ยังไม่รู้ผล)

1. **การวิเคราะห์ระบอบข้อมูล (data regime analysis)** — ตาราง §7.2.1 ที่แสดงว่า "หลักพันคู่" จริง ๆ แล้วคือ 40–60 คลัสเตอร์อิสระ พร้อมเหตุผล นี่เป็นการวิเคราะห์ที่มีค่าในตัวเอง
2. **ตาราง landscape ของวิธี** (§1) พร้อมคอลัมน์ "ทำได้บน free tier ไหม" — เป็นการ survey ที่ใช้ได้จริง
3. **ระยะห่างจาก SOTA เชิงปริมาณ** — RLAES ใช้ ASAP 12,978 essays + A100/H20 แปดใบ vs. ทีมมี ~300–1,000 คู่และ 0 GPU = **ห่างกันสองอันดับของขนาดในทั้งสองแกน** (§6) การเขียนตัวเลขนี้ตรง ๆ ทำให้กรรมการเห็นว่าทีมรู้ว่าตัวเองยืนอยู่ตรงไหน
4. **เพดานมนุษย์ที่วัดเอง** (§7.3.4) — inter-rater QWK ของอาจารย์ในบริบทจริงของวิชานี้ **เป็น contribution ที่ไม่มีใครมีมาก่อน** เพราะไม่มีใครวัด IRR ของ rubric UX/UI ของวิชานี้มาก่อน
5. **การจัดหมวดวิธีการให้ถูกชื่อ** (§5.1–5.2) — ตารางที่แยก Self-Refine / Reflexion / TextGrad / OPRO / GEPA / bandit ออกจากกัน พร้อมข้อสรุปว่าอะไรเป็น RL อะไรไม่ใช่ นี่คือส่วนที่กันการโดนซักในห้องสอบ
6. **ข้อจำกัดเชิงโครงสร้าง** — องค์ประกอบ MDP ที่ **ไม่มี** ในระบบนี้ (ไม่มี state transition ที่ agent ควบคุม, ไม่มี credit assignment ข้ามหลาย step, ไม่มี gradient เข้า LLM weights) ตามตาราง §5.4
7. **log ของ kill criteria** (§7.4) เป็นภาคผนวก

#### 7.6.2 ถ้าสาย RL ได้ผล (bandit ชนะ / P3 ขยับ QWK ขึ้นจริง)

- รายงาน **effect size พร้อม cluster-bootstrap CI** ไม่ใช่แค่ค่ากลาง (§7.3.2)
- แสดง **กราฟ cumulative regret** ของ bandit เทียบกับ uniform-random arm selection และเทียบกับ oracle (arm ที่ดีที่สุดถ้ารู้ล่วงหน้า)
- แสดงว่า arm ไหนชนะ **ในบริบทไหน** (contextual — เช่น งานที่มีหน้าจอเยอะชอบ strategy A, งานหน้าจอน้อยชอบ B) เพราะนี่คือสิ่งที่ทำให้มันเป็น *contextual* bandit จริง ไม่ใช่ A/B test
- **ยังต้องเขียนข้อจำกัดเรื่อง generalization**: ผลมาจากวิชาเดียว อาจารย์ผู้ตรวจกลุ่มเดียว rubric ชุดเดียว — ห้ามอ้างเกินนี้

#### 7.6.3 ถ้าสาย RL ไม่ได้ผล (ซึ่งมีโอกาสสูงและไม่เป็นไร)

เขียนเป็น **negative result ที่มีหลักฐาน** ไม่ใช่ "ทำไม่ทัน" — สิ่งที่ต้องมี:

1. **ตัวเลขที่แสดงว่าไม่ได้ผลจริง** พร้อม CI ที่คาบเกี่ยว (เช่น K5)
2. **การเทียบกับวรรณกรรมที่ทำนายผลนี้ไว้แล้ว** — จุดนี้แข็งมากเพราะมีเปเปอร์รองรับตรง ๆ:
   - *"preference optimization is limited in extremely low-resource settings"* (arXiv:2603.20100, §1) — งานนั้นพบว่า DPO เหนือ SFT แค่ < 0.6 จุด และบนชุด sonnet ที่มีแค่ 131 ตัวอย่าง *"DPO provides only minor gains over the SFT baseline"*
   - Gao, Schulman & Hilton, *Scaling Laws for Reward Model Overoptimization* (arXiv:2210.10760, §2.2) — reward model จากข้อมูลน้อยเป็น proxy ที่แย่ และ *"optimizing its value too much can hinder ground truth performance, in accordance with Goodhart's law"* → **ผลลบของเราสอดคล้องกับ scaling law ที่รู้กันอยู่แล้ว ไม่ใช่บั๊ก**
   - Huang et al., ICLR 2024 (arXiv:2310.01798, §5.3) — self-correction ที่ไม่มี external feedback ทำให้ผล *"degrades"*
3. **การเทียบกับเส้นทางที่ได้ผล** — GEPA ชนะ GRPO 6–20% ด้วย rollout น้อยกว่า 35 เท่า (§5.2) เป็นคำอธิบายเชิงวิชาการว่าทำไม **การลงแรงที่ prompt optimization จึงเป็นการตัดสินใจที่ถูก ไม่ใช่การหนี**
4. **ข้อเสนอเชิงระบบว่าต้องมีข้อมูลเท่าไรถึงจะทำได้** — คือตาราง data gate §7.2.3 พร้อมประมาณการว่าต้องเก็บกี่เทอมถึงจะถึง G4/G5 (เช่น G5 ที่ 10,000 pairs ≈ **3–6 เทอมของการใช้งานจริง** ถ้านับเฉพาะ blind mode ก็ยาวกว่านั้นอีกหลายเท่า) — นี่คือ **future work ที่มีตัวเลขรองรับ** ไม่ใช่ future work ลอย ๆ

> **ประโยคที่ควรมีในบทสรุปไม่ว่าทางไหน**: "ระบบที่ส่งมอบคือ calibration loop ที่ทำงานได้จริงกับข้อมูลระดับที่ชั้นเรียนหนึ่งผลิตได้ ส่วนการใช้ reinforcement learning เต็มรูปแบบถูกประเมินอย่างเป็นระบบแล้วพบว่าอยู่นอกระบอบข้อมูลและคอมพิวต์ของโครงงานนี้ พร้อมตัวเลขกำกับว่าต้องมีเท่าไรถึงจะเข้าถึงได้"

### 7.7 กับดักที่ต้องกันไว้ตั้งแต่ออกแบบ

#### 7.7.1 Anchoring — ข้อมูลปนเปื้อนตั้งแต่เกิด

**กลไก**: อาจารย์เห็นคะแนน AI ก่อนให้คะแนนเอง → คะแนนอาจารย์ถูกดึงเข้าหาคะแนน AI → เมื่อเอาข้อมูลนั้นไปเทรน/ปรับ prompt ระบบก็ **เรียนรู้จากเงาของตัวเอง** ไม่ใช่จากอาจารย์ ยิ่งวนยิ่งเข้ารูปตัวเอง (self-confirming loop) และ QWK จะสูงขึ้นเรื่อย ๆ **โดยที่ระบบไม่ได้เก่งขึ้นเลย**

**สิ่งที่ต้องทำ**
- **กฎเหล็ก: test set ต้องมาจาก blind mode 100%** ข้อมูล `ScoreEdit` ที่เห็นคะแนน AI ก่อน ใช้เป็น test set ไม่ได้ในทุกกรณี
- สุ่ม blind **20–30% ของทุกชิ้นงาน** ไม่ใช่กระจุกที่ชิ้นงานแรก ๆ (§7.2.2) เพื่อให้ blind set กระจายทั้งเทอม
- **วัดขนาดของ anchoring แล้วเขียนลงเล่ม**: เทียบการกระจายคะแนนอาจารย์ใน blind mode vs non-blind บนงานลักษณะเดียวกัน ถ้าคะแนน non-blind เกาะคะแนน AI แน่นกว่าอย่างมีนัย นั่นคือ **การวัด anchoring ได้เชิงปริมาณ = contribution หนึ่งชิ้น**
- ในโหมดปกติ ให้ UI **ซ่อนคะแนน AI จนกว่าอาจารย์จะกดเลือกระดับของตัวเองแล้ว** (แสดงเฉพาะหลักฐาน/จุดที่ AI ชี้ ไม่แสดงระดับ) — เก็บ signal ได้เกือบเท่าเดิมโดยลด anchoring ลงมาก

#### 7.7.2 Reward hacking ในบริบทการให้คะแนน

reward ที่ตั้งง่ายที่สุดคือ `r = −|score_AI − score_teacher|` และมันแตกได้หลายทาง:

| รูปแบบการแฮก | อาการที่จะเห็น | วิธีกัน |
|---|---|---|
| **ถอยเข้าค่ากลาง (regression to the mode)** | โมเดลเรียนรู้ว่าให้ "ระดับกลาง" ทุกครั้งแล้ว MAE ต่ำสุด → MAE สวย แต่ **QWK ≈ 0** | รายงาน QWK คู่ MAE เสมอ (§7.3.1); เพิ่ม penalty กับการกระจายที่แคบเกินจริง |
| **หลักฐานเสื่อมคุณภาพ** | คะแนนตรงขึ้นแต่คำอธิบาย/จุดที่ชี้กลายเป็นน้ำ เพราะ reward ไม่ได้มองส่วนนั้นเลย | ใส่ **spot-check 20 ชิ้นต่อรอบให้มนุษย์ให้คะแนนคุณภาพหลักฐาน** และผูกเป็นเงื่อนไข **K8** (§7.4) |
| **ยาวไว้ก่อน (verbosity)** | ถ้าใช้ LLM-as-judge เป็น reward โมเดลจะเรียนรู้ว่าเขียนยาวได้คะแนนดีกว่า | MT-Bench (arXiv:2306.05685) ระบุ **verbosity bias, position bias, self-enhancement bias** ไว้ตรง ๆ — ต้องควบคุมความยาว และสลับตำแหน่งเมื่อเทียบคู่ |
| **เข้าข้างตัวเอง (self-enhancement)** | ถ้าโมเดลที่ตรวจกับโมเดลที่เป็น judge เป็นตัวเดียวกัน คะแนนจะพองโดยอัตโนมัติ | **ห้ามใช้โมเดลเดียวกันเป็นทั้ง grader และ judge** — ถ้าเลี่ยงไม่ได้ต้องประกาศเป็น limitation |
| **Goodhart ระดับโครงสร้าง** | ยิ่ง optimize proxy หนัก ผลจริงยิ่งแย่ | Gao et al. (arXiv:2210.10760) แสดงว่านี่เป็น **คุณสมบัติเชิงโครงสร้างของ reward model ที่เล็ก/ข้อมูลน้อย ไม่ใช่ปัญหา hyperparameter** → วิธีกันที่ได้ผลจริงคือ **ไม่ optimize หนัก** (best-of-n ที่ n เล็ก, KL penalty, early stopping) |

> **ทางออกที่ปลอดภัยที่สุดในบริบทนี้** (จาก §2.3): ใช้ **verifiable reward ที่คำนวณได้ตรง ๆ** (|score_AI − score_teacher|, exact-match รายเกณฑ์) แทนการเทรน neural reward model — เพราะ *ไม่มีพารามิเตอร์ให้ overfit* จึงไม่มี reward model ให้แฮก เหลือแค่ต้องกันการแฮกเชิงพฤติกรรมในตารางข้างบน

#### 7.7.3 Distribution shift — อาจารย์เปลี่ยน rubric กลางเทอม

**กลไก**: rubric ในวิชาออกแบบมีการปรับถ้อยคำ/เพิ่มเกณฑ์/เปลี่ยนน้ำหนักระหว่างเทอมเป็นเรื่องปกติ เมื่อ rubric เปลี่ยน **คู่คะแนนเก่าไม่ใช่ข้อมูลของงานเดียวกันอีกต่อไป** แต่โค้ดจะยังนับรวมอยู่ดีถ้าไม่ได้ออกแบบไว้

**สิ่งที่ต้องทำตั้งแต่ P0 (ราคาถูกมากถ้าทำก่อน แพงมากถ้าทำทีหลัง)**
- **ให้ rubric มี version id** และผูก `rubric_version` ไว้กับ **ทุก** คู่คะแนน — ไม่ใช่แค่กับ rubric
- **ห้ามรวมข้อมูลข้าม rubric version โดยอัตโนมัติ** ต้องเป็นการตัดสินใจที่มีคนกดยืนยันและมีบันทึก
- **ตรวจ shift แบบเฝ้าระวัง**: plot การกระจายคะแนนอาจารย์รายเกณฑ์ต่อสัปดาห์ และ QWK แบบ rolling window ถ้าตกฮวบพร้อมกันหลายเกณฑ์ ให้สงสัย rubric/นโยบายการให้คะแนนเปลี่ยนก่อนสงสัยโมเดล
- **สำหรับ bandit โดยเฉพาะ**: rubric ที่เปลี่ยนคือ **non-stationary reward** ซึ่ง bandit มาตรฐานรับมือไม่ได้ → ต้องใช้ **discounted / sliding-window** version ของ Thompson sampling หรือ **รีเซ็ต posterior เมื่อ rubric version เปลี่ยน** และเขียนเหตุผลนี้ลงเล่ม (เป็นจุดวิเคราะห์ที่ทำให้ส่วน RL ดูลึกขึ้นมาก)
- **งานเก่าที่ต้อง re-grade**: ต้อง re-grade ด้วย rubric version ปัจจุบันเท่านั้น และติดธงว่าเป็น retrospective — เพราะอาจารย์ที่ตรวจย้อนหลังจำงานได้/มีบริบทต่างจากตอนตรวจสด

#### 7.7.4 กับดักที่เหลือ (สั้น แต่พลาดกันบ่อย)

| กับดัก | วิธีกัน |
|---|---|
| **Test set รั่วผ่านการ tune prompt** — ทีมรัน eval บน test set ทุกครั้งที่แก้ prompt จนกลายเป็นการ fit test set ด้วยมือ | แยก dev/test ตั้งแต่ P0, **แตะ test set ได้ไม่เกิน 3 ครั้งตลอดโครงงาน** และบันทึกทุกครั้งที่แตะ |
| **การกระจายคะแนนเบ้จนดูเก่งเกินจริง** — ถ้า 70% ของคะแนนอยู่ระดับเดียว accuracy 70% ได้ฟรี | ต้องมี baseline B0 (majority class) เสมอ (§7.3.3) |
| **อาจารย์กด accept รัว ๆ เพราะเหนื่อย** (rubber-stamping) | อย่าใช้ accept-rate เป็น reward เดี่ยว ๆ; ตรวจว่าเวลาที่ใช้ต่อชิ้นต่ำผิดปกติไหม; ให้ blind sample เป็นตัวสอบทาน |
| **`ScoreEdit` ที่ไม่มีการแก้ = ไม่มีข้อมูล** — เก็บแค่ตอน "ต่าง" ทำให้ dataset เอียงไปทางเคสที่ AI ผิด | ต้องเก็บ **ทุกการตรวจ** รวมทั้งที่เห็นตรงกัน ไม่ใช่เฉพาะที่ต่าง ไม่งั้นคำนวณ QWK ไม่ได้เลย |
| **เหตุผลที่อาจารย์บังคับเขียน กลายเป็นข้อความสั้น ๆ ซ้ำ ๆ** | ยอมรับว่าจะเกิด และวางแผนใช้เป็น **สัญญาณเชิงคุณภาพ** (จัดกลุ่มเหตุผลเพื่อหา failure mode ของ AI) มากกว่าใช้เป็น training text |
| **PPO/GRPO ถูกหยิบมาทำเพราะ "ฟังดูเป็น RL มากกว่า"** | ตอบด้วย §5.2: contextual bandit **เป็น RL จริงตามนิยาม** และ GEPA (ICLR 2026 Oral) ชนะ GRPO ด้วย rollout น้อยกว่า 35 เท่า — ทางเลือกนี้มีวรรณกรรมหนุน ไม่ใช่ทางลัด |
| **ลืม `modules_to_save=["score"]` ตอนใช้ LoRA กับ reward head** | ระบุไว้แล้วใน §2.1 — reward head จะไม่ถูกเทรนเลยและผลจะเป็น noise ล้วน โดยไม่มี error แจ้ง |
