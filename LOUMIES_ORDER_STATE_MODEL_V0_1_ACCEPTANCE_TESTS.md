# LOUMIES Order State Model V0.1 — Acceptance Tests

**Status:** Specification/validation artifact only. Not executable code, not a test framework.
**Companion:** `LOUMIES_ORDER_STATE_MODEL_V0_1_PATCH.yaml` (the additive delta these tests validate) and the approved baseline `LOUMIES_ORDER_STATE_MODEL_V0.yaml` (blob SHA `1e3ae49440ee05c560dd70752ba402d5bd8cab69`, unmodified).
**Source:** the 15 required cases from the governing "LOUMIES — ORDER STATE MODEL V0.1 PATCH SPEC & ACCEPTANCE TESTS — 2026-09-04" Google Doc, reproduced here in Given/When/Then form with an explicit result against the patch as written.

Every test below is evaluated by tracing the patched model (V0 baseline + `LOUMIES_ORDER_STATE_MODEL_V0_1_PATCH.yaml`) — not by running code, since this repository is a specification artifact set with no application/test harness to execute. Each result cites the exact patch element(s) that make it hold.

---

## TEST 01 — Standard Table direct confirmation

**Given:** a 10-person Tier 2 Table, date available, under cutoff, below the date-level auto-confirm review threshold.
**When:** the customer configures the package and pays successfully.
**Then:** `order_type = group_table`; `commercial_workflow = DIRECT`; lifecycle `DRAFT → CONFIRMED`; `payment_status = PAID_IN_FULL`; `procurement_status = NOT_APPLICABLE`; no `QUOTE_SENT` or `CUSTOMER_ACCEPTED` in the order's history; the capacity hold converts atomically with confirmation.

**Result: PASS.** `ADD-01` defines `group_table`; `EXT-05` (extended `L01`) carries `DRAFT → CONFIRMED` for `group_table` unconditionally; `EXT-06` (extended `gates.to_CONFIRMED`) requires `payment_status = PAID_IN_FULL` for `group_table` and the DIRECT hold-conversion condition; `EXT-06C` adds `group_table` to `procurement_states.NOT_APPLICABLE`; the transition's `truth_rule` explicitly forbids synthesizing `QUOTE_SENT`/`CUSTOMER_ACCEPTED`.

---

## TEST 02 — Table payment failure

**Given:** the same valid Table checkout as Test 01.
**When:** payment fails.
**Then:** lifecycle does not reach `CONFIRMED`; the temporary hold releases; `payment_status` reflects failure; no booked revenue or confirmed capacity remains.

**Result: PASS.** Unchanged baseline mechanics apply unmodified: `payment_transitions` (`PAYMENT_PENDING → PAYMENT_FAILED`) and `capacity_hold.hold_transitions` (`ACTIVE → RELEASED` on payment failure) are untouched by the patch and already generalize to every DIRECT flow via `EXT-07`. `EXT-05`'s `L01` never fires without a satisfied payment gate, so lifecycle stays at `DRAFT` (and may proceed to `EXPIRED` via `EXT-06B` if the hold subsequently lapses).

---

## TEST 03 — Table exceeding the $3,000 date-level auto-confirm rule

**Given:** existing confirmed scheduled/advance revenue on the date, such that adding the new Table would exceed the aggregate review threshold.
**When:** the customer attempts checkout.
**Then:** automatic confirmation stops; the order moves to the owner-review/exception path per approved experience logic; the system does not silently accept the order or double-count capacity.

**Result: PASS.** `ADD-04` (`aggregate_date_level_review_threshold`) is checked as part of `EXT-05`'s trigger, and — per the 2026-09-04 closeout addendum's owner-locked cross-path scope — this Table's contribution is checked against the *same* aggregate that already includes any other CONFIRMED scheduled/advance business on the date (scheduled_preorder, other group_table orders, DIRECT or REVIEWED group_catering once CONFIRMED, and legitimately confirmed institutional orders), not a Table-only or DIRECT-only counter. When the aggregate would be exceeded without explicit owner authorization, the order does not cross `L01` into `CONFIRMED`; it takes unmodified baseline transition `L12` (`→ EXCEPTION`) instead. Its capacity hold stays `ACTIVE` (subject to normal expiry) rather than being force-converted or force-released, per `ADD-04.effect_when_threshold_would_be_exceeded`. No new lifecycle state is introduced — `EXCEPTION` is the existing V0 state, resolved onward via unmodified `L13` (owner authorizes → `CONFIRMED`) or `L14` (→ `CANCELLED`).

---

## TEST 04 — Standard direct Catering confirms without quote states

**Given:** the customer selects only published standard Catering offers/quantities/prices within standard rules.
**When:** availability passes and payment succeeds.
**Then:** `order_type = group_catering`; `commercial_workflow = DIRECT`; lifecycle `DRAFT → CONFIRMED`; no quote states appear in the order's history; the temporary hold converts atomically.

**Result: PASS.** `EXT-05`'s broadened `L01` applies to `group_catering` specifically `applicable_when commercial_workflow = DIRECT`. `EXT-06`'s `payment_condition_by_order_type.group_catering_DIRECT` requires `PAID_IN_FULL`. `EXT-06A`'s `commercial_workflow_required: REVIEWED` field on `L02`–`L09` means none of those quote-path transitions are reachable for a `DIRECT` `group_catering` order — it has no path into `INQUIRY`/`QUALIFICATION`/`QUOTE_SENT`/`CUSTOMER_ACCEPTED` at all.

---

## TEST 05 — Custom Catering remains REVIEWED

**Given:** the customer asks for a nonstandard quantity/service/pricing arrangement requiring LOUMIES judgment.
**When:** the request is submitted.
**Then:** `order_type = group_catering`; `commercial_workflow = REVIEWED`; lifecycle may use `INQUIRY`/`QUALIFICATION`; LOUMIES issues `QUOTE_SENT`; `CUSTOMER_ACCEPTED` is separately recorded; no automatic capacity hold exists during the request/quote stage.

**Result: PASS.** Unmodified baseline `L02`–`L07` (now carrying `commercial_workflow_required: REVIEWED` per `EXT-06A`, which a `REVIEWED` order trivially satisfies) provide exactly this path, unchanged from V0. `capacity_hold`'s `explicitly_out_of_scope` bullets (unchanged, reaffirmed by `EXT-07` and new guardrail `G-18`) confirm no automatic hold exists at inquiry/quote stages.

---

## TEST 06 — Reviewed Catering quote does not hold the date

**Given:** a quote has been sent but not paid or confirmed.
**When:** another customer buys available standard business on the same date.
**Then:** the second customer may proceed if capacity permits; the quoted order has no confirmed capacity and no booked revenue.

**Result: PASS.** `QUOTE_SENT` is not a `counts_as_confirmed_revenue`-equivalent state in the baseline (`reporting_dimensions.pipeline_value` explicitly includes `QUOTE_SENT`, separate from `confirmed_booked_revenue`, unchanged by this patch). No capacity layer (`ALLOCATED`, `TEMPORARILY_HELD`, `CONFIRMED`) is touched by reaching `QUOTE_SENT` — `capacity_hold` is never created for a REVIEWED quote (`G-18`). The second, unrelated DIRECT or REVIEWED sale proceeds against actual remaining capacity, independently.

---

## TEST 07 — Reviewed Catering rechecks date/capacity before payment/confirmation

**Given:** the customer accepted a reviewed Catering quote, but date conditions changed before payment.
**When:** the customer attempts to pay.
**Then:** the system rechecks date/capacity before confirmation; the stale quote does not bypass the current date-level rule.

**Result: PASS.** New guardrail `G-19` states this explicitly: capacity and any applicable owner-review rule are rechecked concurrency-safely against *current* state immediately before payment/confirmation, not against the state at quote-issuance time. `L08`'s gate (`gates.to_CONFIRMED`, `EXT-06`) is evaluated at the moment of the `CUSTOMER_ACCEPTED → CONFIRMED` transition attempt, not cached from an earlier point — and per the closeout addendum, the "applicable owner-review rule" explicitly includes the `aggregate_date_level_review_threshold` (`ADD-04`): a REVIEWED Catering order checks the *same* cross-path date-level aggregate as a DIRECT order does, immediately before `CONFIRMED`, not only date/item capacity. A stale quote that would now push the date over the threshold is routed to `EXCEPTION` (`L12`) at this recheck, exactly as a DIRECT order would be.

---

## TEST 08 — Two simultaneous DIRECT Catering checkouts, one unit remaining

**Given:** only one unit/slot of the relevant sellable capacity remains.
**When:** two DIRECT Catering customers reach checkout concurrently.
**Then:** only one `ACTIVE` hold can reserve the final capacity; the other checkout sees the unit as unavailable/routes to review rather than both payments succeeding.

**Result: PASS.** `EXT-07` generalizes `capacity_hold` — including its unmodified guardrail ("a unit with an ACTIVE capacity_hold or CONFIRMED_CAPACITY status is not offered as available to a second checkout attempt") — to every DIRECT flow, `group_catering`-DIRECT included. The first checkout to create an `ACTIVE` hold on the last unit removes it from availability; the second checkout attempt sees no available unit to hold. This is the same mechanism V0 already used to close Case 1 (last same-day item), now applying to DIRECT Catering as well.

---

## TEST 09 — DIRECT group orders have no fake quote events

**Given:** any DIRECT `group_table` or DIRECT `group_catering` order.
**Then:** the audit trail contains no `QUOTE_SENT` and no `CUSTOMER_ACCEPTED`, unless a genuine later reviewed amendment creates a separate, distinctly audited event/path. The system never synthesizes those states merely for compatibility.

**Result: PASS.** Structural, not merely policy: `EXT-06A` restricts `L02`–`L09` (the only transitions that can produce `QUOTE_SENT`/`CUSTOMER_ACCEPTED`) to `commercial_workflow_required: REVIEWED`. A DIRECT order has no transition in the model capable of reaching either state. New guardrail `G-16` states this as an explicit, standalone rule. A later reviewed amendment (per `ADD-02`'s guardrails) would be a separate, distinctly audited record — never a retroactive rewrite of the original DIRECT order's history.

---

## TEST 10 — Institutional procurement remains orthogonal

**Given:** institutional Catering requiring PO authorization.
**Then:** `commercial_workflow = REVIEWED`; `order_type` remains `institutional`; `procurement_status = PENDING` blocks `CONFIRMED`; after authorized PO and other gates, the order may confirm under existing payment terms. No new commercial-workflow value is needed.

**Result: PASS.** `ADD-03`'s matrix fixes `institutional → REVIEWED` (the only allowed value). Baseline `procurement_states`/`procurement_transitions` and `gates.to_CONFIRMED`'s procurement condition ("never PENDING or DECLINED") are entirely unchanged by this patch — see `explicitly_unchanged`. `EXT-06`'s institutional payment branch is quoted verbatim from V0, unmodified. No `PROCUREMENT_AUTHORIZED`-as-workflow or any third `commercial_workflow` value is introduced anywhere in the patch (`self_check` confirms exactly two values). Per the closeout addendum, once this order's procurement and payment gates are otherwise satisfied it *also* checks the cross-path `aggregate_date_level_review_threshold` (`ADD-04`) immediately before `CONFIRMED`, exactly like DIRECT and REVIEWED Catering do — institutional confirmed business is not a silent exception to the date-level aggregate, only `same_day` is.

---

## TEST 03/07/10 CLOSEOUT NOTE

These three tests were amended by the 2026-09-04 closeout addendum, which corrected `ADD-04.scope_of_aggregation_status` from OPEN to OWNER-LOCKED cross-path scope: the $3,000 aggregate always counted *all* CONFIRMED scheduled/advance business for the date (DIRECT and REVIEWED alike, institutional included), never only DIRECT orders. No other test in this suite changes, and no 16th test was added.

---

## TEST 11 — Company/institution customer using ordinary direct checkout

**Given:** a Cornell department or company user buys an ordinary published direct offer by card, and no procurement process applies to that specific transaction.
**Then:** the customer/account may retain institutional/company identity, but the order uses the truthful DIRECT operational `order_type`; `procurement_status = NOT_APPLICABLE`. Identity does not force institutional order semantics.

**Result: PASS.** `ADD-03`'s `customer_identity_guardrail` states this exactly: order_type is determined by the transaction, never by customer/account identity. The resulting order is `scheduled_preorder`, `same_day`, `group_table`, or DIRECT `group_catering` (whichever the transaction is) with `procurement_status = NOT_APPLICABLE` per `EXT-06C`, while the customer/account record separately retains its institutional identity — a fact this state-model patch does not touch (customer/account identity lives outside the order-record dimensions this patch modifies).

---

## TEST 12 — Mixed customer relationship

**Given:** the same customer has a DIRECT Table, a REVIEWED Catering request, and a scheduled Regular Menu preorder.
**Then:** frontstage may present one coherent LOUMIES relationship; backstage has linked order records with independent `order_type`/`commercial_workflow`/state; one component does not block another merely because its workflow differs.

**Result: PASS, by construction.** Nothing in V0 or this patch couples one order record's lifecycle/payment/procurement/fulfillment to another order record's. Each of the three orders in this scenario is an independent record governed entirely by its own `order_type` and `commercial_workflow` (`group_table`/DIRECT, `group_catering`/REVIEWED, `scheduled_preorder`/DIRECT respectively) and proceeds through the model exactly as Tests 01, 05, and V0's original scheduled-preorder case already establish, with no cross-order gate anywhere in either document.

---

## TEST 13 — Cancellation does not leave phantom capacity

**Given:** a confirmed DIRECT Table or Catering order is legitimately reduced/cancelled.
**Then:** active confirmed/production quantities and date-level capacity update; any retained cancellation money remains payment/financial truth and does not continue consuming production capacity.

**Result: PASS.** Unmodified baseline `L16` (`→ CANCELLED`, applicable to `[all]` order types, so `group_table` and `group_catering` are covered without any patch change) removes the order from `PRODUCTION_COMMITTED`/the active manifest as part of the same transition (baseline guardrail, unchanged) and releases previously-held/confirmed capacity (baseline `capacity_hold` and `capacity_layers` mechanics, generalized to DIRECT flows by `EXT-07`). `payment_status` (e.g. retained via `REFUND_OR_REVERSAL_PENDING`/`REFUNDED_OR_REVERSED`, unchanged) is tracked entirely independently of capacity/production state — money retained on cancellation is a payment-dimension fact, never a production-capacity fact, consistent with V0's four-orthogonal-dimension design that this patch preserves.

---

## TEST 14 — Same-day remains a separate pool

**Given:** same-day released inventory is available while scheduled confirmed business is at or near the $3,000 threshold.
**Then:** the same-day order uses `order_type = same_day`, `commercial_workflow = DIRECT`, its own released-inventory/hold logic, and remains outside the scheduled $3,000 booked-revenue counter, as owner-approved.

**Result: PASS.** `ADD-04`'s `aggregate_date_level_review_threshold` explicitly lists `explicitly_excluded: [same_day]`, with the exclusion rationale citing same-day's status as a separate released-inventory pool (unchanged baseline `capacity_layers`/`pool_reallocation`, reaffirmed by `EXT-08` and left otherwise untouched). `same_day`'s `commercial_workflow` is fixed to `DIRECT` by `ADD-03`'s matrix, consistent with its existing baseline treatment.

---

## TEST 15 — No new lifecycle state machine

**Validation rule:** V0.1 introduces no new lifecycle states. Existing payment/procurement/fulfillment states remain intact. New behavior is expressed through `group_table`, `commercial_workflow`, transition applicability, and the generalized DIRECT checkout hold/gates.

**Result: PASS.** `self_check` in the patch file confirms: 11 lifecycle states before and after; `payment_states`, `procurement_states` (except one applicability-list addition), and `fulfillment_states` are listed under `explicitly_unchanged`; `group_table` and DIRECT `group_catering` reuse the exact same `lifecycle_states`, `lifecycle_transitions` (extended in place, not duplicated), `payment_states`, `procurement_states`, and `fulfillment_states` as every other order type — confirmed programmatically: parsing both `LOUMIES_ORDER_STATE_MODEL_V0.yaml` and `LOUMIES_ORDER_STATE_MODEL_V0_1_PATCH.yaml` shows zero new lifecycle-state identifiers introduced by the patch.

---

## Summary

| Test | Result |
|---|---|
| 01 — Standard Table direct confirmation | PASS |
| 02 — Table payment failure | PASS |
| 03 — Table over $3,000 date-level rule | PASS |
| 04 — Standard direct Catering | PASS |
| 05 — Custom Catering remains REVIEWED | PASS |
| 06 — Reviewed quote does not hold date | PASS |
| 07 — Reviewed Catering payment recheck | PASS |
| 08 — Two simultaneous DIRECT checkouts | PASS |
| 09 — No fake quote events on DIRECT orders | PASS |
| 10 — Institutional procurement orthogonal | PASS |
| 11 — Company customer, ordinary direct checkout | PASS |
| 12 — Mixed customer relationship | PASS |
| 13 — Cancellation releases capacity, not money-truth | PASS |
| 14 — Same-day separate pool, outside $3,000 counter | PASS |
| 15 — No new lifecycle state machine | PASS |

**15 of 15 required acceptance cases pass** against the patched model (approved V0 baseline + `LOUMIES_ORDER_STATE_MODEL_V0_1_PATCH.yaml`), traced structurally against the patch's explicit elements rather than executed, since this repository is a specification artifact set with no application code or test harness to run.

---

## Aggregation-scope note — resolved by the 2026-09-04 closeout addendum

An earlier version of this document flagged the $3,000 aggregate rule's scope (`ADD-04`) as an open question: whether REVIEWED `group_catering` or `institutional` confirmed revenue counts toward the same-date aggregate alongside the DIRECT order types. That question was never actually open — Rania had already owner-locked it as a cross-path rule before this patch was first written. `ADD-04.scope_of_aggregation_status` is now `OWNER-LOCKED`: the aggregate counts ALL CONFIRMED scheduled/advance business for the date — `scheduled_preorder`, `group_table`, `group_catering` (DIRECT or REVIEWED, once CONFIRMED), and legitimately confirmed `institutional` orders — with `same_day` as the sole explicit exclusion (Test 14, unchanged). Tests 03, 07, and 10 above reflect this cross-path scope.
