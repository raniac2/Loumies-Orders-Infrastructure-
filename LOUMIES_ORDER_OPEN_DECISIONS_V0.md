# LOUMIES Order Infrastructure — Open Decisions Register (V0, corrected)

Companion to `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` and `LOUMIES_ORDER_STATE_MODEL_V0.yaml`.

This register lists every material business rule referenced in the V0 specification that is not a permanent owner decision. Nothing here is resolved by this document. Each row states what would close it and whether it currently blocks specification work, implementation, or activation of the proof — or blocks nothing yet.

**Classification key:** OWNER-LOCKED (decided by Rania) · WORKING ASSUMPTION (being modeled/tested) · CONFIGURABLE (must remain changeable, value not yet set) · OPEN (not yet decided or not yet evidenced).

**Blocks key:** *Specification* = this V0 cannot be finalized without it · *Implementation* = a future vendor/build cannot start without it · *Activation* = the proof cannot go live without it · *None* = safe to leave open for now.

**Correction note:** This register has been adjudicated by Rio against the wider LOUMIES governance record, across two correction passes. First pass: rows A3, A4, A5, A6, B4, C1, D1, F1, G4, and I1 corrected from the original V0. Final surgical patch (this version): A4 and D1–D3 corrected on **blocking classification only** — none of these open decisions blocks Order #1 or the two-normal-day launch; and a new payment-terms row (C6) reflects the addition of `PAYMENT_NOT_YET_DUE` to the payment dimension as architecture only, not a resolution of any institutional payment-term question.

---

## A. Proof Parameters

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| A1 | Whether the Ithaca proof proceeds at all | WORKING ASSUMPTION | Everything downstream depends on it | Rania's go decision | None (this V0 is useful either way) |
| A2 | Proof duration (currently modeled 3–6 months) | WORKING ASSUMPTION | Affects how much metric history is meaningful (§14) | Rania's decision, informed by early proof data | None |
| **A3** | **Corrected.** Two prior conflated questions, now split: **(a)** the proof-day *architecture* — 2 normal bookable operating days per week plus 1 conditional/bookable third day; **(b)** the specific *weekday identities* assigned to those days. | **(a) OWNER-LOCKED** — the day-count architecture is decided. **(b) OPEN / CONFIGURABLE** — which weekdays remain unset. | The architecture drives date-capacity configuration (§10); the weekday assignment must be changeable without touching that architecture | (a) already closed by Rania. (b) Rania's operating decision, and may change during the proof | (a) none — closed. (b) Activation |
| **A4** | Conditions that make the third day "sufficiently firm" to activate | **OPEN — no numerical threshold implied or invented.** | Determines when/how the operator opens the conditional day (§13) | Rania's judgment call, informed by demand signals as the proof runs — not a formula to be derived here | **Corrected: blocks only use/opening of the conditional third day. Does NOT block Order #1 or activation of the two normal (OWNER-LOCKED, per A3a) operating days — LOUMIES can launch and accept orders on those two days with A4 unresolved.** |
| **A5** | Founder day maximum (12 hours) and paid kitchen block (at least 10 hours per normal operating day) | **OWNER-LOCKED for the current proof** — corrected from WORKING ASSUMPTION. | Upper-bounds labor-sensitive capacity (§10) and the proof's operating envelope | Already closed by Rania for this proof; reopening would be an explicit future decision, not a default drift | None — closed for the proof's duration |
| **A6** | Helper use: whether occasional help is used, under what terms, and whether helper labor enters proof economics | **OPEN — reopened by Rania conceptually.** Do not state "no helper ever" or "helpers freely available"; both are incorrect defaults. | Affects labor-sensitive capacity (§10), compliance posture, and cost/contribution metrics (§14) | Rania's decision if/when help is actually engaged | Implementation (only if/when helpers are used) |

---

## B. Menu, Pricing, and Product Classification

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| B1 | Final proof menu (which items are sold) | OPEN | Item capacity (§10) and production manifest (§12) are per-item; nothing to configure until items exist | Rania's menu decision | Implementation |
| B2 | 2026 selling prices | OPEN | Channel-specific economics (§11) needs values, not just structure | Rania's pricing decision | Activation |
| B3 | Product classification against labor-sensitive capacity categories (batch-scalable / shaping-intensive / assembly-intensive / oven-constrained / vessel-constrained / cooling-constrained) | OPEN | §10's categories are illustrative only; no product has been assigned | Physical production measurement plus Rania's classification decision | Implementation |
| **B4** | Numerical physical, sellable, and allocated capacity limits (date, item, labor-sensitive) | **OPEN — corrected framing.** Full numerical physical-capacity knowledge is **not** required before activation. What is required is a defensible **safe sellable capacity** (§10), which can start conservative and be revised using physical proof data as the proof runs. | §10 defines the five capacity layers and the levers; no test values are set | An initial conservative sellable-capacity number Rania is comfortable exposing, revised iteratively — not a single upfront physical-capacity study | Activation (a safe sellable number is required; the full physical ceiling is not) |
| B5 | Discount policy (whether/when discounts are permitted) | OPEN | §8 reserves a field; no rule exists | Rania's decision, if ever raised | None |
| B6 | Tax treatment | OPEN, legal/vendor-dependent | §8 reserves a placeholder field only | Legal/accounting guidance, likely vendor-dependent | Implementation |

---

## C. Scheduled Preorder

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| **C1** | Preorder cutoff timing | **CONFIGURABLE / working hypothesis — the "noon the day prior" example is illustrative only and is not locked in any form.** | Gates which dates are offered (§4) and when lifecycle reaches `EXPIRED` | Rania's decision, likely tested and adjusted during the proof | Activation |
| C2 | Order/booking modification rules once `CONFIRMED` | OPEN | §4 flags this must be capacity-aware, not a silent overwrite, but does not define allowed scope | Rania's policy decision | Implementation |
| C3 | Cancellation/refund policy for scheduled preorders (fees, windows, exceptions) | OPEN | §4 requires cancellation to be *capturable*; the consequence (refund/fee, and `payment_status` transition to `REFUND_OR_REVERSAL_PENDING`) is undefined | Rania's policy decision | Activation |
| C4 | Consumer information required at checkout (beyond what's structurally necessary) | OPEN | Affects §8 field completeness for this order type | Rania's decision, possibly informed by fulfillment/compliance needs | Implementation |
| C5 | Capacity-hold expiration duration at checkout | CONFIGURABLE / vendor-dependent | §10's temporary-hold mechanism requires a duration; none is set here by design | Rania's decision in consultation with whatever checkout mechanism is later selected | Implementation |
| **C6** | Institutional/group-catering payment terms: net terms, invoice timing, payment deadline, and which counterparties/orders use `payment_status = PAYMENT_NOT_YET_DUE` at all vs. requiring payment/deposit at booking | **OPEN.** The `PAYMENT_NOT_YET_DUE` state added to the payment dimension (YAML) is **architecture only** — it provides a truthful place to represent pay-on-terms arrangements once approved, and does **not** itself resolve net terms, invoice timing, payment deadlines, deposit policy, or any counterparty-specific rule. | Whether a given institutional order can legitimately reach `CONFIRMED` via the `PAYMENT_NOT_YET_DUE` branch of `gates.to_CONFIRMED` depends entirely on this being resolved per counterparty | Rania's decision / counterparty negotiation, per institution, as relationships develop (same discovery process as F1) | Activation (for this channel, per counterparty) |

---

## D. Same-Day Ordering

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| **D1** | Total same-day order capacity per operating date | **WORKING ASSUMPTION / CONFIGURABLE — the assignment's "5 orders" is one test scenario only, never a permanent rule.** Same-day ordering itself remains an experimental/configurable hypothesis (infrastructure doc §2), not an owner-locked channel. | §5 requires the system to test 0, 5, 10, or any value without redesign | Rania's test decisions across proof weeks | **Corrected: blocks same-day channel/experiment only. LOUMIES may operate with same-day capacity set to zero — this does NOT block Order #1 or activation of scheduled-preorder, group/catering, or institutional order types.** |
| **D2** | Which operating dates offer same-day ordering at all | CONFIGURABLE | §5 requires this to be operator-selectable per date, not automatic | Rania's per-date decision | **Corrected: blocks same-day channel/experiment only, same as D1. Not a dependency for any other channel.** |
| **D3** | Same-day fulfillment eligibility (pickup/delivery availability by date) | CONFIGURABLE | §5, §10 | Rania's operational decision | **Corrected: blocks same-day channel/experiment only, same as D1. Not a dependency for any other channel.** |
| D4 | Rules/limits governing deliberate reallocation between the scheduled-preorder and same-day pools (e.g., how close to service reallocation remains sensible) | OPEN | §5, §10 establish that reallocation is permitted; operational limits on when/how much are not set | Rania's operating decision, likely refined during the proof | None yet |

---

## E. Group / Catering

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| E1 | Minimum order size | OPEN | §6 explicitly defers this | Rania's decision, likely informed by batch-efficiency testing | Activation (for this channel) |
| E2 | Deposit percentage / whether a deposit is required at all | OPEN | Determines whether `payment_status` passes through `DEPOSIT_RECEIVED` for a given booking | Rania's decision | Activation (for this channel) |
| E3 | Required lead time for catering requests | OPEN | Affects qualification (§6) | Rania's decision | Activation (for this channel) |
| E4 | Cancellation policy for group/catering | OPEN | §6 explicitly defers this | Rania's decision | Activation (for this channel) |
| E5 | Final group package pricing / whether standardized packages exist yet | OPEN | §6 supports both custom and standardized package models; neither is populated | Rania's decision | Implementation |
| E6 | Whether LOUMIES offers an optional manual date-hold for a serious group/institutional prospect ahead of formal confirmation | **OPEN — explicitly not implemented as a rule in this V0.** | §5/§10's capacity-hold mechanism is scoped to consumer checkout only; an operator-initiated hold for a serious prospect is a plausible future tool, not a default | Rania's operating decision, if she wants this capability | None yet |

---

## F. Institutional

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| **F1** | Which institutional counterparties require formal PO/procurement authorization vs. accept written/verbal acceptance as sufficient | **OPEN, strictly per counterparty — corrected framing.** Do not describe verbal or written acceptance as sufficient for institutions in general; counterparty-specific rules control, represented by `procurement_status = NOT_REQUIRED` only for a counterparty confirmed not to need formal authorization. | Determines whether a given institutional order's lifecycle can cross into `CONFIRMED` on acceptance alone, or must wait for `procurement_status = AUTHORIZED` | Discovery per institution as relationships develop (the Cornell case is the governing reference: PO required, acceptance alone is not sufficient) | Activation (for this channel, per counterparty) |
| F2 | Invoice payment terms (net terms, timing) | OPEN | Affects `INVOICED_OUTSTANDING` → `PAID_IN_FULL` timing | Rania's decision / counterparty negotiation | Activation (for this channel) |
| F3 | Institutional cancellation policy | OPEN | Not addressed in the assignment; not invented here | Rania's decision | Activation (for this channel) |

---

## G. Common Model / Economics

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| G1 | Delivery charge structure | OPEN | §8 reserves a placeholder field | Rania's decision | Activation |
| G2 | Service charge (if any) | OPEN | §8 reserves a placeholder field | Rania's decision | Activation |
| G3 | Packaging charge (if any) | OPEN | §8 reserves a placeholder field | Rania's decision | Activation |
| **G4** | Whether/when a third-party marketplace channel is adopted | **OPEN — and explicitly not required for the proof.** | §3, §11 require the architecture to *permit* this without assuming it | Rania's strategic decision, likely post-proof | None yet |
| G5 | Channel-specific fee structures once a marketplace or payment vendor exists | OPEN, vendor-dependent | §11, §14 | Vendor selection (explicitly out of scope here) and its fee schedule | Implementation |

---

## H. Metrics / Success Criteria

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| H1 | GO/NO-GO numerical threshold for the proof | OPEN — explicitly not set by design | §14 captures data across six distinct reporting dimensions (pipeline / booked / paid / fulfilled / outstanding / cancelled-reversed); no target is defined | Rania's decision, made with real proof data, not invented in advance | None (deliberately deferred) |
| H2 | Food cost and contribution by channel | OPEN, pending menu/pricing/vendor decisions (B1, B2, G5) | §14 | Resolution of B1, B2, G5 first | None yet |

---

## I. Cross-Check / Governance

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| **I1** | Cross-check against any wider existing LOUMIES governance, Customer Interaction OS, public-experience, or operating-model artifacts | **PARTIALLY CLOSED.** This correction pass constitutes a Rio/Rania governance adjudication for the specific items explicitly addressed here: the OWNER-LOCKED proof-architecture facts (A3a, A5), the reopening of helper use (A6), the reframing of scheduled-preorder and same-day as working hypotheses rather than locked channels, and the Cornell-referenced institutional procurement pattern (F1). **Anything not explicitly adjudicated in this pass remains OPEN** and cross-check against the broader governance record remains required before implementation. | Prevents this V0 from silently drifting away from, or contradicting, governing LOUMIES material held elsewhere | Continued review with Rio against the full governance record as it becomes available in-repository or otherwise | Implementation (for anything not explicitly adjudicated here) |

---

## Summary — Items Blocking What

- **Blocks nothing yet (safe to leave open):** A1, A6 (until helpers used), B5, D4, E6, G4, H1, H2 (pending upstream items)
- **Blocks Order #1 / launch of the two normal operating days:** none of the items in this register. A4 and D1–D3 are explicitly corrected below to confirm they do not sit on this critical path.
- **Blocks Activation of a specific channel/experiment beyond the base two-day launch:** A4 (conditional third day only), B2, B4 (safe sellable number only, not full physical study), C1, C3, C5, C6 (institutional pay-on-terms channel only), D1–D3 (same-day channel/experiment only), E1–E4, F1–F3, G1–G3
- **Blocks Implementation (a future vendor build):** A6 (if used), B1, B3, B4 (physical-capacity refinement), B6, C2, C4, E5, G5, I1 (for anything not adjudicated here)
- **Blocks Specification (this V0 itself):** none — every item above has been deliberately left open rather than blocking the V0's completion, per the assignment's decision discipline.
- **Closed by prior correction pass:** A3a (day-count architecture), A5 (founder-hour and kitchen-hour envelope), and the portion of I1 covering those items plus A6's reopening and F1's per-counterparty framing.
- **Corrected by this final surgical patch (blocking classification only, not the underlying OPEN status):** A4 now reads as blocking only the conditional third day, never Order #1 or the two normal days. D1, D2, D3 now read as blocking only the same-day channel/experiment, never scheduled preorder, group/catering, or institutional. New row C6 added for institutional/catering payment-terms questions that `PAYMENT_NOT_YET_DUE` gives a place to represent but does not resolve.
