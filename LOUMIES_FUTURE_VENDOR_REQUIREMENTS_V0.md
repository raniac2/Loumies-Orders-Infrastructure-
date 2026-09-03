# LOUMIES — Future Vendor-Neutral Requirements Checklist (V0, corrected)

**Correction note (Rio adjudication, two passes):** this checklist reflects (1) the orthogonal lifecycle/payment/procurement/fulfillment architecture, a temporary checkout capacity-hold mechanism, and reallocatable (not permanently isolated) scheduled-preorder/same-day pools; and (2) a final surgical patch adding `PAYMENT_NOT_YET_DUE` to the payment dimension (§8, §12, §14) and clarifying that the still-open third-day trigger and same-day settings never block Order #1 or other channels (§1, §5). See the corresponding corrections in `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` and `LOUMIES_ORDER_STATE_MODEL_V0.yaml`.

**Purpose:** Translate `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` and `LOUMIES_ORDER_STATE_MODEL_V0.yaml` into a checklist that can be used to evaluate *any* future commerce/payment/ordering vendor against LOUMIES's own requirements.

**This document names no vendor, ranks no vendor, scores no vendor, and recommends no vendor.** It is a checklist of required capabilities only. Evaluate candidates against it later; do not read a candidate's marketing back into it now.

Each item below is a capability requirement, phrased as "the system must be able to…". A "✓ / must confirm" checklist format is used deliberately so this can be walked line-by-line against any candidate later.

---

## 1. Scheduled Orders

- [ ] Present only operating dates that are currently open, within cutoff, and have available date-level capacity.
- [ ] Support a configurable cutoff per operating date (not a single global hardcoded cutoff).
- [ ] Block submission once cutoff has passed for a given date, with a clear customer-facing reason.
- [ ] Support order modification pre-confirmation without corrupting capacity accounting.
- [ ] Support a capacity-aware, non-silent modification path post-confirmation (or explicitly disallow modification post-confirmation, per §4/C2 of the open-decisions register, until that policy is set).
- [ ] Represent lifecycle status (`CONFIRMED`, `PRODUCTION_COMMITTED`, etc.) independently of payment status — never infer one from the other (see §8 below).
- [ ] Support opening and accepting orders on the two normal operating days independently of whether the conditional third day's activation trigger has been resolved — the conditional day's open status must never block launch or Order #1 on the two normal days.

## 2. Cutoff Control

- [ ] Allow the operator to change the cutoff time/rule per operating date without a code change or vendor support ticket.
- [ ] Apply cutoff independently per operating date (not one fixed rule for the whole calendar).
- [ ] Distinguish cutoff-driven expiration (`EXPIRED`, per the state model) from cancellation.

## 3. Order / Date Capacity

- [ ] Support a configurable ceiling on total orders (or order volume) per operating date, independent of item-level limits.
- [ ] Stop offering a date for new orders once its allocated-and-unheld capacity is reached, while leaving other dates unaffected.
- [ ] Allow the operator to adjust sellable/allocated date capacity up or down at any time, subject to not revoking already-held or already-confirmed units (see §21).
- [ ] Decrement into confirmed capacity exactly once per order, atomically with the order's lifecycle transition into `CONFIRMED` (never at cart/submission, and never twice).
- [ ] Release held or confirmed date capacity on hold-expiry, hold-abandonment, cancellation, or expiration.

## 4. Item Inventory

- [ ] Support per-item, per-date quantity limits independent of date-level capacity.
- [ ] Mark an item sold out for a given date once its quantity reaches zero, without affecting other items on the same date.
- [ ] Allow the operator to set/adjust item availability manually.
- [ ] Support independent capacity tracking by operational category (e.g., a labor-intensive item constrained separately from a batch-scalable item) — see `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` §10-C. No specific categories or numbers need be pre-built; the requirement is that item-level and labor-sensitive limits can be modeled as separate dimensions, not collapsed into one number.

## 5. Same-Day Availability

- [ ] Support a same-day allocation pool that is separately identifiable and accounted for from the scheduled-preorder pool — never silently double-counted — while remaining **explicitly reallocatable** between the two by the operator (corrected: not a permanently walled-off pool; see §22).
- [ ] Support a configurable total same-day capacity per date, settable to any value including zero (i.e., same-day ordering fully off for a date) — no built-in assumption of a fixed quantity.
- [ ] Support per-item same-day quantity limits, independent of the overall same-day ceiling.
- [ ] Support immediate manual pause/closure of same-day ordering as a whole, independent of remaining numeric capacity.
- [ ] Support enabling same-day ordering only on operator-selected dates, not automatically on every operating day.
- [ ] Support fulfillment-method eligibility (pickup/delivery) that can differ by date and by order type.
- [ ] Support the same temporary capacity-hold mechanism at same-day checkout as scheduled preorder (see §21) — same-day is not exempt from the oversell-race correction.
- [ ] Support same-day capacity being set to zero (fully OFF) for a date, or for the entire proof, **without impairing** scheduled-preorder, group/catering, or institutional order types in any way — same-day is optional for the proof, not a dependency for other channels.

## 6. Pickup

- [ ] Support a pickup fulfillment method as one of potentially several fulfillment options per order — never assumed as the only option, and never conflated with order type.
- [ ] Support a pickup window/schedule that can be surfaced on a pickup manifest (see §12).
- [ ] Support recording pickup completion, and recording a pickup exception (no-show) as a distinct state, not silently treated as fulfilled or silently dropped.

## 7. Delivery

- [ ] Support a delivery fulfillment method alongside pickup, selectable per order where eligible.
- [ ] Support delivery eligibility that can be restricted by date, order type, or channel.
- [ ] Support recording delivery completion, and recording a delivery exception (failed attempt) as a distinct state.
- [ ] Support a placeholder for a delivery charge (value not set here — see open decisions G1).

## 8. Payment Capability — corrected: four orthogonal status dimensions

- [ ] Support **lifecycle status** (`DRAFT`/`INQUIRY`/…/`CONFIRMED`/`PRODUCTION_COMMITTED`/…/`CLOSED`), **payment status** (`NOT_STARTED`/`PAYMENT_NOT_YET_DUE`/`PAYMENT_PENDING`/`PAYMENT_FAILED`/`DEPOSIT_RECEIVED`/`PAID_IN_FULL`/`INVOICED_OUTSTANDING`/refund states), **procurement status** (`NOT_APPLICABLE`/`NOT_REQUIRED`/`PENDING`/`AUTHORIZED`/`DECLINED`), and **fulfillment status** (`NOT_READY`/`READY`/`FULFILLED`/`FULFILLMENT_EXCEPTION`) as four independent fields on the order record — never inferring one from another.
- [ ] Support a scheduled-preorder order that is `lifecycle = CONFIRMED` and `payment = PAID_IN_FULL` at the same moment (payment required before confirmation for this order type), not only after fulfillment.
- [ ] Support an institutional order that is `fulfillment = FULFILLED` while `payment = INVOICED_OUTSTANDING`, without treating this as an error state.
- [ ] Support `payment = PAYMENT_NOT_YET_DUE` as **distinct from both `NOT_REQUIRED` and `INVOICED_OUTSTANDING`** — payment IS required, but is not yet due and no invoice/receivable currently exists. Support an institutional order reaching `lifecycle = CONFIRMED` with `procurement = AUTHORIZED` and `payment = PAYMENT_NOT_YET_DUE`, i.e. confirmed and authorized before any cash is collected, where the counterparty's approved payment terms permit it. Support the later transitions `PAYMENT_NOT_YET_DUE → INVOICED_OUTSTANDING → PAID_IN_FULL` as the order progresses. Never count `PAYMENT_NOT_YET_DUE` as paid revenue or as an outstanding receivable (see §14).
- [ ] Support a payment-failure path that leaves the order's lifecycle short of `CONFIRMED`, non-capacity-consuming, and excluded from the production manifest.
- [ ] Support deposit-based payment (partial now, balance later) for group/catering and institutional orders, without forcing full payment at booking for those order types.
- [ ] Support invoicing as a distinct downstream step from fulfillment for institutional (and optionally group/catering) orders.
- [ ] Support recording procurement authorization as its own field with a clean affirmative `AUTHORIZED` value (not merely the absence of a blocker), distinct from both payment status and customer acceptance — see §12.
- [ ] Support representing different payment/fee structures per channel (§11) without a schema change per channel added later.
- [ ] **Not** assumed: any specific payment processor, integration, or API. This document defines only the capability surface a processor must satisfy.

## 9. Refunds

- [ ] Support recording a refund as a distinct, auditable event tied to a cancellation reason.
- [ ] Support partial refunds (e.g., deposit-only scenarios) as well as full refunds, structurally — no refund *policy* (percentage, window, fee) is defined here; see open decisions C3/E4/F3.

## 10. Order Modification

- [ ] Support modifying an order pre-confirmation (item/quantity/date changes) with live capacity re-validation.
- [ ] Support an explicit, non-silent path for any post-confirmation modification, or an explicit block on post-confirmation modification if that is the eventual policy — either must be a deliberate configuration, not an accidental default.
- [ ] Log modification history (what changed, when, by whom — customer or operator) for audit and metrics purposes.

## 11. Catering Workflow

- [ ] Support a request-and-review workflow structurally distinct from cart checkout: inquiry → qualification → quote → acceptance → (deposit) → confirmed booking.
- [ ] Support capturing catering-specific intake fields (date, headcount, organization, fulfillment method, location, requested package, budget, dietary notes, contact info, special notes) at first contact, expandable as the request progresses.
- [ ] Support both custom (individually quoted) and standardized-package catering within the same workflow, without a separate parallel system for each.
- [ ] Support issuing a quote directly from a single complete initial request **without requiring separately recorded inquiry/qualification waypoints** when the request already contains what those steps would have captured — corrected: do not force artificial state theater (see infrastructure doc §6).
- [ ] Nonetheless keep inquiry, quote, acceptance, confirmed booking, production commitment, fulfillment, and payment as distinct recorded events/timestamps, even when they occur close together — evidentiary integrity, not bureaucratic ceremony.
- [ ] Ensure inquiry- and quote-stage catering records are excluded from confirmed-revenue reporting and from the production manifest until lifecycle reaches `CONFIRMED`.

## 12. Institutional Workflow

- [ ] Support a procurement-style workflow distinct from both cart checkout and standard catering: inquiry → quote/SOW → acceptance → procurement status → confirmed order → fulfillment → invoice → paid.
- [ ] Support a **clean affirmative `AUTHORIZED` procurement status** (not merely the absence of a blocker) as distinct from customer acceptance, with the ability to require it before confirmation for counterparties that need it, and represent `NOT_REQUIRED` for counterparties that don't — **per counterparty, never a universal institutional default** (see open decision F1).
- [ ] Support post-fulfillment invoicing with configurable payment terms (net terms placeholder — value not set here).
- [ ] Prevent an order whose procurement status is `PENDING` or `DECLINED` from reaching `CONFIRMED` or `PRODUCTION_COMMITTED` under any circumstance, regardless of customer acceptance **or approved payment terms** — `PAYMENT_NOT_YET_DUE` never substitutes for or bypasses procurement authorization.

## 13. Exports

- [ ] Support exporting the production manifest (per operating date) in a form usable outside the vendor's own interface (e.g., file export), so LOUMIES is not locked into viewing production data only inside one vendor's UI.
- [ ] Support exporting pickup and delivery manifests separately.
- [ ] Support exporting raw order/event data for independent metrics calculation (§14 of the narrative doc), not only vendor-native dashboards.

## 14. Reporting — corrected: six distinct figures, never silently combined

- [ ] Support reporting, as separate figures that are never silently combined into one "revenue" or "sales" number: **pipeline value** (lifecycle pre-`CONFIRMED`), **confirmed/booked revenue** (lifecycle ≥ `CONFIRMED`, regardless of payment), **paid revenue** (`payment = PAID_IN_FULL`, regardless of lifecycle/fulfillment), **fulfilled revenue** (`fulfillment = FULFILLED`, regardless of payment), **outstanding receivable** (`payment = INVOICED_OUTSTANDING` or a deposit with balance remaining), and **cancelled/reversed value**.
- [ ] Support slicing each of the six figures above by order type/channel (scheduled preorder, same-day, group/catering, institutional).
- [ ] Support `payment = PAYMENT_NOT_YET_DUE` counting toward confirmed/booked revenue but toward **neither** paid revenue **nor** outstanding receivable — booked, but not yet payable or invoiced (no seventh mandatory metric; a clarification of how the six figures treat this state).
- [ ] Support reporting same-day sell-through (allocated vs. sold), unsold same-day inventory, and the effect of any deliberate operator reallocation between pools (see §22).
- [ ] Support reporting cancellations, failed payments, and fulfillment failures as discrete, filterable event types, each with reason code and initiating party where applicable.
- [ ] Support reporting revenue confirmed before vs. after preorder cutoff for a given date.

## 15. APIs / Integration Capability

- [ ] Support some form of programmatic or file-based data access (API, export, or equivalent) so LOUMIES's own order/production data is not permanently trapped in one vendor's proprietary interface.
- [ ] Support the possibility of a future marketplace channel feeding into the same order model without requiring a parallel, disconnected system (§3, §11) — capability only; no marketplace is chosen or assumed active.

## 16. Data Ownership

- [ ] LOUMIES (Rania) retains ownership of and export access to all order, customer, and production data generated during the proof, independent of which vendor is later selected.
- [ ] No vendor lock-in on the underlying order records themselves — the vendor's system implements the state model and fields defined in this V0; it does not redefine them.

## 17. Operator Controls

All items from `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` §13 must be exposed in *some* form (interface unspecified — not a UI requirement, a capability requirement):

- [ ] Open/close an operating date, and reassign which weekday it falls on (day-count architecture is fixed; weekday identity is not — see open decision A3b)
- [ ] Change the preorder cutoff
- [ ] Change fulfillment windows
- [ ] Set sellable capacity per date/item (which may sit below physical capacity), and deliberately **reallocate** allocated-but-unheld capacity between the scheduled-preorder and same-day pools, with the reallocation recorded as an auditable event
- [ ] Set item availability / mark items sold out
- [ ] Pause all ordering / reopen ordering
- [ ] Review pending group/catering and institutional inquiries
- [ ] Record quote status and customer acceptance
- [ ] Record institutional procurement status (`PENDING`/`AUTHORIZED`/`DECLINED`/`NOT_REQUIRED`) per counterparty
- [ ] Mark an order's lifecycle confirmed / production-committed, and mark fulfillment ready / fulfilled
- [ ] Cancel an order with a required reason code
- [ ] Identify outstanding exceptions (lifecycle `EXCEPTION` and `FULFILLMENT_EXCEPTION`)
- [ ] View the active (regenerating) production, pickup, and delivery manifests
- [ ] Distinguish pipeline value, confirmed/booked revenue, paid revenue, and outstanding receivable from one another in any summary view (see §14)

## 18. Transaction Fees

- [ ] Support representing transaction/platform fees as a distinct field, separable by channel (§11), so channel-level contribution (§14) can eventually be calculated net of fees.
- [ ] **No fee structure, processor, or rate is assumed, compared, or recommended here.**

## 19. Marketplace Support

- [ ] Support the *possibility* of a future third-party marketplace channel being added as an additional channel value on the existing order model, without requiring the scheduled-preorder, same-day, catering, and institutional workflows to be rebuilt.
- [ ] Support channel-specific economics (§11) applying to a marketplace channel the same way they apply to direct channels, if/when adopted.
- [ ] **No marketplace is named, evaluated, or assumed adopted.** This is capability-only, per the governing context (§2 of the narrative doc): "may or may not be used."

## 20. Production Exports / Manifests

- [ ] Generate a production manifest per operating date containing only orders with lifecycle `PRODUCTION_COMMITTED` (never inquiry- or quote-stage demand — see state model guardrails).
- [ ] Include in the manifest: confirmed order counts and revenue by order type; product quantities and totals; same-day allocation (reflecting any reallocation); pickup/delivery schedules; production notes; dietary/allergen notes where collected; packaging quantities where derivable; flagged exceptions; and payment/procurement exceptions currently blocking production commitment.
- [ ] Treat the manifest as a **live, regenerating view** — corrected: automatically exclude/remove any order that transitions to `CANCELLED` or `EXPIRED`, or that is deliberately walked back out of `PRODUCTION_COMMITTED` before production begins, and regenerate the active manifest to reflect current valid commitments, while optionally retaining prior snapshots for audit (never presenting a stale snapshot as current).

## 21. Capacity Hold / Checkout-Race Prevention

- [ ] Support a temporary capacity reservation ("hold") created the moment a customer begins checkout against a specific unit of scheduled-preorder or same-day capacity, before payment completes.
- [ ] Ensure a held unit is not offered as available to a second, simultaneous checkout attempt — the mechanism that makes it structurally impossible for two customers to both successfully purchase the same final unit (Case 1 in the infrastructure doc's validation set).
- [ ] Support **atomic conversion** of a hold into confirmed capacity in the same operation that confirms the order, only on payment success.
- [ ] Support **automatic release** of a hold — back to its allocation pool — on payment failure, customer abandonment, or hold expiry, with a configurable expiry duration (value not set here; see open decision C5).
- [ ] **Not** required: automatic capacity holds during group/catering or institutional inquiry/quote stages. An optional, manually operator-initiated date hold for a serious prospect is a plausible future tool, tracked as open (see open decision E6), not implemented as a default rule.

## 22. Capacity Layers & Pool Reallocation

- [ ] Support representing capacity in (at minimum) five distinct layers per date/item/labor-sensitive category: **physical** (production ceiling), **sellable** (what the operator chooses to expose — may be below physical), **allocated** (sellable capacity divided among pools), **temporarily held** (active checkout holds), and **confirmed** (consumed by `CONFIRMED`-or-later orders).
- [ ] Never automatically expose all physically possible production as sellable capacity — sellable capacity is always an operator choice, which may sit below the physical ceiling.
- [ ] Support the operator deliberately reallocating allocated-but-unheld-and-unconfirmed capacity between the scheduled-preorder and same-day pools at any time, as a discrete auditable event (source, destination, quantity, date/item, timestamp, operator identity).
- [ ] Prevent reallocation from ever revoking a unit already under an active hold or already confirmed.
- [ ] Prevent total allocated capacity, across all pools for a given date/item, from ever exceeding sellable capacity, regardless of how many reallocations occur.

---

## How to Use This Checklist Later

When a vendor is eventually evaluated (a separate, future exercise — not part of this V0), walk each numbered section against that vendor's actual capabilities. A "no" on any item is not automatically disqualifying — it may instead surface a workaround, a manual process, or a genuine gap to weigh against other candidates. What this checklist prevents is the reverse failure mode: designing LOUMIES's business rules around whatever one vendor happens to support well, rather than checking vendors against rules LOUMIES already knows it needs.
