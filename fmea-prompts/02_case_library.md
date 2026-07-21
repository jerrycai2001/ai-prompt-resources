# HISTORICAL FAILURE CASE LIBRARY
<!--
HOW TO USE (ChatGPT Enterprise, chat window):
Paste in the first message of a session alongside 01_rules_constraints.md and
04_analysis_system_prompt.md. The model retrieves the nearest cases by tag
and cites case_id in Tier-2 findings.

MAINTENANCE RULES:
- One case = one real failure, anonymized. Target 20–50 cases.
- Follow the canonical template EXACTLY. The fields are the retrieval keys.
- The single most valuable field is `veteran_question` — the question a
  10-year manager would have asked that would have caught it. Write it as a
  literal, askable question, not a moral.
- Tags come from the closed tag vocabulary below. No freeform tags.
-->

## 0. Authority Statement (model-facing)

Cases are precedent, not law. Use them to justify Tier-2 (suspected gap) and
Tier-3 (questions) findings by analogy: "this quote resembles CASE-003 because
X and Y." Never cite a case as if it were a hard rule — hard rules live in the
rules document. If no case is similar, say so; do not force an analogy.

## 1. Tag Vocabulary (closed)

`software-version` · `license-subscription` · `mechanical-coupling` · `io-wiring` ·
`power-budget` · `lead-time` · `missing-service` · `gxp-validation` ·
`vendor-substitution` · `qty-error` · `unit-of-measure` · `retrofit` · `new-install`

## 2. Canonical Case Template

```
### CASE-XXX — <short title>
tags: <2–4 tags from vocabulary>
severity_realized: BLOCKER | MAJOR | MINOR
rule_refs: <rule_ids this case motivated, or NONE if not yet codified>

SITUATION:        <1–3 sentences of business context>
DOCUMENT SHOWED:  <what the BoM/quote actually contained — the surface view>
ACTUALLY WRONG:   <the underlying fault, stated precisely>
DETECTION CUE:    <the surface signal a veteran noticed — what "looked off">
VETERAN QUESTION: "<the literal question that would have caught it>"
CONSEQUENCE:      <what it cost — time, money, escalation>
```

## 3. Cases

<!-- SYNTHETIC EXAMPLES demonstrating the format — replace with real history. -->

### CASE-001 — v15 software quoted against Rev B gateway
tags: software-version, license-subscription, retrofit
severity_realized: BLOCKER
rule_refs: R-003, R-004, C-001

SITUATION:        Existing customer upgrading control software during annual service renewal.
DOCUMENT SHOWED:  One line item: SW-CTRL v15.2, qty 1. No hardware lines, no license SKU.
ACTUALLY WRONG:   Site gateway was HW-GW Rev B; v15 licensing stack requires Rev C+. Account was on a legacy perpetual license — v15 activation needs subscription conversion.
DETECTION CUE:    A major-version software upgrade quoted with zero hardware and zero license lines. Upgrades that touch the licensing model are never one-line quotes.
VETERAN QUESTION: "What gateway revision is installed at this site, and is this account still on perpetual licensing?"
CONSEQUENCE:      Software delivered, could not activate. Emergency gateway order at expedite pricing, 5-week slip, customer escalation to district manager.

### CASE-002 — Weighing module without coupling or rewire
tags: mechanical-coupling, io-wiring, retrofit
severity_realized: BLOCKER
rule_refs: R-001, R-002, C-002

SITUATION:        Retrofit of WM-2400 weighing module into an existing filling line.
DOCUMENT SHOWED:  WM-2400 module, mounting hardware kit, freight. Looked complete to the inside-sales rep.
ACTUALLY WRONG:   Base frame requires coupling kit CPL-40 (not in the generic mounting kit), and legacy panel I/O had to be rewired from the old terminal block — a labor line, not a part.
DETECTION CUE:    "Mounting hardware kit" on a retrofit quote. Generic kits fit new installs; retrofits almost always need the model-specific coupling plus rewiring labor.
VETERAN QUESTION: "Is this going onto an existing base — and if so, who is quoted to move the I/O off the old terminal block?"
CONSEQUENCE:      Field tech on site with an unmountable module. Return trip, 3 weeks, ~$8k unbilled labor.

### CASE-003 — Power supplies under-quoted for I/O expansion
tags: power-budget, qty-error
severity_realized: MAJOR
rule_refs: R-005, C-003

SITUATION:        Line expansion adding six IO-8CH remote I/O modules.
DOCUMENT SHOWED:  IO-8CH qty 6, PS-24-10A qty 1.
ACTUALLY WRONG:   Ratio rule is 1 supply per 4 modules → 2 supplies required. Single supply browned out under load.
DETECTION CUE:    Any I/O quantity above 4 with exactly one power supply. The ratio is tribal — it is not on the datasheet's first page.
VETERAN QUESTION: "Six modules on one supply — has anyone run the 24V power budget?"
CONSEQUENCE:      Intermittent comms faults misdiagnosed as firmware for two weeks before power was checked.

### CASE-004 — Dependent part lead time exceeded parent need date
tags: lead-time, mechanical-coupling
severity_realized: MAJOR
rule_refs: R-006

SITUATION:        New install; customer had a hard commissioning date.
DOCUMENT SHOWED:  All parts present and correct. WM-2400 lead time 4 wk; CPL-40 coupling lead time 9 wk. Need date 6 wk out.
ACTUALLY WRONG:   Nothing was missing — the timeline was impossible. The cheap dependent part was the schedule constraint, not the expensive parent.
DETECTION CUE:    A complete BoM can still fail. Scan the MAX lead time across each dependency cluster, not per line.
VETERAN QUESTION: "What's the longest lead time in this dependency chain, and does it clear the commissioning date?"
CONSEQUENCE:      Module sat in a crate on the customer's dock for 5 weeks; storage fees and a slipped go-live.

<!-- Continue CASE-005 … CASE-050 using the identical template. -->
