# LOUMIES Order Infrastructure — V0 Vendor-Neutral Operating Contract

**Scope:** Limited paid proof, Ithaca, NY, approx. 3–6 months.
**Status:** Specification only. No vendor selected. No application built. No deployment performed.
**Audience:** Rania (owner/final authority), Rio (continuity/positioning), Mona (future technical implementation), and any future vendor evaluated against this document.

---

## 0. How to Read This Document

Every material business rule below carries one of four tags:

| Tag | Meaning |
|---|---|
| **[OWNER-LOCKED]** | Explicitly decided by Rania. Treat as fixed unless she reopens it. |
| **[WORKING ASSUMPTION]** | Currently being modeled or tested for the proof. Not a permanent rule. |
| **[CONFIGURABLE]** | The future system must let this change without redesign. A value, not an architecture. |
| **[OPEN]** | Not yet decided, or not yet supported by evidence. No answer is invented here. |

Untagged structural statements (e.g., "an order has a status") are architecture, not business decisions, and are not subject to this classification.

Every OPEN and WORKING ASSUMPTION item referenced here is tracked in full in `LOUMIES_ORDER_OPEN_DECISIONS_V0.md`. This document does not re-argue those items; it only marks where they attach.

**Governing-artifact cross-check:** No existing LOUMIES governance, Customer Interaction OS, public-experience, or operating-model artifacts were present in this repository at the time this V0 was written (the repository had no prior commits). This document therefore uses the assignment prompt as its governing source. If governing LOUMIES artifacts exist elsewhere (with Rio, in another repository, in prior conversations), **cross-checking this V0 against them remains required before implementation.** Any conflict found in that cross-check should be surfaced, not silently resolved.

---

## 1. Purpose

This document specifies — independent of any commerce or payment vendor — how LOUMIES needs its ordering operation to work during the Ithaca proof: how orders are accepted, how different kinds of demand are distinguished, how orders are confirmed, how capacity and same-day availability are controlled, how group/catering and institutional demand are handled, how payment/procurement status is recorded, how production commitments and production information are generated for the founder, how fulfillment is recorded, how cancellations and exceptions are handled, and what proof-stage metrics must be capturable.

The purpose is to let LOUMIES evaluate any future software/vendor against LOUMIES's own requirements, rather than redesigning the business around whatever a vendor happens to offer.

---

## 2. Governing Proof Context

These facts govern this V0 and are treated as given, not re-derived:

- LOUMIES is evaluating a **limited paid proof** in Ithaca, NY. **[WORKING ASSUMPTION]** — the proof itself, including whether it proceeds, is a decision outside this document's scope.
- Approximate proof duration: **3–6 months**. **[WORKING ASSUMPTION]**
- **Two normal bookable operating days per week**, plus **one conditional/bookable third day** usable when sufficiently firm demand justifies it. **[WORKING ASSUMPTION]** — the day count is a test parameter; the *capability* to configure operating days is architecture. See §10.
- **No walk-in storefront.** **[WORKING ASSUMPTION]** for the proof.
- Production occurs from a **shared commercial-kitchen model**. **[WORKING ASSUMPTION]**
- **Rania Chidiac Kaldi** is founder and primary operator.
- Founder day maximum currently modeled at **12 hours**. **[WORKING ASSUMPTION]**
- Normal paid kitchen block currently modeled at **at least 10 hours per operating day**. **[WORKING ASSUMPTION]**
- **Pickup and delivery are fulfillment methods, not separate demand sources** — this is a structural decision reflected throughout the order model (see §8, §9).
- Historical demand is already substantially proved and is **not** part of this assignment.
- **Final proof menu: OPEN.**
- **2026 selling prices: OPEN.**
- **Technology/vendor selection: deliberately not yet decided** — out of scope for this document by design.
- **Deployment: deliberately out of scope** for this document by design.
- **Helper use during the proof:** Rania has stated occasional help may be operationally possible if needed. Whether helpers are used, what that implies for compliance, and whether helper labor must enter proof economics are **[OPEN]** — do not assume "founder-only" or "helpers freely available." See open-decisions register.

### Current Intended Commercial Channels

1. Scheduled individual preorder
2. Limited same-day ordering
3. Group/catering
4. Institutional purchasing

A future third-party marketplace channel **may** be used later. The architecture must permit it (see §11) but must not assume it — no marketplace channel is modeled as active in V0.

### Current Working Commercial Hypotheses **[WORKING ASSUMPTION — all of the following]**

- Scheduled preorder is the primary individual-consumer channel.
- Same-day ordering is deliberately capacity-limited, treated as incremental upside rather than required base revenue.
- Group/catering can provide concentrated revenue through batch-efficient production.
- Institutional purchasing may require quote/SOW/PO workflows rather than ordinary consumer checkout.

These are hypotheses the proof is designed to test, not locked commitments.

---

## 3. Channels vs. Order Types vs. Fulfillment — A Structural Distinction

Three concepts must stay separate in the data model, or the system will silently misclassify demand:

- **Order type** — *what kind of demand this is*: `scheduled_preorder`, `same_day`, `group_catering`, `institutional`. Determines the workflow (§4–§7) and the state machine path (see YAML).
- **Channel** — *where the order originated*: direct (LOUMIES-controlled ordering surface) today; a third-party marketplace potentially later. Determines economics (§11), not workflow logic.
- **Fulfillment method** — *how the order physically reaches the customer*: pickup or delivery. Never a demand source in its own right — every order type can carry either fulfillment method (subject to date/channel eligibility).

An order record therefore always carries all three as independent attributes (see §8).

---

## 4. Order Flow — Scheduled Individual Preorder

This is the primary individual-consumer channel under the current working hypothesis (§2).

### Conceptual sequence

```
customer sees eligible offering
  → selects available operating date
  → selects items and quantities
  → selects eligible pickup/delivery option
  → provides required customer information
  → order/payment attempt
  → successful confirmation
  → production commitment
  → production manifest
  → pickup/delivery
  → fulfilled
  → closed
```

"Eligible offering" means: only operating dates that are open, within the cutoff window, and carrying available date/item capacity are presented. What is "eligible" at any moment is a live function of the capacity model (§10), not a fixed calendar.

### Required behavior at each stress point

| Situation | Required conceptual behavior |
|---|---|
| **Cutoff has passed** | The date must no longer be offered as selectable for new preorders. An in-progress cart that crosses cutoff before submission must be blocked at submission with a clear "cutoff passed" outcome, not silently accepted. Exact cutoff timing is **[CONFIGURABLE / WORKING ASSUMPTION]** — see open decisions. |
| **Date capacity exhausted** | The date must stop being offered for new orders (or new order volume) once date-level capacity (§10-A) is reached, independent of whether individual items still have stock. |
| **Item capacity exhausted** | The specific item must be marked unavailable for that date while other items on the same date remain orderable. |
| **Item sold out** | Same as item capacity exhausted, but framed as a terminal state for that date (no more units regardless of remaining capacity elsewhere). |
| **Fulfillment date no longer available** | If a previously offered date is withdrawn (operator closes it — §13), any customer mid-flow on that date must be stopped before submission with a clear reason, not allowed to complete against a date that no longer exists. |
| **Customer attempts modification** | Modification of a `SUBMITTED`/`PAYMENT_PENDING` order before confirmation may be allowed to fail cleanly back to cart. Modification of a `CONFIRMED` or later order requires an explicit, capacity-aware re-check (it is not a silent overwrite) — final modification rules are **[OPEN]**. |
| **Customer requests cancellation** | Must be capturable as a distinct event with reason, and must release any capacity/inventory the order held (§9, §10). Refund/fee consequences of cancellation are **[OPEN]**. |
| **LOUMIES must cancel** | Same capacity-release requirement, tagged as `CANCELLED_BY_LOUMIES` with a reason, distinguishable from customer-initiated cancellation for later metrics (§14). |
| **Payment fails** | Order must remain in a non-confirmed state (`PAYMENT_FAILED`); it must **not** consume date/item capacity, and must **not** appear on any production manifest. |
| **Payment succeeds but an operational exception occurs** | Order must be able to enter an `EXCEPTION_HOLD` state that is visibly flagged for operator review without being silently treated as either fully confirmed-and-clean or lost. |
| **Fulfillment fails** (no-show, failed delivery attempt) | Must be recorded as a distinct fulfillment exception, not silently marked `FULFILLED`, and not silently left open with no record. |

No final refund, cancellation-fee, or modification policy is defined here — see `LOUMIES_ORDER_OPEN_DECISIONS_V0.md`.

---

## 5. Order Flow — Limited Same-Day Ordering

Same-day ordering must **not** behave like unlimited restaurant ordering. It is deliberately controlled availability, structurally separate from scheduled-preorder inventory.

### Required capabilities (all **[CONFIGURABLE]** — none of the numbers below are rules)

- Configurable total same-day order capacity (a ceiling on same-day orders for a given operating date).
- Configurable same-day item quantities (per-item ceilings independent of the overall same-day ceiling).
- Decrementing available same-day quantity at the moment an order is **confirmed** (not merely submitted — see the state model's prerequisite for capacity decrement).
- A sold-out state once a same-day quantity reaches zero, surfaced immediately to would-be customers.
- Manual adjustment of same-day capacity by the operator, at any point before it is exhausted.
- Manual immediate pause/closure of same-day ordering as a whole, independent of remaining capacity.
- Same-day ordering offered only on operator-selected operating dates (not automatically on every operating day).
- Fulfillment eligibility (pickup/delivery) that can differ by date and by order type.
- **Separate accounting** of scheduled-preorder inventory versus same-day speculative inventory — a unit reserved for same-day exposure is never silently drawn from, or double-counted against, scheduled-preorder stock, and vice versa.

### Explicit non-rule

**Five same-day orders per operating day is one possible proof-test scenario, not a permanent capacity rule.** The architecture must support testing 0, 5, 10, or any other exposed quantity without changing the underlying model — same-day capacity is a configuration value read by the same mechanism regardless of what that value is set to.

---

## 6. Order Flow — Group / Catering

Group/catering is structurally distinct from ordinary individual checkout — it is a request-and-review workflow, not a cart-and-checkout workflow.

### Conceptual sequence

```
inquiry/request
  → qualification
  → required information captured
  → LOUMIES review
  → quote
  → customer acceptance
  → deposit/payment requirement (if applicable)
  → confirmed booking
  → production commitment
  → fulfillment
  → final payment (if applicable)
  → closed
```

### Information the request must be capable of capturing (eventually — not all fields are mandatory at inquiry stage)

Requested date; headcount; organization/customer; fulfillment method; location; requested food/package; budget if supplied; dietary requirements/notes; contact information; special operating notes.

### Structural requirement

The architecture must support **both**:

- **(A)** Custom catering requiring individualized review and quotation, and
- **(B)** Potentially standardized group packages later,

without one precluding the other. A standardized package is simply a quote whose price/contents are pre-populated rather than manually composed; the underlying state machine (inquiry → quote → acceptance → confirmation) is the same either way.

### Not defined here (all **[OPEN]** — see register)

Minimum order size; deposit percentage; lead time requirements; cancellation policy; final group pricing.

---

## 7. Order Flow — Institutional

Institutional purchasing must support procurement workflows and must **not** force an institution through ordinary consumer checkout.

### Conceptual sequence

```
qualified institutional inquiry
  → date/headcount/scope
  → quote and/or SOW
  → institutional acceptance
  → required procurement authorization / PO / equivalent
  → confirmed institutional order
  → production commitment
  → fulfillment
  → invoice/payment workflow
  → paid
  → closed
```

### Hard requirements

The state model **must distinguish**, as separately observable states (see YAML):

`inquiry` → `quote requested` → `quote sent` → `customer verbal/written acceptance` → `procurement authorization` → `confirmed production commitment` → `fulfilled` → `invoiced` → `paid`.

Three rules follow directly from the assignment and are treated as structural, not merely stylistic:

1. **A quote is not booked demand.** An institutional order does not count toward confirmed revenue or production planning while it sits at `QUOTE_SENT`.
2. **A verbal yes is not procurement authorization** when an institution's process requires formal authorization (PO or equivalent). `CUSTOMER_ACCEPTED` and `PROCUREMENT_AUTHORIZED` are distinct states; the second does not follow automatically from the first.
3. **Unpaid/unconfirmed institutional demand is never counted as confirmed production.** Only orders that have reached the state model's confirmed-production gate (see §12, §9) enter the production manifest.

Whether a given institutional counterparty *requires* formal PO authorization, or whether written acceptance is sufficient, is itself institution-specific and is **[OPEN]** per counterparty until determined.

---

## 8. Common Internal Order Model

A single, vendor-neutral conceptual order record must be capable of representing all four order types. At minimum it carries fields for:

- Unique order ID
- Channel (direct today; marketplace-capable field for later — §11)
- Order type (`scheduled_preorder` / `same_day` / `group_catering` / `institutional`)
- Customer name/identifier
- Organization/account (if applicable — group/institutional)
- Source/referral source (if known)
- Requested fulfillment date
- Fulfillment method (pickup/delivery)
- Fulfillment window
- Order status (see state model)
- Payment status (independent of order status — see §9)
- Procurement/PO status (institutional; independent field, not folded into payment status)
- Product/item lines
- Quantities
- Unit/channel price (channel-specific — §11; no price values are set here)
- Discounts (placeholder only — none permitted or defined yet)
- Tax (placeholder only — no tax rules defined here; legal/vendor-dependent)
- Delivery charge (placeholder)
- Service charge (placeholder, if applicable)
- Packaging charge (placeholder, if applicable)
- Production commitment status (whether this order is currently reflected in a production manifest)
- Cancellation state
- Cancellation reason
- Exception status
- Operator notes
- Timestamps (created, each state transition, fulfilled, closed)
- Scheduled-vs-same-day classification (redundant with order type, but called out because §5 requires it to never be conflated with scheduled-preorder inventory accounting)

No tax, discount, or fee **rule** is defined by this document — only the fields that must exist so a future rule (legal, vendor, or owner-decided) has somewhere to attach without a schema redesign.

---

## 9. Formal Order State Model

The full formal state machine is specified in the companion artifact **`LOUMIES_ORDER_STATE_MODEL_V0.yaml`**, which is the authoritative source for states, transitions, prerequisites, and effects. This section summarizes its purpose and guardrails; the YAML governs on any conflict of detail.

### Purpose

The state model exists specifically to make the following errors structurally impossible, not just discouraged by convention:

- Inquiry-stage demand counted as booked revenue.
- Quote-stage demand counted as confirmed revenue.
- Failed payment treated as confirmed.
- Institutional order counted before required authorization.
- Cancelled order remaining on the production manifest.
- Sold-out inventory continuing to accept orders.
- Capacity decremented twice for one order.
- An order fulfilled without ever passing through a valid confirmed state.
- A fulfilled order automatically assumed paid when payment may legitimately occur later (e.g., institutional invoicing).

### What the YAML defines

- The full set of valid states, with which order types each applies to.
- Allowed transitions between states, including trigger, acting party (customer / system / operator), and prerequisites.
- Transitions explicitly marked as requiring operator action versus those that could later be automated.
- The effects of each transition on capacity/inventory (§10) and on production commitment (§12).
- Exception states and how an order enters/exits them.
- Guardrail rules stating invalid transitions outright (e.g., no direct `INQUIRY → PRODUCTION_COMMITTED`).

No webhook, vendor API, or executable trigger is defined — the YAML is a specification artifact, read by humans and future engineers, not a runtime configuration.

---

## 10. Capacity Control Model

The system must support multiple **independent** kinds of capacity that can each constrain an order without the others being touched.

### A. Date capacity
Can LOUMIES accept another order for this operating date, at all — independent of which item is being ordered. Governed at the operating-date level.

### B. Item capacity
How many additional units of a particular item can still be sold for a given date, independent of whether the date itself still has room for other items.

### C. Labor-sensitive capacity
The system must eventually allow labor-intensive products to be constrained **independently** from highly batch-scalable products — i.e., a date could be "full" for a shaping-intensive item while still open for a batch-scalable one, or vice versa. Potential operational categories that may eventually be used to model this (none of these are adopted as final classifications):

- batch-scalable
- shaping-intensive
- assembly-intensive
- oven-constrained
- vessel/equipment-constrained
- cooling/holding-constrained

**These categories are illustrative only.** No product is assigned to any of them in this document, and no numerical capacity is calculated here. Populating actual limits requires physical production measurement or an explicit owner decision — both **[OPEN]**.

### Capacity is checked, and decremented, at defined points

- **Availability check** happens continuously as customers browse (a date/item that has hit its ceiling stops being offered).
- **Decrement** happens once, at the transition into a `CONFIRMED`-class state (see YAML) — never at submission, and never twice for the same order. This is the mechanism that prevents double-decrement and prevents unconfirmed carts from silently consuming real capacity.
- **Release** happens on cancellation (customer- or LOUMIES-initiated) from any state that had already decremented capacity, restoring it for resale where operationally appropriate.

### Explicit non-goal

This section does not calculate a single final number for any date, item, or labor category. It specifies *that* the system must be able to hold and enforce such numbers once they exist, and that date capacity, item capacity, and labor-sensitive capacity are three separate levers, not one.

---

## 11. Channel-Specific Economics

LOUMIES must not be forced into one universal price/economic structure across all channels. The data model must be capable of representing, for a single product, different values across channels for:

- Selling price
- Packaging requirements
- Platform/payment fees
- Service/delivery charges
- Unit economics generally

across scheduled direct preorder, same-day direct ordering, a future third-party marketplace (if adopted), group/catering, and institutional.

This is a **capability requirement only.** No price is set, no marketplace is chosen, and no pricing premium is recommended by this document. The requirement exists so that, whenever prices and fees *are* decided (owner decision, vendor terms, marketplace terms), the order model already has a place for the value to live per channel — it does not need to be redesigned to add that dimension later.

---

## 12. Production Commitment and Production Manifest

The ordering infrastructure's most operationally critical output is telling Rania what must actually be produced, for which date, in what quantity — and nothing else.

### Definition

A **production commitment** is the state an order reaches only once it has satisfied every prerequisite the state model defines for its order type (payment/deposit as applicable, procurement authorization as applicable, customer acceptance as applicable — see YAML). Reaching production commitment is what makes an order eligible to appear on a production manifest.

### Production Manifest — required contents (conceptually)

For a given operating date, after the relevant cutoff, the manifest is capable of containing:

- Operating date
- Confirmed scheduled-order count
- Confirmed scheduled-order revenue
- Confirmed group/catering orders
- Confirmed institutional orders
- Product quantities required, and totals by product
- Same-day inventory deliberately reserved (as distinct from what was actually sold — §14)
- Pickup schedule
- Delivery schedule
- Order-specific production notes
- Dietary/allergen notes where collected
- Packaging quantities, where derivable from the above
- Exceptions requiring operator review
- Payment/procurement exceptions that are currently **preventing** an otherwise-ready order from reaching production commitment

### Critical rule

**Inquiry-stage and quote-stage demand must never inflate production requirements.** Only orders that have reached a production-commitment-eligible state in the YAML state model are summed into the manifest. An institutional quote sitting at `QUOTE_SENT`, or a catering inquiry at `QUALIFICATION_IN_PROGRESS`, contributes zero units and zero revenue to the manifest, however likely it seems.

---

## 13. Operator Control Requirements

This section defines the minimum controls Rania needs to operate the business behind the ordering surface. **This is not a UI design assignment** — no screens, layouts, or interface mockups are specified; these are capability requirements only.

The operator must eventually be able to:

- Open/close an operating date
- Change the preorder cutoff
- Change fulfillment windows
- Set or reduce same-day capacity
- Set item availability
- Mark an item sold out
- Pause all ordering
- Reopen ordering, where appropriate
- Review pending group/catering inquiries
- Record quote status
- Record customer acceptance
- Record institutional procurement authorization
- Mark an order confirmed
- Mark an order fulfilled
- Cancel an order, with a required reason
- Identify outstanding exceptions
- View the production manifest
- View the pickup manifest
- View the delivery manifest
- Distinguish committed revenue from potential pipeline (i.e., see confirmed-production totals separately from inquiry/quote-stage totals)

Every item above is a capability the future system must expose in *some* form; how it is exposed (dashboard, spreadsheet export, admin panel) is a vendor/implementation decision, explicitly deferred.

---

## 14. Paid-Proof Data and Metrics

The system must capture enough underlying event/order data, at the point each event occurs, to calculate the following later. No success threshold or GO/NO-GO number is defined here.

- Scheduled preorder: order count and revenue
- Same-day: order count and revenue
- Group/catering: order count and revenue
- Institutional: order count and revenue
- Average order value, by channel
- Same-day inventory exposed vs. same-day inventory sold (sell-through)
- Unsold same-day inventory
- Cancellation count (with reason and initiating party, per §4/§9)
- Failed payment count
- Fulfillment failures
- Revenue confirmed **before** the order enters the kitchen (i.e., prior to production commitment cutoff) vs. revenue generated **after** preorder cutoff (same-day)
- Channel-specific fees, once known
- Food cost by channel, once costs are known
- Contribution by channel, once the above are known
- Repeat-customer behavior, where legally and technically available

Capturing the *capacity* to compute these later is the requirement. No threshold, target, or GO/NO-GO number is set by this document — that is an owner decision made with real proof data, not a specification artifact.

---

## 15. Related Specification Files

- `LOUMIES_ORDER_STATE_MODEL_V0.yaml` — authoritative formal state machine (§9)
- `LOUMIES_ORDER_OPEN_DECISIONS_V0.md` — full decision register for every OPEN/WORKING ASSUMPTION item referenced above
- `LOUMIES_FUTURE_VENDOR_REQUIREMENTS_V0.md` — vendor-neutral requirements checklist derived from this specification

---

## 16. Non-Goals / Boundary Reminder

This document does not select a vendor, does not compare or research vendors, does not build or scaffold an application, does not create a frontend/backend/database, does not write payment or integration code, does not deploy anything, does not set final menu items or prices, does not set catering minimums or deposit percentages, does not set permanent same-day capacity or preorder cutoff values, and does not set a GO/NO-GO threshold. Anything that felt tempting to decide along the way and is not an explicit owner decision has been routed to `LOUMIES_ORDER_OPEN_DECISIONS_V0.md` instead of being resolved here.
