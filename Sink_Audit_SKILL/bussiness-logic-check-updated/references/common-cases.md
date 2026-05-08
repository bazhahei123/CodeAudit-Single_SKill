# Business Logic Candidate Sink Common Cases

## Purpose

This file defines shared candidate sink logic for business logic abuse review across languages and frameworks.

Use this file as the base reference before loading a language-specific reference.

This file explains:
- how to model business logic sinks for graph-database and taint-tracking workflows,
- how to combine entry candidates, business-effect candidates, object/value keywords, and control keywords,
- how the seven business logic scenarios map to candidate search terms,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code matches a candidate keyword. Always verify the real business object, real rule, real state-changing behavior, and real missing control in code.

---

# 1. Business Logic Sink Model

## 1.1 Why business logic sinks are different

Business logic bugs do not usually end at one universal dangerous API.

Instead, candidate sinks are business-effect points:
- state changes
- money movement
- balance or ledger writes
- entitlement grants
- reward issuance
- identity binding
- workflow transitions
- quota consumption
- external callback processing
- queue or retry side effects
- expensive downstream tool invocation

## 1.2 Four candidate groups

Use four groups for graph search:

### Entry candidates
Where an actor, request, callback, job, queue message, admin tool, or external event enters the code path.

### Business-effect candidates
Functions, methods, handlers, or operations that change business outcome.

### Object/value keywords
Domain objects and fields that carry state, amount, quota, identity, beneficiary, entitlement, or workflow meaning.

### Required-control keywords
Expected rules that should appear in or near the path before the effect happens.

## 1.3 Graph search recipe

Useful graph search seeds often look like:

```text
<entry candidate> + <business-effect verb>
<business-effect verb> + <object/value keyword>
<candidate sink> without nearby <required-control keyword>
<callback/job entry> + <state-changing verb>
<write operation> + <amount/status/beneficiary keyword>
```

Candidate examples:

```text
post + refund
callback + settle
queue + grantReward
webhook + updateStatus
mutation + bindAccount
retry + fulfill
admin + replay + processEvent
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

## 2.2 Generic business-effect verbs

Search for:
- create
- update
- save
- insert
- delete
- remove
- submit
- commit
- confirm
- approve
- reject
- cancel
- close
- reopen
- finalize
- publish
- archive
- restore
- enable
- disable
- bind
- unbind
- verify
- reset
- issue
- grant
- redeem
- consume
- allocate
- transfer
- charge
- pay
- refund
- settle
- capture
- withdraw
- deposit
- fulfill
- ship
- deliver
- enqueue
- dispatch
- process
- handle
- replay
- retry
- sync
- import
- export

## 2.3 Generic business object and value keywords

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
- role
- permission
- beneficiary
- recipient
- payer
- owner
- invitee
- inviter
- coupon
- promotion
- reward
- points
- quota
- limit
- usage
- entitlement
- subscription
- license
- status
- state
- workflow
- approval
- callback
- webhook
- event
- job
- idempotency
- nonce
- signature
- amount
- price
- currency
- quantity
- inventory

## 2.4 Generic required-control keywords

Search for:
- validate
- check
- verify
- authorize
- permission
- policy
- can
- allowed
- owner
- tenant
- state
- status
- transition
- idempotent
- idempotency
- dedupe
- duplicate
- processed
- nonce
- replay
- signature
- hmac
- lock
- transaction
- atomic
- unique
- quota
- limit
- rate
- cooldown
- attempt
- recompute
- calculate
- reconcile
- bind
- beneficiary
- expiry
- expired
- used
- consumed

---

# 3. Seven Scenario Candidate Dimensions

## 3.1 Payment and settlement

Entry candidates:
- payment API
- checkout endpoint
- payment callback
- payment webhook
- refund endpoint
- settlement job
- reconciliation job
- order callback
- admin refund tool

Business-effect candidates:
- pay
- charge
- capture
- refund
- reverse
- void
- settle
- reconcile
- updatePaymentStatus
- markPaid
- markRefunded
- createLedgerEntry
- updateBalance
- grantEntitlement
- fulfill
- ship

Object/value keywords:
- order
- payment
- paymentIntent
- transaction
- invoice
- refund
- settlement
- balance
- wallet
- ledger
- amount
- currency
- discount
- payable
- status
- entitlement
- fulfillment
- shipment

Required-control keywords:
- signature
- idempotency
- transactionId
- paymentId
- processed
- state
- status
- amount
- recompute
- currency
- lock
- unique
- ledger
- reconcile
- beneficiary

## 3.2 Authentication, verification, and account binding

Entry candidates:
- login endpoint
- OTP send
- OTP verify
- email verify
- password reset
- account recovery
- bind account
- bind phone
- bind email
- step-up verification

Business-effect candidates:
- sendCode
- verifyCode
- verifyOtp
- resetPassword
- bindAccount
- bindPhone
- bindEmail
- changeEmail
- changePhone
- enableMfa
- disableMfa
- issueToken
- createSession
- markVerified

Object/value keywords:
- user
- account
- session
- challenge
- otp
- code
- token
- email
- phone
- device
- contact
- purpose
- reset
- binding
- verified

Required-control keywords:
- issued
- verified
- purpose
- target
- user
- session
- expiry
- expired
- used
- consumed
- attempt
- limit
- lockout
- cooldown
- nonce
- oneTime

## 3.3 Rate, quota, and anti-abuse

Entry candidates:
- send endpoint
- verify endpoint
- invite endpoint
- export endpoint
- search endpoint
- report endpoint
- notification endpoint
- worker trigger
- retry endpoint

Business-effect candidates:
- send
- verify
- invite
- export
- generate
- notify
- enqueue
- dispatch
- consumeQuota
- incrementUsage
- createJob
- scheduleTask
- processBatch

Object/value keywords:
- quota
- limit
- usage
- attempt
- target
- recipient
- email
- phone
- IP
- device
- tenant
- user
- payload
- cost
- job
- report

Required-control keywords:
- rate
- quota
- limit
- cooldown
- attempt
- lockout
- dedupe
- inFlight
- unique
- window
- daily
- hourly
- target
- tenant
- size

## 3.4 Workflow, approval, and lifecycle

Entry candidates:
- workflow endpoint
- approval endpoint
- publish endpoint
- cancel endpoint
- archive endpoint
- admin action
- batch transition
- queue transition

Business-effect candidates:
- approve
- reject
- submit
- resubmit
- publish
- archive
- cancel
- close
- reopen
- finalize
- assign
- resolve
- transition
- updateStatus
- setState
- changeState

Object/value keywords:
- workflow
- status
- state
- approval
- task
- ticket
- document
- article
- order
- request
- draft
- pending
- approved
- rejected
- closed
- archived
- finalized

Required-control keywords:
- currentState
- expectedState
- transition
- terminal
- allowed
- role
- permission
- approver
- owner
- sequence
- prerequisite
- lock
- idempotency

## 3.5 Promotion, coupon, reward, and referral

Entry candidates:
- claim endpoint
- redeem endpoint
- apply coupon
- referral callback
- invite endpoint
- campaign endpoint
- reward job
- points update

Business-effect candidates:
- claim
- redeem
- applyCoupon
- calculateDiscount
- issueCoupon
- grantReward
- addPoints
- creditReward
- createReferral
- bindInvite
- markEligible
- consumeBenefit

Object/value keywords:
- coupon
- promotion
- campaign
- reward
- points
- referral
- invite
- inviter
- invitee
- eligibility
- firstOrder
- newUser
- discount
- benefit
- redemption
- claim

Required-control keywords:
- eligibility
- firstUse
- firstOrder
- newUser
- limit
- unique
- redeemed
- claimed
- consumed
- stacking
- policy
- beneficiary
- selfReferral
- idempotency
- lock

## 3.6 Resource consumption and downstream tool usage

Entry candidates:
- export endpoint
- report endpoint
- OCR endpoint
- AI endpoint
- LLM endpoint
- translation endpoint
- notification fanout
- preview endpoint
- batch trigger
- worker trigger

Business-effect candidates:
- generate
- export
- process
- analyze
- translate
- transcribe
- classify
- summarize
- enqueue
- dispatch
- fanout
- notify
- callModel
- callVendor
- createTask
- startJob

Object/value keywords:
- report
- export
- file
- document
- image
- OCR
- LLM
- model
- token
- cost
- quota
- job
- task
- batch
- vendor
- notification
- recipient

Required-control keywords:
- quota
- rate
- limit
- dedupe
- hash
- inFlight
- size
- cost
- tenant
- daily
- monthly
- budget
- retry
- idempotency
- admission

## 3.7 Third-party callback and integration

Entry candidates:
- webhook
- callback
- provider event
- partner sync
- import event
- reconciliation job
- replay tool
- manual sync
- queue consumer

Business-effect candidates:
- handleWebhook
- processCallback
- processEvent
- sync
- import
- reconcile
- updateExternalStatus
- applyProviderState
- markDelivered
- settle
- fulfill
- grantReward
- retry
- replay

Object/value keywords:
- provider
- partner
- integration
- externalId
- eventId
- callbackId
- webhookId
- tenant
- account
- merchant
- order
- payment
- shipment
- status
- payload
- signature

Required-control keywords:
- signature
- hmac
- timestamp
- nonce
- replay
- processed
- eventId
- idempotency
- tenant
- merchant
- account
- state
- reconcile
- freshness
- source

---

# 4. Finding Guidance

Report a finding only when candidate evidence and missing-control evidence line up.

Confirmed findings usually include:
- candidate entry point,
- candidate business-effect sink,
- affected business object,
- missing or weak rule,
- proof of attacker influence or replay,
- specific business impact.

Use `Suspected` or `Not enough evidence` if:
- candidate sink exists but control may be hidden in another layer,
- naming suggests business effect but side effect is not visible,
- a check exists but its coverage across alternate paths is unclear,
- the graph search found a candidate but taint path is incomplete.
