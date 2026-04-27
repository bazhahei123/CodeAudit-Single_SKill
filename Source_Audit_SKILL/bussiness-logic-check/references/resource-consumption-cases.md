# Resource Consumption and Downstream Tool Source Cases

## Purpose

This file defines resource-consumption source points for business logic source discovery.

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

This reference is guidance, not proof. Always verify the real trigger source, quota logic, dedupe controls, and downstream invocation boundaries in code.

---

# 1. Resource Source Discovery Points

Prioritize these source values and events:
- trigger route, job, callback, import, or replay tool
- actor ID, tenant ID, account ID, API key
- target object, recipient, document, file, URL, report scope, or search query
- workload size, file size, page count, token estimate, model/tool type
- content hash, payload hash, dedupe key, in-flight job key
- queue job ID, retry count, redelivery event, worker state
- downstream vendor/API request metadata
- quota bucket, cost counter, tenant plan, usage limit

Source questions:
- Which source triggers expensive work?
- Which source determines workload size or cost?
- Which source identifies duplicate or in-flight equivalent work?
- Which source enforces per-user, per-target, per-tenant, or vendor quota?
- Which alternate paths can trigger the same downstream tool?

---

# 2. Resource Source Patterns

## RC-S1. Expensive trigger source
Example idea:
- API request, queue message, import row, callback, or admin replay starts OCR, LLM, export, or processing.

Audit relevance:
The trigger source defines who can cause resource consumption and through which path.

Follow-up:
- verify admission control, authentication, authorization, rate limit, and quota checks.

## RC-S2. Workload size source
Example idea:
- document size, page count, prompt size, export scope, row count, or model/tool type.

Audit relevance:
Workload sources determine downstream cost and capacity impact.

Follow-up:
- verify size-aware quota, plan limits, and expensive-model gates.

## RC-S3. Dedupe and in-flight source
Example idea:
- content hash, request hash, task key, file ID, report key, or in-flight job record.

Audit relevance:
Dedupe sources determine whether repeated identical work is suppressed.

Follow-up:
- verify dedupe is checked before enqueue or downstream invocation.

## RC-S4. Retry and queue source
Example idea:
- job ID, retry count, redelivery metadata, callback retry, or worker state.

Audit relevance:
Retry sources determine whether repeated processing multiplies cost.

Follow-up:
- verify retries reuse idempotency state and do not repeat full downstream side effects unnecessarily.

## RC-S5. Target and beneficiary source
Example idea:
- recipient, notification target, external account, tenant, or third-party destination.

Audit relevance:
Target sources matter for harassment, quota abuse, and cross-tenant cost shifting.

Follow-up:
- verify per-target and per-tenant throttles.

---

# 3. What to Inspect in Code

Inspect:
- job enqueue service
- downstream tool invocation wrapper
- OCR/LLM/export task creation
- content-hash or request-hash dedupe logic
- in-flight task suppression
- per-user, per-target, and per-tenant quota enforcement
- retry and redelivery handling
- queue admission rules
- per-target throttles for outbound actions
- batch/admin/replay tools that can trigger the same downstream work

---

# 4. Source-Level Danger Signals

High-priority source signals:
- same request payload can create repeated identical jobs
- expensive request lacks content hash, task key, or in-flight source
- user-facing route has limiter but worker/replay path does not share the source
- workload size source is ignored before expensive processing
- retry metadata is not used to distinguish retry from first execution
- outbound action limiter is keyed only by initiator, not recipient or target
- tenant or plan source is not used before downstream vendor calls

---

# 5. Case Templates

## Case RC-S-1: Expensive work trigger source

Source focus:
Identify route/job/callback source, actor, target object, workload, and downstream tool type.

Follow-up:
Verify quota, admission control, and cost-aware gating.

## Case RC-S-2: Dedupe source

Source focus:
Identify payload hash, content hash, target key, task key, and in-flight job state.

Follow-up:
Verify equivalent work cannot be repeatedly scheduled or invoked.

## Case RC-S-3: Queue retry source

Source focus:
Identify job ID, retry count, redelivery metadata, and processed-job state.

Follow-up:
Verify retries do not multiply downstream cost or side effects.

## Case RC-S-4: Target abuse source

Source focus:
Identify recipient, destination, tenant, or third-party target.

Follow-up:
Verify per-target quota and harassment controls.

## Case RC-S-5: Alternate path source

Source focus:
Identify UI, API, batch, replay, admin, and async trigger paths for the same work.

Follow-up:
Compare whether all paths use equivalent limiter and dedupe sources.
