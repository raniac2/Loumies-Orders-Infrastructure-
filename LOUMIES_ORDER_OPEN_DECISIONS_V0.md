# LOUMIES Order Infrastructure — Open Decisions Register (V0)

Companion to `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` and `LOUMIES_ORDER_STATE_MODEL_V0.yaml`.

This register lists every material business rule referenced in the V0 specification that is not a permanent owner decision. Nothing here is resolved by this document. Each row states what would close it and whether it currently blocks specification work, implementation, or activation of the proof — or blocks nothing yet.

**Classification key:** OWNER-LOCKED (decided by Rania) · WORKING ASSUMPTION (being modeled/tested) · CONFIGURABLE (must remain changeable, value not yet set) · OPEN (not yet decided or not yet evidenced).

**Blocks key:** *Specification* = this V0 cannot be finalized without it · *Implementation* = a future vendor/build cannot start without it · *Activation* = the proof cannot go live without it · *None* = safe to leave open for now.

---

## A. Proof Parameters

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| A1 | Whether the Ithaca proof proceeds at all | WORKING ASSUMPTION | Everything downstream depends on it | Rania's go decision | None (this V0 is useful either way) |
| A2 | Proof duration (currently modeled 3–6 months) | WORKING ASSUMPTION | Affects how much metric history is meaningful (§14) | Rania's decision, informed by early proof data | None |
| A3 | Number and identity of operating days per week (currently 2 firm + 1 conditional) | WORKING ASSUMPTION | Drives date-capacity configuration (§10-A); the architecture must accept any count without redesign | Rania's decision, tested against founder-hour model | Activation |
| A4 | Conditions that make the third day "sufficiently firm" to activate | OPEN | Determines when/how the operator opens the conditional day (§13) | Rania's decision on what threshold/signal justifies it | Activation |
| A5 | Founder day maximum (currently modeled 12 hours) and paid kitchen block (currently modeled ≥10 hours) | WORKING ASSUMPTION | Upper-bounds labor-sensitive capacity (§10-C) indirectly | Rania's operating decision, validated in early proof weeks | None yet |
| A6 | Helper use: whether occasional help is used, under what terms, and whether helper labor enters proof economics | OPEN | Affects labor-sensitive capacity (§10-C), compliance posture, and cost/contribution metrics (§14) | Rania's decision if/when help is actually engaged | Implementation (only if/when helpers are used) |

---

## B. Menu, Pricing, and Product Classification

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| B1 | Final proof menu (which items are sold) | OPEN | Item capacity (§10-B) and production manifest (§12) are per-item; nothing to configure until items exist | Rania's menu decision | Implementation |
| B2 | 2026 selling prices | OPEN | Channel-specific economics (§11) needs values, not just structure | Rania's pricing decision | Activation |
| B3 | Product classification against labor-sensitive capacity categories (batch-scalable / shaping-intensive / assembly-intensive / oven-constrained / vessel-constrained / cooling-constrained) | OPEN | §10-C's categories are illustrative only; no product has been assigned | Physical production measurement plus Rania's classification decision | Implementation |
| B4 | Numerical date, item, and labor-sensitive capacity limits | OPEN | §10 defines the levers; no test values are set | Physical trial/measurement in the shared kitchen, plus Rania's decision on what to expose vs. hold in reserve | Activation |
| B5 | Discount policy (whether/when discounts are permitted) | OPEN | §8 reserves a field; no rule exists | Rania's decision, if ever raised | None |
| B6 | Tax treatment | OPEN, legal/vendor-dependent | §8 reserves a placeholder field only | Legal/accounting guidance, likely vendor-dependent | Implementation |

---

## C. Scheduled Preorder

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| C1 | Preorder cutoff timing (e.g., "noon the day prior" is only an illustrative example in the assignment) | CONFIGURABLE / WORKING ASSUMPTION | Gates which dates are offered (§4) and when EXPIRED (T34) fires | Rania's decision, likely tested and adjusted during the proof | Activation |
| C2 | Order/booking modification rules once CONFIRMED | OPEN | §4 flags this must be capacity-aware, not a silent overwrite, but does not define allowed scope | Rania's policy decision | Implementation |
| C3 | Cancellation/refund policy for scheduled preorders (fees, windows, exceptions) | OPEN | §4 requires cancellation to be *capturable*; the consequence (refund/fee) is undefined | Rania's policy decision | Activation |
| C4 | Consumer information required at checkout (beyond what's structurally necessary) | OPEN | Affects §8 field completeness for this order type | Rania's decision, possibly informed by fulfillment/compliance needs | Implementation |

---

## D. Same-Day Ordering

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| D1 | Total same-day order capacity per operating date (the assignment's "5 orders" is one test scenario only) | WORKING ASSUMPTION / CONFIGURABLE | §5 requires the system to test 0, 5, 10, or any value without redesign | Rania's test decisions across proof weeks | Activation |
| D2 | Which operating dates offer same-day ordering at all | CONFIGURABLE | §5 requires this to be operator-selectable per date, not automatic | Rania's per-date decision | Activation |
| D3 | Same-day fulfillment eligibility (pickup/delivery availability by date) | CONFIGURABLE | §5, §10 | Rania's operational decision | Activation |

---

## E. Group / Catering

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| E1 | Minimum order size | OPEN | §6 explicitly defers this | Rania's decision, likely informed by batch-efficiency testing | Activation (for this channel) |
| E2 | Deposit percentage / whether a deposit is required at all | OPEN | Determines whether T15/T19 (DEPOSIT_CONFIRMED path) is used for a given booking | Rania's decision | Activation (for this channel) |
| E3 | Required lead time for catering requests | OPEN | Affects qualification (§6, T09-T11) | Rania's decision | Activation (for this channel) |
| E4 | Cancellation policy for group/catering | OPEN | §6 explicitly defers this | Rania's decision | Activation (for this channel) |
| E5 | Final group package pricing / whether standardized packages exist yet | OPEN | §6 supports both custom and standardized package models; neither is populated | Rania's decision | Implementation |

---

## F. Institutional

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| F1 | Which institutional counterparties require formal PO/procurement authorization vs. accept written/verbal acceptance as sufficient | OPEN, per counterparty | Determines whether T15-T18 (procurement-authorization path) applies to a given institutional order, or whether T20-style direct-to-CONFIRMED-after-acceptance is appropriate | Discovery per institution as relationships develop | Activation (for this channel, per counterparty) |
| F2 | Invoice payment terms (net terms, timing) | OPEN | Affects INVOICED → PAID timing (T30) | Rania's decision / counterparty negotiation | Activation (for this channel) |
| F3 | Institutional cancellation policy | OPEN | Not addressed in the assignment; not invented here | Rania's decision | Activation (for this channel) |

---

## G. Common Model / Economics

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| G1 | Delivery charge structure | OPEN | §8 reserves a placeholder field | Rania's decision | Activation |
| G2 | Service charge (if any) | OPEN | §8 reserves a placeholder field | Rania's decision | Activation |
| G3 | Packaging charge (if any) | OPEN | §8 reserves a placeholder field | Rania's decision | Activation |
| G4 | Whether/when a third-party marketplace channel is adopted | OPEN | §3, §11 require the architecture to *permit* this without assuming it | Rania's strategic decision, likely post-proof | None yet |
| G5 | Channel-specific fee structures once a marketplace or payment vendor exists | OPEN, vendor-dependent | §11, §14 | Vendor selection (explicitly out of scope here) and its fee schedule | Implementation |

---

## H. Metrics / Success Criteria

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| H1 | GO/NO-GO numerical threshold for the proof | OPEN — explicitly not set by design | §14 captures data to compute metrics; no target is defined | Rania's decision, made with real proof data, not invented in advance | None (deliberately deferred) |
| H2 | Food cost and contribution by channel | OPEN, pending menu/pricing/vendor decisions (B1, B2, G5) | §14 | Resolution of B1, B2, G5 first | None yet |

---

## I. Cross-Check / Governance

| # | Issue | Status | Why it matters | What would close it | Blocks |
|---|---|---|---|---|---|
| I1 | Cross-check against any existing LOUMIES governance, Customer Interaction OS, public-experience, or operating-model artifacts held outside this repository | OPEN | Section 16 of the assignment requires this cross-check before implementation; none were found in this repository at the time of writing (no prior commits existed) | Locating and reviewing those artifacts (with Rio or elsewhere), and reconciling any conflicts found | Implementation |

---

## Summary — Items Blocking What

- **Blocks nothing yet (safe to leave open):** A1, A6 (until helpers used), B5, G4, H1, H2 (pending upstream items)
- **Blocks Activation of the proof / a given channel:** A3, A4, B2, B4, C1, C3, D1, D2, D3, E1–E4, F1–F3, G1–G3
- **Blocks Implementation (a future vendor build):** A6 (if used), B1, B3, B4, B6, C2, E5, G5, I1
- **Blocks Specification (this V0 itself):** none — every item above has been deliberately left open rather than blocking the V0's completion, per the assignment's decision discipline.
