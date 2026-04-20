# Payment and Settlement Cases

## Purpose

This file defines payment-specific business abuse patterns, code review targets, anti-patterns, and audit cases.

Use it when the target workflow involves:
- order creation
- payment initiation
- payment callback or webhook handling
- capture / refund / reversal
- settlement
- reconciliation
- balance or ledger updates
- entitlement delivery after payment

This reference is guidance, not proof. Always verify the real order state, payment status, callback validation, and side effects in code.

---

# 1. Typical Payment Objects and Rules

Common objects:
- order
- payment intent
- transaction
- invoice
- ledger entry
- refund record
- settlement record
- delivery / entitlement state

Typical rules:
- one order should not be paid twice
- one payment callback should not settle twice
- payment amount must be derived server-side
- state changes must follow valid payment transitions
- refund and settlement should not exceed available amount
- benefits should be granted only after trusted payment success

---

# 2. Payment Abuse Patterns

## P1. Duplicate payment callback processing
Risk:
The same callback, webhook, or retryable confirmation message is processed more than once.

Look for:
- missing idempotency key
- no unique transaction reference
- no terminal-state guard
- callback handler that updates order and ledger repeatedly

## P2. Missing signature or trust validation before state update
Risk:
Payment status is updated before callback authenticity is verified.

Look for:
- state update before signature check
- logging / parsing / state write before trust validation
- weak verification branch that still triggers business effect

## P3. Client-trusted amount, discount, or currency
Risk:
Server trusts amount, payable total, coupon result, or currency from the client.

Look for:
- request-provided total
- request-provided discount
- payment request built from client amount
- no server-side recomputation from authoritative order lines

## P4. Invalid state transitions
Risk:
Order moves from the wrong state to paid, refunded, canceled, or settled.

Look for:
- no current-state check
- no terminal-state guard
- refund allowed before payment finalization
- repeated delivery after already-completed payment

## P5. Duplicate entitlement or fulfillment
Risk:
Payment success causes repeated shipping, repeated credit, repeated subscription activation, or repeated balance increment.

Look for:
- callback writes business effect directly without dedupe
- fulfillment job triggered more than once
- separate code paths can both grant the same entitlement

## P6. Partial accounting mismatch
Risk:
Order state changes but ledger, balance, stock, invoice, or settlement updates are inconsistent.

Look for:
- multiple side effects with no transaction boundary
- order marked paid even if downstream credit fails
- ledger updated on retry without state reconciliation

---

# 3. What to Inspect in Code

Inspect:
- callback or webhook handler
- signature verification order
- transaction ID uniqueness checks
- order state transition checks
- idempotency / dedupe storage
- refund and settlement limits
- server-side amount recomputation
- entitlement / shipment / subscription activation logic
- transaction boundaries and rollback scope
- retry and queue consumer behavior

---

# 4. Code-Level Danger Signals

High-risk signals:
- `if (paid) return success;` missing before side effects
- update order state and ledger before verifying callback
- recomputing nothing on server, trusting client total
- repeated callback path lacking a processed-event table
- refund amount accepted from request without bound check
- granting benefits in both sync path and async callback path

---

# 5. Case Templates

## Case Pay-1: Duplicate callback settlement

Expected rule:
One payment notification should only settle the order once.

Audit focus:
Verify whether callback processing is idempotent and tied to a unique payment reference.

## Case Pay-2: Client-trusted total price

Expected rule:
Amount, discount, and payable total should come from authoritative server-side calculation.

Audit focus:
Verify whether the server recomputes payable amount from order items and valid promotion state.

## Case Pay-3: Payment before signature validation

Expected rule:
No business state or ledger change should occur before callback authenticity is validated.

Audit focus:
Verify order of operations in webhook handlers.

## Case Pay-4: Repeated entitlement after payment success

Expected rule:
Benefits should be granted once even under retries, duplicate callbacks, or queue redelivery.

Audit focus:
Verify idempotency around fulfillment logic, not only payment-state update.
