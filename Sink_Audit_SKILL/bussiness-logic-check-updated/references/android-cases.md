# Android Business Logic Candidate Sink Cases

## Purpose

This file defines Android-specific candidate search terms for business logic abuse review.

Use it when the target application includes Android code, especially:
- exported activities, services, receivers, and content providers
- deep links and app links
- Binder, AIDL, Messenger, and other IPC
- WorkManager, JobScheduler, foreground services, and background sync
- WebView JavaScript bridges
- mobile payment, account binding, promotion, reward, and entitlement flows

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. Android Entry Candidate Baseline

Use these Android entry candidates across all scenarios:
- `AndroidManifest.xml`
- `<activity>`
- `<service>`
- `<receiver>`
- `<provider>`
- `android:exported`
- `intent-filter`
- deep links
- app links
- `Activity#onCreate`
- `Activity#onNewIntent`
- `getIntent()`
- `Intent.getData()`
- `getStringExtra`
- `getParcelableExtra`
- `Service#onStartCommand`
- `onBind`
- `BroadcastReceiver#onReceive`
- `ContentProvider#query`
- `ContentProvider#insert`
- `ContentProvider#update`
- `ContentProvider#delete`
- `ContentProvider#call`
- `WorkManager`
- `Worker#doWork`
- `JobService`
- `FirebaseMessagingService#onMessageReceived`
- `PendingIntent`
- `Binder`
- `AIDL`
- `Messenger`
- `addJavascriptInterface`
- `@JavascriptInterface`

Baseline write/effect candidates:
- `startActivity`
- `startService`
- `sendBroadcast`
- `enqueue`
- `doWork`
- `call`
- `insert`
- `update`
- `delete`
- `sync`
- `submit`
- `confirm`
- `process`
- `handle`
- `dispatch`
- `send`
- `notify`

Baseline control candidates:
- `checkCallingPermission`
- `checkCallingOrSelfPermission`
- `enforceCallingPermission`
- `Binder.getCallingUid`
- `PackageManager#getPackagesForUid`
- `android:permission`
- `signature`
- `nonce`
- `state`
- `status`
- `idempotency`
- `processed`
- `rateLimit`
- `quota`
- `accountId`
- `userId`
- `tenantId`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- payment deep links
- checkout activities
- payment result activities
- push notification payment callbacks
- `FirebaseMessagingService#onMessageReceived`
- exported services handling payment result intents
- WorkManager payment sync workers
- WebView bridge methods for payment confirmation

## Business-effect sink candidates
- `pay`
- `confirmPayment`
- `onPaymentResult`
- `handlePaymentCallback`
- `markPaid`
- `refund`
- `settle`
- `syncPaymentStatus`
- `updateBalance`
- `grantEntitlement`
- `activateSubscription`
- `fulfill`
- `deliver`

## Object, state, amount, permission, and entitlement keywords
- `orderId`
- `paymentId`
- `transactionId`
- `invoiceId`
- `refundId`
- `amount`
- `currency`
- `status`
- `paid`
- `refunded`
- `wallet`
- `balance`
- `subscription`
- `entitlement`
- `sku`
- `purchaseToken`

## Required-control keywords
- `verifySignature`
- `purchaseToken`
- `acknowledgePurchase`
- `idempotency`
- `processed`
- `state`
- `amount`
- `serverVerify`
- `accountId`
- `userId`
- `tenantId`
- `nonce`

## Graph search recipes
```text
deep link + confirmPayment
onMessageReceived + markPaid
WebView bridge + grantEntitlement
purchaseToken + activateSubscription
refund + processed
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- login activities
- OTP activities
- account recovery deep links
- password reset app links
- phone/email binding screens
- push verification callbacks
- exported receivers for verification actions
- WebView bridge methods for account actions

## Business-effect sink candidates
- `sendOtp`
- `verifyOtp`
- `verifyCode`
- `resetPassword`
- `bindAccount`
- `bindPhone`
- `bindEmail`
- `changeEmail`
- `changePhone`
- `issueToken`
- `createSession`
- `markVerified`
- `enableMfa`
- `disableMfa`

## Object, state, identity, and permission keywords
- `accountId`
- `userId`
- `sessionId`
- `otp`
- `code`
- `token`
- `resetToken`
- `email`
- `phone`
- `deviceId`
- `purpose`
- `target`
- `verified`
- `used`
- `expired`

## Required-control keywords
- `serverVerify`
- `purpose`
- `target`
- `expiresAt`
- `usedAt`
- `attempt`
- `cooldown`
- `nonce`
- `oneTime`
- `sameUser`
- `signature`

## Graph search recipes
```text
deep link + resetPassword
verifyOtp + usedAt
bindPhone + target
WebView bridge + issueToken
sendOtp + cooldown
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send OTP button handlers
- invite/share actions
- notification triggers
- export/report actions
- WorkManager jobs
- retry workers
- Firebase message handlers
- background sync services

## Business-effect sink candidates
- `sendOtp`
- `sendSms`
- `sendEmail`
- `invite`
- `share`
- `export`
- `generateReport`
- `enqueue`
- `doWork`
- `notify`
- `incrementUsage`
- `consumeQuota`
- `schedule`

## Object, quota, and cost keywords
- `quota`
- `limit`
- `usage`
- `attempt`
- `recipient`
- `phone`
- `email`
- `deviceId`
- `accountId`
- `userId`
- `job`
- `report`
- `notification`
- `cost`

## Required-control keywords
- `rateLimit`
- `cooldown`
- `attempt`
- `dedupe`
- `unique`
- `inFlight`
- `daily`
- `hourly`
- `target`
- `payloadHash`
- `quota`

## Graph search recipes
```text
button + sendOtp
WorkManager + export
enqueue + without quota
notify + recipient + cooldown
doWork + processBatch
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approval deep links
- moderation activities
- task action screens
- exported components for workflow actions
- push notification action intents
- WorkManager state sync jobs

## Business-effect sink candidates
- `approve`
- `reject`
- `submit`
- `publish`
- `archive`
- `cancel`
- `close`
- `reopen`
- `finalize`
- `assign`
- `resolve`
- `updateStatus`
- `setStatus`
- `setState`
- `syncState`

## Object, state, and permission keywords
- `taskId`
- `ticketId`
- `approvalId`
- `documentId`
- `status`
- `state`
- `draft`
- `pending`
- `approved`
- `rejected`
- `closed`
- `archived`
- `finalized`
- `role`
- `permission`

## Required-control keywords
- `currentState`
- `expectedState`
- `allowed`
- `terminal`
- `approver`
- `owner`
- `permission`
- `serverVerify`
- `idempotency`

## Graph search recipes
```text
deep link + approve
notification action + updateStatus
setState + terminal
doWork + syncState
approve + permission
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- coupon claim activities
- checkout coupon screens
- invite/referral deep links
- push campaigns
- reward notification handlers
- background reward sync jobs

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
- `consumeBenefit`

## Object, reward, and entitlement keywords
- `couponId`
- `promotionId`
- `campaignId`
- `rewardId`
- `points`
- `referralCode`
- `inviteCode`
- `inviter`
- `invitee`
- `eligibility`
- `firstOrder`
- `newUser`
- `discount`
- `benefit`

## Required-control keywords
- `eligibility`
- `unique`
- `redeemed`
- `claimed`
- `consumed`
- `stacking`
- `beneficiary`
- `selfReferral`
- `idempotency`
- `serverVerify`

## Graph search recipes
```text
deep link + bindInvite
applyCoupon + discount
grantReward + beneficiary
claim + unique
redeem + consumed
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- export actions
- report actions
- OCR / AI feature buttons
- preview screens
- notification fanout triggers
- WorkManager workers
- JobService tasks

## Business-effect sink candidates
- `generate`
- `export`
- `process`
- `analyze`
- `translate`
- `transcribe`
- `summarize`
- `enqueue`
- `doWork`
- `notify`
- `callModel`
- `callVendor`
- `startJob`

## Object, quota, and downstream cost keywords
- `reportId`
- `exportId`
- `fileId`
- `documentId`
- `imageId`
- `ocr`
- `llm`
- `model`
- `token`
- `cost`
- `quota`
- `job`
- `task`
- `vendor`
- `recipient`

## Required-control keywords
- `quota`
- `rateLimit`
- `dedupe`
- `payloadHash`
- `inFlight`
- `size`
- `cost`
- `budget`
- `retry`
- `idempotency`

## Graph search recipes
```text
button + export
doWork + callModel
enqueue + without quota
notify + fanout
retry + callVendor
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- push provider callbacks
- payment SDK result handlers
- login SDK callbacks
- share SDK callbacks
- Firebase message handlers
- exported receivers
- WorkManager sync jobs
- WebView callback bridges

## Business-effect sink candidates
- `handleCallback`
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
- `provider`
- `partner`
- `sdk`
- `externalId`
- `eventId`
- `callbackId`
- `accountId`
- `merchantId`
- `orderId`
- `paymentId`
- `shipmentId`
- `status`
- `payload`
- `signature`

## Required-control keywords
- `signature`
- `hmac`
- `timestamp`
- `nonce`
- `replay`
- `processed`
- `eventId`
- `idempotency`
- `accountId`
- `state`
- `freshness`
- `source`

## Graph search recipes
```text
onActivityResult + applyProviderState
onMessageReceived + processEvent
WebView bridge + handleCallback
replay + grantReward
callback + signature
```
