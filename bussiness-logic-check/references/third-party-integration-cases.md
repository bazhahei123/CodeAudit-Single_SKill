# Third-Party Callback and Integration Abuse Cases

## Purpose

This file defines third-party callback, webhook, and integration-specific business abuse patterns, code review targets, anti-patterns, and audit cases.

Use it when the target workflow involves:
- payment webhooks
- shipping or logistics callbacks
- identity verification callbacks
- messaging or email provider events
- SaaS integrations
- internal service callbacks
- partner APIs
- sync / import / reconciliation jobs driven by external data

This reference is guidance, not proof. Always verify the real trust boundary, signature validation, replay protection, state guard, and downstream side effects in code.

---

# 1. Typical Integration Rules

Typical rules:
- external callbacks must be authenticated or signed
- external events must be replay-resistant
- the same event should not be processed more than once
- external state updates must be validated against current internal state
- integration-triggered side effects should be idempotent
- partner data should not overwrite authoritative internal state blindly
- sync/import jobs should reconcile rather than trust raw event order

---

# 2. Integration Abuse Patterns

## T1. Side effects before authenticity validation
Risk:
Business state changes before webhook or callback authenticity is fully validated.

Look for:
- parsing and state update before signature check
- fallback processing path after failed validation
- logging or queueing event as trusted before verification

## T2. Replayable event processing
Risk:
The same signed or unsigned event can be processed repeatedly.

Look for:
- no event ID dedupe
- no replay cache
- no processed-event table
- no timestamp / nonce freshness check

## T3. Blind external state overwrite
Risk:
External callback directly overwrites internal state without reconciling against valid workflow state.

Look for:
- callback status applied without current-state guard
- external event order trusted blindly
- stale or duplicate event accepted as authoritative

## T4. Duplicate side effects from retries
Risk:
Redelivered callback or sync job causes repeated reward, shipment, ledger update, or status transition.

Look for:
- side effects before idempotent state record
- no one-time action table
- no distinction between event receipt and effect execution

## T5. Weak source segmentation
Risk:
One integration's event can affect objects or tenants outside its intended scope.

Look for:
- tenant / account / merchant binding missing
- callback target not bound to expected integration account
- external identifier lookup too broad

## T6. Alternate ingest path bypass
Risk:
Main webhook path validates strongly, but replay tool, import tool, manual sync, or internal callback path skips equivalent checks.

Look for:
- admin replay or importer bypasses signature/replay logic
- internal queue consumer trusts previously accepted event without own dedupe/state guard

---

# 3. What to Inspect in Code

Inspect:
- webhook and callback handlers
- signature verification order
- event ID / nonce / timestamp verification
- processed-event storage
- current-state guard before state update
- integration account / tenant / merchant binding
- side-effect execution after event handling
- import / replay / resync tooling
- retry and redelivery behavior
- sync reconciliation logic

---

# 4. Code-Level Danger Signals

High-risk signals:
- update business state before verifying callback
- signed callback accepted multiple times
- no event-id uniqueness check
- external status blindly mapped to internal terminal state
- replay/import tool skips main verification logic
- integration identifier resolves target object without tenant/account binding

---

# 5. Case Templates

## Case TPI-1: Signed but replayable webhook

Expected rule:
A valid event should still only be processed once.

Audit focus:
Verify replay protection and processed-event dedupe beyond signature validation.

## Case TPI-2: Callback updates state before trust validation

Expected rule:
No state transition or side effect should occur before callback authenticity is validated.

Audit focus:
Verify exact order of operations in handlers.

## Case TPI-3: Stale or duplicate event overwrites valid state

Expected rule:
External events should only apply if they are valid for the current internal state.

Audit focus:
Verify current-state reconciliation and event ordering assumptions.

## Case TPI-4: Integration account / tenant mismatch

Expected rule:
An external event should affect only the correct merchant, tenant, or account scope.

Audit focus:
Verify binding between integration credentials, event identity, and local object scope.

## Case TPI-5: Replay or resync tool bypass

Expected rule:
Replay/import/resync tools should enforce equivalent trust, dedupe, and state-validation rules.

Audit focus:
Compare normal webhook path vs replay, admin, and import tooling.
