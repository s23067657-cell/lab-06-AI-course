# Lab 06 Design Justification: Ethical Principles, Legal Instruments & Engineering Trade-Offs

## 1. Pre-Search Prediction: Hardest Control to Tie to a Legal Provision
Before searching legal texts, the hardest control to tie to a specific statutory provision is **`validate_input`** (and the prompt-injection sub-rule inside **`refuse`**). Legal frameworks governing personal data and AI—such as Kazakhstan's Law *On Personal Data and Their Protection* and the EU AI Act—establish high-level duties regarding data accuracy, data security, and robustness against manipulation, but they do not prescribe domain-specific schema bounds (e.g., `GPA` in `0.0–4.0` or blocking substrings like `"ignore all previous instructions"`). Consequently, mapping technical input sanitization requires bridging general statutory obligations of data reliability and cybersecurity to concrete engineering validation rules.

---

## 2. System Boundaries: What the System May Do, Must Refuse, and Must Escalate

### 2.1 Permitted Scope (`May Do`)
* Screen adult applicants (`age >= 18`) who have explicitly consented (`consent_to_automated_processing == True`) strictly for the authorized purpose of `scholarship_screening`.
* Evaluate blinded merit attributes (`program`, `gpa`, `exam_score`, `household_income_bracket`, `achievements`) to generate a structured preliminary score (`0–100`), recommendation (`shortlist` or `reject`), and brief rationale.

### 2.2 Mandatory Refusals (`Must Refuse` — 0 Model Calls)
The harness enforces deterministic pre-inference refusals before any personal data reaches the LLM:
1. **Scope Violation (`purpose_not_allowed`):** Any purpose other than `scholarship_screening` (e.g., `rank_by_ethnicity`).
2. **Age Ineligibility (`minor`):** Applicants under 18 years old (`age < 18`).
3. **Lack of Consent (`no_consent`):** Profiles where `consent_to_automated_processing` is not `True`.
4. **Schema / Range Violation (`invalid_input`):** Out-of-range or non-numeric values (e.g., `gpa=9.9`, `exam_score=-5`).
5. **Adversarial Input (`prompt_injection_or_malformed_text` via `my_extra_refusal`):** Free-text fields (`name`, `university`, `program`) exceeding 120 characters or containing instruction-override markers (`"ignore previous"`, `"ignore all"`, `"system:"`, `"shortlist"`, `"score:"`, newlines).

### 2.3 Human Escalation (`Must Escalate` to Human Reviewer)
1. **Borderline Uncertainty Band (`borderline = (40, 49)`):** Any parsed decision whose score falls in `[40, 49]`—the empirical overlap zone where baseline rejects (`max=45`) and shortlists (`min=45`) collide (`33 R, 31 S`).
2. **Unparseable Output:** Any malformed LLM response that fails structured parsing.
3. **Merit-Contradiction & Rationale Audit (`my_extra_escalation`):**
   * `missing_rationale`: Model returned an empty explanation string.
   * `high_merit_rejection_audit`: Model rejected an applicant with `gpa >= 3.7` or `achievements >= 3`.
   * `low_merit_shortlist_audit`: Model shortlisted an applicant with `gpa < 2.5` and `exam_score < 60`.

---

## 3. Accountability Record (`decisions.jsonl` & `control_register.json`)
* **Privacy-Preserving Audit Log (`decisions.jsonl`):** Records one JSON object per request containing `record_id`, `application_id`, a cryptographic hash of the exact prompt text shown to the model, the lists of **shown** and **blinded field names** (never raw PII values), model/prompt version, active harness flags, raw model reply, score, outcome, deterministic `reason_code`, `escalated_to_human` boolean, and UTC timestamp. An auditor reviewing the log a year later can reconstruct *why* every decision was made without storing a duplicate database of applicant PII or re-running the LLM.
* **Control Register (`control_register.json`):** Maps all five active controls (`validate_input`, `blind`, `refuse`, `log`, `escalate`) to named ethical principles and legal provisions:
  * **`blind`:** *Fairness and Non-Discrimination* (UNESCO AI Ethics Recommendation, 2021, Sec. III.2, Para. 28) & **Constitution of the Republic of Kazakhstan (30 Aug 1995), Article 14, Paragraph 2** (prohibiting discrimination by origin, social status, gender, or language)—directly justified by Part 1's `+2.5` point flagship university bias (`sign p = 0.002`).
  * **`refuse`:** *Lawfulness, Consent, and Purpose Limitation* (OECD AI Principles, 2019/2024, Principle 1.2) & **Law of the Republic of Kazakhstan No. 94-V "On Personal Data and Their Protection" (21 May 2013), Article 7, Para. 1 & Article 8** (purpose limitation and mandatory subject consent).
  * **`validate_input`:** *Robustness and Data Quality* (UNESCO AI Ethics Recommendation, 2021, Sec. III.2, Para. 32) & **Law of the Republic of Kazakhstan No. 94-V (21 May 2013), Article 7, Para. 4** (accuracy and reliability of personal data).
  * **`log`:** *Transparency and Auditability* (UNESCO AI Ethics Recommendation, 2021, Sec. III.2, Para. 37) & **Regulation (EU) 2024/1689 (EU AI Act, 13 June 2024), Article 12, Para. 1** (automatic event logging for high-risk AI systems, applied here as an international governance benchmark).
  * **`escalate`:** *Human Oversight* (UNESCO AI Ethics Recommendation, 2021, Sec. III.2, Para. 35) & **Regulation (EU) 2024/1689 (EU AI Act, 13 June 2024), Article 14, Para. 1 & Annex III, Point 3(a)** (human oversight for high-risk educational/scholarship access systems, applied as an international benchmark).

---

## 4. Accepted Engineering Trade-Offs (ABET PI 4.1 / 4.2)

1. **Eliminating Prestige Bias vs. Losing Institutional Grading Context (`blind`):**
   * *Accepted Trade-Off:* Hiding `university` eliminates the `+2.5` point regional university penalty (`0/40` flips, `+0.0` score gap), at the cost of preventing the model from adjusting for grade inflation or rigor differences across institutions. This trade-off is justified because in our baseline run, unblinded institutional access acted as a statistically significant proxy bias rather than a calibrated merit adjustment, and academic rigor remains represented via standardized `exam_score` and `achievements`.
2. **Human Reviewer Workload vs. Borderline Error Risk (`escalate`):**
   * *Accepted Trade-Off:* Setting `BORDERLINE = (40, 49)` routes **9.0% (`36/400`)** of applications to human reviewers (down from the `16.0%` baseline estimate due to post-blinding score shifts). A narrower band would reduce human labor but leave borderline applicants exposed to prompt-layout noise (`mean |delta| = 14.01` across runs); a wider band (e.g., `40–60`) would overwhelm reviewers. A 9.0% review rate balances fairness protection with operational feasibility, though the fund must periodically recalibrate the band as score distributions shift.
3. **Prompt-Injection Security vs. False-Refusal Risk (`my_extra_refusal`):**
   * *Accepted Trade-Off:* Deterministic keyword matching (`"ignore all"`, `"system:"`, `"shortlist"`, etc.) and a 120-character length cap blocked `100%` of tested prompt injections (`0` model calls) with `0/400` (`0.0%`) false refusals on our benchmark. However, keyword rules risk false positives if a legitimate university or program name contains a blocked substring or exceeds 120 characters. We accept this residual risk because refused requests are explicitly logged with `reason_code = "prompt_injection_or_malformed_text"` in `decisions.jsonl`, enabling rapid human override and rule refinement.
4. **Zero Extra Compute Cost vs. Runtime Counterfactual Auditing (`counterfactual_check = False`):**
   * *Accepted Trade-Off:* Leaving `counterfactual_check` disabled kept inference cost at **1.00 model call/profile** (`400` calls, `77,922` prompt tokens) rather than doubling compute (`2.00` calls/profile). Because `name` and `university` are already blinded before inference, counterfactual swapping of those attributes is redundant by construction.
