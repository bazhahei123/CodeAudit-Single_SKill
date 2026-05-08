# Android Business Logic Candidate Source Cases

## Purpose

This file defines Android-specific candidate search terms for business logic source discovery.

Use it when Android code can introduce business-rule-relevant inputs, especially:
- exported activities, services, receivers, and providers
- deep links and app links
- WebView JavaScript bridges
- SDK callbacks and push notification handlers
- WorkManager jobs, AlarmManager jobs, and background workers
- Binder, AIDL, Messenger, PendingIntent, and IPC paths
- mobile API clients that pass identity, object, value, state, quota, or beneficiary values to backends

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. Android Entry Candidate Baseline

Use these Android entry candidates across all scenarios:
- `android:exported="true"`
- `<intent-filter>`
- deep links and app links
- `Activity.onCreate`
- `onNewIntent`
- `Service.onStartCommand`
- `BroadcastReceiver.onReceive`
- `ContentProvider.query`, `insert`, `update`, `delete`
- `WorkManager`, `Worker`, `CoroutineWorker`
- `FirebaseMessagingService.onMessageReceived`
- `PendingIntent`
- `ActivityResultLauncher`
- `onActivityResult`
- `@JavascriptInterface`
- `addJavascriptInterface`
- Binder/AIDL methods
- SDK callback interfaces
- Retrofit, OkHttp, Apollo GraphQL, gRPC client calls

Baseline source extraction candidates:
- `getIntent()`, `Intent.getData()`, `Uri.getQueryParameter(...)`
- `getStringExtra`, `getLongExtra`, `getBooleanExtra`, `getParcelableExtra`, `getSerializableExtra`
- `Bundle.getString`, `Bundle.getLong`, `Bundle.getParcelable`
- push payload keys, notification action extras, QR/NFC/share target payloads
- WebView bridge method arguments
- Binder/AIDL method arguments
- Retrofit `@Path`, `@Query`, `@Body`, `@Header`, `@Field`, `@Part`
- local session/token stores, `SharedPreferences`, encrypted preferences

Baseline downstream mapping candidates:
- `enqueue`, `startWork`, `startActivity`, `startService`, `sendBroadcast`
- `execute`, `runAction`, `submit`, `confirm`, `cancel`, `refund`, `redeem`
- API calls through Retrofit/OkHttp/Apollo/gRPC
- local database writes through Room or SQLite

---

# 2. Payment and Settlement Candidates

## Entry candidates
- payment deep links, app links, SDK payment callbacks, wallet screens, refund actions, subscription callbacks
- push notification actions that open payment/refund/order flows
- WebView bridge methods that submit orders or payment choices
- WorkManager jobs syncing payment status

## Business source candidates
- Intent extras or deep-link params for order ID, amount, currency, payment method, transaction ID, payer, merchant, or callback status
- SDK callback payloads for provider event ID, payment status, paid amount, refund amount, and signature-like metadata
- mobile session, tenant, merchant, wallet, or account context passed into backend calls

## Object, state, amount, identity, and beneficiary keywords
- `orderId`, `paymentId`, `transactionId`, `refundId`, `invoiceId`, `merchantId`, `payerId`, `payeeId`, `beneficiaryId`
- `amount`, `payableAmount`, `totalAmount`, `refundAmount`, `currency`, `discount`, `quantity`, `status`
- `Order`, `Payment`, `Wallet`, `Balance`, `Subscription`, `Entitlement`

## Trust-boundary and replay keywords
- `idempotencyKey`, `requestId`, `eventId`, `providerEventId`, `transactionId`
- `signature`, `timestamp`, `nonce`, `processed`, `alreadyPaid`, `alreadyRefunded`
- `calculatePayable`, `verifySignature`, `validateCallback`

## Graph search recipes
```text
deep link + orderId/amount + Retrofit @Body
SDK callback + providerEventId/status + updatePaymentStatus API
@JavascriptInterface + amount/paymentMethod + submitOrder
push action + refundId + refund API
WorkManager + transactionId + sync payment
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- login, OTP, reset, bind, recovery, MFA, device binding, and account linking screens
- SMS retriever callbacks, push verification callbacks, biometric result handlers
- deep links for reset token or magic login
- WebView bridge methods for auth or binding

## Business source candidates
- Intent/deep-link extras for token, code, phone, email, target user, purpose, or channel
- SMS/push payload values, device IDs, account IDs, session tokens, and biometric success state
- bridge method arguments that bind phone/email/account or issue login tokens

## Object, state, identity, and permission keywords
- `otp`, `code`, `token`, `resetToken`, `challengeId`, `mfa`, `deviceId`, `sessionId`
- `userId`, `accountId`, `targetUserId`, `phone`, `email`, `purpose`, `channel`
- `verified`, `used`, `expired`, `attemptCount`, `step`

## Trust-boundary and replay keywords
- `expiresAt`, `usedAt`, `attempts`, `nonce`, `oneTime`, `purpose`, `target`
- `verifyOtp`, `validateToken`, `markVerified`, `issueToken`, `bindPhone`, `bindEmail`, `resetPassword`

## Graph search recipes
```text
deep link + resetToken + reset password API
SMS callback + code + verifyOtp
@JavascriptInterface + phone/email + bind API
Intent extra + targetUserId + account linking
deviceId + issueToken + session
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send code, invite, export, report, upload, sync, notification, AI/LLM, and batch actions
- WorkManager jobs and retryable background workers
- push notification actions and SDK callbacks that trigger repeated work

## Business source candidates
- limiter keys from user ID, tenant ID, device ID, IP-like metadata, target phone/email, operation, route, or payload
- request fields that define workload size, target count, export type, model, provider, or retry count

## Object, quota, and cost keywords
- `rateKey`, `quotaKey`, `userId`, `tenantId`, `deviceId`, `target`, `operation`
- `batchSize`, `pageCount`, `targetCount`, `model`, `provider`, `cost`, `credits`, `retryCount`

## Trust-boundary and replay keywords
- `rateLimit`, `quota`, `throttle`, `cooldown`, `dedupe`, `requestId`, `jobId`, `WorkRequest`, `BackoffPolicy`
- `enqueue`, `sendNotification`, `consumeCredits`, `dispatch`

## Graph search recipes
```text
button/deep link + target phone/email + sendOtp API
WorkManager + retryCount/jobId + expensive API
Intent extra + batchSize/ids + enqueue
AI screen + prompt/model + quota key
notification action + operation + dispatch
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approve, publish, cancel, archive, assign, disable, or restore screens
- deep links and notification actions for workflow decisions
- WebView bridge methods that run workflow actions
- offline queue sync jobs that replay state changes

## Business source candidates
- Intent extras, bridge args, API body fields, or local pending-action records containing object ID, action, target state, reason, approver, or assignee
- local database records representing queued transitions

## Object, state, and permission keywords
- `workflowId`, `orderId`, `ticketId`, `documentId`, `contentId`, `approverId`, `assigneeId`, `ownerId`
- `action`, `status`, `state`, `targetState`, `currentState`, `terminal`, `reason`

## Trust-boundary and replay keywords
- `allowedTransition`, `stateMachine`, `canApprove`, `canPublish`, `version`, `offlineQueue`
- `approve`, `reject`, `publish`, `cancel`, `finalize`, `assign`, `archive`

## Graph search recipes
```text
deep link + action/targetState + workflow API
notification action + orderId + approve/reject
@JavascriptInterface + projectId/action + runAction
Room pending action + targetState + sync
offline queue + eventId + transition
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- coupon claim/redeem screens, referral/invite screens, reward center, checkout discount flows
- deep links carrying campaign/referral/coupon values
- push or SDK callbacks that trigger reward claims
- WebView bridge promotion methods

## Business source candidates
- coupon code, campaign ID, referral code, inviter/referrer, invitee, claimant, beneficiary, reward amount, and discount fields
- local first-use markers and mobile-generated idempotency/request keys

## Object, reward, and entitlement keywords
- `couponCode`, `campaignId`, `referrerId`, `inviteeId`, `beneficiaryId`, `claimantId`
- `discountAmount`, `rewardValue`, `points`, `credits`, `eligible`, `firstUse`, `usageCount`

## Trust-boundary and replay keywords
- `redeem`, `claim`, `applyDiscount`, `issueReward`, `grantPoints`, `eligibility`, `usageLimit`
- `idempotencyKey`, `referralBinding`, `beneficiaryBinding`, `alreadyClaimed`, `stacking`

## Graph search recipes
```text
deep link + couponCode/campaignId + redeem API
invite screen + referrerId/inviteeId + issueReward
push callback + rewardId + claim
checkout + discountAmount + applyDiscount
@JavascriptInterface + beneficiaryId + grant points API
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- export, report, OCR, AI/LLM, translation, notification, upload, import, or batch actions
- WorkManager jobs and offline retry queues
- WebView bridges and SDK callbacks that trigger downstream provider work

## Business source candidates
- file ID, report type, export type, model, prompt, provider, target list, batch IDs, workload size, and quota key fields
- actor, tenant, requester, target, recipient, and device values

## Object, quota, and downstream cost keywords
- `fileId`, `documentId`, `reportId`, `batchIds`, `recipientIds`, `model`, `prompt`, `provider`, `pageCount`, `tokenBudget`
- `cost`, `quota`, `credits`, `tenantId`, `userId`, `deviceId`

## Trust-boundary and replay keywords
- `enqueue`, `dispatch`, `generate`, `export`, `send`, `fanout`, `consumeCredits`, `rateLimit`, `quota`
- `jobId`, `requestId`, `retryCount`, `dedupe`, `costEstimate`

## Graph search recipes
```text
screen/deep link + fileId/reportType + enqueue API
AI feature + prompt/model + consumeCredits
export action + batchIds + generateExport
notification action + recipientIds + fanout
WorkManager + jobId/retryCount + provider call
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- SDK callbacks, push callbacks, payment provider callbacks, partner app callbacks, deep links from external providers
- WorkManager sync jobs, import jobs, and manual replay screens
- WebView bridges receiving external integration data

## Business source candidates
- event ID, provider status, external order ID, external payment ID, integration account, tenant, amount, currency, signature-like metadata, timestamp, and replay fields
- external state mapped into local or backend records

## Object, integration, and state keywords
- `eventId`, `externalId`, `providerEventId`, `providerStatus`, `integrationAccountId`, `tenantId`
- `amount`, `currency`, `status`, `state`, `mappingId`, `externalOrderId`, `externalPaymentId`

## Trust-boundary and replay keywords
- `signature`, `timestamp`, `nonce`, `processedEvent`, `dedupe`, `replay`, `validateCallback`
- `sync`, `reconcile`, `updateStatus`, `createLocalRecord`, `mapExternalId`

## Graph search recipes
```text
SDK callback + eventId/status + update backend
deep link + externalId + sync/reconcile
WorkManager + providerEventId + processedEvent
manual replay + externalOrderId + reconcile
@JavascriptInterface + integrationAccountId + sync
```
