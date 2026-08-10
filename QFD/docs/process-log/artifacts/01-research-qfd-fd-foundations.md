# QFD → Functional Decomposition Pipeline for Undergraduate Software + AI Senior Projects

**Research summary for course design (student guidelines, advisor guidelines, rubrics, self-check prompts).**
Date: 2026-07-14. Prepared for a Computer Engineering senior-project course.

## How to read this document

- Each of the six required questions has its own section, followed by a **"Recommended pipeline for this course"** synthesis and a **"Sources"** list.
- **Citation integrity note.** Most factual claims below are grounded in primary sources identified by author/title/year/standard-number and URL. Several of those sources are paywalled standards (ISO, IEEE, ACM, HBR) that could not be read in full; where that is the case I cite the source by identity and use only well-established characterizations of its content. I do **not** attribute invented clause numbers, page numbers, or verbatim quotations to sources I could not read in full.
- **Synthesis is labelled.** The ID scheme, the worked micro-example, the reduced "pragmatic" version, and the recommended stage list are my own synthesis for this course context and are marked **[COURSE SYNTHESIS]**. They are defensible engineering choices, not claims about what any cited author wrote.
- **One example threads the whole document.** To make the pipeline teachable, a single plausible project — **"StudyMate," a study-assistant web app whose core is an ML classifier that labels a practice question as Easy / Medium / Hard** — is traced from customer requirement all the way to a verification test in every section. Watch the IDs `CR-01 → TC-02 → QR-03 → F-04 → C-05 → T-06` reappear.

---

## Q1. The canonical manufacturing QFD sequence

### 1.1 Origin and the "House of Quality"

Quality Function Deployment originated in Japan in the late 1960s–1972, developed by **Yoji Akao** and **Shigeru Mizuno**, with early application at Mitsubishi's Kobe shipyard. The method was introduced to a wide Western management audience by **John R. Hauser and Don Clausing, "The House of Quality," *Harvard Business Review*, May–June 1988** (hbr.org/1988/05/the-house-of-quality). Hauser & Clausing describe the House of Quality as a conceptual map / matrix on which a cross-functional team organizes customer needs and the engineering decisions that will meet them, and set target values for design.

### 1.2 The classic front-end flow (Voice of Customer → prioritized technical characteristics)

The canonical sequence that a manufacturing QFD team runs through:

1. **Voice of the Customer (VoC).** Collect raw customer statements, in customer language, through interviews, surveys, observation, complaints.
2. **Affinity / KJ diagram.** Group the raw VoC statements into a hierarchy of themes (the KJ method, after Jiro Kawakita). This yields structured **customer requirements ("WHATs")** at a manageable level of abstraction.
3. **Prioritize / weight the customer requirements.** Assign an importance weight to each WHAT — either by simple direct rating (e.g., 1–5, or 100-point distribution) or by a pairwise method such as the **Analytic Hierarchy Process (AHP)** (see 1.4).
4. **Build the House of Quality (HoQ)** — see 1.3 for its "rooms."
5. **Output: prioritized technical characteristics ("HOWs").** Multiplying customer-requirement weights through the relationship matrix produces an importance score for each engineering characteristic, telling the team which technical parameters matter most.

### 1.3 The rooms of the House of Quality

The HoQ is drawn as a house because it stacks several sub-matrices ("rooms"):

- **Left wall — WHATs:** the prioritized customer requirements, each carrying an importance weight.
- **Ceiling — HOWs:** the engineering / technical characteristics (measurable design parameters) the team can act on.
- **Body — Relationship matrix:** for each (WHAT × HOW) cell, the strength of relationship (commonly Strong / Medium / Weak, weighted e.g. 9/3/1).
- **Roof — Correlation matrix:** the triangular matrix on top showing how the HOWs interact with **each other** (synergy `+` vs. trade-off/conflict `−`). This is where design trade-offs are surfaced.
- **Right wall — Competitive / planning assessment:** how the product and competitors are rated by customers against each WHAT (the "customer competitive assessment"), plus planning columns (target, improvement ratio, sales point).
- **Basement / floor — "How much" (target values) and technical benchmarking:** for each HOW, its target value / measurable objective, plus a technical competitive comparison and the computed **importance of each HOW**.

### 1.4 Prioritizing customer requirements: simple weighting vs. AHP

- **Simple weighting** (direct rating or point allocation) is fast and is the norm in lightweight/software use.
- **AHP (Saaty)** derives weights from pairwise comparisons and yields a consistency check; it has been repeatedly recommended and studied specifically for prioritizing the customer voice in the HoQ (e.g., the QFD-and-AHP literature reviewed in *IJAHP* and *IIE Transactions*; fuzzy-AHP variants exist for imprecise judgments). Trade-off: AHP is more rigorous but requires *n(n−1)/2* comparisons, which grows quickly. **[COURSE SYNTHESIS: for a solo/pair student project, direct 1–5 rating or 100-point allocation is usually sufficient; AHP is worth teaching as an optional rigor upgrade when there are ~5–8 requirements and a real client to interview.]**

### 1.5 The four-phase QFD model — and the fact that its naming varies

Full QFD does not stop at one house. It cascades the outputs of one matrix into the inputs (left wall) of the next, forming a **matrix-of-matrices** across four phases. **Important caveat the course should teach honestly: the phase names are not standardized across sources.** Common renderings:

| Phase | "Four Houses" / ASI-style rendering | Alternative rendering seen in the literature |
|---|---|---|
| 1 | **House of Quality** — customer requirements → technical/design characteristics | Product Planning |
| 2 | **Parts deployment** — technical characteristics → part/component characteristics | Product / Part Design deployment |
| 3 | **Process planning** — part characteristics → key process operations | Process Planning |
| 4 | **Production / process control planning** — process operations → production/control requirements | Production Planning |

The through-line is constant even though labels differ: **the HOWs of one house become the WHATs of the next**, so priority "flows down" from customer voice to shop-floor control. Cite: Akao's QFD (matrix-of-matrices) tradition; the "four houses" popularization associated with the American Supplier Institute (ASI) and GOAL/QPC training materials; Hauser & Clausing (1988) for phase 1. Because the naming is genuinely inconsistent in the literature, the course should **pick one labelling and state that others exist** rather than present any single version as canonical.

**StudyMate trace, Q1:** VoC statement *"I want practice that isn't too easy or too hard for me"* → affinity-grouped into customer requirement **CR-01 "Questions are matched to my skill level"** (weight 5/5). In the HoQ, CR-01 relates strongly to the technical characteristic **TC-02 "Difficulty-classification accuracy"** and to "response latency." The roof shows a trade-off: a heavier, more accurate model conflicts with low latency.

---

## Q2. Software QFD adaptation

### 2.1 The core reframing: software is valued for what it *does*

Software QFD keeps QFD's spine (VoC → weighted requirements → matrix → prioritized characteristics) but reinterprets the content. **Richard Zultner** — the principal popularizer of Software QFD — frames the shift with the maxim that *software is valued not for what it is, but for what it does*, so **function deployment** (what the software must do) sits alongside **quality deployment** (how well). Zultner also created **Blitz QFD**, a deliberately reduced subset for projects constrained in time, people, and money, focused on quickly identifying the highest-value customer needs, product characteristics, and project tasks (see Q7 for why this matters to students). Cite: Zultner, R., writings on Software QFD / Blitz QFD (1990s); overview in Herzwurm/Schockert materials below.

Primary and named sources for Software QFD:
- **Georg Herzwurm, Sixten Schockert, Werner Mellis — *Joint Requirements Engineering: QFD for Rapid Customer-Focused Software Development*** (Vieweg; ISBN 3-528-05736-X / 3-528-15577-9). Thesis: the RE process must integrate customer and developer know-how and focus on *only the features important for success*, not everything technically possible. Also Herzwurm & Schockert, "QFD for customer-focused requirements engineering" (IEEE). A first-party University of Stuttgart summary is at qfd-id.de/wp-content/uploads/2020/02/qfd_software_english.pdf.
- **Stephen Haag, M. K. Raja, L. L. Schkade, "Quality function deployment usage in software development," *Communications of the ACM* 39(1), 1996, pp. 41–49** (dl.acm.org/doi/10.1145/234173.234178) — an empirical study of QFD-in-software practice and of the barriers to adoption.
- The Stuttgart material notes that software teams commonly recognize **Zultner, Shindo, Ohmori, and Herzwurm/Schockert** as the established software-specific QFD models.

*(Barnett & Raja and various Liu et al. QFD-software papers exist in this line; they are not cited individually here because I could not verify their exact bibliographic details against a primary source in this pass. The course should verify before citing them by name.)*

### 2.2 What maps naturally

| Manufacturing QFD element | Natural software mapping |
|---|---|
| Voice of Customer | user/stakeholder needs, user stories, complaints, market asks |
| Customer requirements (WHATs) | desired **features** and **quality attributes** (usability, performance, reliability…) |
| Technical characteristics (HOWs) | measurable **software requirements**: functional requirements, and especially **quantifiable non-functional requirements** (response time, error rate, availability %) |
| "How much" / target values | measurable acceptance targets on those requirements |
| Cascading houses | feature → function → module/design → test-plan deployment |

A clean and teachable move: map the HoQ's HOWs onto the quality characteristics of **ISO/IEC 25010:2023 (SQuaRE product-quality model)** — functional suitability, performance efficiency, compatibility, usability, reliability, security, maintainability, portability, **and safety (added in the 2023 edition; the 2011 edition had eight characteristics, 2023 has nine)**. This gives students a ready-made, standardized vocabulary of technical characteristics instead of inventing ad-hoc ones. Cite: ISO/IEC 25010:2023 (iso.org/standard/78176.html).

### 2.3 What is dropped or awkward

- **The correlation "roof" is often simplified or dropped.** In software, trade-off relationships between technical characteristics (e.g., accuracy vs. latency, security vs. usability) are real but harder to pin down than physical engineering conflicts, so software practitioners frequently reduce or omit the roof. **[COURSE SYNTHESIS: keep a *lightweight* roof for AI projects specifically — the accuracy/latency/cost/interpretability trade-offs are pedagogically valuable and genuinely constrain the design.]**
- **Manufacturing "parts deployment" and the production/process-control phases (phases 3–4) map poorly.** There is no physical bill-of-materials or shop-floor process. These are usually collapsed into **design/architecture deployment** and **build/test deployment**.
- **Heavy multi-matrix cascades are usually too much** for a student team; Blitz QFD exists precisely because comprehensive QFD is impractical under tight constraints.

### 2.4 What is added or reinterpreted

- HOWs are reinterpreted as a mix of **functional requirements (FRs)** and **non-functional requirements (NFRs)**; use cases / user stories become the functional carrier.
- Quality characteristics are anchored to **ISO/IEC 25010** subcharacteristics, making them measurable.
- The key teachable output: **every technical characteristic must become a *measurable software requirement* with a target value** (this is exactly the bridge into Q4/Q5's "Quantifiable Requirement").

**StudyMate trace, Q2:** CR-01 (customer requirement) maps to technical characteristic **TC-02 "Difficulty-classification accuracy,"** which is an ISO/IEC 25010 *functional suitability / correctness*-flavored characteristic. Its sibling HOW is *performance efficiency → time behavior* (latency). The roof records the accuracy↔latency trade-off. TC-02 is not yet testable until it gets a target value — that happens in Q4.

---

## Q3. Functional decomposition after technical requirements

### 3.1 From prioritized characteristics to a function structure

Once the HoQ has produced prioritized technical characteristics, the design question becomes *what must the system **do*** to meet them. Functional decomposition answers this by breaking the system's **overall function** into a hierarchy of **sub-functions**.

- **Systematic engineering design — Gerhard Pahl & Wolfgang Beitz, *Engineering Design: A Systematic Approach*** (Springer; later editions with Feldhusen & Grote). Conceptual design *"establishes function structures"* by decomposing the overall function into sub-functions according to the flow of **energy, material, and signals**. The clarification phase produces a **requirements/specification list** that drives this. This is the classic engineering-design justification for a **function tree**.
- **IDEF0 activity decomposition — NIST FIPS PUB 183, *Integration Definition for Function Modeling (IDEF0)*** (issued Dec 1993, effective 1994; withdrawn as a mandatory federal standard in 2008 but still widely used and referenced). IDEF0 models a function as a **box** with **ICOM arrows** — **I**nputs (left), **C**ontrols (top), **O**utputs (right), **M**echanisms (bottom). *Decomposition* is the partitioning of a parent function into its component functions on a child diagram, giving a strict parent→child hierarchy with numbered nodes (A0 → A1, A2, A3 …). This node numbering is itself a ready-made traceability ID scheme. Cite: FIPS 183 (csrc.nist.gov / everyspec.com FIPS_PUB_183).

### 3.2 Software decomposition and the FR/NFR distinction

For the software portion, decomposition becomes **feature → function → sub-function → component/module**:

- **David L. Parnas, "On the Criteria To Be Used in Decomposing Systems into Modules," *Communications of the ACM* 15(12), 1972, pp. 1053–1058** (dl.acm.org/doi/10.1145/361598.361623). Parnas's key result: decompose modules around **information hiding** — each module hides a *design decision likely to change* — rather than around the steps of a flowchart. This is the criterion that turns a function tree into a *good* module structure: functions that share a volatile secret (a data format, an algorithm choice, a model) belong in one module.
- **Functional requirements (FRs)** — what the system does — decompose naturally into the function tree (each leaf function realizes one or more FRs).
- **Non-functional requirements (NFRs)** — how well (performance, security, reliability…) — are generally **cross-cutting**: they constrain many functions rather than becoming a single leaf. Teach students to attach NFRs as **constraints/targets on functions or on the architecture**, not as tree leaves. This aligns with the ML-NFR literature (Q4).

### 3.3 Preserving the upward mapping

The decomposition is only useful if each function still points *up* to the requirement it satisfies. Two mechanisms:

1. **Hierarchical node IDs** (IDEF0 A1.2.3 style, or a `F-xx` scheme) so parentage is explicit.
2. **A "satisfies" link** recorded per function back to its originating technical characteristic / Quantifiable Requirement (this is the traceability spine of Q6). **[COURSE SYNTHESIS: require every function node to name the requirement ID it derives from; a function with no upward link is either dead work or a missing requirement.]**

**StudyMate trace, Q3:** TC-02 "classification accuracy" drives function **F-04 "Classify question difficulty."** F-04 decomposes into sub-functions: F-04.1 *extract question features*, F-04.2 *run difficulty model*, F-04.3 *map model output to Easy/Medium/Hard label*, F-04.4 *return label + confidence*. By Parnas's criterion, the *model itself* (the volatile design decision — the algorithm will change) is hidden behind F-04.2's module boundary, so swapping models doesn't ripple. Each sub-function records "satisfies: TC-02".

---

## Q4. Where AI/ML semi-research projects fit

### 4.1 A model metric IS a target value / Quantifiable Requirement in the HoQ

Yes — for an AI/ML core, a model performance metric (accuracy, F1, precision/recall, AUC, latency, throughput, calibration) is best treated as the **"how much" / target value** attached to a technical characteristic in the House of Quality, i.e., a **Quantifiable Requirement (QR)**. This is the clean way to make an otherwise fuzzy "the model should be good" into something that satisfies the ISO/IEC/IEEE 29148 quality criterion of being **verifiable/measurable** (Q5).

Practical rule for students: **a technical characteristic (TC) names the dimension; the Quantifiable Requirement (QR) states the metric + operator + threshold + dataset + condition.**

> QR-03: *On the held-out StudyMate test set (n ≥ 500, stratified by topic), the difficulty classifier achieves **macro-F1 ≥ 0.80** and **median inference latency ≤ 200 ms** on the target hardware.*

Note the components that make it testable: **metric** (macro-F1), **operator/threshold** (≥ 0.80), **dataset/population** (held-out, stratified, sized), and **operating condition** (target hardware, latency). Omit any of these and the requirement is not verifiable.

### 4.2 Where training and experiments sit

- **Training and experimentation are implementation/realization activities**, not requirements activities. In pipeline terms they sit **inside the function that owns the model** (StudyMate's F-04.2) during the build phase.
- The **requirement** (QR-03) is fixed up front as the acceptance bar; **experiments are the means of hitting it**. This distinction is exactly the "status quo" finding of RE-for-ML surveys: RE for ML is often done informally inside notebooks by data scientists, and NFRs cluster around **data quality, model reliability, and explainability** — which is precisely why forcing an explicit, testable target up front adds value. Cite: "Status Quo and Problems of Requirements Engineering for Machine Learning: Results from an International Survey" (Springer, link.springer.com/chapter/10.1007/978-3-031-49266-2_11); Habibullah et al., "Non-functional requirements for machine learning: understanding current use and challenges among practitioners" (arxiv.org/abs/2109.00872).

### 4.3 ML metrics as verification / acceptance criteria

An ML evaluation metric doubles as an **acceptance test**: the model is "accepted" when it meets QR-03 on a *frozen, representative* evaluation set. Teach two guardrails:

- **Data quality is a first-class requirement.** A metric target is meaningless on bad data. Anchor data expectations to **ISO/IEC 25012 (Data Quality model)** — accuracy, completeness, consistency, credibility, currentness (inherent), plus system-dependent characteristics (iso25000.com/.../iso-25012). Students should state a data-quality requirement alongside the metric.
- **Evaluation must be honest and reproducible.** Held-out/test split declared, no leakage, fixed seed, versioned data. Documentation practices such as **model cards** (Mitchell et al., 2019) and **datasheets for datasets** (Gebru et al., 2018) give students a lightweight, citable way to record the model's intended use, evaluation data, and measured performance as evidence for the acceptance decision.

### 4.4 Stating an ML target as a testable requirement — the student template

**[COURSE SYNTHESIS]** Give students this fill-in-the-blanks form:

> On **[named evaluation dataset, size, how sampled]**, the **[model/function]** shall achieve **[metric] [≥/≤] [threshold]** under **[operating condition/hardware]**, measured by **[evaluation procedure/script]**, with the dataset meeting **[data-quality condition]**.

A "research/stretch" flavor is allowed for semi-research projects: pair a **must-pass** threshold (the acceptance bar) with a **target/stretch** threshold (the research ambition), so a project is not judged a failure for missing an aspirational number while still hitting a credible bar.

**StudyMate trace, Q4:** TC-02 gets its target as **QR-03 (macro-F1 ≥ 0.80, latency ≤ 200 ms, on a stratified held-out set ≥ 500)**. Training experiments (feature choices, model family, hyperparameters) happen inside F-04.2 during the build term; QR-03 is the bar they must clear.

---

## Q5. Verification closes the traceability loop

### 5.1 Every Quantifiable Requirement maps to a verification

The rule: **each QR is paired with a verification action that produces objective evidence it is met.** The four standard verification methods (INCOSE Systems Engineering Handbook; ISO/IEC/IEEE 15288 and 12207 life-cycle processes) are:

- **Test** — exercise the system under controlled conditions and measure (e.g., run the eval script, measure macro-F1).
- **Demonstration** — operate the system to show a capability (e.g., show the app returning a difficulty label live).
- **Inspection** — examine the artifact/code/docs (e.g., confirm a config, read the model card).
- **Analysis** — reason/model/simulate where direct test is impractical (e.g., latency budget analysis, statistical confidence intervals on the metric).

*(Note: some secondary summaries garble this list — the correct four are Test, Demonstration, Inspection, Analysis; a fifth, "Analysis by similarity," is sometimes broken out. Cite by identity: INCOSE SE Handbook; ISO/IEC/IEEE 15288.)*

### 5.2 "Verifiable" is a quality criterion for a good requirement

**ISO/IEC/IEEE 29148** (Systems and software engineering — Requirements engineering) defines characteristics of a well-formed requirement — commonly rendered as **necessary, appropriate, unambiguous, complete, singular, feasible, verifiable, correct, conforming**, and (as a set) **consistent, complete, traceable**. On verifiability: a requirement is **verifiable if and only if there exists a finite, cost-effective process by which a person or machine can check that the system meets it**, and **verifiability is enhanced when the requirement is stated in measurable terms.** This is the standards-backed reason the course insists every QR carry a metric + threshold. Cite: ISO/IEC/IEEE 29148:2011 (iso.org/standard/45171.html); 2018 revision exists.

### 5.3 Requirements Traceability Matrix (RTM) and acceptance criteria

- The **Requirements Traceability Matrix (RTM)** is the artifact that records, per requirement, its source (upstream) and its verification (downstream), ensuring **end-to-end coverage** — no requirement without a test, no test without a requirement. Cite: INCOSE SE Handbook; 29148.
- **Acceptance criteria** are the pass/fail conditions on each QR; meeting all acceptance criteria for the requirements allocated to a build is what closes the loop. For ML, the acceptance criterion *is* the metric threshold on the frozen eval set (Q4).

**StudyMate trace, Q5:** QR-03 is verified by **T-06 "Difficulty-classifier evaluation test"** — method: **Test** (run `eval.py` on the frozen held-out set; assert macro-F1 ≥ 0.80) plus **Analysis** (report a 95% CI on F1) and **Demonstration** (live labelling in the UI, method: Demonstration for the latency-in-context claim). T-06's result row in the RTM points back to QR-03, which points back to TC-02, which points back to CR-01. The loop is closed.

---

## Q6. The traceability spine

### 6.1 The end-to-end chain

The whole pipeline is bound by a single directed chain in which each artifact **references the one before it**:

```
Customer Requirement ──(weight)──▶ HoQ Technical Characteristic ──▶ Quantifiable Requirement ──▶ Function (decomposition) ──▶ Component/Module ──▶ Test/Verification
        CR-01                              TC-02                             QR-03                        F-04                        C-05                     T-06
```

Read upward it answers "*why does this test exist?*"; read downward it answers "*is this customer need actually delivered and proven?*". This bidirectional traceability — from stakeholder need through to verification — is exactly what **ISO/IEC/IEEE 29148** (traceable as a requirement characteristic) and the **INCOSE Systems Engineering Handbook** (RTM / requirements traceability practice) call for.

### 6.2 A unique ID scheme that makes the spine mechanical

**[COURSE SYNTHESIS — proposed for this course]** Give every artifact a typed, numbered ID, and require each row to name its parent ID:

| Prefix | Artifact | References upward |
|---|---|---|
| `CR-xx` | Customer Requirement (weighted) | — (root; from VoC) |
| `TC-xx` | HoQ Technical Characteristic | one or more `CR` |
| `QR-xx` | Quantifiable Requirement (metric + target) | its `TC` |
| `F-xx` | Function / sub-function (decomposition node) | the `QR`/`TC` it satisfies |
| `C-xx` | Component / module | the `F`(s) it implements |
| `T-xx` | Test / verification action (+ method) | the `QR` it verifies |

Because each ID names its parent, the RTM can be **generated and checked mechanically**: any `CR` with no downstream `T` is an unverified promise; any `T` with no upstream `QR` is gold-plating; any `F` with no `QR`/`TC` is scope creep. This turns "traceability" from a documentation chore into a rubric the advisor can grade in minutes.

### 6.3 The complete StudyMate spine (worked micro-example)

| ID | Artifact | Content | ↑ Parent |
|---|---|---|---|
| CR-01 | Customer Requirement (wt 5/5) | "Questions are matched to my skill level" | VoC |
| TC-02 | Technical Characteristic | Difficulty-classification accuracy (ISO 25010 functional correctness); sibling: latency | CR-01 |
| QR-03 | Quantifiable Requirement | macro-F1 ≥ 0.80 AND median latency ≤ 200 ms on stratified held-out set (n ≥ 500); data meets ISO 25012 accuracy/completeness | TC-02 |
| F-04 | Function | "Classify question difficulty" → F-04.1 extract features, F-04.2 run model (hides the model — Parnas), F-04.3 map to label, F-04.4 return label+confidence | QR-03 |
| C-05 | Component | `difficulty_classifier` module (model artifact + inference service) | F-04 |
| T-06 | Test (Test + Analysis + Demonstration) | `eval.py` asserts macro-F1 ≥ 0.80 with 95% CI; live demo checks in-context latency | QR-03 |

---

## Q7. Recommended pipeline for this course (3-term software/AI senior project)

**[COURSE SYNTHESIS]** A pragmatic, teachable, reduced-QFD pipeline. It keeps QFD's spine and traceability while dropping manufacturing baggage that does not serve a solo/pair student team. Reduced version rationale is grounded in Zultner's **Blitz QFD** philosophy (comprehensive QFD is impractical under tight time/people/money) and Herzwurm/Schockert's *"only the features important for success."*

### Stage list (name → artifact produced → owning term)

| # | Stage name | Artifact(s) produced | Term |
|---|---|---|---|
| S0 | **Project framing & stakeholder identification** | problem statement, stakeholder list, client contact | 1 |
| S1 | **Voice of Customer capture** | raw VoC log (interview notes, user stories) | 1 |
| S2 | **Affinity grouping & requirement structuring** | KJ/affinity diagram → `CR-xx` list | 1 |
| S3 | **Prioritize customer requirements** | weighted `CR` table (direct rating; AHP optional) | 1 |
| S4 | **Reduced House of Quality** | HoQ with WHATs, HOWs (`TC-xx` mapped to ISO 25010), relationship matrix, *lightweight* roof (esp. AI trade-offs), target column | 1 |
| S5 | **Quantifiable Requirements** | `QR-xx` spec (metric+target+dataset+condition; ML targets per Q4 template; data-quality reqs per ISO 25012) | 1→2 |
| S6 | **Functional decomposition** | function tree / IDEF0-style model → `F-xx` nodes, each linked to a `QR`/`TC` | 2 |
| S7 | **Architecture / module mapping** | components `C-xx` (Parnas information-hiding), FR/NFR allocation | 2 |
| S8 | **Implementation + ML experimentation** | working system; for AI core: training/eval experiments inside model-owning function; model card | 2→3 |
| S9 | **Verification & RTM close-out** | `T-xx` tests (Test/Demo/Inspect/Analyze), completed RTM, acceptance report | 3 |
| S10 | **Validation & defense** | demo, results vs. targets, traceability walkthrough | 3 |

### Where classic QFD is overkill (and the pragmatic reduction)

- **Full four-phase matrix-of-matrices → collapse to ONE reduced House of Quality.** Parts/process/production-control phases have no software analog; fold "design deployment" into the functional decomposition (S6) and "build/test deployment" into S8–S9.
- **AHP for weighting → default to direct 1–5 rating or 100-point allocation.** Teach AHP only as an optional rigor upgrade with a real client.
- **Full correlation roof → keep only a lightweight roof, and mainly for AI trade-offs** (accuracy vs. latency vs. cost vs. interpretability), which are genuinely design-shaping.
- **Large VoC surveys → a handful of real stakeholder interviews** are enough for a solo/pair scope (Blitz-QFD spirit).
- **The one thing NOT to cut: the ID'd traceability spine (Q6).** It is cheap, it is gradable, and it is the pedagogical core — it forces students to justify every line of work by a customer need and prove every need with a test.

### Self-check prompts this enables (preview for the rubric work)

- Does every `CR` have a weight and at least one downstream `T`? (coverage)
- Is every `QR` measurable — metric + operator + threshold + dataset + condition? (29148 verifiability)
- Does every `F` name the `QR`/`TC` it satisfies? (no orphan functions)
- For the AI core: is there a frozen eval set, a stated data-quality condition, and a must-pass vs. stretch target? (Q4)
- Does the RTM close every loop CR→…→T with no orphans at either end? (Q6)

---

## Sources

**QFD origins & House of Quality**
- J. R. Hauser & D. Clausing, "The House of Quality," *Harvard Business Review*, May–June 1988. https://hbr.org/1988/05/the-house-of-quality
- Y. Akao and S. Mizuno — QFD origin (Mitsubishi Kobe shipyard, late 1960s–1972); matrix-of-matrices / four-phase QFD tradition. Overview: "Quality Function Deployment: A Comprehensive Review of Its Concepts and Methods" (ResearchGate 245495796); "The history of the QFD method" (ResearchGate 321388254).
- American Supplier Institute (ASI) / GOAL-QPC — "four houses" popularization (training-material tradition; naming varies, see Q1.5).

**Prioritization**
- QFD + AHP joint application review, *IJAHP* / *IIE Transactions* (e.g., tandfonline.com/doi/abs/10.1080/07408179408966620; fuzzy-AHP: tandfonline.com/doi/abs/10.1080/07408170304355). T. Saaty's AHP underlies these.

**Software QFD**
- R. Zultner — Software QFD and Blitz QFD (1990s); "software is valued not for what it is but for what it does."
- G. Herzwurm, S. Schockert, W. Mellis, *Joint Requirements Engineering: QFD for Rapid Customer-Focused Software Development*, Vieweg (ISBN 3-528-05736-X). Stuttgart summary: https://qfd-id.de/wp-content/uploads/2020/02/qfd_software_english.pdf
- S. Haag, M. K. Raja, L. L. Schkade, "Quality function deployment usage in software development," *Communications of the ACM* 39(1), 1996, 41–49. https://dl.acm.org/doi/10.1145/234173.234178
- ISO/IEC 25010:2023, *SQuaRE — Product quality model*. https://www.iso.org/standard/78176.html

**Functional decomposition**
- G. Pahl & W. Beitz, *Engineering Design: A Systematic Approach*, Springer. https://books.google.com/books/about/Engineering_Design.html?id=4uvSBwAAQBAJ
- NIST FIPS PUB 183, *Integration Definition for Function Modeling (IDEF0)*, 1993/1994 (withdrawn as federal standard 2008). http://everyspec.com/NIST/NIST-FIPS/FIPS_PUB_183_3469/ ; https://www.idef.com/idefo-function_modeling_method/
- D. L. Parnas, "On the Criteria To Be Used in Decomposing Systems into Modules," *Communications of the ACM* 15(12), 1972, 1053–1058. https://dl.acm.org/doi/10.1145/361598.361623

**AI/ML requirements & data quality**
- "Status Quo and Problems of Requirements Engineering for Machine Learning: Results from an International Survey," Springer 2023. https://link.springer.com/chapter/10.1007/978-3-031-49266-2_11
- K. M. Habibullah et al., "Non-functional requirements for machine learning: understanding current use and challenges among practitioners," 2021/2022. https://arxiv.org/pdf/2109.00872 ; workshop version: https://dl.acm.org/doi/10.1145/3526073.3527589
- ISO/IEC 25012, *Data Quality model*. https://iso25000.com/index.php/en/iso-25000-standards/iso-25012
- M. Mitchell et al., "Model Cards for Model Reporting," FAT* 2019; T. Gebru et al., "Datasheets for Datasets," 2018/2021 (documentation/evaluation practice).

**Verification, traceability, life-cycle standards**
- ISO/IEC/IEEE 29148, *Systems and software engineering — Requirements engineering* (2011; rev. 2018). https://www.iso.org/standard/45171.html
- ISO/IEC/IEEE 15288 (system life-cycle processes) and ISO/IEC/IEEE 12207 (software life-cycle processes).
- INCOSE, *Systems Engineering Handbook* — verification methods (Test/Demonstration/Inspection/Analysis) and requirements traceability / RTM. https://www.incose.org/ (V&V tutorial: https://www.incose.org/docs/default-source/enchantment/200827-miller-v-v-tutorial.pdf)

---

*End of research summary. Synthesis items (ID scheme, StudyMate worked example, reduced pipeline, self-check prompts) are course-design proposals, not claims about cited authors.*
