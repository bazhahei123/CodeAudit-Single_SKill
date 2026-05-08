# Java Business Logic Candidate Sink Cases

## Purpose

This file defines Java-specific candidate search terms for business logic abuse review.

Use it when the target application is primarily implemented in Java, especially:
- Spring / Spring Boot
- Spring MVC / WebFlux
- Spring Security
- GraphQL Java / Spring GraphQL
- gRPC Java
- Kafka / Rabbit / JMS listeners
- Quartz / Spring scheduled jobs
- MyBatis, JPA, Hibernate, QueryDSL, or repository/service layers

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. Java Entry Candidate Baseline

Use these Java entry candidates across all scenarios:
- `@RestController`
- `@Controller`
- `@RequestMapping`
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@PatchMapping`
- `@DeleteMapping`
- `RouterFunction`
- `RouterFunctions.route(...)`
- `HandlerFunction`
- `@RequestBody`
- `@RequestParam`
- `@PathVariable`
- `@RequestHeader`
- `@CookieValue`
- `@QueryMapping`
- `@MutationMapping`
- `@SchemaMapping`
- `DataFetcher`
- `@MessageMapping`
- `@KafkaListener`
- `@RabbitListener`
- `@JmsListener`
- `@EventListener`
- `ApplicationListener`
- `@Scheduled`
- `Job`, `JobExecutionContext`
- `CommandLineRunner`, `ApplicationRunner`
- gRPC `BindableService`
- service methods called from controllers, listeners, workers, or admin tools

Baseline write/effect candidates:
- `save`
- `saveAndFlush`
- `update`
- `delete`
- `deleteById`
- `insert`
- `merge`
- `flush`
- `execute`
- `publishEvent`
- `send`
- `convertAndSend`
- `enqueue`
- `dispatch`
- `process`
- `handle`

Baseline control candidates:
- `validate`
- `check`
- `verify`
- `can`
- `authorize`
- `hasPermission`
- `hasRole`
- `idempotent`
- `idempotency`
- `dedupe`
- `processed`
- `lock`
- `transaction`
- `@Transactional`
- `unique`
- `limit`
- `quota`
- `signature`
- `hmac`
- `state`
- `status`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- `@PostMapping` on checkout, pay, charge, refund, capture, settlement, or callback paths
- `@RequestMapping` under `/pay`, `/payment`, `/order`, `/checkout`, `/refund`, `/wallet`, `/settlement`
- `@KafkaListener` / `@RabbitListener` consuming payment callbacks or order events
- `@EventListener` processing payment success, order paid, refund finished, or settlement events
- `@Scheduled` reconciliation jobs
- GraphQL mutations for payment, refund, wallet, or order state

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
- `PaymentIntent`
- `Transaction`
- `Invoice`
- `Refund`
- `Settlement`
- `Wallet`
- `Balance`
- `Ledger`
- `Entitlement`
- `Subscription`
- `Fulfillment`
- `Shipment`
- `amount`
- `payableAmount`
- `totalAmount`
- `currency`
- `discount`
- `status`
- `PAID`
- `PENDING`
- `REFUNDED`
- `SETTLED`
- `CANCELLED`

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
- `@Transactional`
- `lock`
- `version`
- `unique`
- `stateMachine`
- `beneficiary`
- `ownerId`
- `tenantId`

## Graph search recipes
```text
@PostMapping + refund
@KafkaListener + markPaid
@EventListener + grantEntitlement
updatePaymentStatus + without idempotencyKey
createLedgerEntry + amount
refund + alreadyRefunded
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- `@PostMapping` on login, OTP, verify, reset, bind, recovery, MFA, or step-up paths
- `@RequestMapping` under `/auth`, `/login`, `/verify`, `/otp`, `/password`, `/reset`, `/bind`, `/mfa`
- `@KafkaListener` / `@RabbitListener` for verification messages
- GraphQL mutations for password reset, contact binding, or token issue

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
- `sessionId`
- `accountId`
- `userId`

## Required-control keywords
- `issued`
- `verify`
- `purpose`
- `target`
- `bindTarget`
- `session`
- `expire`
- `expiresAt`
- `usedAt`
- `consume`
- `attemptLimit`
- `cooldown`
- `lockout`
- `nonce`
- `oneTime`
- `rateLimit`
- `sameUser`

## Graph search recipes
```text
@PostMapping + resetPassword
verifyOtp + without usedAt
bindPhone + target
issueToken + verifyCode
sendOtp + rateLimit
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- `@PostMapping` on send, verify, invite, export, report, search, notification, or task trigger paths
- `@Scheduled` tasks that process user-triggered work
- `@KafkaListener` / `@RabbitListener` workers consuming user-triggered jobs
- admin endpoints that replay or bulk-trigger work

## Business-effect sink candidates
- `send`
- `sendSms`
- `sendEmail`
- `verify`
- `invite`
- `export`
- `generateReport`
- `createJob`
- `scheduleTask`
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
@PostMapping + sendSms
@PostMapping + export
createJob + without quota
sendEmail + target + cooldown
@KafkaListener + processBatch + limit
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- `@PostMapping` / `@PatchMapping` on approve, reject, submit, publish, archive, cancel, close, reopen, assign, or resolve paths
- GraphQL mutations for lifecycle transitions
- queue consumers and scheduled jobs changing business state
- admin or batch endpoints that modify state

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
- `Article`
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
@PatchMapping + updateStatus
approve + without currentState
publish + approval
setState + terminal
@KafkaListener + transition
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- `@PostMapping` on claim, redeem, coupon, reward, referral, invite, campaign, or points paths
- checkout discount application endpoints
- callback or event listeners issuing rewards
- scheduled reward settlement jobs

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
- `CouponTemplate`
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
@PostMapping + redeem
applyCoupon + discount
grantReward + beneficiary
claim + unique
createReferral + selfReferral
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- `@PostMapping` on export, report, OCR, AI, LLM, translation, notification, preview, or batch paths
- queue producers and consumers for expensive tasks
- scheduled retries for downstream jobs
- admin endpoints triggering replay or batch work

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
@PostMapping + generateReport
enqueue + callModel
export + without quota
notify + fanout
@KafkaListener + callVendor + retry
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- `@PostMapping` under webhook, callback, provider, partner, sync, import, or reconcile paths
- `@KafkaListener` / `@RabbitListener` for provider events
- admin replay endpoints
- scheduled sync and reconciliation jobs

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
@PostMapping + handleWebhook
processEvent + without eventId
replay + applyProviderState
@KafkaListener + fulfill
callback + signature
```
