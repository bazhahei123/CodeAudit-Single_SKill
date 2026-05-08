# Business Logic Candidate Source Common Cases

## Purpose

This file defines shared candidate source logic for business logic abuse review across languages and frameworks.

Use this file as the base reference before loading a language-specific reference.

This file explains:
- how to model business logic sources for graph-database and taint-tracking workflows,
- how to combine entry candidates, actor/intent candidates, object/value/beneficiary keywords, and downstream mapping keywords,
- how the seven business logic scenarios map to candidate source terms,
- when to record `Confirmed source`, `Suspected source`, `Not enough evidence`, or `Probably irrelevant`.

This reference is guidance, not proof. Do not report a vulnerability only because code matches a candidate keyword. Always verify the real source origin, real trust boundary, real business object, and real downstream business-effect path.

---

# 1. Business Logic Source Model

## 1.1 Why business logic sources are different

Business logic bugs do not usually start at one universal source API.

Instead, candidate sources are business-intent points:
- actor identity
- target object identity
- tenant, merchant, account, or integration scope
- amount, quantity, price, discount, or currency
- requested action or target state
- workflow step markers
- idempotency keys, event IDs, nonces, and replay markers
- quota dimensions and workload size
- beneficiary, recipient, payer, invitee, or referrer
- provider callback status or imported external state

A source point is an audit starting point, not proof of a vulnerability.

## 1.2 Four candidate groups

Use four groups for graph search:

### Entry candidates
Where an actor, request, callback, job, queue message, admin tool, mobile entry, IPC payload, or external event enters the code path.

### Actor and intent source candidates
Values that identify who acts, what operation is requested, which target is affected, which step is being attempted, or which workflow state is requested.

### Object, value, and beneficiary keywords
Domain objects and fields that carry amount, quantity, state, quota, identity, tenant, entitlement, recipient, payer, reward, or workflow meaning.

### Trust-boundary and downstream mapping keywords
Source-origin markers, replay identifiers, provider metadata, session context, persisted records, and nearby business-effect operations that prove why the source matters.

## 1.3 Graph search recipe

Useful graph search seeds often look like:

```text
<entry candidate> + <actor/intent source>
<entry candidate> + <object/value/beneficiary keyword>
<source keyword> + <downstream business-effect verb>
<callback/job entry> + <event id/status/amount/tenant keyword>
<client-controlled value> + <state/action/beneficiary keyword>
<external event field> + <idempotency/replay/control keyword>
```

Candidate examples:

```text
post + amount + createOrder
callback + providerStatus + updatePaymentStatus
mutation + targetState + approve
queue + eventId + grantReward
deepLink + orderId + cancel
admin + replay + externalEventId
```

---

# 2. Generic Candidate Token Groups

## 2.1 Generic entry candidates

Search for:
- route
- controller
- handler
- API
- endpoint
- GraphQL mutation
- GraphQL resolver
- RPC method
- webhook
- callback
- queue consumer
- message listener
- scheduled job
- retry endpoint
- replay tool
- admin action
- import job
- sync job
- batch job
- worker
- event listener
- mobile deep link
- exported component
- WebView bridge
- IPC method
- CLI command

## 2.2 Generic actor and intent source candidates

Search for:
- `user_id`
- `userId`
- `uid`
- `actor_id`
- `actorId`
- `account_id`
- `accountId`
- `tenant_id`
- `tenantId`
- `org_id`
- `orgId`
- `merchant_id`
- `merchantId`
- `role`
- `permission`
- `scope`
- `action`
- `operation`
- `intent`
- `target`
- `target_id`
- `targetId`
- `target_state`
- `targetState`
- `status`
- `state`
- `step`
- `stage`
- `purpose`
- `reason`
- `channel`
- `source`

## 2.3 Generic business object and value source keywords

Search for:
- order
- payment
- transaction
- invoice
- refund
- settlement
- balance
- wallet
- ledger
- account
- user
- tenant
- organization
- workspace
- beneficiary
- recipient
- payer
- payee
- merchant
- provider
- integration
- subscription
- entitlement
- reward
- coupon
- promotion
- referral
- invite
- workflow
- approval
- status
- state
- amount
- price
- total
- currency
- quantity
- discount
- points
- credits
- quota
- limit
- workload
- batch
- file
- report
- export

## 2.4 Generic trust-boundary and downstream mapping keywords

Search for:
- request
- query
- body
- form
- header
- cookie
- session
- claims
- principal
- token
- jwt
- webhook
- signature
- hmac
- timestamp
- nonce
- event_id
- eventId
- message_id
- messageId
- job_id
- jobId
- idempotency_key
- idempotencyKey
- transaction_id
- transactionId
- provider_event_id
- providerEventId
- external_id
- externalId
- callback
- replay
- retry
- processed
- dedupe
- verified
- recompute
- calculate
- validate
- stateMachine
- quota
- rateLimit
- lock
- unique

---

# 3. Seven Scenario Candidate Dimensions

## 3.1 Payment and settlement

Entry source candidates:
- checkout route
- payment creation route
- payment callback
- refund route
- settlement job
- reconciliation job
- provider webhook
- wallet API
- subscription callback

Actor and intent source candidates:
- payer ID
- payee ID
- merchant ID
- buyer ID
- seller ID
- refund requester
- payment method
- requested action
- callback status

Object, value, and beneficiary keywords:
- order ID
- payment ID
- transaction ID
- refund ID
- invoice ID
- amount
- total
- payable amount
- refund amount
- currency
- balance delta
- ledger entry
- entitlement
- subscription
- beneficiary account

Trust-boundary and downstream mapping keywords:
- idempotency key
- provider event ID
- callback signature
- transaction reference
- payment status
- recompute amount
- calculate payable
- mark paid
- refund
- update balance
- grant entitlement
- create ledger entry

## 3.2 Authentication, verification, and account binding

Entry source candidates:
- login route
- OTP send route
- OTP verify route
- password reset route
- email/phone bind route
- MFA route
- account recovery route
- device binding route

Actor and intent source candidates:
- user ID
- account ID
- target user
- phone
- email
- device ID
- session ID
- purpose
- channel
- step

Object, value, and beneficiary keywords:
- OTP
- verification code
- reset token
- challenge ID
- recovery token
- binding token
- MFA code
- new password
- target contact
- verified flag
- account binding

Trust-boundary and downstream mapping keywords:
- expiry
- one time use
- attempt count
- nonce
- token purpose
- token target
- mark verified
- issue token
- create session
- bind phone
- bind email
- reset password

## 3.3 Rate, quota, and anti-abuse

Entry source candidates:
- send code route
- invite route
- export route
- report route
- search route
- batch route
- AI/LLM route
- notification route
- queue trigger

Actor and intent source candidates:
- user ID
- tenant ID
- IP
- device ID
- session ID
- target phone
- target email
- operation
- request type

Object, value, and beneficiary keywords:
- quota key
- rate key
- limit key
- workload size
- batch size
- export type
- report type
- model name
- target count
- cost unit
- downstream provider

Trust-boundary and downstream mapping keywords:
- rate limit
- quota
- throttle
- token bucket
- window
- cooldown
- dedupe
- enqueue
- dispatch
- send notification
- create job
- consume credits

## 3.4 Workflow, approval, and lifecycle

Entry source candidates:
- approval route
- publish route
- cancel route
- archive route
- finalize route
- assignment route
- moderation route
- admin action
- GraphQL mutation
- queue event

Actor and intent source candidates:
- actor ID
- approver ID
- assignee ID
- owner ID
- role
- action
- transition
- target state
- reason

Object, value, and beneficiary keywords:
- workflow ID
- order ID
- ticket ID
- document ID
- content ID
- current state
- target state
- status
- terminal flag
- approval record
- lifecycle stage

Trust-boundary and downstream mapping keywords:
- state machine
- allowed transition
- terminal state
- approve
- reject
- publish
- cancel
- finalize
- assign
- lock
- version

## 3.5 Promotion, coupon, reward, and referral

Entry source candidates:
- coupon claim route
- coupon redeem route
- referral route
- invite route
- reward callback
- campaign job
- checkout discount route

Actor and intent source candidates:
- claimant ID
- inviter ID
- invitee ID
- referrer ID
- recipient ID
- beneficiary ID
- campaign ID
- coupon code
- redemption intent

Object, value, and beneficiary keywords:
- coupon
- promotion
- reward
- referral
- invite
- discount
- points
- credit
- eligibility
- first use
- usage count
- stacking rule
- reward value

Trust-boundary and downstream mapping keywords:
- redeem
- claim
- issue reward
- grant points
- apply discount
- eligibility check
- usage limit
- idempotency key
- referral binding
- beneficiary binding

## 3.6 Resource consumption and downstream tool usage

Entry source candidates:
- export route
- report generation route
- OCR route
- AI/LLM route
- translation route
- notification fan-out route
- batch import route
- queue job
- scheduled sync

Actor and intent source candidates:
- user ID
- tenant ID
- target ID
- requester ID
- operation
- tool type
- provider
- model
- prompt source

Object, value, and beneficiary keywords:
- file ID
- document ID
- report ID
- batch IDs
- workload size
- page count
- recipient list
- export type
- model name
- token budget
- downstream cost
- quota key

Trust-boundary and downstream mapping keywords:
- enqueue
- dispatch
- generate
- export
- send
- fanout
- rate limit
- quota
- dedupe
- retry
- cost estimate
- consume credits

## 3.7 Third-party callback and integration

Entry source candidates:
- webhook route
- callback route
- provider event consumer
- partner sync job
- reconciliation job
- manual replay tool
- import job
- queue consumer

Actor and intent source candidates:
- provider account
- integration account
- tenant ID
- merchant ID
- external user
- external object
- event type
- provider status

Object, value, and beneficiary keywords:
- event ID
- external order ID
- external payment ID
- provider transaction ID
- amount
- currency
- callback status
- imported state
- integration account
- mapping record

Trust-boundary and downstream mapping keywords:
- signature
- hmac
- timestamp
- nonce
- replay
- processed event
- dedupe
- verify callback
- sync
- reconcile
- update status
- create local record

---

# 4. Source Point Guidance

Record a candidate as `Confirmed source` only when:
- the source origin is visible,
- the value reaches business-rule-relevant code,
- the trust boundary can be classified,
- and the downstream business-effect path or decision is visible enough to explain relevance.

Record `Suspected source` when:
- the source origin is visible but downstream use is partially hidden,
- the downstream effect is visible but source derivation is partially hidden,
- middleware, queue infrastructure, provider adapters, or service helpers may rewrite or verify the value.

Record `Not enough evidence` when:
- critical entry, origin, trust, or downstream evidence is missing.

Record `Probably irrelevant` when:
- the value does not influence a business rule, protected workflow, side effect, quota, amount, state, identity binding, or beneficiary outcome.
