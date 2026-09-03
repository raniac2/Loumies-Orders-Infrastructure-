# LOUMIES Order Infrastructure — V0 Vendor-Neutral Operating Contract

**Scope:** Limited paid proof, Ithaca, NY, approx. 3–6 months.
**Status:** Specification only. No vendor selected. No application built. No deployment performed.
**Audience:** Rania (owner/final authority), Rio (continuity/positioning), Mona (future technical implementation), and any future vendor evaluated against this document.

---

## Correction Log

This version incorporates a Rio adjudication/correction pass against the wider LOUMIES governance record. Corrections made:

1. **Governance reclassification** — several proof parameters previously marked WORKING ASSUMPTION are OWNER-LOCKED for the current proof architecture (§2). "Number of operating days" is split from "which weekdays."
2. **Orthogonal state model** — the order model no longer forces commercial lifecycle, payment, procurement, and fulfillment into one linear sequence. They are four independent dimensions on every order record (§9; full formal detail in the YAML).
3. **Capacity-hold mechanism** — a temporary checkout reservation closes an oversell race the prior version left open (§5, §10).
4. **Reallocatable pools** — scheduled-preorder and same-day capacity must be separately accounted for but are not permanently walled off from each other; the operator can deliberately reallocate unused, unconfirmed capacity between them (§5, §10).
5. **Five-layer capacity model** — physical, sellable, allocated, held, and confirmed capacity are distinguished; LOUMIES may deliberately sell below physical capacity (§10).
6. **Catering/institutional shortcut** — a complete initial request does not require artificial intermediate states before a quote can issue (§6, §7).
7. **Production manifest as a live view** — the active manifest regenerates when legitimate commitments change; it is not an irreversible historical record (§12).
8. **Reporting correction** — pipeline, booked, paid, fulfilled, outstanding, and cancelled/reversed value are kept as separate reportable figures, never silently combined (§14).

**Final surgical patch (this version) adds:**

9. **Launch-blocking correction** — the still-open third-day activation trigger (A4) and the still-open same-day capacity/date/fulfillment settings (D1–D3) do not block Order #1 or the launch of the two normal (owner-locked) operating days, and do not block scheduled-preorder, group/catering, or institutional activation. They block only the conditional third day and the same-day channel/experiment, respectively (§2, §5).
10. **`PAYMENT_NOT_YET_DUE`** — a fourth payment-status value representing a legitimately confirmed, procurement-authorized institutional (or approved pay-on-terms) order whose payment is required but not yet due, distinct from `NOT_REQUIRED` and from `INVOICED_OUTSTANDING` (§7, §9, §14).

This document does not re-argue those items further in prose; each correction is reflected inline in the relevant section below and in the companion YAML/decision register.

---

## 0. How to Read This Document

Every material business rule below carries one of four tags:

| Tag | Meaning |
|---|---|
| **[OWNER-LOCKED]** | Explicitly decided by Rania. Treat as fixed unless she reopens it. |
| **[WORKING ASSUMPTION]** | Currently being modeled or tested for the proof. Not a permanent rule. |
| **[CONFIGURABLE]** | The future system must let this change without redesign. A value, not an architecture. |
| **[OPEN]** | Not yet decided, or not yet supported by evidence. No answer is invented here. |

Untagged structural statements (e.g., "an order has a lifecycle status") are architecture, not business decisions, and are not subject to this classification.

Every OPEN and WORKING ASSUMPTION item referenced here is tracked in full in `LOUMIES_ORDER_OPEN_DECISIONS_V0.md`. This document does not re-argue those items; it only marks where they attach.

**Governing-artifact cross-check:** This V0 was originally written without the wider LOUMIES governance record in the repository. This correction pass constitutes a Rio/Rania governance adjudication for the specific items addressed here (§2, and the corresponding rows in the open-decisions register). A broader cross-check against any other existing LOUMIES governance, Customer Interaction OS, public-experience, or operating-model artifacts remains required before implementation for anything not explicitly adjudicated in this pass.

---

## 1. Purpose

This document specifies — independent of any commerce or payment vendor — how LOUMIES needs its ordering operation to work during the Ithaca proof: how orders are accepted, how different kinds of demand are distinguished, how orders are confirmed, how capacity and same-day availability are controlled, how group/catering and institutional demand are handled, how payment/procurement status is recorded, how production commitments and production information are generated for the founder, how fulfillment is recorded, how cancellations and exceptions are handled, and what proof-stage metrics must be capturable.

The purpose is to let LOUMIES evaluate any future software/vendor against LOUMIES's own requirements, rather than redesigning the business around whatever a vendor happens to offer.

---

## 2. Governing Proof Context

### OWNER-LOCKED for the current proof architecture (unless Rania explicitly reopens them)

- **Geography:** Ithaca, NY. **[OWNER-LOCKED]**
- **Two normal bookable operating days per week**, plus **one conditional/bookable third day**. **[OWNER-LOCKED]** — this is the *architecture* (day count and its conditional structure). The specific weekday identities assigned to those days are a separate question — see below.
- **Founder day maximum: 12 hours.** **[OWNER-LOCKED]** for the current proof.
- **At least 10 paid kitchen hours per normal operating day.** **[OWNER-LOCKED]** for the current proof.
- **No walk-in storefront model.** **[OWNER-LOCKED]** for the proof.
- **Pickup and delivery are fulfillment methods, not separate demand channels.** **[OWNER-LOCKED]** — structural, reflected throughout the order model (§8, §9).
- **Group/catering and institutional purchasing are permitted proof order types.** **[OWNER-LOCKED]** — their operating parameters (minimums, deposits, lead time, per-counterparty procurement rules) remain open; that they are in scope as order types is not.

### Split explicitly: day count vs. weekday identity

- *How many* operating days (2 normal + 1 conditional) is **[OWNER-LOCKED]**.
- *Which weekdays* those are is **[OPEN / CONFIGURABLE]** — not decided here, and the system must allow the weekday assignment to change without touching the day-count architecture.

### Still WORKING ASSUMPTION / OPEN

- Production occurs from a **shared commercial-kitchen model**. **[WORKING ASSUMPTION]**
- **The condition/threshold that justifies activating the conditional third day is [OPEN].** No numerical trigger is invented here — it is a judgment call for Rania, informed by demand signals as the proof runs. **This open item blocks only the opening of the conditional third day — it does not block Order #1 or the launch of the two normal operating days**, which are owner-locked and independently activatable (see open decisions A4).
- **Final proof menu: OPEN.**
- **2026 selling prices: OPEN.**
- **Technology/vendor selection: deliberately not yet decided** — out of scope for this document by design.
- **Deployment: deliberately out of scope** for this document by design.
- **Helper use during the proof** has been reopened by Rania conceptually. Do not state "Rania alone, no helper ever." Do not state that helpers are freely available. Whether helpers are used, on what terms, what that implies for compliance, and whether/how helper labor enters proof economics are **[OPEN]** until specifically resolved. See open-decisions register.

### Current Intended Commercial Channels

1. Scheduled individual preorder
2. Limited same-day ordering
3. Group/catering
4. Institutional purchasing

A future third-party marketplace channel **may** be used later. The architecture must permit it (see §11) but must not assume it — no marketplace channel is modeled as active in V0.

### Current Working Commercial Hypotheses — corrected framing

- **Scheduled preorder as the primary individual-consumer channel is a WORKING COMMERCIAL HYPOTHESIS currently being tested, not an owner-locked permanent requirement.** **[WORKING ASSUMPTION]**
- **Limited same-day ordering is an EXPERIMENTAL / CONFIGURABLE WORKING HYPOTHESIS, not an owner-locked permanent channel requirement.** **[WORKING ASSUMPTION]** — the proof is explicitly testing whether and how much same-day capacity to expose, including the possibility of exposing none on a given date (see §5).
- Group/catering can provide concentrated revenue through batch-efficient production. **[WORKING ASSUMPTION]**
- Institutional purchasing may require quote/SOW/PO workflows rather than ordinary consumer checkout. **[WORKING ASSUMPTION — the general pattern; per-counterparty procurement requirements are OPEN, see §7]**

These are hypotheses the proof is designed to test, not locked commitments — distinct from the OWNER-LOCKED proof-architecture facts above.

---

## 3. Channels vs. Order Types vs. Fulfillment — A Structural Distinction

Three concepts must stay separate in the data model, or the system will silently misclassify demand:

- **Order type** — *what kind of demand this is*: `scheduled_preorder`, `same_day`, `group_catering`, `institutional`. Determines the workflow (§4–§7) and which lifecycle path applies (see YAML).
- **Channel** — *where the order originated*: direct (LOUMIES-controlled ordering surface) today; a third-party marketplace potentially later. Determines economics (§11), not workflow logic.
- **Fulfillment method** — *how the order physically reaches the customer*: pickup or delivery. Never a demand source in its own right, and never encoded as a status — it is a field on the order, independent of the fulfillment *status* dimension (see §9-D).

An order record therefore always carries all three as independent attributes (see §8).

---

## 4. Order Flow — Scheduled Individual Preorder

This is the working-hypothesis primary individual-consumer channel (§2) — being tested, not assumed proven.

### Conceptual sequence

```
customer sees eligible offering
  → selects available operating date
  → selects items and quantities
  → selects eligible pickup/delivery option
  → provides required customer information
  → availability check → temporary capacity hold (see §5, §10)
  → payment attempt
  → hold atomically converts to confirmed capacity on payment success
  → order = CONFIRMED, payment = PAID_IN_FULL
  → production commitment
  → production manifest
  → pickup/delivery → fulfillment = FULFILLED
  → CLOSED (once fulfilled + paid + any procurement dimension resolved)
```

"Eligible offering" means: only operating dates that are open, within the cutoff window, and carrying available date/item capacity are presented. What is "eligible" at any moment is a live function of the capacity model (§10), not a fixed calendar.

**Critical correction:** a scheduled consumer preorder that must be paid before confirmation reaches `lifecycle = CONFIRMED` and `payment = PAID_IN_FULL` **at the same moment** — not after fulfillment. See the formal gate definition in the YAML (`gates.to_CONFIRMED`).

### Required behavior at each stress point

| Situation | Required conceptual behavior |
|---|---|
| **Cutoff has passed** | The date must no longer be offered as selectable for new preorders. An in-progress cart that crosses cutoff before submission must be blocked at submission with a clear "cutoff passed" outcome, not silently accepted. Exact cutoff timing is **[CONFIGURABLE / WORKING ASSUMPTION]** — see open decisions. |
| **Date capacity exhausted** | The date must stop being offered for new orders (or new order volume) once date-level capacity (§10) is reached, independent of whether individual items still have stock. |
| **Item capacity exhausted** | The specific item must be marked unavailable for that date while other items on the same date remain orderable. |
| **Item sold out** | Same as item capacity exhausted, but framed as a terminal state for that date (no more units regardless of remaining capacity elsewhere). |
| **Two customers reach for the last unit simultaneously** | The temporary capacity hold mechanism (§5, §10) makes this race structurally impossible to resolve as a double-sale: the first checkout to hold the unit blocks the second from holding the same unit, rather than both racing to pay first. |
| **Fulfillment date no longer available** | If a previously offered date is withdrawn (operator closes it — §13), any customer mid-flow on that date must be stopped before submission with a clear reason, not allowed to complete against a date that no longer exists. |
| **Customer attempts modification** | Modification of a pre-hold or held-but-unconfirmed order may be allowed to fail cleanly back to cart. Modification of a `CONFIRMED` or later order requires an explicit, capacity-aware re-check (it is not a silent overwrite) — final modification rules are **[OPEN]**. |
| **Customer requests cancellation** | Must be capturable as a distinct event with reason, and must release any capacity/hold the order held (§9, §10). Refund/fee consequences of cancellation are **[OPEN]**. |
| **LOUMIES must cancel** | Same capacity-release requirement, with `reason_code = loumies_cancelled`, distinguishable from customer-initiated cancellation for later metrics (§14). |
| **Payment fails** | `payment_status = PAYMENT_FAILED`. The order must remain outside `lifecycle = CONFIRMED`; its capacity hold, if any, is released or allowed to expire; it must **not** appear on any production manifest. |
| **Payment succeeds but an operational exception occurs** | The order enters `lifecycle = EXCEPTION`, visibly flagged for operator review, without being silently treated as either fully confirmed-and-clean or lost. |
| **Fulfillment fails** (no-show, failed delivery attempt) | `fulfillment_status = FULFILLMENT_EXCEPTION` — a distinct, recorded state, not silently marked `FULFILLED` and not silently left open with no record. |

No final refund, cancellation-fee, or modification policy is defined here — see `LOUMIES_ORDER_OPEN_DECISIONS_V0.md`.

---

## 5. Order Flow — Limited Same-Day Ordering

Same-day ordering must **not** behave like unlimited restaurant ordering. It is deliberately controlled, experimental availability (§2) — separately accounted for from scheduled-preorder allocation, but not permanently walled off from it.

### Required capabilities (all **[CONFIGURABLE]** — none of the numbers below are rules)

- Configurable total same-day order capacity (a ceiling on same-day allocation for a given operating date), settable to zero.
- Configurable same-day item quantities (per-item ceilings independent of the overall same-day ceiling).
- A temporary capacity hold at checkout (see §10), converted to confirmed capacity only on successful payment — same race-prevention mechanism as scheduled preorder.
- A sold-out state once same-day allocated-and-unheld quantity reaches zero, surfaced immediately to would-be customers.
- Manual adjustment of same-day capacity by the operator, at any point before it is held or confirmed.
- Manual immediate pause/closure of same-day ordering as a whole, independent of remaining capacity.
- Same-day ordering offered only on operator-selected operating dates (not automatically on every operating day).
- Fulfillment eligibility (pickup/delivery) that can differ by date and by order type.

### Corrected principle: separately accounted, not permanently isolated

Scheduled-preorder allocation and same-day allocation must be **separately identifiable and accounted for**, and protected against silent double-counting. They are **not** required to be permanently isolated pools.

- The operator may deliberately **reallocate** unheld, unconfirmed allocated capacity between the scheduled-preorder pool and the same-day pool at any time.
- Reallocation is a discrete, auditable event (source pool, destination pool, quantity, date/item, timestamp, operator identity).
- Reallocation can never pull capacity out from under an active hold or a confirmed order.
- Total allocation across both pools can never exceed sellable capacity for that date/item (see §10).

**Worked example:** if 10 extra same-day portions were allocated but Rania decides before service to expose only 5, the remaining 5 (assuming none are held or confirmed) are reallocated — not permanently trapped in the same-day pool simply because that is where they started.

### Explicit non-rule

**Five same-day orders per operating day is one possible proof-test scenario, not a permanent capacity rule.** The architecture must support testing 0, 5, 10, or any other exposed quantity without changing the underlying model.

### Same-day is optional for the proof — it is not a launch dependency

Because same-day capacity may legitimately be set to zero, none of the still-open same-day settings (total capacity, which dates offer it, fulfillment eligibility — see open decisions D1–D3) are prerequisites for launching LOUMIES. **LOUMIES can accept Order #1, and operate scheduled-preorder, group/catering, and institutional order types, with same-day ordering fully switched off.** A future ordering system must support turning same-day fully OFF without impairing any other order type — this is a capability requirement, not merely a convenience.

This does not remove same-day from the specification, does not make it owner-locked, and does not set a same-day quantity — same-day remains a real order type (§9 of the state model) with its own capacity mechanics; it is simply not required to be "on" for the proof to launch.

---

## 6. Order Flow — Group / Catering

Group/catering is structurally distinct from ordinary individual checkout — it is a request-and-review workflow, not a cart-and-checkout workflow. **This does not mean every conceptual step must become a mandatory, sequentially forced technical state.**

### Conceptual sequence

```
inquiry/request (may be minimal or already complete)
  → qualification (may be instantaneous when the request is already sufficient)
  → quote sent
  → customer acceptance
  → deposit/payment requirement (if applicable)
  → confirmed booking
  → production commitment
  → fulfillment
  → final payment if applicable
  → closed
```

### Corrected principle: evidentiary integrity, not bureaucratic ceremony

A customer who submits a complete request (date, headcount, package, and an explicit ask for pricing) should not be forced through an artificial separate "quote requested" event before LOUMIES can issue a quote. Inquiry and qualification may be traversed together, or even skipped as separately recorded waypoints, when the initial submission already contains what they would have captured.

What must **remain distinct, recorded events** regardless of how quickly they happen: **inquiry/request intake ≠ quote ≠ acceptance ≠ confirmed booking ≠ production commitment ≠ fulfilled ≠ paid.** Compressing the *steps a human must click through* is fine; collapsing the *evidence that a quote is not yet an acceptance, or an acceptance is not yet a confirmed booking* is not.

### Information the request must be capable of capturing (eventually — not all fields are mandatory at first contact)

Requested date; headcount; organization/customer; fulfillment method; location; requested food/package; budget if supplied; dietary requirements/notes; contact information; special operating notes.

### Structural requirement

The architecture must support **both**:

- **(A)** Custom catering requiring individualized review and quotation, and
- **(B)** Potentially standardized group packages later,

without one precluding the other. A standardized package is simply a quote whose price/contents are pre-populated rather than manually composed; the underlying lifecycle (request → quote → acceptance → confirmation) is the same either way.

### Not defined here (all **[OPEN]** — see register)

Minimum order size; deposit percentage; lead time requirements; cancellation policy; final group pricing; whether LOUMIES offers an optional manual date-hold for a serious prospect ahead of formal confirmation (a plausible future operating tool, not implemented as a rule here — see open decisions).

---

## 7. Order Flow — Institutional

Institutional purchasing must support procurement workflows and must **not** force an institution through ordinary consumer checkout.

### Conceptual sequence

```
qualified institutional inquiry (may already be a specific, complete ask)
  → quote and/or SOW
  → institutional acceptance
  → required procurement authorization / PO / equivalent (per-counterparty — see below)
  → confirmed institutional order
  → production commitment
  → fulfillment
  → invoice/payment workflow
  → paid
  → closed
```

### Hard requirements

The order model **must distinguish**, as independently observable status values, not one linear sequence:

- **Lifecycle**: inquiry → quote sent → customer acceptance → confirmed → production committed → closed
- **Procurement status**, independently: `NOT_APPLICABLE` / `NOT_REQUIRED` / `PENDING` / `AUTHORIZED` / `DECLINED`
- **Payment status**, independently: pending / failed / deposit received / paid in full / invoiced-outstanding
- **Fulfillment status**, independently: not ready / ready / fulfilled / fulfillment exception

Three rules follow directly and are structural, not merely stylistic:

1. **A quote is not booked demand.** An institutional order does not count toward confirmed revenue or production planning while its lifecycle sits at `QUOTE_SENT`.
2. **A verbal or written yes is not procurement authorization** when a counterparty's process requires formal authorization (PO or equivalent). `CUSTOMER_ACCEPTED` and `procurement_status = AUTHORIZED` are distinct, and the order's lifecycle cannot cross into `CONFIRMED` on acceptance alone if that counterparty requires authorization.
3. **Unpaid/unconfirmed institutional demand is never counted as confirmed production.** Only orders whose lifecycle has reached the production-commitment gate (§12, §9) enter the production manifest.

**Corrected framing:** whether a given institutional counterparty requires formal PO authorization, or whether written/verbal acceptance is sufficient for *that* counterparty, is **per-counterparty** and is never generalized as a universal institutional rule. The `procurement_status = NOT_REQUIRED` value exists precisely to represent a counterparty that does not require formal authorization, without ever assuming that by default for institutions generally.

The governing reference case: a department accepts a quote, but the required PO has not arrived — the order's procurement status is `PENDING`, and the order cannot cross into `CONFIRMED` or `PRODUCTION_COMMITTED` until it reaches `AUTHORIZED`.

Also note: fulfillment can complete before payment settles. An institutional order can legitimately be `fulfillment = FULFILLED` while `payment = INVOICED_OUTSTANDING` — this is not a contradiction, and the order does not close until payment is resolved (see §9, gate to `CLOSED`).

**Confirmed before cash is collected — `PAYMENT_NOT_YET_DUE`.** Institutional procurement can also legitimately produce this sequence: acceptance recorded, required PO/authorization received, order is genuinely `CONFIRMED` — but under that counterparty's approved terms, payment is not yet due and no invoice yet exists. This is neither `NOT_REQUIRED` (payment *is* required) nor `INVOICED_OUTSTANDING` (no invoice exists yet). The payment dimension carries a distinct value, `PAYMENT_NOT_YET_DUE`, for exactly this state (see §9). It never substitutes for or bypasses procurement authorization — an order with `procurement = PENDING` cannot reach `CONFIRMED` regardless of its payment terms — and it is never counted as paid revenue or as an outstanding receivable (§14). Whether a given institutional order actually uses this state, versus requiring payment or a deposit at booking, is a per-counterparty term, not an assumption made here (see open decision C6).

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
- Fulfillment method (pickup/delivery) — a field, never a status
- Fulfillment window
- **Lifecycle status** (dimension A — see §9)
- **Payment status** (dimension B — independent of lifecycle, see §9)
- **Procurement/PO status** (dimension C — independent field, institutional-focused, see §9)
- **Fulfillment status** (dimension D — independent of lifecycle and payment, see §9)
- Product/item lines
- Quantities
- Unit/channel price (channel-specific — §11; no price values are set here)
- Discounts (placeholder only — none permitted or defined yet)
- Tax (placeholder only — no tax rules defined here; legal/vendor-dependent)
- Delivery charge (placeholder)
- Service charge (placeholder, if applicable)
- Packaging charge (placeholder, if applicable)
- Capacity-hold reference (if applicable — links to the temporary hold that preceded confirmation; see §10)
- Cancellation state and reason code (see YAML `reason_code_catalog`)
- Exception status
- Operator notes
- Timestamps (created, each status-dimension transition, fulfilled, closed)
- Scheduled-vs-same-day classification (order type already carries this; called out because §5 requires it never be conflated with which pool currently holds the capacity, since pools are reallocatable)

No tax, discount, or fee **rule** is defined by this document — only the fields that must exist so a future rule (legal, vendor, or owner-decided) has somewhere to attach without a schema redesign.

---

## 9. Formal Order State Model

The full formal state model is specified in the companion artifact **`LOUMIES_ORDER_STATE_MODEL_V0.yaml`**, which is the authoritative source for states, transitions, prerequisites, and effects. This section summarizes its structure and guardrails; the YAML governs on any conflict of detail.

### Structural correction: four orthogonal dimensions, not one sequence

The original V0 modeled commercial lifecycle, payment, procurement, and fulfillment as a single, largely linear chain. That made it impossible to cleanly represent legitimate real states like "confirmed and paid, not yet fulfilled" or "fulfilled, invoiced, not yet paid." The corrected model treats every order record as carrying **four independent status dimensions**, plus a separate small entity for checkout capacity reservation:

- **A — Lifecycle status**: `DRAFT` → (for catering/institutional: `INQUIRY` → `QUALIFICATION`, both skippable as distinct recorded steps → `QUOTE_SENT` → `CUSTOMER_ACCEPTED`) → `CONFIRMED` → `PRODUCTION_COMMITTED` → `CLOSED`, with `EXCEPTION`, `CANCELLED`, and `EXPIRED` reachable as needed. Tells you *how far the commercial/production story has progressed.*
- **B — Payment/settlement status**: `NOT_REQUIRED` / `NOT_STARTED` / `PAYMENT_NOT_YET_DUE` / `PAYMENT_PENDING` / `PAYMENT_FAILED` / `DEPOSIT_RECEIVED` / `PAID_IN_FULL` / `INVOICED_OUTSTANDING` / `REFUND_OR_REVERSAL_PENDING` / `REFUNDED_OR_REVERSED`. Tells you *what has actually been paid*, independent of lifecycle. `PAYMENT_NOT_YET_DUE` (institutional/approved pay-on-terms only) represents payment that is required but not yet due under approved terms — booked, but not yet payable or invoiced (see §7).
- **C — Procurement status** (institutional-focused): `NOT_APPLICABLE` / `NOT_REQUIRED` / `PENDING` / `AUTHORIZED` / `DECLINED`. Tells you *whether formal purchasing authorization exists*, independent of customer acceptance.
- **D — Fulfillment status**: `NOT_READY` / `READY` / `FULFILLED` / `FULFILLMENT_EXCEPTION`. Tells you *whether the food actually reached the customer*, independent of payment.

A given order's lifecycle transition into `CONFIRMED`, `PRODUCTION_COMMITTED`, or `CLOSED` is gated by conditions checked **across** these dimensions (the YAML's `gates` section) — this is how the model prevents, for example, an institutional order from being confirmed while procurement is still `PENDING`, without forcing every order type through identical payment timing.

### Purpose

The state model exists specifically to make the following errors structurally impossible, not just discouraged by convention:

- Inquiry-stage or quote-stage demand counted as booked revenue.
- Failed payment treated as confirmed.
- Institutional order counted before required authorization.
- Cancelled or expired order remaining on the production manifest.
- Sold-out inventory continuing to accept orders.
- Two customers both successfully purchasing the same final unit of capacity.
- Capacity decremented twice for one order.
- An order fulfilled without ever passing through a valid confirmed/production-committed lifecycle state.
- A fulfilled order automatically assumed paid when payment may legitimately occur later (institutional invoicing).
- One reported figure silently combining pipeline, booked, paid, fulfilled, and outstanding revenue.

### What the YAML defines

- The full set of valid states for all four dimensions, with which order types each applies to.
- Allowed transitions within each dimension, including trigger and acting party (customer / system / operator).
- The cross-dimension gates that govern lifecycle transitions into `CONFIRMED`, `PRODUCTION_COMMITTED`, and `CLOSED`.
- The temporary capacity-hold entity and its own small state set (`ACTIVE` / `CONVERTED` / `EXPIRED` / `RELEASED`).
- The five-layer capacity model and pool-reallocation rules (§10).
- Guardrail rules stating invalid transitions/combinations outright.
- Six distinct reporting dimensions (pipeline / booked / paid / fulfilled / outstanding / cancelled-reversed) that must never be silently combined.

No webhook, vendor API, or executable trigger is defined — the YAML is a specification artifact, read by humans and future engineers, not a runtime configuration.

---

## 10. Capacity Control Model

### Corrected: the checkout race, and how the hold mechanism closes it

The original model decremented capacity only after successful payment. That leaves an oversell race: two customers could both begin checkout on the last available unit, and both could successfully pay before the system discovers only one unit existed.

**Correction — temporary capacity reservation:** for scheduled preorder and same-day checkout, the future system must support a short-lived hold on a specific unit of capacity, created the moment a customer begins checkout against it:

```
availability check
  → temporary capacity hold created (order not yet confirmed, not yet booked revenue)
  → payment attempt
  → if payment succeeds: hold ATOMICALLY converts into confirmed capacity,
    in the same operation that confirms the order
  → if payment fails, customer abandons, or the hold expires:
    held capacity is released back to its pool
```

A hold is **not** confirmed demand, **not** booked revenue, and **not** a production commitment — it exists purely so that while one hold is active against the last unit, a second checkout attempt sees that unit as unavailable rather than racing to pay first. The hold's expiration duration is **[CONFIGURABLE / vendor-dependent]** — not set here.

This mechanism is **not** extended automatically to group/catering or institutional inquiry/quote stages — an automatic hold at inquiry would misrepresent unqualified interest as reserved capacity. An optional, manually operator-initiated date hold for a serious group/institutional prospect is a plausible future operating tool, tracked as **[OPEN]** rather than implemented as a rule here.

### Five distinct capacity layers

The system must support several **independent** kinds of capacity that can each constrain an order without the others being touched:

1. **Physical capacity** — what production can actually support for a date/item/labor-sensitive category. A ceiling, not calculated here.
2. **Sellable capacity** — how much of physical capacity Rania chooses to expose for sale. **May be, and by default should be assumed to be, less than physical capacity.** The system must never automatically expose all physically possible production.
3. **Allocated capacity** — how sellable capacity is divided among pools (scheduled-preorder, same-day). Sum of all pools must never exceed sellable capacity.
4. **Temporarily held capacity** — units currently reserved by an active checkout hold.
5. **Confirmed capacity** — units consumed by orders whose lifecycle has reached `CONFIRMED` or later. This is what feeds the production manifest.

### A. Date capacity
Can LOUMIES accept another order for this operating date, at all — independent of which item is being ordered. Governed at the operating-date level, using the same five-layer model.

### B. Item capacity
How many additional units of a particular item can still be sold for a given date, independent of whether the date itself still has room for other items.

### C. Labor-sensitive capacity
The system must eventually allow labor-intensive products to be constrained **independently** from highly batch-scalable products. Potential operational categories that may eventually be used to model this (illustrative only — no product is assigned to any of them here):

- batch-scalable
- shaping-intensive
- assembly-intensive
- oven-constrained
- vessel/equipment-constrained
- cooling/holding-constrained

Populating actual limits requires physical production measurement or an explicit owner decision — both **[OPEN]**. What is required before accepting orders is a **defensible safe sellable capacity**, not full knowledge of theoretical physical maximums — sellable capacity can begin conservatively and be revised as proof data accumulates.

### Capacity is checked, held, decremented, and released at defined points

- **Availability check** happens continuously as customers browse (a date/item at zero allocated-and-unheld capacity stops being offered).
- **Hold** happens when a customer begins checkout against a specific unit (scheduled preorder / same-day only).
- **Decrement into confirmed capacity** happens exactly once, atomically with the lifecycle transition into `CONFIRMED` — never at browse/cart time, and never twice for the same order.
- **Release** happens on hold expiry/abandonment, or on cancellation/expiration from any state that had already held or confirmed capacity, restoring it to its pool for resale where operationally appropriate.
- **Reallocation** happens only on allocated-but-unheld-and-unconfirmed capacity, moved deliberately by the operator between the scheduled-preorder and same-day pools, recorded as an auditable event, never exceeding sellable capacity (§5).

### Explicit non-goal

This section does not calculate a single final number for any date, item, or labor category. It specifies *that* the system must be able to hold and enforce such numbers once they exist, and that date capacity, item capacity, and labor-sensitive capacity are three separate levers, layered across five capacity states, not one pooled number.

---

## 11. Channel-Specific Economics

LOUMIES must not be forced into one universal price/economic structure across all channels. The data model must be capable of representing, for a single product, different values across channels for:

- Selling price
- Packaging requirements
- Platform/payment fees
- Service/delivery charges
- Unit economics generally

across scheduled direct preorder, same-day direct ordering, a future third-party marketplace (if adopted), group/catering, and institutional.

This is a **capability requirement only.** No price is set, no marketplace is chosen, and no pricing premium is recommended by this document.

---

## 12. Production Commitment and Production Manifest

### Corrected: `CONFIRMED` and `PRODUCTION_COMMITTED` are related but not identical

A **confirmed order** (lifecycle = `CONFIRMED`) is a legitimate booked commercial commitment — every cross-dimension gate the order type requires has been satisfied. A **production-committed order** (lifecycle = `PRODUCTION_COMMITTED`) means that confirmed order is currently admitted into the **active** production plan for its relevant production cycle. The two are related — production commitment requires confirmation first — but not the same event.

### The production manifest is a live view, not a frozen historical record

The active production manifest for an operating date is the current set of `PRODUCTION_COMMITTED` orders, summarized by product/quantity/schedule/notes. **It regenerates whenever the underlying set legitimately changes** — an order is newly admitted, or a confirmed order legitimately changes or cancels before production begins and is removed. Prior snapshots may be retained for audit purposes, but the manifest the operator or kitchen actually works from is always the current, regenerated active version, never a stale one presented as current.

### Production Manifest — required contents (conceptually)

For a given operating date, after the relevant cutoff, the manifest is capable of containing:

- Operating date
- Confirmed scheduled-order count and revenue
- Confirmed group/catering orders
- Confirmed institutional orders
- Product quantities required, and totals by product
- Same-day inventory deliberately allocated (as distinct from what was actually sold — §14), reflecting any reallocation
- Pickup schedule
- Delivery schedule
- Order-specific production notes
- Dietary/allergen notes where collected
- Packaging quantities, where derivable from the above
- Exceptions requiring operator review
- Payment/procurement exceptions that are currently **preventing** an otherwise-ready order from reaching production commitment

### Critical rule

**Inquiry-stage and quote-stage demand must never inflate production requirements.** Only orders whose lifecycle has reached `PRODUCTION_COMMITTED` are summed into the manifest. An institutional quote sitting at `QUOTE_SENT`, or a catering request still at `QUALIFICATION`, contributes zero units and zero revenue, however likely it seems. Likewise, an order that legitimately cancels before production begins is removed from the manifest as part of that same transition, and its confirmed capacity is released back to its pool for deliberate resale where operationally appropriate.

---

## 13. Operator Control Requirements

This section defines the minimum controls Rania needs to operate the business behind the ordering surface. **This is not a UI design assignment** — no screens, layouts, or interface mockups are specified; these are capability requirements only.

The operator must eventually be able to:

- Open/close an operating date, and assign/reassign which weekday it falls on
- Change the preorder cutoff
- Change fulfillment windows
- Set sellable capacity (which may sit below physical capacity), and reallocate allocated capacity between the scheduled-preorder and same-day pools
- Set item availability / mark items sold out
- Pause all ordering / reopen ordering
- Review pending group/catering and institutional inquiries
- Record quote status and customer acceptance
- Record institutional procurement status (pending / authorized / declined / not required — per counterparty)
- Mark an order's lifecycle confirmed / production-committed
- Mark fulfillment ready / fulfilled, and resolve fulfillment exceptions
- Cancel an order, with a required reason code
- Identify outstanding exceptions (lifecycle `EXCEPTION` and `FULFILLMENT_EXCEPTION`)
- View the active production, pickup, and delivery manifests
- Distinguish pipeline value from confirmed/booked revenue from paid revenue from outstanding receivables in any summary view (§14)

Every item above is a capability the future system must expose in *some* form; how it is exposed (dashboard, spreadsheet export, admin panel) is a vendor/implementation decision, explicitly deferred.

---

## 14. Paid-Proof Data and Metrics

### Corrected: six distinct reportable figures, never silently combined

No single number stands in for all financial truth. The system must be able to report, separately:

- **Pipeline value** — orders at lifecycle `INQUIRY` / `QUALIFICATION` / `QUOTE_SENT` / `CUSTOMER_ACCEPTED` — not yet booked.
- **Confirmed/booked revenue** — orders at lifecycle `CONFIRMED` or later, regardless of payment status.
- **Paid revenue** — orders with `payment = PAID_IN_FULL`, regardless of lifecycle or fulfillment status.
- **Fulfilled revenue** — orders with `fulfillment = FULFILLED`, regardless of payment status.
- **Outstanding receivable** — orders with `payment = INVOICED_OUTSTANDING` (or a deposit received with balance remaining).
- **Cancelled/reversed value** — orders at lifecycle `CANCELLED`, or refunded/reversed payments — tracked separately, never netted silently against the figures above.

**Worked example:** a $2,000 quote not yet accepted is pipeline only. A $700 paid scheduled preorder is confirmed/booked revenue and paid revenue. A $1,200 PO-authorized institutional booking whose payment is not yet due under approved terms (`payment = PAYMENT_NOT_YET_DUE`) is confirmed/booked revenue only — **not** paid revenue, and **not** outstanding receivable until an actual invoice/receivable becomes due (payment status moves to `INVOICED_OUTSTANDING`). No report may combine these into one figure without labeling which dimension it represents. `PAYMENT_NOT_YET_DUE` does not create a seventh mandatory top-line metric — it is a clarification of how the existing six figures treat that payment state: booked, but not yet payable or invoiced.

### Additional event/data capture required

- Scheduled preorder, same-day, group/catering, and institutional: order count and revenue by each of the six reporting dimensions above
- Average order value, by channel
- Same-day inventory allocated vs. sold (sell-through), including the effect of any deliberate reallocation
- Unsold same-day inventory
- Cancellation count (with reason code and initiating party)
- Failed payment count
- Fulfillment failures
- Revenue confirmed **before** the order enters the kitchen (prior to production-commitment) vs. revenue generated **after** preorder cutoff (same-day)
- Channel-specific fees, once known
- Food cost by channel, once costs are known
- Contribution by channel, once the above are known
- Repeat-customer behavior, where legally and technically available

Capturing the *capacity* to compute these later is the requirement. No threshold, target, or GO/NO-GO number is set by this document.

---

## 15. Related Specification Files

- `LOUMIES_ORDER_STATE_MODEL_V0.yaml` — authoritative formal state model (§9): four orthogonal dimensions, cross-dimension gates, capacity-hold entity, five-layer capacity model, reporting dimensions
- `LOUMIES_ORDER_OPEN_DECISIONS_V0.md` — full decision register for every OPEN/WORKING ASSUMPTION item referenced above
- `LOUMIES_FUTURE_VENDOR_REQUIREMENTS_V0.md` — vendor-neutral requirements checklist derived from this specification

---

## 16. Non-Goals / Boundary Reminder

This document does not select a vendor, does not compare or research vendors, does not build or scaffold an application, does not create a frontend/backend/database, does not write payment or integration code, does not deploy anything, does not set final menu items or prices, does not set catering minimums or deposit percentages, does not set permanent same-day capacity or preorder cutoff values, and does not set a GO/NO-GO threshold. Anything that felt tempting to decide along the way and is not an explicit owner decision has been routed to `LOUMIES_ORDER_OPEN_DECISIONS_V0.md` instead of being resolved here.
