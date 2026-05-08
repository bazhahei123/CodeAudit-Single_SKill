# C++ Business Logic Candidate Source Cases

## Purpose

This file defines C++-specific candidate search terms for business logic source discovery.

Use it when the target application includes C++ code, especially:
- HTTP servers and REST APIs
- CGI/FastCGI modules
- gRPC / Thrift / custom RPC services
- WebSocket handlers
- native IPC services, daemons, agents, and plugins
- desktop or embedded applications that forward business inputs to backend services
- C++ data-access or service layers that receive state, amount, identity, tenant, quota, or callback event values

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. C++ Entry Candidate Baseline

Use these C++ entry candidates across all scenarios:
- cpp-httplib `Get`, `Post`, `Put`, `Patch`, `Delete`
- Crow `CROW_ROUTE`
- Drogon `HttpController`, `ADD_METHOD_TO`, `METHOD_ADD`, `PATH_ADD`
- oatpp `ENDPOINT`, `ENDPOINT_ASYNC`
- Pistache, Restbed, Wt handlers
- CGI/FastCGI `FCGX_Accept`
- gRPC service methods, Thrift handlers, custom RPC dispatch
- WebSocket message handlers
- DBus, named pipe, Unix socket, local TCP admin API, Qt signal/slot IPC
- plugin callbacks, worker jobs, queue consumers, CLI/admin commands

Baseline source extraction candidates:
- route captures, path variables, query parsers, form/body parsers, headers, cookies
- JSON fields from nlohmann/json, RapidJSON, JsonCpp, Boost.JSON
- protobuf/thrift request fields
- IPC message fields, WebSocket message fields, CLI args, environment variables in CGI
- session/JWT/API-key/mTLS context objects

Baseline downstream mapping candidates:
- `save`, `update`, `delete`, `insert`, `execute`, `commit`
- `process`, `handle`, `dispatch`, `enqueue`, `publish`
- `transition`, `calculate`, `recompute`, `verify`, `validate`, `authorize`
- database transaction, lock, unique, dedupe, idempotency helpers

---

# 2. Payment and Settlement Candidates

## Entry candidates
- checkout, pay, charge, refund, capture, settlement, wallet, or callback HTTP/RPC handlers
- payment provider callbacks, reconciliation jobs, queue consumers, local admin replay tools
- IPC/plugin paths that submit or reconcile payment status

## Business source candidates
- request/message fields for amount, currency, coupon, order ID, payment method, payer, payee, merchant, or beneficiary
- provider callback fields such as status, event ID, transaction ID, paid amount, refund amount, and signature
- idempotency headers/fields, request IDs, session, tenant, merchant, or wallet context

## Object, state, amount, identity, and beneficiary keywords
- `Order`, `Payment`, `Transaction`, `Invoice`, `Refund`, `Settlement`, `Wallet`, `Balance`, `Ledger`, `Entitlement`, `Subscription`
- `orderId`, `paymentId`, `transactionId`, `refundId`, `invoiceId`, `merchantId`, `payerId`, `payeeId`, `beneficiaryId`
- `amount`, `payableAmount`, `totalAmount`, `refundAmount`, `currency`, `discount`, `quantity`, `status`

## Trust-boundary and replay keywords
- `idempotencyKey`, `requestId`, `eventId`, `providerEventId`, `transactionId`
- `verifySignature`, `validateCallback`, `hmac`, `timestamp`, `nonce`
- `processed`, `alreadyPaid`, `alreadyRefunded`, `recomputeAmount`, `calculatePayable`, `lock`, `version`, `unique`

## Graph search recipes
```text
HTTP route + JSON amount + createOrder
callback handler + eventId/status/amount + updatePaymentStatus
header idempotencyKey + refund
queue consumer + providerEventId + markPaid
session context + merchantId/orderId + refund
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- login, OTP, verify, reset, bind, recovery, MFA, or step-up HTTP/RPC handlers
- queue consumers sending or validating verification messages
- native IPC or plugin methods that bind accounts or issue session tokens

## Business source candidates
- request/message fields for OTP, token, phone, email, password, target user, device ID, purpose, or channel
- session, principal, JWT claim, challenge record, or verification event values

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
HTTP/RPC handler + code/token + verifyOtp
JSON phone/email + bindPhone/bindEmail
resetPassword + token + userId/targetUserId
sendOtp + target phone/email + rateLimit
issueToken + challengeId/sessionId + markVerified
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send code, invite, export, search, report, AI/LLM, notification, upload, or batch HTTP/RPC handlers
- queue consumers and scheduled jobs that trigger repeated or expensive work
- native IPC/admin commands that fan out downstream work

## Business source candidates
- limiter keys from user ID, tenant ID, IP, device, target, operation, route, or payload
- request fields that define workload size, target count, export type, model, provider, or retry count

## Object, quota, and cost keywords
- `RateLimit`, `Quota`, `Bucket`, `Window`, `Usage`, `Credit`, `Limit`, `Counter`
- `userId`, `tenantId`, `ip`, `deviceId`, `target`, `operation`, `payloadHash`
- `batchSize`, `pageCount`, `targetCount`, `model`, `provider`, `cost`, `credits`

## Trust-boundary and replay keywords
- `rateLimiter`, `throttle`, `quotaService`, `consume`, `remaining`, `cooldown`, `dedupe`
- `requestId`, `jobId`, `retryCount`, `window`, `ttl`

## Graph search recipes
```text
HTTP route + target phone/email + sendOtp + rateLimiter
export/report handler + tenantId/userId + quota
JSON ids/batchSize + enqueue
AI handler + model/prompt + consumeCredits
queue consumer + retryCount/jobId + dispatch
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approve, reject, publish, cancel, archive, restore, finalize, assign, moderate, disable, or enable HTTP/RPC handlers
- WebSocket/IPC methods and admin commands for state transitions
- queue events and scheduled jobs that transition workflow objects

## Business source candidates
- request/message fields for object IDs, action, status, target state, actor, approver, assignee, reason, tenant, and current state
- local persisted pending-action records or imported transition events

## Object, state, and permission keywords
- `Workflow`, `Approval`, `Ticket`, `Order`, `Document`, `Content`, `Lifecycle`, `StateMachine`
- `objectId`, `orderId`, `ticketId`, `documentId`, `approverId`, `assigneeId`, `ownerId`
- `action`, `status`, `state`, `targetState`, `currentState`, `terminal`, `reason`

## Trust-boundary and replay keywords
- `allowedTransition`, `stateMachine`, `canApprove`, `canPublish`, `lock`, `version`, `terminalState`
- `approve`, `reject`, `publish`, `cancel`, `finalize`, `assign`, `archive`

## Graph search recipes
```text
HTTP/RPC handler + action/targetState + transition
PATCH/update handler + status + updateStatus
path/request id + approve/reject/publish
queue event + state + finalize
currentState + targetState + stateMachine
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- coupon claim, redeem, referral, invite, reward, checkout discount, campaign, or loyalty handlers
- queue consumers issuing rewards or credits
- IPC/plugin methods that submit referral or campaign state

## Business source candidates
- coupon code, campaign ID, referral code, referrer, invitee, claimant, beneficiary, reward amount, and discount fields
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
HTTP route + couponCode + redeem/applyDiscount
referral handler + referrerId/inviteeId + issueReward
reward consumer + eventId + grantPoints
checkout + discountAmount + calculatePayable
campaignId + beneficiaryId + claim
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- export, report, OCR, AI/LLM, translation, notification, search, import, or batch handlers
- queue workers and scheduled jobs that trigger downstream providers
- admin replay, CLI, IPC, or plugin tools that rerun expensive tasks

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
HTTP/RPC handler + fileId/reportType + enqueue
AI handler + prompt/model + consumeCredits
export handler + batchIds + generateExport
notification handler + recipientIds + fanout
queue worker + jobId/retryCount + provider call
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- webhook handlers, callback handlers, partner sync jobs, provider event consumers, reconciliation jobs, manual replay tools, import jobs
- queue consumers handling provider or partner events
- native admin tools that import external state

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
webhook handler + eventId/status + updateStatus
callback + signature/timestamp + validateCallback
queue consumer + providerEventId + processedEvent
replay tool + externalId + reconcile
integrationAccountId + tenantId + sync
```
