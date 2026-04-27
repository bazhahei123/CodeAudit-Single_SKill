# Resource Consumption and Downstream Tool Abuse Cases

## Purpose

This file defines resource-consumption abuse patterns, code review targets, anti-patterns, and audit cases.

Use it when the target workflow involves:
- OCR
- LLM / AI calls
- translation
- transcription
- image or document processing
- report or export generation
- notification fan-out
- search or ranking jobs
- batch jobs
- expensive third-party API calls
- any feature where repeated triggering consumes cost, capacity, or downstream quota

This reference is guidance, not proof. Always verify the real trigger path, quota logic, dedupe controls, and downstream invocation boundaries in code.

---

# 1. Typical Resource-Abuse Rules

Typical rules:
- expensive work should have rate limits and quotas
- equivalent work should be deduplicated
- inflight duplicate tasks should be suppressed
- repeated trigger attempts should not enqueue unbounded work
- retries should not multiply cost without need
- per-user, per-target, and per-tenant limits should be enforced
- downstream vendor/API quotas should be respected proactively

---

# 2. Resource Abuse Patterns

## RC1. No dedupe for identical expensive work
Risk:
The same payload or same target can trigger repeated OCR, LLM, export, or processing work.

Look for:
- no idempotency key
- no content hash dedupe
- no inflight-task check
- same input repeatedly schedules expensive jobs

## RC2. Unlimited async enqueue
Risk:
A cheap frontend/API request creates unbounded queue or background work.

Look for:
- no queue admission control
- no backlog threshold check
- no per-user or per-tenant queue cap
- no suppression of repeated identical jobs

## RC3. Missing multi-dimensional quota
Risk:
There is some rate limiting, but no meaningful business quota over user, tenant, target, or workload size.

Look for:
- per-request checks only
- no daily or monthly cap
- no size-aware quota
- no separate cap for expensive models / tools / report types

## RC4. Retry amplification
Risk:
Network retry, callback retry, or worker retry multiplies cost or repeated downstream calls.

Look for:
- no idempotency on downstream call path
- no replay key
- no processed-job state
- no distinction between retry and first execution

## RC5. Downstream target or beneficiary harassment
Risk:
The attacker can cause repeated costly or disruptive actions against a third-party target.

Look for:
- repeated notifications, emails, or external actions against same destination
- no per-target throttle
- no abuse control for outbound actions

## RC6. Alternate path bypass
Risk:
Main UI path is limited, but API, batch, preview, retry, or admin path can trigger same expensive work without equivalent controls.

Look for:
- shared downstream tool called from multiple entry points
- limiter only attached to one path
- background replay tool with no quota or dedupe

---

# 3. What to Inspect in Code

Inspect:
- job enqueue service
- downstream tool invocation wrapper
- OCR/LLM/export task creation
- content-hash or request-hash dedupe logic
- inflight-task suppression
- per-user / per-tenant quota enforcement
- retry and redelivery handling
- queue admission rules
- per-target throttles for outbound actions
- batch/admin/replay tools that can trigger the same downstream work

---

# 4. Code-Level Danger Signals

High-risk signals:
- same request can create repeated identical jobs
- no dedupe key around expensive downstream call
- user-facing limiter exists but worker/replay path bypasses it
- no size-sensitive cost gating for expensive operations
- retries re-run full downstream action with no replay protection
- outbound actions keyed only by initiator, not by recipient or target

---

# 5. Case Templates

## Case RC-1: Duplicate LLM/OCR invocation

Expected rule:
Equivalent expensive work should not be triggered repeatedly without dedupe or quota checks.

Audit focus:
Verify idempotency key design, content-based dedupe, and inflight suppression.

## Case RC-2: Unbounded queue growth from cheap trigger

Expected rule:
A low-cost request should not enqueue unlimited expensive background work.

Audit focus:
Verify queue admission control, backlog caps, and per-user / per-tenant quotas.

## Case RC-3: Retry amplification

Expected rule:
Retries should not multiply downstream cost or business side effects.

Audit focus:
Verify downstream idempotency, replay detection, and worker retry behavior.

## Case RC-4: Recipient-target abuse

Expected rule:
Repeated costly or disruptive actions against the same third-party target should be throttled.

Audit focus:
Verify per-target cooldowns and target-based quota enforcement.

## Case RC-5: Alternate path limiter bypass

Expected rule:
All equivalent paths to the same expensive operation should enforce the same anti-abuse controls.

Audit focus:
Compare UI, API, batch, replay, admin, and async trigger paths.
