# Business Logic Abuse Common Cases

## Purpose

This file defines shared business logic abuse concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across business domains and implementation stacks.

Use this file as the base reference for business logic abuse review before loading any scenario-specific reference.

This file explains:
- what business logic abuse is,
- what counts as a business object, workflow, rule, state, and abuse surface,
- how to reason about sequence, idempotency, rate and quota control, value integrity, and beneficiary consistency,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real business object, real rule, and real state-changing behavior in the target code.

---

# 1. Core Concepts

## 1.1 What business logic abuse is
Business logic abuse occurs when an attacker can use legitimate features in an unintended way because business rules are missing, weak, inconsistent, or incorrectly enforced.

The core question is:

**Can an attacker use allowed functionality with the wrong state, wrong order, wrong frequency, wrong value, wrong beneficiary, or wrong retry behavior to obtain an unintended business outcome?**

## 1.2 Business object, workflow, rule, and abuse surface

### Business object
A domain object whose state, ownership, or value matters.

Examples:
- order
- payment
- invoice
- coupon
- reward
- verification challenge
- account binding
- shipment
- task
- approval request

### Workflow
A sequence of steps the system expects to happen in a controlled order.

Examples:
- register -> verify -> bind
- create order -> pay -> confirm -> fulfill
- draft -> review -> approve -> publish
- request OTP -> verify OTP -> reset password

### Rule
A condition that must remain true for safe operation.

Examples:
- one coupon can only be redeemed once
- one callback must not settle the same order twice
- one phone number can only receive a limited number of OTP messages
- a request must come after successful verification
- only one beneficiary should receive the reward

### Abuse surface
A place where the attacker can influence workflow or business state.

Examples:
- state-changing API
- webhook handler
- retry endpoint
- queue consumer
- admin replay tool
- expensive downstream trigger
- alternate HTTP method
- background job redelivery

---

# 2. Shared Business Abuse Categories

## 2.1 State and transition abuse
The attacker causes a business object to enter an invalid, repeated, skipped, or terminally incorrect state.

Examples:
- paying or settling an already-paid order
- canceling after irreversible fulfillment
- reusing a coupon already marked redeemed
- re-approving or re-finalizing an object

## 2.2 Sequence and workflow abuse
The attacker skips a required step, changes order, or reaches a later phase without completing earlier controls.

Examples:
- resetting a password without valid OTP completion
- publishing before review
- binding an email or phone without completing the challenge
- using an entitlement before payment finalization

## 2.3 Idempotency and replay abuse
The attacker repeats the same request, callback, or async event until the side effect occurs more than once.

Examples:
- duplicate payment callback causes duplicate credit
- repeated button click creates multiple identical orders
- repeated redemption grants multiple rewards
- webhook replay reopens fulfillment logic

## 2.4 Rate, quota, and resource abuse
The attacker uses an allowed feature too often, too broadly, or too expensively.

Examples:
- SMS or email bombing
- repeated OCR/LLM/tool invocation
- repeated export generation
- abuse of verification attempts
- mass notification or invite abuse

## 2.5 Value, quantity, and accounting abuse
The attacker manipulates amounts, counts, balances, discounts, or inventory to obtain unintended value.

Examples:
- client-controlled total price
- repeated discount stacking
- negative quantity or negative amount edge cases
- ledger update mismatch
- double decrement / double credit

## 2.6 Beneficiary and scope abuse
The attacker causes business benefit or effect to land on the wrong person, account, tenant, or object.

Examples:
- self-referral abuse
- inviter / invitee mismatch
- account binding confusion
- payer and beneficiary divergence
- delegated action applied to wrong object

---

# 3. Shared Attack Surfaces

Prioritize these attack surfaces first:

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

---

# 4. Shared Anti-Patterns

These are common danger signals across business domains.

## A1. UI-only sequencing
High-risk pattern:
- frontend shows step order
- backend accepts later-step action directly

Why risky:
The expected order exists in product flow, but not in enforceable backend logic.

## A2. Terminal action without state guard
High-risk pattern:
- settle, finalize, cancel, publish, redeem, or approve action executes without checking current state

Why risky:
Repeated or invalid transitions may become possible.

## A3. Missing idempotency or replay protection
High-risk pattern:
- repeated callback, retry, or job can repeat side effects
- no deduplication or unique processing key

Why risky:
One intended action can create multiple outcomes.

## A4. Client-trusted critical values
High-risk pattern:
- amount, total, discount, quantity, beneficiary, or target is trusted from the request

Why risky:
The attacker may change the value without server-side recomputation or binding.

## A5. Incomplete rate / quota enforcement
High-risk pattern:
- some limiter exists, but only by one dimension
- no cooldown, no per-target limit, or no per-user/device/IP split

Why risky:
The feature may still be abusable at scale or against third parties.

## A6. Weak actor-target-beneficiary binding
High-risk pattern:
- actor performs action on one object, but reward or effect lands elsewhere
- inviter, redeemer, payer, recipient, and owner are not consistently tied together

Why risky:
Abuse may not look like classic authorization failure, but still causes wrong business outcome.

## A7. Async / callback trust assumption
High-risk pattern:
- webhook, queue, or internal event is assumed trustworthy
- state checks or dedupe omitted because the source is "internal"

Why risky:
Replay, redelivery, or attacker-influenced producer paths may still exist.

## A8. Rule exists in one path only
High-risk pattern:
- normal path checks the rule
- admin path, batch path, import path, retry path, or alternate method skips it

Why risky:
Attackers or operators can reach the weakest equivalent path.

---

# 5. Shared Protection Model

## 5.1 What strong protection usually looks like

Strong protections typically include:
- explicit state-machine validation
- server-side workflow ordering
- idempotency keys or unique processing constraints
- replay detection
- deduplicated callback and queue processing
- multi-dimensional rate or quota enforcement
- server-side recomputation of critical values
- strong actor/object/beneficiary binding
- transactional updates or locking for shared-value changes
- consistent business-rule enforcement across all equivalent paths

## 5.2 What does not automatically mean the code is safe

The following do **not** automatically mean the code is safe:
- the user is authenticated
- the endpoint requires authorization
- the frontend forces a sequence
- there is some retry logic
- there is some limiter somewhere
- the feature is internal-only
- a callback is signed but not idempotent
- a state field exists in the database
- a transaction exists around only part of the workflow

## 5.3 Business abuse often spans multiple layers
A rule may be split across:
- controller
- service
- repository
- queue consumer
- callback handler
- scheduled job
- third-party integration adapter

Do not assume safety from seeing only one layer.

---

# 6. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the relevant rule is clearly enforced server-side,
- the apparently missing check is enforced in a shared service or workflow layer,
- the operation is intentionally repeatable and side-effect-safe,
- the value is recomputed server-side and not actually trusted from input,
- the limiter, quota, dedupe, or idempotency control is clearly applied and covers the relevant dimensions.

Use `Suspected` or `Not enough evidence` if:
- the workflow is visible but the true rule-enforcement layer is hidden,
- the callback or job path exists but dedupe logic may exist elsewhere,
- the state field is visible but transition validation is abstracted,
- rate limiting appears incomplete but its real scope or storage layer is unclear,
- the beneficiary relationship is suspicious but final binding logic is not visible.

Do not over-claim based only on:
- the existence of a business feature,
- the existence of a retry path,
- the absence of one controller-level check,
- an internal callback or queue path by name alone,
- one repeated action without proving repeated side effects.

---

# 7. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- a business rule is absent, weak, or inconsistently enforced,
- the attacker can reach the relevant workflow or object,
- and an unintended business outcome is plausibly achievable.

## Suspected
Use `Suspected` when:
- a high-risk workflow pattern exists,
- the expected rule is not visible where it should normally appear,
- but hidden enforcement may still exist elsewhere.

## Not enough evidence
Use `Not enough evidence` when:
- the workflow or state model is only partially visible,
- enforcement may exist in another service or integration layer,
- or critical side-effect / dedupe / quota logic cannot be verified.

## Probably safe
Use `Probably safe` when:
- the path is visible,
- the expected rule is clearly enforced,
- and no weaker equivalent path or replay / alternate path is evident.

---

# 8. What Good Evidence Looks Like

Strong business logic abuse findings usually include:
- the exact entry point
- the business object or workflow
- the expected rule
- the actual rule observed
- the state, retry, quota, value, or beneficiary mismatch
- the reason abuse can produce an unintended business outcome

Good findings usually answer:
1. What can the attacker do repeatedly, out of order, or with altered scope?
2. What business rule should stop that behavior?
3. What code shows the rule is absent, weak, or inconsistent?
4. Why is abuse plausible in practice?

---

# 9. Shared Remediation Guidance

Preferred fixes include:
- enforce state transitions explicitly on the backend
- enforce step ordering server-side
- add idempotency keys, unique constraints, or dedupe records
- validate callbacks and protect against replay
- use multi-dimensional rate and quota controls
- recompute price, discount, and totals server-side
- bind actor, object, and beneficiary consistently
- wrap critical state/value updates in proper transactional logic
- apply the same rule across normal, retry, batch, callback, and admin paths

Avoid weak fixes such as:
- relying on UI step order only
- rate limiting by a single easy-to-rotate dimension only
- trusting client-provided totals or beneficiary fields
- checking only one path and leaving alternate paths open
- assuming internal systems never replay or redeliver events

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is the business object's allowed state transition enforced?
- Is workflow order enforced on the backend?
- Can the same action be repeated safely?
- Is replay or duplicate processing prevented?
- Are expensive or abuse-prone actions rate-limited and quota-limited?
- Are value-critical fields computed server-side?
- Are actor, target, and beneficiary consistently bound?
- Are callback, retry, batch, and async paths as strict as the main path?
