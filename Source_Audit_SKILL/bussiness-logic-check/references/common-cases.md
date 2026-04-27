# Business Logic Source Common Cases

## Purpose

This file defines shared business logic source concepts, source classification logic, false-positive controls, and evidence standards that apply across business domains and implementation stacks.

Use this file as the base reference for business logic source review before loading any scenario-specific reference.

This file explains:
- what a business logic source point is,
- how to distinguish source points from business rules and side effects,
- how to classify client, external, server, mixed, and unclear trust boundaries,
- which source categories matter most for business logic abuse review,
- when to record `Confirmed source`, `Suspected source`, `Not enough evidence`, or `Probably irrelevant`.

This reference is guidance, not proof. Do not report a vulnerability only because code contains a source described here. Always verify the real business object, real source origin, and real downstream behavior in the target code.

---

# 1. Core Concepts

## 1.1 Business logic source point
A business logic source point is where a value, event, or signal that can influence a business rule enters or becomes available to the backend code path.

Examples:
- order status
- payment callback event
- OTP code
- target state
- coupon code
- discount amount
- idempotency key
- retry event
- quota key
- beneficiary account

A source point is an audit starting point, not proof of a vulnerability.

## 1.2 Source, rule, and side effect

### Source
The origin of a value or event used in business-rule-sensitive code.

Examples:
- `amount` from request body
- `event_id` from webhook payload
- `status` from database row
- `target_state` from API input
- `coupon_code` from checkout request
- `recipient` from notification request

### Rule
The condition that should govern safe business behavior.

Examples:
- transition must be allowed from current state
- callback must be authentic and processed once
- OTP must be bound to user, target, purpose, and expiry
- quota must cover actor and target
- price must be recomputed server-side

### Side effect
The protected business outcome reached after the source is used.

Examples:
- settle order
- issue reward
- reset password
- bind phone
- enqueue OCR job
- send SMS
- update ledger
- publish content

Source discovery should document enough downstream use to explain why the source matters.

## 1.3 Trust boundary

Classify every source by trust boundary:

### Client-controlled
The value comes from request path, query string, request body, form data, headers, cookies, GraphQL variables, RPC arguments, uploaded files, or import rows.

### External-system-controlled
The value comes from a webhook, callback, queue message, partner event, provider payload, or integration sync. It is only trusted after authenticity, freshness, scope, and replay handling are verified.

### Server-trusted
The value comes from persisted server-side state, verified session context, recomputed business data, trusted workflow state, database-derived values and records, or validated event records.

### Mixed
The value combines client/external input with server-trusted data, or an untrusted value is validated against authoritative state before use.

### Unclear
The origin cannot be determined from visible code.

---

# 2. Shared Business Source Categories

## 2.1 State and transition sources
Values that identify current state, target state, or requested transition.

Examples:
- `status`
- `state`
- `target_state`
- `action`
- terminal flags
- approval status

## 2.2 Sequence and workflow step sources
Values that indicate whether a prerequisite step is complete.

Examples:
- verification-complete flag
- challenge ID
- approval record
- onboarding step
- review status
- session flow marker

## 2.3 Idempotency and replay sources
Values used to identify duplicate requests, retries, callbacks, jobs, or events.

Examples:
- idempotency key
- event ID
- transaction reference
- nonce
- timestamp
- retry count
- job ID
- processed-event record

## 2.4 Rate, quota, and resource trigger sources
Values used to calculate abuse limits or trigger expensive work.

Examples:
- user ID
- IP
- device ID
- target phone/email
- tenant ID
- report type
- model/tool type
- workload size

## 2.5 Value, quantity, and accounting sources
Values that influence money, quantity, inventory, balance, discount, or ledger behavior.

Examples:
- amount
- total
- discount
- currency
- quantity
- refund amount
- balance delta
- reward value

## 2.6 Identity, scope, and beneficiary sources
Values that connect actor, target, tenant, payer, recipient, inviter, invitee, or beneficiary.

Examples:
- actor ID
- target object ID
- payer ID
- recipient ID
- inviter ID
- invitee ID
- tenant ID
- merchant ID
- integration account ID

---

# 3. Shared Source Surfaces

Prioritize these source surfaces first:

- state-changing APIs
- retryable actions
- callbacks and webhooks
- queue consumers and async jobs
- OTP, SMS, and email challenge flows
- payment, refund, settlement, and reconciliation paths
- coupon, referral, reward, and promotion flows
- approval, publish, archive, and finalize actions
- expensive downstream tools or report generation paths
- import, replay, and admin operational tooling
- alternate HTTP methods or alternate endpoints for the same feature
- batch operations and bulk state changes

---

# 4. Shared Source Patterns

These patterns are common source signals across business domains. They are not automatic proof of a vulnerability.

## S1. Client-supplied target state
Example idea:
- request body contains `status`, `state`, `action`, or `target_state`

Audit relevance:
State and transition inputs determine whether later business-rule checks must validate the current state and allowed transition.

## S2. Server-side current state
Example idea:
- current object status is loaded from database before transition

Audit relevance:
This is a stronger source for rule enforcement if the transition check actually uses it before side effects.

## S3. Replay or idempotency identifier
Example idea:
- webhook `event_id`, payment transaction ID, idempotency key, job ID, nonce, or timestamp

Audit relevance:
These values are central to dedupe and replay analysis.

## S4. Client-trusted critical value
Example idea:
- amount, quantity, discount, total, refund value, or reward value comes from request input

Audit relevance:
Critical values should usually be recomputed or bounded server-side before accounting or fulfillment.

## S5. Actor-target-beneficiary tuple
Example idea:
- actor ID, order ID, recipient ID, inviter ID, or reward owner appear in the same workflow

Audit relevance:
Business logic review must verify that the initiator, target, and beneficiary remain consistently bound.

## S6. Quota dimension source
Example idea:
- limiter uses user, IP, device, tenant, recipient, or workload size

Audit relevance:
The selected dimensions determine whether abuse can rotate around the limiter.

## S7. External event payload
Example idea:
- callback payload provides status, amount, external object ID, merchant ID, or event time

Audit relevance:
External sources need authenticity, freshness, scope binding, replay protection, and state reconciliation.

## S8. Alternate trigger source
Example idea:
- normal API, admin replay tool, queue consumer, import job, and webhook can trigger the same business effect

Audit relevance:
Equivalent paths should be compared because source origins and controls often differ.

---

# 5. Shared Source Handling Model

## 5.1 Stronger source handling usually looks like

Stronger source handling typically includes:
- current state loaded from authoritative storage before transition
- target state validated against an explicit transition table or workflow guard
- idempotency keys or event IDs stored before side effects
- replay detection tied to event identity and freshness
- quota keys built from actor, target, tenant, device/IP, and workload dimensions as appropriate
- critical values recomputed server-side from authoritative records
- actor, object, and beneficiary bound through server-side state
- callbacks validated before event data becomes trusted
- equivalent normal, retry, batch, callback, and admin paths using the same source interpretation

## 5.2 What does not automatically make a source safe

The following do not automatically make a source safe:
- user is authenticated
- endpoint is authorized
- frontend controls the sequence
- parameter is validated for type only
- callback is signed but not deduped
- state field exists but is not checked
- value is passed through a helper
- limiter exists on one route only
- transaction wraps only part of the workflow
- path is called internal, admin, retry, or replay

## 5.3 Business source points often combine multiple values

Important business logic review often depends on combinations such as:
- current state + target state
- idempotency key + side effect
- actor + target + beneficiary
- amount + currency + order lines
- coupon + user + order + promotion policy
- callback event ID + external object ID + merchant account
- quota key + target + workload size

Record the combination when the code shows values being used together.

---

# 6. False-Positive Controls

Do not record a source point as high-priority if:
- the value is not connected to a business object, workflow, side effect, quota decision, value calculation, or beneficiary relationship,
- the value is static configuration unrelated to business rules,
- the operation is intentionally repeatable and side-effect-free,
- the untrusted value is clearly overwritten by server-side state before sensitive use,
- the downstream operation is intentionally public or non-sensitive and code evidence supports that.

Use `Suspected source` or `Not enough evidence` if:
- the source origin is visible but downstream business use is hidden,
- the downstream side effect is visible but the value origin is hidden,
- a service, workflow engine, integration layer, or database constraint may enforce the real rule,
- queue or callback behavior is partially visible,
- dedupe, limiter, quota, or transaction behavior is abstracted away.

Do not over-claim based only on:
- feature existence,
- parameter naming,
- endpoint naming,
- a retry path by name alone,
- one state-changing method without seeing side effects,
- a familiar anti-pattern.

---

# 7. Source Classification

## Confirmed source
Use `Confirmed source` when there is clear evidence that:
- the value or event origin is visible,
- it reaches business-rule-relevant code,
- and the trust boundary can be classified.

## Suspected source
Use `Suspected source` when:
- a value or event appears business-rule-relevant,
- the source or downstream use is partially visible,
- but hidden workflow, service, storage, integration, or policy behavior may change the classification.

## Not enough evidence
Use `Not enough evidence` when:
- the value origin cannot be determined,
- the downstream business use cannot be determined,
- or critical callback, queue, workflow, or storage behavior is not visible.

## Probably irrelevant
Use `Probably irrelevant` when:
- the value is visible,
- the downstream code is visible,
- and the value does not influence a business object, workflow, side effect, quota decision, value calculation, or beneficiary relationship.

---

# 8. What Good Evidence Looks Like

Strong business source findings usually include:
- the exact entry point
- the source value or event name
- the source origin
- the trust boundary
- the business object or workflow it influences
- the first downstream business-rule-relevant use
- the scenario-specific follow-up check needed

Good source points usually answer:
1. Where does the value or event enter?
2. Who controls the value or event?
3. What business object, workflow, side effect, quota, value, or beneficiary can it influence?
4. What code proves that connection?
5. What should the next business logic audit verify?

---

# 9. Shared Follow-up Guidance

After source discovery, the business logic audit should verify:
- whether state transitions are explicitly validated,
- whether workflow order is enforced server-side,
- whether repeated requests, callbacks, and jobs are idempotent,
- whether replay, retry, and duplicate processing are prevented,
- whether rate and quota controls cover the relevant dimensions,
- whether critical values are recomputed or bounded server-side,
- whether actor, target, scope, and beneficiary are consistently bound,
- whether normal, retry, callback, batch, and admin paths enforce the same business rules.

Avoid weak conclusions such as:
- source exists, therefore vulnerability exists,
- validation exists, therefore business rule is enforced,
- signed callback exists, therefore replay is safe,
- limiter exists, therefore all dimensions are covered,
- transaction exists, therefore all side effects are consistent.

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Where does current state come from?
- Where does target state or action come from?
- Are workflow step markers client-controlled or server-trusted?
- Which values identify duplicate requests, callbacks, or jobs?
- Which dimensions are used for rate and quota decisions?
- Are amount, total, discount, quantity, and reward values client-controlled or recomputed?
- Are actor, target, payer, recipient, inviter, invitee, and beneficiary consistently sourced?
- Are callbacks, queue messages, imports, and replay tools treated as separate source surfaces?
- What business rule should review each source next?
