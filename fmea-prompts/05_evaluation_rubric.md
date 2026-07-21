# EVALUATION FRAMEWORK & RUBRIC
<!--
HOW TO USE:
This file is for YOU (and optionally a second ChatGPT session acting as judge).
Do not paste it into the analysis session — the analyst must not see the
answer key or the scoring criteria.
-->

## 1. Golden Dataset Design

**Composition (initial target: 16 documents):**

| set | count | description |
|---|---|---|
| Seeded-fault BoMs | 10 | Real historical BoMs (anonymized) or realistic synthetics, each with 1–3 labeled faults |
| Clean BoMs | 4 | Genuinely correct documents — measures false-positive rate |
| Adversarial | 2 | "Too clean" documents: complete parts list but a composite/timeline fault only (CASE-004 pattern) — tests multi-hop, not lookup |

**Fault class codes** (label every seeded fault with exactly one):

| code | fault class | example |
|---|---|---|
| F-DEP | Dependency omission | Weighing module without coupling kit |
| F-VER | Version/compatibility violation | v15 software with Rev B gateway |
| F-LIC | License/entitlement gap | Missing subscription conversion SKU |
| F-WIR | Wiring/labor omission | Retrofit with no rewiring labor line |
| F-QTY | Quantity/ratio error | 6 I/O modules, 1 power supply |
| F-LT | Lead-time infeasibility | Dependent part gates the schedule |
| F-SVC | Required service missing | No IQ/OQ line for GxP instrument |
| F-STR | Structural defect | Missing units, ambiguous quantities |

**Answer key format** (one row per seeded fault, kept OUT of the analysis session):

```
doc_id | fault_id | class | ground truth description | expected tier (1/2) |
expected rule_id(s) | reference veteran question
```

## 2. Metrics

Run the full golden set through a fresh chat session per document. Score:

| metric | definition | target |
|---|---|---|
| Recall (per class) | seeded faults flagged in Tier 1 or Tier 2 / total seeded, computed per fault class | ≥ 90% F-DEP, F-VER, F-QTY; ≥ 75% F-LT, adversarial |
| Precision | true findings / all Tier-1 + HIGH-confidence Tier-2 findings | ≥ 85% |
| Clean-doc FPR | clean docs with ≥1 Tier-1 or HIGH Tier-2 finding / clean docs | ≤ 1 of 4 |
| Tier accuracy | faults reported in the correct tier / faults detected | ≥ 80% |
| Citation validity | findings citing a real, correctly-applied rule_id / all Tier-1 findings | 100% (a fabricated citation is an automatic run failure) |
| Coverage compliance | runs producing a complete Pass A–G coverage table / all runs | 100% |
| Question quality | mean rubric score (Section 3) across Tier-3 questions | ≥ 2.0 |

Scoring rule for partial credit: a fault detected but mis-tiered counts toward
recall, fails tier accuracy. A fault "detected" only as a vague Tier-3 question
counts as 0.5 recall.

## 3. Question-Quality Rubric (0–3, per question)

| score | anchor |
|---|---|
| 0 | Generic or unanswerable ("Are there any other issues?"), or asks for information already in the document |
| 1 | Topically relevant but unfocused — names an area, not a fact ("Can you confirm the software details?") |
| 2 | Names the specific missing fact AND is answerable by the account team ("What version of SW-CTRL is currently installed?") |
| 3 | Score-2 quality PLUS names the decision it gates or the failure it prevents — matches or exceeds the reference veteran question ("What gateway revision is at this site? v15 won't license on Rev A/B, and this quote has no gateway line.") |

Bonus signal (track, don't score): a question that surfaces a fault class NOT
seeded in the document but genuinely present — evidence of real generalization.

## 4. LLM-as-Judge Protocol

Use a separate chat session. Human spot-checks 20% of judge outputs; if human
and judge disagree on >15% of spot checks, discard the judge scores and rescore
manually.

**Judge prompt (paste verbatim, then the materials):**

```
You are grading an AI analyst's BoM fault-detection report against an answer key.
You will receive: (1) the answer key for one document, (2) the analyst's report.

For each fault in the answer key, output:
fault_id | DETECTED / MISSED / PARTIAL | tier reported | correct rule cited (Y/N) | evidence quote from report

Then list every Tier-1 and HIGH-confidence Tier-2 finding in the report that
does NOT correspond to an answer-key fault, and classify each:
TRUE-EXTRA (genuinely valid finding not in key) | FALSE-ALARM

Then score each Tier-3 question 0–3 using this rubric: [paste Section 3 table].
For each question, quote the rubric anchor that justifies the score.

Output a final line: recall_hits, false_alarms, mean_question_score.
Do not give the analyst benefit of the doubt: vague language that could be read
as detecting a fault is PARTIAL at best. Judge only what is written.
```

## 5. Regression Protocol

- **Trigger:** any edit to the system prompt, rules doc, checklist, or case
  library → rerun the full golden set before the change is "released."
- **One variable at a time.** Never edit the prompt and the rules doc in the
  same revision — you will not know which change moved the metrics.
- **Log every run:**

| run_id | date | doc_id | prompt_ver | rules_ver | recall | precision | FPR | mean Q score | notes |
|---|---|---|---|---|---|---|---|---|---|

- **Variance check:** run 3 repeats on 2 documents per revision. If findings
  differ materially across repeats, the instability is a finding in itself —
  usually a sign the rule or check wording is ambiguous.
- **Release gate:** no revision ships if any target in Section 2 regresses,
  even if others improve.

## 6. Failure Triage (where to fix what)

| observed failure | fix location — in this order |
|---|---|
| Missed dependency/version fault | Rules doc: is the rule present and machine-decidable? Then checklist: does a check point at it? Prompt last. |
| Missed multi-hop fault | Add a Composite Rule (C-xxx) pre-composing the hops. Do not try to prompt your way to better composition. |
| False alarms | Tighten the rule's `condition` column; add a clean counter-example case to the library |
| Vague questions | Add reference veteran questions to more cases; the model imitates the case library's question style |
| Missing coverage table / format drift | System prompt — strengthen the output contract |
| Inconsistent across repeats | Ambiguous rule or check wording — make the condition decidable |

## 7. Known Limits of This Harness

- LLM-as-judge inherits the judge model's blind spots — hence the mandatory
  human spot-check floor.
- 16 documents detects gross regressions, not small ones. Statistical
  confidence at this n is low; treat metric movements < ~10 points as noise
  until the set grows to 30+.
- Recall is measured against SEEDED faults. Track TRUE-EXTRA findings over
  time — they are the only measure of detection beyond what you already knew
  to test for.
