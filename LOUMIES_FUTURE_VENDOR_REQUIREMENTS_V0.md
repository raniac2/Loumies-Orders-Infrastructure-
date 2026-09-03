# LOUMIES — Future Vendor-Neutral Requirements Checklist (V0)

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

## 2. Cutoff Control

- [ ] Allow the operator to change the cutoff time/rule per operating date without a code change or vendor support ticket.
- [ ] Apply cutoff independently per operating date (not one fixed rule for the whole calendar).
- [ ] Distinguish cutoff-driven expiration (`EXPIRED`, per the state model) from cancellation.

## 3. Order / Date Capacity

- [ ] Support a configurable ceiling on total orders (or order volume) per operating date, independent of item-level limits.
- [ ] Stop offering a date for new orders once date capacity is reached, while leaving other dates unaffected.
- [ ] Allow the operator to adjust date capacity up or down at any time before it's exhausted.
- [ ] Decrement date capacity exactly once per order, only at the point the order reaches a `CONFIRMED`-equivalent state (never at cart/submission).
- [ ] Release date capacity on cancellation from any state that had already decremented it.

## 4. Item Inventory

- [ ] Support per-item, per-date quantity limits independent of date-level capacity.
- [ ] Mark an item sold out for a given date once its quantity reaches zero, without affecting other items on the same date.
- [ ] Allow the operator to set/adjust item availability manually.
- [ ] Support independent capacity tracking by operational category (e.g., a labor-intensive item constrained separately from a batch-scalable item) — see `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` §10-C. No specific categories or numbers need be pre-built; the requirement is that item-level and labor-sensitive limits can be modeled as separate dimensions, not collapsed into one number.

## 5. Same-Day Availability

- [ ] Support a same-day order type that is structurally and numerically separate from scheduled-preorder inventory (never shares a pool, never double-counts).
- [ ] Support a configurable total same-day capacity per date, settable to any value including zero (i.e., same-day ordering fully off for a date) — no built-in assumption of a fixed quantity.
- [ ] Support per-item same-day quantity limits, independent of the overall same-day ceiling.
- [ ] Support immediate manual pause/closure of same-day ordering as a whole, independent of remaining numeric capacity.
- [ ] Support enabling same-day ordering only on operator-selected dates, not automatically on every operating day.
- [ ] Support fulfillment-method eligibility (pickup/delivery) that can differ by date and by order type.

## 6. Pickup

- [ ] Support a pickup fulfillment method as one of potentially several fulfillment options per order — never assumed as the only option, and never conflated with order type.
- [ ] Support a pickup window/schedule that can be surfaced on a pickup manifest (see §12).
- [ ] Support recording pickup completion, and recording a pickup exception (no-show) as a distinct state, not silently treated as fulfilled or silently dropped.

## 7. Delivery

- [ ] Support a delivery fulfillment method alongside pickup, selectable per order where eligible.
- [ ] Support delivery eligibility that can be restricted by date, order type, or channel.
- [ ] Support recording delivery completion, and recording a delivery exception (failed attempt) as a distinct state.
- [ ] Support a placeholder for a delivery charge (value not set here — see open decisions G1).

## 8. Payment Capability

- [ ] Support recording payment status as a field independent of order status (an order can be `CONFIRMED` while payment is `PAID`, or `FULFILLED` while payment is still pending, per the state model's `INVOICED`/`PAID` split).
- [ ] Support a payment-failure path that leaves the order unconfirmed, non-capacity-consuming, and excluded from the production manifest.
- [ ] Support deposit-based payment (partial now, balance later) for group/catering and institutional orders, without forcing full payment at booking for those order types.
- [ ] Support invoicing as a distinct downstream step from fulfillment for institutional (and optionally group/catering) orders.
- [ ] Support recording procurement authorization (PO or equivalent) as a field distinct from both payment status and customer acceptance.
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
- [ ] Support capturing catering-specific intake fields (date, headcount, organization, fulfillment method, location, requested package, budget, dietary notes, contact info, special notes) at the inquiry stage, expandable as the request progresses.
- [ ] Support both custom (individually quoted) and standardized-package catering within the same workflow, without a separate parallel system for each.
- [ ] Ensure inquiry- and quote-stage catering records are excluded from confirmed-revenue reporting and from the production manifest until they reach a `CONFIRMED`-equivalent state.

## 12. Institutional Workflow

- [ ] Support a procurement-style workflow distinct from both cart checkout and standard catering: inquiry → quote/SOW → acceptance → procurement authorization → confirmed order → fulfillment → invoice → paid.
- [ ] Support representing procurement authorization (PO or equivalent) as a state distinct from customer acceptance, with the ability to require it before confirmation for institutions that need it, and to skip it for institutions that don't (see open decision F1).
- [ ] Support post-fulfillment invoicing with configurable payment terms (net terms placeholder — value not set here).
- [ ] Prevent an unauthorized or unpaid institutional order from being counted as confirmed production under any circumstance.

## 13. Exports

- [ ] Support exporting the production manifest (per operating date) in a form usable outside the vendor's own interface (e.g., file export), so LOUMIES is not locked into viewing production data only inside one vendor's UI.
- [ ] Support exporting pickup and delivery manifests separately.
- [ ] Support exporting raw order/event data for independent metrics calculation (§14 of the narrative doc), not only vendor-native dashboards.

## 14. Reporting

- [ ] Support reporting confirmed revenue separately by order type/channel (scheduled preorder, same-day, group/catering, institutional).
- [ ] Support reporting pipeline (inquiry/quote-stage) separately from confirmed revenue — never commingled in a single "sales" figure.
- [ ] Support reporting same-day sell-through (exposed vs. sold) and unsold same-day inventory.
- [ ] Support reporting cancellations, failed payments, and fulfillment failures as discrete, filterable event types, each with reason and initiating party where applicable.
- [ ] Support reporting revenue confirmed before vs. after preorder cutoff for a given date.

## 15. APIs / Integration Capability

- [ ] Support some form of programmatic or file-based data access (API, export, or equivalent) so LOUMIES's own order/production data is not permanently trapped in one vendor's proprietary interface.
- [ ] Support the possibility of a future marketplace channel feeding into the same order model without requiring a parallel, disconnected system (§3, §11) — capability only; no marketplace is chosen or assumed active.

## 16. Data Ownership

- [ ] LOUMIES (Rania) retains ownership of and export access to all order, customer, and production data generated during the proof, independent of which vendor is later selected.
- [ ] No vendor lock-in on the underlying order records themselves — the vendor's system implements the state model and fields defined in this V0; it does not redefine them.

## 17. Operator Controls

All items from `LOUMIES_ORDER_INFRASTRUCTURE_V0.md` §13 must be exposed in *some* form (interface unspecified — not a UI requirement, a capability requirement):

- [ ] Open/close an operating date
- [ ] Change the preorder cutoff
- [ ] Change fulfillment windows
- [ ] Set/reduce same-day capacity
- [ ] Set item availability / mark items sold out
- [ ] Pause all ordering / reopen ordering
- [ ] Review pending group/catering inquiries
- [ ] Record quote status and customer acceptance
- [ ] Record institutional procurement authorization
- [ ] Mark an order confirmed / fulfilled
- [ ] Cancel an order with a required reason
- [ ] Identify outstanding exceptions
- [ ] View production, pickup, and delivery manifests
- [ ] Distinguish committed revenue from pipeline in any summary view

## 18. Transaction Fees

- [ ] Support representing transaction/platform fees as a distinct field, separable by channel (§11), so channel-level contribution (§14) can eventually be calculated net of fees.
- [ ] **No fee structure, processor, or rate is assumed, compared, or recommended here.**

## 19. Marketplace Support

- [ ] Support the *possibility* of a future third-party marketplace channel being added as an additional channel value on the existing order model, without requiring the scheduled-preorder, same-day, catering, and institutional workflows to be rebuilt.
- [ ] Support channel-specific economics (§11) applying to a marketplace channel the same way they apply to direct channels, if/when adopted.
- [ ] **No marketplace is named, evaluated, or assumed adopted.** This is capability-only, per the governing context (§2 of the narrative doc): "may or may not be used."

## 20. Production Exports / Manifests

- [ ] Generate a production manifest per operating date containing only orders that have reached a `PRODUCTION_COMMITTED`-equivalent state (never inquiry- or quote-stage demand — see state model guardrails).
- [ ] Include in the manifest: confirmed order counts and revenue by order type; product quantities and totals; same-day inventory reserved; pickup/delivery schedules; production notes; dietary/allergen notes where collected; packaging quantities where derivable; flagged exceptions; and payment/procurement exceptions currently blocking production commitment.
- [ ] Automatically exclude/remove any order that transitions to a cancelled or expired state from the manifest, even if it was previously included.

---

## How to Use This Checklist Later

When a vendor is eventually evaluated (a separate, future exercise — not part of this V0), walk each numbered section against that vendor's actual capabilities. A "no" on any item is not automatically disqualifying — it may instead surface a workaround, a manual process, or a genuine gap to weigh against other candidates. What this checklist prevents is the reverse failure mode: designing LOUMIES's business rules around whatever one vendor happens to support well, rather than checking vendors against rules LOUMIES already knows it needs.
