# Third-Party Callback and Integration Source Cases

## Purpose

This file defines third-party callback, webhook, and integration source points for business logic source discovery.

Use it when the target workflow involves:
- payment webhooks
- shipping or logistics callbacks
- identity verification callbacks
- messaging or email provider events
- SaaS integrations
- internal service callbacks
- partner APIs
- sync / import / reconciliation jobs driven by external data

This reference is guidance, not proof. Always verify the real trust boundary, signature validation, replay protection, state guard, scope binding, and downstream side effects in code.

---

# 1. Integration Source Discovery Points

Prioritize these source values and events:
- webhook route, callback handler, import job, replay tool, or sync task
- signature header, API key, secret ID, mTLS identity, or provider credential
- event ID, nonce, timestamp, delivery ID, retry count
- external object ID, transaction ID, shipment ID, verification ID, or ticket ID
- external status, amount, account, tenant, merchant, workspace, or provider user ID
- local object lookup key and integration account binding
- processed-event record, replay cache, or idempotency marker
- downstream side-effect trigger such as fulfillment, ledger update, reward, notification, or local state change

Source questions:
- Which source proves the external event is authentic?
- Which source identifies duplicate or stale events?
- Which external source maps to the local business object?
- Which source binds provider account, merchant, tenant, or workspace to local scope?
- Which alternate ingest paths can apply the same event or state update?

---

# 2. Integration Source Patterns

## T-S1. Authenticity source
Example idea:
- signature header, API key, provider credential, certificate identity, or shared secret ID.

Audit relevance:
Authenticity sources decide whether an external event can become trusted.

Follow-up:
- verify authenticity is validated before state changes or side effects.

## T-S2. Replay and freshness source
Example idea:
- event ID, delivery ID, nonce, timestamp, sequence number, or processed-event record.

Audit relevance:
Replay sources distinguish new events from duplicates, retries, and stale deliveries.

Follow-up:
- verify dedupe and freshness checks before business effects.

## T-S3. External object mapping source
Example idea:
- external transaction ID, shipment ID, account ID, verification ID, or partner object ID.

Audit relevance:
Mapping sources determine which local business object is affected.

Follow-up:
- verify lookup is scoped by integration account, tenant, merchant, or workspace.

## T-S4. External state source
Example idea:
- callback status, event type, amount, fulfillment state, verification result, or provider lifecycle state.

Audit relevance:
External state sources may drive local state transitions.

Follow-up:
- verify current internal state reconciliation and allowed transition checks.

## T-S5. Alternate ingest source
Example idea:
- normal webhook, replay tool, import job, manual sync, queue consumer, or internal callback.

Audit relevance:
Alternate ingest paths may bypass the primary validation, dedupe, or state guard.

Follow-up:
- compare source interpretation and rule enforcement across all ingest paths.

---

# 3. What to Inspect in Code

Inspect:
- webhook and callback handlers
- signature verification order
- event ID / nonce / timestamp verification
- processed-event storage
- current-state guard before state update
- integration account / tenant / merchant binding
- external-to-local object mapping
- side-effect execution after event handling
- import / replay / resync tooling
- retry and redelivery behavior
- sync reconciliation logic

---

# 4. Source-Level Danger Signals

High-priority source signals:
- callback payload status reaches state update before authenticity source is checked
- signed event has no event ID or processed-event source
- external object ID lookup is not scoped by merchant, tenant, or integration account
- external status blindly overwrites local terminal state
- replay/import tool accepts raw event payload without the same source checks
- queue consumer trusts event source but lacks its own dedupe or state guard
- timestamp or nonce source exists but is not used for freshness

---

# 5. Case Templates

## Case TPI-S-1: Webhook trust source

Source focus:
Identify signature/API key source, event payload source, and exact point where event becomes trusted.

Follow-up:
Verify no state transition or side effect occurs before trust validation.

## Case TPI-S-2: Replay source

Source focus:
Identify event ID, delivery ID, nonce, timestamp, and processed-event storage.

Follow-up:
Verify valid events still process only once.

## Case TPI-S-3: External state source

Source focus:
Identify external status, event type, sequence, local current state, and mapping to local object.

Follow-up:
Verify stale or duplicate events cannot overwrite valid internal state.

## Case TPI-S-4: Integration scope source

Source focus:
Identify provider account, merchant, tenant, workspace, external object ID, and local object lookup.

Follow-up:
Verify external event affects only the correct local scope.

## Case TPI-S-5: Replay/import path source

Source focus:
Identify normal webhook path and replay/import/resync path for the same event type.

Follow-up:
Compare trust, dedupe, freshness, scope, and state-validation sources across paths.
