# Java Business Logic Candidate Source Cases

## Purpose

This file defines Java-specific candidate search terms for business logic source discovery.

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
- `HandlerFunction`
- `@RequestBody`
- `@RequestParam`
- `@PathVariable`
- `@RequestHeader`
- `@CookieValue`
- `HttpServletRequest`
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

Baseline source extraction candidates:
- DTO fields bound by `@RequestBody`
- `request.getParameter(...)`
- `request.getHeader(...)`
- `request.getCookies()`
- `Authentication`
- `Principal`
- `SecurityContextHolder`
- `@AuthenticationPrincipal`
- JWT claims
- message payload fields
- protobuf request fields
- mapper/service method parameters

Baseline downstream mapping candidates:
- `save`
- `update`
- `delete`
- `process`
- `handle`
- `publishEvent`
- `send`
- `enqueue`
- `dispatch`
- `transition`
- `calculate`
- `recompute`
- `verify`
- `validate`
- `authorize`
- `@Transactional`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- `@PostMapping` on checkout, pay, charge, refund, capture, settlement, or callback paths
- `@RequestMapping` under `/pay`, `/payment`, `/order`, `/checkout`, `/refund`, `/wallet`, `/settlement`
- `@KafkaListener` / `@RabbitListener` consuming payment callbacks or order events
- `@EventListener` processing payment success, order paid, refund finished, or settlement events
- `@Scheduled` reconciliation jobs
- GraphQL mutations for payment, refund, wallet, or order state

## Business source candidates
- `@RequestBody` amount, currency, coupon, order ID, payment method, payer, payee, merchant, or beneficiary fields
- provider callback payload fields such as status, event ID, transaction ID, paid amount, refund amount, and signature
- idempotency headers or request IDs
- authenticated user, tenant, merchant, or wallet context

## Object, state, amount, identity, and beneficiary keywords
- `Order`, `Payment`, `PaymentIntent`, `Transaction`, `Invoice`, `Refund`, `Settlement`, `Wallet`, `Balance`, `Ledger`, `Entitlement`, `Subscription`
- `orderId`, `paymentId`, `transactionId`, `refundId`, `invoiceId`, `merchantId`, `payerId`, `payeeId`, `beneficiaryId`
- `amount`, `payableAmount`, `totalAmount`, `refundAmount`, `currency`, `discount`, `quantity`
- `status`, `PAID`, `PENDING`, `REFUNDED`, `SETTLED`, `CANCELLED`

## Trust-boundary and replay keywords
- `idempotencyKey`, `requestId`, `eventId`, `providerEventId`, `transactionId`
- `verifySignature`, `validateCallback`, `hmac`, `timestamp`, `nonce`
- `processed`, `alreadyPaid`, `alreadyRefunded`, `recomputeAmount`, `calculatePayable`, `lock`, `version`, `unique`

## Graph search recipes
```text
@PostMapping + @RequestBody + amount + createOrder
@KafkaListener + eventId/status/amount + updatePaymentStatus
@RequestHeader + idempotencyKey + refund
callback payload + providerEventId + markPaid
SecurityContextHolder + merchantId/orderId + refund
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- `@PostMapping` on login, OTP, verify, reset, bind, recovery, MFA, or step-up paths
- `@RequestMapping` under `/auth`, `/login`, `/verify`, `/otp`, `/password`, `/reset`, `/bind`, `/mfa`
- GraphQL mutations for password reset, contact binding, token issue, or MFA
- queue/listener paths that send or validate verification messages

## Business source candidates
- `@RequestBody` OTP, token, phone, email, password, target user, device ID, purpose, or channel fields
- session, principal, JWT claim, or challenge record values
- verification event payloads and retry metadata

## Object, state, identity, and permission keywords
- `Otp`, `VerificationCode`, `Challenge`, `ResetToken`, `Mfa`, `Device`, `Session`, `AccountBinding`
- `userId`, `accountId`, `targetUserId`, `phone`, `email`, `deviceId`, `purpose`, `channel`
- `code`, `token`, `verified`, `used`, `expired`, `attemptCount`, `step`

## Trust-boundary and replay keywords
- `expiresAt`, `usedAt`, `attempts`, `nonce`, `challengeId`, `sessionId`
- `verifyCode`, `validateToken`, `markVerified`, `issueToken`, `bindPhone`, `bindEmail`, `resetPassword`
- `rateLimit`, `lock`, `oneTime`, `purpose`, `target`

## Graph search recipes
```text
@PostMapping + code/token + verifyOtp
@RequestBody + phone/email + bindPhone/bindEmail
resetPassword + token + userId/targetUserId
sendOtp + target phone/email + rateLimit
issueToken + challengeId/sessionId + markVerified
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send code, invite, export, search, report, AI/LLM, notification, upload, or batch routes
- queue consumers and scheduled jobs that trigger expensive or repeated work
- GraphQL mutations or RPC methods that fan out downstream work

## Business source candidates
- limiter keys from user ID, tenant ID, IP, device, target, operation, route, or payload
- request fields that define workload size, target count, export type, model, or provider
- headers or context values used as rate dimensions

## Object, quota, and cost keywords
- `RateLimit`, `Quota`, `Bucket`, `Window`, `Usage`, `Credit`, `Limit`, `Counter`
- `userId`, `tenantId`, `ip`, `deviceId`, `target`, `operation`, `payloadHash`
- `batchSize`, `pageCount`, `targetCount`, `model`, `provider`, `cost`, `credits`

## Trust-boundary and replay keywords
- `rateLimiter`, `throttle`, `quotaService`, `consume`, `remaining`, `cooldown`, `dedupe`
- `requestId`, `jobId`, `retryCount`, `window`, `ttl`

## Graph search recipes
```text
@PostMapping + target phone/email + sendOtp + rateLimiter
export/report route + tenantId/userId + quota
@RequestBody + batchSize/ids + enqueue
AI route + model/prompt + consumeCredits
@KafkaListener + retryCount/jobId + dispatch
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approve, reject, publish, cancel, archive, restore, finalize, assign, moderate, disable, or enable routes
- GraphQL mutations and RPC methods for state transitions
- queue events, admin actions, and scheduled jobs that transition workflow objects

## Business source candidates
- `@PathVariable` object IDs and `@RequestBody` action/status/target state fields
- actor, approver, assignee, role, reason, and tenant context values
- current state loaded from database or event payload

## Object, state, and permission keywords
- `Workflow`, `Approval`, `Ticket`, `Order`, `Document`, `Content`, `Lifecycle`, `StateMachine`
- `objectId`, `orderId`, `ticketId`, `documentId`, `approverId`, `assigneeId`, `ownerId`
- `action`, `status`, `state`, `targetState`, `currentState`, `terminal`, `reason`

## Trust-boundary and replay keywords
- `allowedTransition`, `stateMachine`, `canApprove`, `canPublish`, `lock`, `version`, `terminalState`
- `approve`, `reject`, `publish`, `cancel`, `finalize`, `assign`, `archive`

## Graph search recipes
```text
@PostMapping + action/targetState + transition
@PatchMapping + status + updateStatus
@PathVariable id + approve/reject/publish
@EventListener + state + finalize
currentState + targetState + stateMachine
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- coupon claim, redeem, referral, invite, reward, checkout discount, campaign, or loyalty routes
- queue/listener paths issuing rewards or credits
- GraphQL mutations for coupon, reward, referral, or invite actions

## Business source candidates
- coupon code, campaign ID, referral code, referrer, invitee, claimant, beneficiary, reward amount, and discount request fields
- first-use, eligibility, usage count, stacking, and tenant values

## Object, reward, and entitlement keywords
- `Coupon`, `Promotion`, `Campaign`, `Reward`, `Referral`, `Invite`, `Discount`, `Credit`, `Point`, `Entitlement`
- `couponCode`, `campaignId`, `referrerId`, `inviteeId`, `beneficiaryId`, `claimantId`
- `discountAmount`, `rewardValue`, `points`, `credits`, `eligible`, `firstUse`, `usageCount`

## Trust-boundary and replay keywords
- `redeem`, `claim`, `applyDiscount`, `issueReward`, `grantPoints`, `eligibility`, `usageLimit`
- `idempotencyKey`, `referralBinding`, `beneficiaryBinding`, `alreadyClaimed`, `stacking`

## Graph search recipes
```text
@PostMapping + couponCode + redeem/applyDiscount
referral route + referrerId/inviteeId + issueReward
reward listener + eventId + grantPoints
checkout + discountAmount + calculatePayable
campaignId + beneficiaryId + claim
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- export, report, OCR, AI/LLM, translation, notification, search, import, or batch routes
- queue workers and scheduled jobs that trigger downstream providers
- admin replay or operational tools that rerun expensive tasks

## Business source candidates
- file ID, report type, export type, model, prompt, provider, target list, batch IDs, workload size, and quota key fields
- actor, tenant, requester, target, and recipient values

## Object, quota, and downstream cost keywords
- `Export`, `Report`, `Ocr`, `Ai`, `LLM`, `Translation`, `Notification`, `Search`, `BatchJob`, `File`, `Document`
- `fileId`, `reportId`, `batchIds`, `recipientIds`, `model`, `prompt`, `provider`, `pageCount`, `tokenBudget`
- `cost`, `quota`, `credits`, `tenantId`, `userId`

## Trust-boundary and replay keywords
- `enqueue`, `dispatch`, `generate`, `export`, `send`, `fanout`, `consumeCredits`, `rateLimit`, `quota`
- `jobId`, `requestId`, `retryCount`, `dedupe`, `costEstimate`

## Graph search recipes
```text
@PostMapping + fileId/reportType + enqueue
AI route + prompt/model + consumeCredits
export route + batchIds + generateExport
notification route + recipientIds + fanout
@Scheduled/@KafkaListener + jobId + retryCount
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- webhook routes, callback routes, partner sync jobs, provider event consumers, reconciliation jobs, manual replay tools, import jobs
- `@KafkaListener` / `@RabbitListener` consuming provider or partner events
- GraphQL/RPC admin tools that import external state

## Business source candidates
- event ID, provider status, external order ID, external payment ID, integration account, tenant, amount, currency, signature, timestamp, and replay fields
- imported external state mapped to local records

## Object, integration, and state keywords
- `Webhook`, `Callback`, `ProviderEvent`, `Integration`, `Partner`, `Sync`, `Reconciliation`, `ExternalOrder`, `ExternalPayment`
- `eventId`, `externalId`, `providerEventId`, `providerStatus`, `integrationAccountId`, `tenantId`
- `amount`, `currency`, `status`, `state`, `mappingId`

## Trust-boundary and replay keywords
- `verifySignature`, `hmac`, `timestamp`, `nonce`, `processedEvent`, `dedupe`, `replay`, `validateCallback`
- `sync`, `reconcile`, `updateStatus`, `createLocalRecord`, `mapExternalId`

## Graph search recipes
```text
webhook route + eventId/status + updateStatus
callback + signature/timestamp + validateCallback
@KafkaListener + providerEventId + processedEvent
replay tool + externalId + reconcile
integrationAccountId + tenantId + sync
```
