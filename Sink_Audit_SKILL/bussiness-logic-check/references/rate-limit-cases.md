# Rate Limit and Anti-Abuse Cases

## Purpose

This file defines rate-limit, quota, replay, and anti-abuse patterns for code review.

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

# 1. Typical Anti-Abuse Rules

Typical rules:
- limit repeated sends by user, IP, device, and target
- limit repeated verification attempts
- add cooldown windows
- cap daily / hourly usage
- dedupe identical expensive tasks
- prevent replay or repeated downstream tool triggers
- apply consistent controls across sync and async paths

---

# 2. Abuse Patterns

## R1. Single-dimension limiting only
Risk:
The system rate-limits only by one easily rotated dimension.

Examples:
- by IP only
- by user only
- by phone only

Look for:
- missing combined dimensions
- attacker can rotate target or account while keeping same effect

## R2. No per-target protection
Risk:
An attacker can harass third parties by repeatedly targeting the same email/phone/account.

Look for:
- no target-based cooldown
- no send-frequency cap per destination
- no abuse tracking by recipient

## R3. No verify-attempt limit
Risk:
OTP or challenge verification can be brute-forced.

Look for:
- no attempt counter
- no lockout or backoff
- no expiry shortening after repeated failure

## R4. No dedupe for expensive downstream jobs
Risk:
Repeated requests can trigger the same expensive OCR/LLM/export/task multiple times.

Look for:
- no idempotency key
- no task dedupe key
- no inflight-job suppression
- same payload creates repeated downstream calls

## R5. Replayable callback or request
Risk:
Same request or callback can be re-sent to trigger repeated work.

Look for:
- no nonce
- no replay cache
- no processed-request store
- no time-window enforcement

---

# 3. What to Inspect in Code

Inspect:
- limiter middleware or service
- limiter key composition
- cooldown logic
- per-user, per-IP, per-device, and per-target counters
- verify-attempt counters
- dedupe tables or caches
- job scheduling and retry behavior
- replay detection
- abuse logging and threshold handling
- alternate endpoints that bypass the main limiter

---

# 4. Code-Level Danger Signals

High-risk signals:
- send limiter keyed only by IP
- verify endpoint has no counter
- resend endpoint bypasses main send limiter
- queue submit path lacks dedupe while UI path has cooldown
- downstream tool trigger callable repeatedly with identical input
- webhook or task consumer lacks replay memory

---

# 5. Case Templates

## Case Rate-1: SMS bombing via missing recipient throttle

Expected rule:
Sending to the same target should be capped regardless of who initiates.

Audit focus:
Verify target-based cooldown and per-destination quota.

## Case Rate-2: OTP brute force due to missing attempt cap

Expected rule:
Verification attempts should be limited and eventually blocked or delayed.

Audit focus:
Verify counters, expiry interaction, and lockout logic.

## Case Rate-3: Repeated expensive tool invocation

Expected rule:
Equivalent expensive work should not be triggered repeatedly without dedupe or quota control.

Audit focus:
Verify task dedupe keys, inflight suppression, and per-user / per-target quotas.

## Case Rate-4: Replayable webhook or job trigger

Expected rule:
Repeated delivery of the same event should not repeat work.

Audit focus:
Verify replay cache, processed-event store, or event-id uniqueness checks.
