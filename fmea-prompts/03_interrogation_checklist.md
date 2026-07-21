# BoM / QUOTE INTERROGATION PROTOCOL
<!--
HOW TO USE (ChatGPT Enterprise, chat window):
Paste in the first message alongside 01_rules_constraints.md and
04_analysis_system_prompt.md. The system prompt instructs the model to execute
these passes IN ORDER and to report coverage per pass.

MAINTENANCE RULES:
- Every check has a stable check_id (CHK-A1, CHK-B2, …). Findings cite check_ids.
- A check is a question with a decidable answer, not advice.
- If a check repeatedly produces false alarms, tighten its condition here —
  do not soften the system prompt.
-->

## 0. Execution Contract (model-facing)

Run every pass, in order, on every document. A pass is either:
- **COMPLETED** — every check evaluated, results reported; or
- **BLOCKED** — required information absent from the document; report exactly
  which fields are missing and generate the corresponding Tier-3 question.

Never skip a pass silently. Never report a document clean without a per-pass
coverage table.

## PASS A — Structural Integrity (is the document analyzable?)

| check_id | check |
|---|---|
| CHK-A1 | Every line has: part number, description, quantity, unit price, lead time. List lines with missing fields. |
| CHK-A2 | Units of measure are explicit and consistent (each vs. box vs. meter; no bare numbers on cable/consumables). |
| CHK-A3 | Quantities are plausible integers/ratios (no qty 0, no fractional "each"). |
| CHK-A4 | Document identifies: customer site, install type (new vs. retrofit), need date. If absent → Tier-3 question, and flag every downstream check that depends on it as BLOCKED. |

## PASS B — Dependency Completeness (what must be here?)

| check_id | check |
|---|---|
| CHK-B1 | For every line, look up all `DEPENDENCY` rules whose scope matches. Confirm each required companion appears as its own line. Cite rule_id per confirmation or violation. |
| CHK-B2 | For every anchor family in the rules doc (Absence Anchors table), confirm each expected companion category is present or explicitly waived. |
| CHK-B3 | For every `SERVICE_REQUIRED` rule in scope (calibration, IQ/OQ, commissioning, training), confirm the service SKU is on the quote or a written waiver is referenced. |
| CHK-B4 | Labor is a line item, not an assumption: any retrofit, rewire, or panel modification implied by the parts must have a corresponding labor/service line. |

## PASS C — Compatibility & Versioning (does what's here work together?)

| check_id | check |
|---|---|
| CHK-C1 | Evaluate every `INCOMPATIBILITY` rule against all line-item pairs. |
| CHK-C2 | Evaluate every `VERSION_CONSTRAINT` rule: extract software/firmware versions from descriptions; if a version is unstated, that is a Tier-3 question, not a pass. |
| CHK-C3 | Evaluate every `LICENSE` rule: subscription vs. perpetual entitlement, conversion SKUs, seat counts vs. quoted users. |
| CHK-C4 | Evaluate every Composite Rule (C-xxx) explicitly, one by one. Do not treat a composite as satisfied because its parts were individually checked. |

## PASS D — Electrical / Physical Budgets

| check_id | check |
|---|---|
| CHK-D1 | Evaluate every `QTY_RATIO` rule (power supplies per module, controllers per node count, terminals per conductor count). Show the arithmetic. |
| CHK-D2 | Cabling: every powered or networked device implies cable lines; confirm cable type, length, and connector/coupling lines exist for each. |
| CHK-D3 | For weighing/instrumentation modules: confirm model-specific coupling to the base and I/O wiring path are covered (parts AND labor). |

## PASS E — Timeline Feasibility

| check_id | check |
|---|---|
| CHK-E1 | Compute max lead time per dependency cluster (parent + all companions), not per line. Compare against need date minus install window. |
| CHK-E2 | Flag any dependent/accessory line whose lead time exceeds its parent's — the cheap part that gates the schedule (see CASE-004 pattern). |
| CHK-E3 | Any expedite fees or partial-ship terms that split a dependency cluster across shipments → flag. |

## PASS F — Pattern & Precedent (the fuzzy tier)

| check_id | check |
|---|---|
| CHK-F1 | Compare the document's overall shape against the case library tags. Report up to 3 nearest cases with the specific matching features — or state "no case resembles this document." |
| CHK-F2 | One-line quotes for anything that changes a system's software, licensing, or mechanical interface are suspicious by default (CASE-001 pattern). |
| CHK-F3 | "Too clean" heuristic: a retrofit quote with zero labor lines, or a multi-instrument order with zero service lines, warrants a Tier-3 question even if no rule fires. |

## PASS G — Question Generation (what can't be answered from this document?)

| check_id | check |
|---|---|
| CHK-G1 | For every BLOCKED check above, emit one precise, askable question naming the missing fact and the decision it gates. |
| CHK-G2 | Rank questions by the severity of the worst finding they could unlock (BLOCKER-gating questions first). |
| CHK-G3 | Cap at the 5 highest-value questions. A 20-question list will be ignored; five sharp ones will be answered. |
