# PHP Business Logic Candidate Sink Cases

## Purpose

This file defines PHP-specific candidate search terms for business logic abuse review.

Use it when the target application is primarily implemented in PHP, especially:
- Laravel
- Symfony
- ThinkPHP
- Yii
- CodeIgniter
- Slim / Mezzio / Laminas
- raw PHP or custom MVC backends
- queue workers, webhook handlers, admin panels, and scheduled jobs

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. PHP Entry Candidate Baseline

Use these PHP entry candidates across all scenarios:
- `Route::get`
- `Route::post`
- `Route::put`
- `Route::patch`
- `Route::delete`
- `Route::match`
- `Route::any`
- `Route::resource`
- `Route::apiResource`
- controller methods such as `index`, `show`, `store`, `update`, `destroy`
- invokable controllers with `__invoke`
- Symfony `#[Route(...)]`
- Symfony `@Route`
- Yii controllers and actions
- ThinkPHP controller/action routes
- CodeIgniter controllers and filters
- Slim route definitions
- GraphQL resolvers and mutations
- webhook controllers
- queue jobs implementing `handle`
- Laravel listeners and subscribers
- scheduled commands
- console commands reachable through admin tools
- legacy `.php` entry files

Baseline write/effect candidates:
- `save`
- `update`
- `insert`
- `delete`
- `destroy`
- `forceDelete`
- `restore`
- `dispatch`
- `event`
- `notify`
- `send`
- `handle`
- `process`
- `execute`
- `DB::transaction`

Baseline control candidates:
- `validate`
- `authorize`
- `Gate::authorize`
- `$this->authorize`
- `can`
- `policy`
- `hasRole`
- `hasPermissionTo`
- `idempotency`
- `dedupe`
- `processed`
- `lockForUpdate`
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
- `Route::post` on checkout, pay, charge, refund, capture, settlement, callback, or webhook paths
- controller methods named `pay`, `checkout`, `refund`, `callback`, `notify`, `webhook`
- Laravel jobs or listeners processing payment events
- scheduled reconciliation commands
- admin refund or replay tools

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
- `payable_amount`
- `total_amount`
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
- `idempotency_key`
- `transaction_id`
- `payment_id`
- `processed`
- `alreadyPaid`
- `alreadyRefunded`
- `recompute`
- `calculatePayable`
- `DB::transaction`
- `lockForUpdate`
- `unique`
- `status`
- `beneficiary`
- `user_id`
- `tenant_id`

## Graph search recipes
```text
Route::post + refund
handle + markPaid
webhook + grantEntitlement
updatePaymentStatus + without idempotency_key
createLedgerEntry + amount
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- `Route::post` on login, OTP, verify, reset, bind, recovery, MFA, or step-up paths
- controller methods named `sendCode`, `verify`, `resetPassword`, `bindPhone`, `bindEmail`
- notification jobs for OTP or verification messages
- account recovery callbacks

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
- `expires_at`
- `used_at`
- `consume`
- `attempt_limit`
- `cooldown`
- `lockout`
- `nonce`
- `one_time`
- `rateLimit`

## Graph search recipes
```text
Route::post + resetPassword
verifyOtp + without used_at
bindPhone + target
issueToken + verifyCode
sendOtp + rateLimit
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- `Route::post` on send, verify, invite, export, report, search, notification, or task trigger paths
- jobs or listeners consuming user-triggered work
- scheduled jobs processing user-created tasks
- admin tools replaying jobs

## Business-effect sink candidates
- `send`
- `sendSms`
- `sendEmail`
- `verify`
- `invite`
- `export`
- `generateReport`
- `createJob`
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
- `RateLimiter`
- `throttle`
- `quota`
- `cooldown`
- `attempt`
- `lockout`
- `dedupe`
- `unique`
- `window`
- `daily`
- `hourly`
- `target`
- `tenant`
- `payload_hash`
- `size`

## Graph search recipes
```text
Route::post + sendSms
Route::post + export
dispatch + without quota
sendEmail + target + cooldown
handle + processBatch + limit
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- `Route::post` / `Route::patch` on approve, reject, submit, publish, archive, cancel, close, reopen, assign, or resolve paths
- GraphQL mutations for lifecycle transitions
- queue jobs changing business state
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
- `current_state`
- `expected_state`
- `allowedTransition`
- `stateMachine`
- `terminal`
- `role`
- `permission`
- `approver`
- `owner`
- `prerequisite`
- `sequence`
- `lockForUpdate`
- `idempotency`

## Graph search recipes
```text
Route::patch + updateStatus
approve + without current_state
publish + approval
setState + terminal
handle + transition
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- `Route::post` on claim, redeem, coupon, reward, referral, invite, campaign, or points paths
- checkout discount application endpoints
- event listeners issuing rewards
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
- `first_use`
- `first_order`
- `new_user`
- `limit`
- `unique`
- `redeemed`
- `claimed`
- `consumed`
- `stacking`
- `policy`
- `beneficiary`
- `self_referral`
- `idempotency`
- `lockForUpdate`

## Graph search recipes
```text
Route::post + redeem
applyCoupon + discount
grantReward + beneficiary
claim + unique
createReferral + self_referral
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- `Route::post` on export, report, OCR, AI, LLM, translation, notification, preview, or batch paths
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
- `RateLimiter`
- `limit`
- `dedupe`
- `hash`
- `in_flight`
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
Route::post + generateReport
dispatch + callModel
export + without quota
notify + fanout
handle + callVendor + retry
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- `Route::post` under webhook, callback, provider, partner, sync, import, or reconcile paths
- queue jobs for provider events
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
- `event_id`
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
Route::post + handleWebhook
processEvent + without event_id
replay + applyProviderState
handle + fulfill
callback + signature
```
