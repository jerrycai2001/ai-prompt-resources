# RULES & CONSTRAINTS KNOWLEDGE BASE
<!--
HOW TO USE (ChatGPT Enterprise, chat window):
Paste this file in the FIRST message of a session, together with
03_interrogation_checklist.md and 04_analysis_system_prompt.md.
Then paste or attach the BoM/quote in the SECOND message.

MAINTENANCE RULES:
- Every rule is one table row. No prose war stories here — those go in the case library.
- Every rule has a stable rule_id. Never reuse a retired ID; mark it STATUS=RETIRED.
- If a check requires combining two rules, DO NOT rely on the model to compose them.
  Write the pre-composed result as a new rule in Section 3 (Composite Rules).
- Record source + last_verified. An unverified compatibility rule is a liability.
-->

## 0. Authority Statement (model-facing)

This document is the authoritative constraint set. When analyzing a BoM or quote:
- A rule here OVERRIDES the model's general knowledge. If general knowledge conflicts with a rule, apply the rule and note the conflict.
- If a situation is not covered by any rule, say so explicitly — do not invent a rule.
- Always cite the `rule_id` for every finding.

## 1. Controlled Vocabulary

**rule_type** (closed enum):
`DEPENDENCY` (A requires B on the same order) · `INCOMPATIBILITY` (A must not appear with B) · `VERSION_CONSTRAINT` (A version X requires B version ≥/≤ Y) · `WIRING` (installation/rework labor or cabling required) · `LEAD_TIME` (temporal constraint between line items) · `QTY_RATIO` (quantity relationship, e.g., 1 controller : 4 modules max) · `SERVICE_REQUIRED` (commissioning, calibration, validation service must be quoted) · `LICENSE` (subscription/entitlement constraint)

**severity** (closed enum):
`BLOCKER` (order will fail or unit is non-functional) · `MAJOR` (rework, delay, or margin loss likely) · `MINOR` (inconvenience, cosmetic, or documentation gap)

**status**: `ACTIVE` · `RETIRED` · `UNVERIFIED`

## 2. Atomic Rules

<!-- EXAMPLE ROWS — replace with elicited rules. Keep the exact column set. -->

| rule_id | rule_type | scope (part / pattern) | condition | requirement | severity | failure_consequence | source | last_verified | status |
|---|---|---|---|---|---|---|---|---|---|
| R-001 | DEPENDENCY | Weighing module WM-2xxx series | Quoted with any base frame | Coupling kit CPL-40 must appear as separate line item | BLOCKER | Module cannot mount; field return, 3–4 wk delay | J. Cai, elicitation 2026-06 | 2026-06-15 | ACTIVE |
| R-002 | WIRING | Weighing module WM-2xxx series | Retrofit into existing panel | I/O rewiring labor line (SVC-RW-IO) required; inputs/outputs move from legacy terminal block | BLOCKER | Unit installed but dead on arrival; emergency service call | J. Cai, elicitation 2026-06 | 2026-06-15 | ACTIVE |
| R-003 | VERSION_CONSTRAINT | Control software SW-CTRL | Version ≥ 15.0 quoted | Gateway hardware must be HW-GW Rev C or later | BLOCKER | v15 will not license on Rev A/B gateways post subscription-model change | Field bulletin FB-2025-11 | 2026-05-02 | ACTIVE |
| R-004 | LICENSE | Control software SW-CTRL v15+ | Customer on legacy perpetual license | Subscription conversion SKU LIC-SUB-CONV must be on the quote | MAJOR | Software ships but cannot activate; billing dispute | Sales ops memo | 2026-04-20 | ACTIVE |
| R-005 | QTY_RATIO | Remote I/O module IO-8CH | Any quantity | 1 power supply PS-24-10A per 4 modules, rounded up | MAJOR | Brownouts, intermittent faults blamed on firmware | 10-yr tribal (mgr interview) | 2026-06-15 | ACTIVE |
| R-006 | LEAD_TIME | Any DEPENDENCY pair | Parent part need-date set | Dependent part lead time must not exceed parent need-date minus install window | MAJOR | Parent arrives, sits uninstallable; storage + schedule slip | PM lessons-learned | 2026-06-15 | ACTIVE |
| R-007 | SERVICE_REQUIRED | Any GxP-impacting instrument | Sold into regulated facility | IQ/OQ service SKU must be quoted or explicitly waived in writing | MAJOR | Instrument unusable until validation; customer escalation | Quality SOP ref | 2026-06-15 | ACTIVE |
| R-008 | INCOMPATIBILITY | Legacy comms card COM-A | Quoted alongside SW-CTRL v14+ | Must not appear on same system | BLOCKER | Driver removed in v14; card is dead weight | Release notes v14 | 2026-03-01 | ACTIVE |

## 3. Composite Rules (pre-composed multi-hop)

<!-- Any conclusion that requires chaining ≥2 atomic rules gets written out here
     explicitly. This is the single highest-leverage section for catching
     second/third-order faults. -->

| rule_id | composed_from | scenario | pre-composed check | severity |
|---|---|---|---|---|
| C-001 | R-003 + R-004 | Quote contains SW-CTRL v15 for an existing customer | Check BOTH: (a) gateway line item is Rev C+, or gateway upgrade SKU present; AND (b) LIC-SUB-CONV present if account is legacy-perpetual. Missing either = BLOCKER | BLOCKER |
| C-002 | R-001 + R-002 + R-006 | WM-2xxx retrofit quote | CPL-40 present, SVC-RW-IO present, AND CPL-40 lead time ≤ WM-2xxx need-date minus 2 wk install window | BLOCKER |
| C-003 | R-005 + R-006 | ≥5 IO-8CH modules quoted | PS-24-10A qty = ceil(modules/4) AND PS lead time within window (PS is chronically long-lead) | MAJOR |

## 4. Absence Anchors

<!-- "What SHOULD be on this document but isn't." For each anchor part family,
     the companion categories a complete quote must contain. This drives the
     Tier-2 "suspected gap" findings. -->

| anchor family | expected companions | typical omission rate | note |
|---|---|---|---|
| Weighing modules WM-* | coupling kit, I/O rewiring labor, calibration service | High | Most-missed trio in history |
| Control software SW-* | license SKU, gateway hardware check, training/commissioning | Medium | License SKU omission spikes after pricing changes |
| Any instrument into GxP site | IQ/OQ service or written waiver | Medium | See R-007 |

## 5. Change Log

| date | rule_id | change | by |
|---|---|---|---|
| 2026-06-15 | R-001..R-008 | Initial elicitation set | J. Cai |
