# REPORT — what the measurement showed, who it affects, and whose responsibility it is

## 1. Measurement read-out

Across our deterministic greedy-decoding evaluation of 200 matched pairs (400 synthetic applicant profiles, `n = 40` pairs per group, `0/400` unparseable replies, `356/400` or `89.0%` baseline shortlist rate, `44/400` rejected), binary outcome flips and continuous score shifts diverged sharply in what they revealed:

* **`name_language` (`n = 40` pairs vs. `null_name` floor):**
  * Binary flips: `2/40` (`5.0%`, 95% CI `[1.4%, 16.5%]`) vs. `null_name` floor `0/40` (`0.0%`, 95% CI `[0.0%, 8.8%]`).
  * Continuous score shifts: `11/40` pairs differed in score, with `mean |b - a| = 3.12` (95% CI `[1.38, 5.25]`). Oriented signed score gaps showed no statistically significant directional bias: Kazakh vs. other (`n = 28`): `+0.7` (`[-1.8, +3.8]`, half-width `±2.8`, sign test `p = 1.000`); Russian vs. other (`n = 26`): `+0.2` (`[-1.7, +2.1]`, half-width `±1.9`, `p = 1.000`); English vs. other (`n = 26`): `-1.0` (`[-4.2, +1.9]`, half-width `±3.1`, `p = 1.000`).

* **`gender_coded_name` (`n = 40` pairs vs. `null_name` floor):**
  * Binary flips: `4/40` (`10.0%`, 95% CI `[4.0%, 23.1%]`, all 4 flips favored female-coded names; exact McNemar `p = 0.125`) vs. `null_name` floor `0/40` (`0.0%`, 95% CI `[0.0%, 8.8%]`).
  * Continuous score shifts: `6/40` pairs differed in score, with `mean |b - a| = 1.12` (95% CI `[0.25, 2.13]`). Oriented signed score gap (male-coded vs. female-coded, `n = 40`): `-0.4` (`[-1.2, +0.6]`, half-width `±0.9`, sign test `p = 0.219`), with male-coded names shortlisted at `80.0%` (mean `61.5`) vs. `90.0%` (mean `61.9`) for female-coded names.

* **`university` (`n = 40` pairs vs. `null_university` floor):**
  * Binary flips: `1/40` (`2.5%`, 95% CI `[0.4%, 12.9%]`, flagship favored; McNemar `p = 1.000`) vs. `null_university` floor `1/40` (`2.5%`, 95% CI `[0.4%, 12.9%]`).
  * Continuous score shifts: `10/40` pairs differed in score, with `mean |b - a| = 2.50` (95% CI `[1.12, 4.12]`). Oriented signed score gap (flagship vs. regional university, `n = 40`): **`+2.5` points** (95% CI `[+1.1, +4.1]`, half-width `±1.5`, **sign test `p = 0.002`**), favoring flagship universities (mean `63.2`, `92.5%` shortlisted) over identical regional university applicants (mean `60.8`, `90.0%` shortlisted).

* **Baseline Noise Floors (`null_name` and `null_university`, `n = 40` pairs each):**
  * `null_name` (swapping two names from the same language and gender group): `0/40` binary flips (`0.0%` `[0.0%, 8.8%]`), `7/40` score differences (`mean |b - a| = 2.00` `[0.62, 3.62]`).
  * `null_university` (swapping two universities from the same prestige tier): `1/40` binary flip (`2.5%` `[0.4%, 12.9%]`), `9/40` score differences (`mean |b - a| = 2.12` `[0.88, 3.62]`).

* **Binary Decisions vs. Continuous Score Shifts**: Against their respective null floors, **no attribute cleared the binary flip threshold**: `name_language` (`2/40`, `5.0%`), `gender_coded_name` (`4/40`, `10.0%`), and `university` (`1/40`, `2.5%`) all had 95% confidence intervals overlapping `null_name` (`0/40`) and `null_university` (`1/40`). However, examining continuous scores exposed a **statistically significant institutional prestige effect**: `university` moved scores by **+2.5 points** (`p = 0.002`) in favor of flagship universities over identical regional university applicants.

* **What Noise Floors Test, What They Leave Untested, and Statistical Power**: Even under greedy decoding, changing tokens inside the same demographic or institutional tier altered scores in `7/40` `null_name` pairs and `9/40` `null_university` pairs. These floors isolate **within-tier lexical/tokenization sensitivity** while holding all merit fields (`program`, `gpa`, `exam_score`, `household_income_bracket`, `achievements`, `age`, `consent`) fixed. They leave untested **intersectional combinations** (e.g., regional university + male-coded name + low income), **prompt-template phrasing variations**, and **bias carried through free-text achievements**. Under an exact two-sided McNemar test (`alpha = 0.05`), `n = 40` pairs requires at least **6 one-sided flips (`15.0%`)** to reach significance. Therefore, finding no significant binary divergence for `gender_coded_name` (`4/40`, `10.0%`, McNemar `p = 0.125`) or `name_language` (`2/40`, `5.0%`) **does not prove fairness or parity**—it only rules out binary flip disparities larger than ~15%; a stronger estimate would require `n >= 200–400` pairs per attribute and multi-template prompt ensembles.

* **Effect of Enabling the Harness (`validate_input`, `blind=("name", "university")`, `refuse`, `log`, `escalate`)**: With the harness active across the same 400 profiles, binary flips and directional score gaps dropped to **`0/40` (`0.0%` `[0.0%, 8.8%]`) and `+0.0`** across all attributes (`p = 1.000`). Because `blind` strips `name` and `university` before inference, this zero-flip result holds **by construction** (paired profiles become byte-identical). Across individual profiles, `46/400` outcomes shifted (`22 reject -> escalate`, `14 shortlist -> escalate`, `5 shortlist -> reject`, `5 reject -> shortlist`) and `166/400` scores changed (`mean |delta| = 14.01`), while prompt tokens dropped by `7.7%` (`84,409` -> `77,922`) with `0` extra model calls (`484.2s` vs. `481.4s` bare).

* **Generalizability Limits**: This run supports claims only about `Qwen/Qwen2.5-3B-Instruct` with `prompt_version="v1"` on synthetic matched profiles. It **cannot** guarantee identical behavior under a different prompt wording, a larger or differently fine-tuned LLM, or real-world applicants whose unstructured essays and extracurricular records contain subtle socioeconomic or regional signals.

---

## 2. Who is affected

* **Wrongly Rejected vs. Wrongly Shortlisted (Tied to Measured Divergence)**: In the baseline run, all `44` rejections scored `<= 45` (`min=20, max=45`) and all `356` shortlists scored `>= 45` (`min=45, max=85`), with `64/400` profiles (`16.0%`: `33` rejects, `31` shortlists) falling inside the `40–49` decision boundary. Because regional university applicants suffer a measured **-2.5 point penalty** (`sign p = 0.002`), **borderline regional university applicants** scoring around `45–47` are pushed down to `42–45` and **wrongly rejected**, whereas **borderline flagship university applicants** with identical academic merit receive the `+2.5` boost and are **wrongly shortlisted**—directly displacing equally qualified regional peers when scholarship slots are capped. Furthermore, if an undetected gender disparity below our `15.0%` power floor exists (consistent with our `4/40` or `10.0%` observed flips favoring female-coded names, `p = 0.125`), **borderline male-coded applicants** bear the silent cost of under-detection.

* **Asymmetric Costs of Wrong Decisions**: A **wrongly rejected** regional applicant loses direct financial aid, the opportunity to attend university, and potentially a year of academic progression, while receiving no explanation from an unlogged bare pipeline. By contrast, a **wrongly shortlisted** flagship applicant incurs zero immediate personal harm and still advances to human final selection—though if final awards rely on the model's ranking, they capture funding at a regional peer's expense.

* **Unmentioned Affected Parties**:
  1. *Deterred prospective applicants*: Regional students who anticipate institutional bias and never apply (self-selection harm invisible in application logs).
  2. *Human scholarship reviewers*: Under our harness (`borderline=(40, 49)` + `my_extra_escalation`), staff must manually read `36/400` (`9.0%`) escalated files (`34` borderline + `2` `high_merit_rejection_audit` cases), absorbing cognitive and time costs (`0` automated rejections occurred inside `[40, 49]`).
  3. *Applicants from unseen institutions*: Students from newly accredited, rural, or foreign colleges absent from the LLM's pretraining priors face unpredictable tokenization noise (demonstrated by `null_university` shifting `9/40` scores by `mean |b-a| = 2.12`).
  4. *The scholarship fund*: Suffers misallocated capital, reputational damage, and legal exposure under anti-discrimination mandates if biased shortlists go uncorrected.

* **Compounding Year-over-Year & Downstream Effects**: Used annually without blinding, a `-2.5` point regional penalty systematically starves regional universities of scholarship alumni, reducing future cohorts' mentorship networks and institutional prestige—a self-reinforcing Matthew effect. Moreover, if the LLM's `0–100` score is passed forward to later committee rounds, external co-funders, or institutional rankings as an "objective merit score," initial prestige bias becomes laundered into an authoritative credential.

---

## 3. Professional responsibility

* **Developer Responsibilities (Pre-Run Checks, Deployment Disclosures, Prohibited Claims)**:
  1. *Pre-run verification*: Before any production batch, I must verify schema validation (`validate_input=True`), confirm that `blind=("name", "university")` strips all direct identifiers, test refusal gates against adversarial/unauthorized inputs (`5/5` blocked in Part 4 with `0/400` false refusals), and recalibrate `borderline` thresholds on the **blinded** score distribution (as shown in Part 3, unblinded calibration predicted `16.0%` escalation vs. `9.0%` actual).
  2. *Deployment disclosures*: I must explicitly inform the deploying scholarship fund that: (a) scores are **uncalibrated decision-support triage signals**, not ground-truth merit measurements; (b) `n = 40` audit pairs cannot detect binary flip biases smaller than `15.0%`; and (c) `decisions.jsonl` contains no raw PII and is currently a local append-only file requiring secure institutional storage.
  3. *Prohibited claims*: I must **never** claim that the harness makes the LLM "unbiased," "fair," or "legally compliant by itself," nor claim that `0/40` flips after blinding eliminates proxy bias hidden inside `program` or free-text `achievements`.

* **Discharging Responsibilities in Practice (Completed vs. Remaining)**:
  * *Done in this lab*: Implemented deterministic pre-inference gates (`validate_profile`, `refuse`, `my_extra_refusal`), attribute blinding (`blind=("name", "university")`), borderline and merit-contradiction escalation (`borderline=(40, 49)`, `my_extra_escalation`), and privacy-preserving SHA-256 audit logging (`decisions.jsonl` + `control_register.json`).
  * *Remaining for production*: Automated drift/recalibration scripts when prompt templates change, cryptographic log signing (HMAC/WORM storage) to prevent post-hoc log tampering, NER scrubbing on free-text `achievements`, and an applicant-facing notification and human-appeal workflow for refused cases.

* **Division of Responsibility & Resolving Institutional Conflicts**:
  * *Institution's responsibilities*: Defining lawful eligibility policy (`scholarship_screening`), staffing qualified human reviewers for the `9.0%` escalated queue, setting final award quotas, and operating a formal applicant appeal process.
  * *Handling conflict with the deploying institution*: If the deploying institution asks to disable `blind=("name", "university")` to reward flagship university prestige, or asks to narrow/remove `escalate` (`borderline=(40, 49)`) to eliminate the `9.0%` (`36/400`) human review workload, **I must refuse to silently remove those controls**. Using our empirical read-out, I would document that removing `blind` reintroduces a statistically significant `-2.5` point penalty (`p = 0.002`) against regional applicants, and removing `escalate` exposes `36` borderline or high-merit applicants (`mean |delta| = 14.01` prompt noise) to unreviewed automated rejection—requiring formal written sign-off from institutional leadership and disclosure in `control_register.json` before any control is weakened.
