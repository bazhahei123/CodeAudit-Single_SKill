# Rate Limit and Anti-Abuse Source Cases

## Purpose

This file defines rate-limit, quota, replay, and anti-abuse source points for business logic source discovery.

Use it when the target workflow involves:
- SMS
- email
- OTP send or verify
- notification delivery
- export generation
- expensive search or report generation
- OCR, LLM, translation, or other downstream tools
- invite or mass-action features

This reference is guidance, not proof. Always verify the real limiter key, dimension, quota store, cooldown, and dedupe logic in code.

---

# 1. Rate and Quota Source Discovery Points

Prioritize these source values and events:
- actor ID, user ID, tenant ID, account ID
- IP, device ID, session ID, API key
- recipient or target phone/email/account
- action name such as send, verify, export, invite, notify, or generate
- workload size, report type, export scope, model/tool type
- time window, cooldown key, counter key, and quota bucket
- attempt count, failure count, lockout state
- request ID, nonce, event ID, replay cache key

Source questions:
- Which dimensions form the limiter key?
- Is the target protected as well as the initiator?
- Which source distinguishes resend from first send, verify from send, or retry from first execution?
- Which source measures workload size or downstream cost?
- Are equivalent paths using the same limiter sources?

---

# 2. Rate and Quota Source Patterns

## R-S1. Initiator limiter source
Example idea:
- limiter key is based on user, account, tenant, IP, device, session, or API key.

Audit relevance:
The initiator source controls how easily an attacker can rotate identity or infrastructure.

Follow-up:
- verify whether additional target, device, tenant, or workload dimensions are needed.

## R-S2. Target limiter source
Example idea:
- SMS/email/notification limit includes destination phone, email, or recipient account.

Audit relevance:
Target sources protect third parties from harassment or unwanted cost.

Follow-up:
- verify per-target cooldown and quota across initiators.

## R-S3. Attempt counter source
Example idea:
- verify path uses challenge ID, user ID, target, or session as attempt-counter key.

Audit relevance:
Attempt sources determine brute-force resistance.

Follow-up:
- verify failure counters, lockout, expiry interaction, and reset behavior.

## R-S4. Expensive work trigger source
Example idea:
- export/report/OCR/LLM request includes payload, target, size, or tool type.

Audit relevance:
These sources determine cost, queue growth, and downstream quota consumption.

Follow-up:
- verify dedupe, quota, and size-aware cost gates.

## R-S5. Replay source
Example idea:
- request ID, nonce, event ID, timestamp, or processed-request record.

Audit relevance:
Replay sources prevent repeated work from identical requests or callbacks.

Follow-up:
- verify replay cache or processed-key persistence covers the side effect.

---

# 3. What to Inspect in Code

Inspect:
- limiter middleware or service
- limiter key composition
- cooldown and time-window logic
- per-user, per-IP, per-device, per-tenant, and per-target counters
- verify-attempt counters
- quota buckets and storage layer
- dedupe tables or caches
- job scheduling and retry behavior
- replay detection
- alternate endpoints that bypass the main limiter

---

# 4. Source-Level Danger Signals

High-priority source signals:
- limiter source is only IP for a user-targeted feature
- send path limiter does not include recipient or target
- verify path has no attempt-counter source
- expensive job trigger lacks payload hash or dedupe key
- cooldown source differs between send and resend paths
- queue submit path lacks the limiter source used by UI path
- replay identifier is accepted but not stored before side effects

---

# 5. Case Templates

## Case Rate-S-1: Recipient throttle source

Source focus:
Identify initiator and recipient dimensions for SMS, email, invite, or notification sends.

Follow-up:
Verify per-target cooldown and quota regardless of initiator rotation.

## Case Rate-S-2: Verify attempt source

Source focus:
Identify the challenge, user, target, session, and failure counter source used for verification attempts.

Follow-up:
Verify lockout, backoff, and expiry behavior.

## Case Rate-S-3: Expensive work dedupe source

Source focus:
Identify payload hash, task key, target, tool type, and in-flight job key.

Follow-up:
Verify equivalent expensive work cannot be repeatedly triggered.

## Case Rate-S-4: Replay request source

Source focus:
Identify nonce, event ID, timestamp, or processed request key.

Follow-up:
Verify replay memory covers the downstream work or side effect.
