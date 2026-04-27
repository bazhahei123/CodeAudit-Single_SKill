# Payment and Settlement Source Cases

## Purpose

This file defines payment-specific source points for business logic source discovery.

Use it when the target workflow involves:
- order creation
- payment initiation
- payment callback or webhook handling
- capture / refund / reversal
- settlement
- reconciliation
- balance or ledger updates
- entitlement delivery after payment

This reference is guidance, not proof. Always verify the real order state, payment status, callback validation, event identity, and side-effect path in code.

---

# 1. Payment Source Discovery Points

Prioritize these source values and events:
- order ID, payment ID, invoice ID, refund ID, transaction ID
- client-provided amount, total, currency, discount, quantity, or payable value
- server-computed order total, payable amount, and settlement amount
- payment provider callback payload
- event ID, transaction reference, nonce, timestamp, and signature header
- current order/payment state and target payment state
- refund amount and refund reason
- entitlement, shipment, subscription, or balance-credit trigger
- ledger entry values and reconciliation identifiers

Source questions:
- Which source selects the order or payment object?
- Which source determines amount, currency, refund, or settlement value?
- Which source identifies duplicate or replayed callbacks?
- Which source triggers fulfillment, credit, shipment, or entitlement?
- Which source binds payer, order owner, merchant, and beneficiary?

---

# 2. Payment Source Patterns

## P-S1. Payment object identifier source
Example idea:
- `order_id`, `payment_id`, or `transaction_id` comes from request, callback payload, route, or queue message.

Audit relevance:
This source selects the business object whose state, ledger, fulfillment, or settlement may change.

Follow-up:
- verify object ownership, merchant binding, and current-state checks.

## P-S2. Client-provided critical value source
Example idea:
- checkout request includes `amount`, `total`, `discount`, `currency`, or `quantity`.

Audit relevance:
Critical value sources influence payment creation and accounting.

Follow-up:
- verify server-side recomputation from authoritative order lines, promotion state, and currency rules.

## P-S3. Provider callback event source
Example idea:
- webhook payload includes external status, amount, transaction ID, merchant ID, and event timestamp.

Audit relevance:
External event data may trigger settlement, ledger updates, or fulfillment.

Follow-up:
- verify signature validation, freshness, replay protection, and external-account binding before side effects.

## P-S4. Idempotency and replay source
Example idea:
- callback `event_id`, payment reference, idempotency key, or processed-event record.

Audit relevance:
This source determines whether duplicate callbacks or retries repeat side effects.

Follow-up:
- verify unique constraints or dedupe records are written before irreversible effects.

## P-S5. Fulfillment or entitlement source
Example idea:
- payment success triggers shipment, subscription activation, points credit, or balance increment.

Audit relevance:
The source connects a payment state to a one-time business benefit.

Follow-up:
- verify fulfillment is idempotent and tied to trusted payment success.

---

# 3. What to Inspect in Code

Inspect:
- payment creation request handlers
- server-side amount calculation
- callback or webhook handlers
- signature verification order
- transaction ID and event ID uniqueness checks
- order/payment state transition checks
- idempotency and processed-event storage
- refund and settlement value bounds
- entitlement, shipment, subscription, credit, and balance update logic
- transaction boundaries and rollback scope
- retry and queue consumer behavior

---

# 4. Source-Level Danger Signals

High-priority source signals:
- payment request accepts amount or discount directly from client input
- callback payload status reaches state update before signature verification
- provider transaction ID is not used as a dedupe source
- refund amount comes from request without visible bound source
- entitlement trigger appears in both sync and callback paths
- order state source is not loaded before settlement or fulfillment
- merchant or tenant ID is taken from callback payload without local binding

---

# 5. Case Templates

## Case Pay-S-1: Client total source

Source focus:
Identify whether total, discount, currency, and payable amount originate from client input or server-side recomputation.

Follow-up:
Verify authoritative price calculation before payment creation.

## Case Pay-S-2: Callback event source

Source focus:
Identify provider event ID, transaction ID, timestamp, signature, and external payment status.

Follow-up:
Verify authenticity, freshness, replay protection, and current-state reconciliation.

## Case Pay-S-3: Settlement side-effect source

Source focus:
Identify which value or event triggers ledger update, order paid state, shipment, subscription, or balance credit.

Follow-up:
Verify idempotency and one-time side-effect handling.

## Case Pay-S-4: Refund value source

Source focus:
Identify refund amount, refund target, original payment reference, and remaining refundable balance.

Follow-up:
Verify server-side bounds, state checks, and accounting consistency.
