# C++ Business Logic Candidate Sink Cases

## Purpose

This file defines C++-specific candidate search terms for business logic abuse review.

Use it when the target application includes C++ services or native components, especially:
- custom HTTP / REST services
- gRPC, Thrift, Cap'n Proto, or custom RPC
- Qt, Boost.Beast, Crow, Drogon, Pistache, Oat++, or cpprestsdk
- native admin interfaces
- DBus, Unix sockets, named pipes, local TCP admin ports, shared memory, or plugin dispatch
- device, desktop, embedded, or high-performance backend workflows

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. C++ Entry Candidate Baseline

Use these C++ entry candidates across all scenarios:
- `CROW_ROUTE`
- `app.route_dynamic`
- Drogon `METHOD_ADD`
- Drogon `ADD_METHOD_TO`
- Pistache `Routes::Get`
- Pistache `Routes::Post`
- Pistache `Routes::Put`
- Pistache `Routes::Delete`
- Oat++ controllers
- Boost.Beast request handlers
- cpprestsdk `http_listener`
- custom `handleGet`
- custom `handlePost`
- `onRequest`
- gRPC service methods
- Thrift handlers
- Cap'n Proto service methods
- DBus methods
- Unix domain socket command handlers
- named pipe command handlers
- local TCP admin handlers
- plugin dispatch tables
- message queue consumers
- scheduled worker loops
- JNI native methods called from Java/Android

Baseline write/effect candidates:
- `save`
- `update`
- `insert`
- `remove`
- `delete`
- `commit`
- `execute`
- `dispatch`
- `send`
- `notify`
- `enqueue`
- `process`
- `handle`
- `apply`
- `submit`

Baseline control candidates:
- `validate`
- `check`
- `verify`
- `authorize`
- `hasPermission`
- `hasRole`
- `can`
- `idempotency`
- `dedupe`
- `processed`
- `mutex`
- `lock`
- `transaction`
- `unique`
- `rateLimit`
- `quota`
- `signature`
- `hmac`
- `status`
- `state`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- HTTP/RPC handlers for checkout, pay, charge, refund, capture, settlement, callback, or webhook actions
- message consumers for payment events
- reconciliation worker loops
- native admin refund or replay commands
- JNI payment result handlers

## Business-effect sink candidates
- `createOrder`
- `submitOrder`
- `pay`
- `charge`
- `capture`
- `confirmPayment`
- `markPaid`
- `updatePaymentStatus`
- `refund`
- `refundOrder`
- `reverse`
- `voidPayment`
- `settle`
- `reconcile`
- `updateBalance`
- `credit`
- `debit`
- `createLedgerEntry`
- `grantEntitlement`
- `activateSubscription`
- `fulfill`
- `ship`
- `deliver`

## Object, state, amount, permission, and entitlement keywords
- `Order`
- `Payment`
- `Transaction`
- `Invoice`
- `Refund`
- `Settlement`
- `Wallet`
- `Balance`
- `Ledger`
- `Entitlement`
- `Subscription`
- `amount`
- `payableAmount`
- `totalAmount`
- `currency`
- `discount`
- `status`
- `paid`
- `pending`
- `refunded`
- `settled`
- `cancelled`

## Required-control keywords
- `verifySignature`
- `validateCallback`
- `idempotencyKey`
- `transactionId`
- `paymentId`
- `processed`
- `alreadyPaid`
- `alreadyRefunded`
- `recomputeAmount`
- `calculatePayable`
- `lock`
- `mutex`
- `transaction`
- `unique`
- `beneficiary`
- `userId`
- `tenantId`

## Graph search recipes
```text
handlePost + refund
RPC + markPaid
consumer + grantEntitlement
updatePaymentStatus + without idempotencyKey
createLedgerEntry + amount
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- HTTP/RPC handlers for login, OTP, verify, reset, bind, recovery, MFA, or step-up
- local IPC methods that modify account state
- JNI handlers for account binding
- background workers sending or verifying challenges

## Business-effect sink candidates
- `sendCode`
- `sendOtp`
- `verifyCode`
- `verifyOtp`
- `verifyEmail`
- `verifyPhone`
- `resetPassword`
- `changePassword`
- `bindAccount`
- `bindPhone`
- `bindEmail`
- `unbind`
- `issueToken`
- `createSession`
- `markVerified`
- `enableMfa`
- `disableMfa`
- `recoverAccount`

## Object, state, identity, and permission keywords
- `User`
- `Account`
- `Session`
- `VerificationCode`
- `Challenge`
- `Otp`
- `ResetToken`
- `Device`
- `Email`
- `Phone`
- `Contact`
- `Mfa`
- `purpose`
- `target`
- `verified`
- `used`
- `expired`
- `attempt`

## Required-control keywords
- `issued`
- `verify`
- `purpose`
- `target`
- `session`
- `expiresAt`
- `usedAt`
- `consume`
- `attemptLimit`
- `cooldown`
- `lockout`
- `nonce`
- `oneTime`
- `rateLimit`

## Graph search recipes
```text
handlePost + resetPassword
verifyOtp + without usedAt
bindPhone + target
issueToken + verifyCode
sendOtp + rateLimit
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- HTTP/RPC handlers for send, verify, invite, export, report, search, notification, or task trigger
- message consumers for expensive work
- scheduled worker loops
- local/admin replay commands

## Business-effect sink candidates
- `send`
- `sendSms`
- `sendEmail`
- `verify`
- `invite`
- `export`
- `generateReport`
- `createJob`
- `enqueue`
- `dispatch`
- `notify`
- `incrementUsage`
- `consumeQuota`
- `createAttempt`
- `processBatch`

## Object, quota, and cost keywords
- `Quota`
- `Limit`
- `Usage`
- `Attempt`
- `RateLimit`
- `Cooldown`
- `Target`
- `Recipient`
- `Phone`
- `Email`
- `Ip`
- `Device`
- `Tenant`
- `User`
- `Job`
- `Report`
- `Notification`
- `Cost`

## Required-control keywords
- `rateLimit`
- `quota`
- `cooldown`
- `attempt`
- `lockout`
- `dedupe`
- `inFlight`
- `unique`
- `window`
- `daily`
- `hourly`
- `target`
- `tenant`
- `payloadHash`
- `size`

## Graph search recipes
```text
handlePost + sendSms
handlePost + export
enqueue + without quota
sendEmail + target + cooldown
consumer + processBatch + limit
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- HTTP/RPC handlers for approve, reject, submit, publish, archive, cancel, close, reopen, assign, or resolve
- message consumers changing object state
- admin batch transition commands
- plugin command handlers that alter workflow state

## Business-effect sink candidates
- `approve`
- `reject`
- `submit`
- `resubmit`
- `publish`
- `unpublish`
- `archive`
- `restore`
- `cancel`
- `close`
- `reopen`
- `finalize`
- `assign`
- `resolve`
- `transition`
- `updateStatus`
- `setStatus`
- `setState`
- `changeState`

## Object, state, and permission keywords
- `Workflow`
- `Status`
- `State`
- `Approval`
- `Task`
- `Ticket`
- `Document`
- `Request`
- `Draft`
- `Pending`
- `Approved`
- `Rejected`
- `Published`
- `Closed`
- `Archived`
- `Finalized`
- `Approver`
- `Role`
- `Permission`

## Required-control keywords
- `currentState`
- `expectedState`
- `allowedTransition`
- `stateMachine`
- `terminal`
- `role`
- `permission`
- `approver`
- `owner`
- `prerequisite`
- `sequence`
- `lock`
- `idempotency`

## Graph search recipes
```text
handlePost + updateStatus
approve + without currentState
publish + approval
setState + terminal
consumer + transition
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- HTTP/RPC handlers for claim, redeem, coupon, reward, referral, invite, campaign, or points
- checkout discount handlers
- event consumers issuing rewards
- scheduled reward settlement workers

## Business-effect sink candidates
- `claim`
- `redeem`
- `applyCoupon`
- `calculateDiscount`
- `issueCoupon`
- `grantReward`
- `addPoints`
- `creditReward`
- `createReferral`
- `bindInvite`
- `markEligible`
- `consumeBenefit`
- `settleReward`

## Object, reward, and entitlement keywords
- `Coupon`
- `Promotion`
- `Campaign`
- `Reward`
- `Points`
- `Referral`
- `Invite`
- `Inviter`
- `Invitee`
- `Eligibility`
- `FirstOrder`
- `NewUser`
- `Discount`
- `Benefit`
- `Redemption`
- `Claim`

## Required-control keywords
- `eligibility`
- `firstUse`
- `firstOrder`
- `newUser`
- `limit`
- `unique`
- `redeemed`
- `claimed`
- `consumed`
- `stacking`
- `policy`
- `beneficiary`
- `selfReferral`
- `idempotency`
- `lock`

## Graph search recipes
```text
handlePost + redeem
applyCoupon + discount
grantReward + beneficiary
claim + unique
createReferral + selfReferral
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- HTTP/RPC handlers for export, report, OCR, AI, LLM, translation, notification, preview, or batch
- queue producers and consumers for expensive tasks
- scheduled retries for downstream jobs
- native admin replay or batch commands

## Business-effect sink candidates
- `generate`
- `export`
- `process`
- `analyze`
- `translate`
- `transcribe`
- `classify`
- `summarize`
- `enqueue`
- `dispatch`
- `fanout`
- `notify`
- `callModel`
- `callVendor`
- `createTask`
- `startJob`

## Object, quota, and downstream cost keywords
- `Report`
- `Export`
- `File`
- `Document`
- `Image`
- `Ocr`
- `Llm`
- `Model`
- `Token`
- `Cost`
- `Quota`
- `Job`
- `Task`
- `Batch`
- `Vendor`
- `Notification`
- `Recipient`

## Required-control keywords
- `quota`
- `rateLimit`
- `limit`
- `dedupe`
- `hash`
- `inFlight`
- `size`
- `cost`
- `tenant`
- `daily`
- `monthly`
- `budget`
- `retry`
- `idempotency`
- `admission`

## Graph search recipes
```text
handlePost + generateReport
enqueue + callModel
export + without quota
notify + fanout
consumer + callVendor + retry
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- HTTP/RPC handlers under webhook, callback, provider, partner, sync, import, or reconcile
- message consumers for provider events
- native replay commands
- scheduled sync and reconciliation workers

## Business-effect sink candidates
- `handleWebhook`
- `processCallback`
- `processEvent`
- `sync`
- `import`
- `reconcile`
- `updateExternalStatus`
- `applyProviderState`
- `markDelivered`
- `settle`
- `fulfill`
- `grantReward`
- `retry`
- `replay`

## Object, integration, and state keywords
- `Provider`
- `Partner`
- `Integration`
- `ExternalId`
- `EventId`
- `CallbackId`
- `WebhookId`
- `Tenant`
- `Account`
- `Merchant`
- `Order`
- `Payment`
- `Shipment`
- `Status`
- `Payload`
- `Signature`

## Required-control keywords
- `signature`
- `hmac`
- `timestamp`
- `nonce`
- `replay`
- `processed`
- `eventId`
- `idempotency`
- `tenant`
- `merchant`
- `account`
- `state`
- `reconcile`
- `freshness`
- `source`

## Graph search recipes
```text
handlePost + handleWebhook
processEvent + without eventId
replay + applyProviderState
consumer + fulfill
callback + signature
```
