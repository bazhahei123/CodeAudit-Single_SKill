# PHP Business Logic Candidate Source Cases

## Purpose

This file defines PHP-specific candidate search terms for business logic source discovery.

Use it when the target application is primarily implemented in PHP, especially:
- Laravel
- Symfony
- ThinkPHP
- Yii
- CodeIgniter
- raw PHP / custom MVC
- queue workers, jobs, events, listeners, and webhook handlers

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. PHP Entry Candidate Baseline

Use these PHP entry candidates across all scenarios:
- Laravel `Route::get`, `Route::post`, `Route::put`, `Route::patch`, `Route::delete`, `Route::match`, `Route::resource`, `Route::apiResource`
- Laravel controllers, invokable controllers, form requests, jobs, listeners, events, commands
- Symfony `#[Route]`, `@Route`, controller actions, commands, Messenger handlers
- ThinkPHP/Yii/CodeIgniter route and controller methods
- raw PHP scripts, custom routers, AJAX endpoints, webhook scripts, admin tools
- GraphQL resolvers, queue consumers, scheduled commands, import jobs

Baseline source extraction candidates:
- Laravel `$request->input`, `$request->query`, `$request->post`, `$request->get`, `$request->header`, `$request->cookie`, `$request->all`, `$request->validated`
- Symfony `Request->query`, `Request->request`, `Request->headers`, `Request->cookies`, `Request->attributes`
- raw `$_GET`, `$_POST`, `$_REQUEST`, `$_COOKIE`, `$_SERVER`, `php://input`
- `auth()->user()`, `Auth::user()`, `$request->user()`, `$this->getUser()`, sessions, JWT claims
- job payloads, event payloads, GraphQL arguments, decoded webhook JSON

Baseline downstream mapping candidates:
- `save`, `update`, `delete`, `insert`, `dispatch`, `handle`, `process`, `queue`
- `transition`, `calculate`, `recompute`, `verify`, `validate`, `authorize`
- `Gate::allows`, `$this->authorize`, `DB::transaction`, Doctrine transactions

---

# 2. Payment and Settlement Candidates

## Entry candidates
- checkout, pay, charge, refund, capture, settlement, wallet, or callback routes
- payment provider webhooks and reconciliation commands
- Laravel jobs/listeners or Symfony Messenger handlers for order paid, refund finished, or settlement events
- GraphQL mutations for payment, refund, wallet, or order state

## Business source candidates
- request/form fields for amount, currency, coupon, order ID, payment method, payer, payee, merchant, or beneficiary
- provider callback payload fields such as status, event ID, transaction ID, paid amount, refund amount, and signature
- idempotency headers, request IDs, authenticated user, tenant, merchant, or wallet context

## Object, state, amount, identity, and beneficiary keywords
- `Order`, `Payment`, `Transaction`, `Invoice`, `Refund`, `Settlement`, `Wallet`, `Balance`, `Ledger`, `Entitlement`, `Subscription`
- `order_id`, `payment_id`, `transaction_id`, `refund_id`, `invoice_id`, `merchant_id`, `payer_id`, `payee_id`, `beneficiary_id`
- `amount`, `payable_amount`, `total_amount`, `refund_amount`, `currency`, `discount`, `quantity`, `status`

## Trust-boundary and replay keywords
- `idempotency_key`, `request_id`, `event_id`, `provider_event_id`, `transaction_id`
- `verifySignature`, `validateCallback`, `hash_hmac`, `timestamp`, `nonce`
- `processed`, `already_paid`, `already_refunded`, `recomputeAmount`, `calculatePayable`, `lockForUpdate`, `transaction`

## Graph search recipes
```text
Route::post + input amount + createOrder
webhook + event_id/status/amount + updatePaymentStatus
header idempotency_key + refund
job/listener + provider_event_id + markPaid
auth()->user + merchant_id/order_id + refund
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- login, OTP, verify, reset, bind, recovery, MFA, or step-up routes
- GraphQL mutations for password reset, contact binding, token issue, or MFA
- jobs/listeners that send or validate verification messages

## Business source candidates
- request/form fields for OTP, token, phone, email, password, target user, device ID, purpose, or channel
- session, auth user, JWT claim, challenge record, or verification event values

## Object, state, identity, and permission keywords
- `otp`, `verification_code`, `challenge`, `reset_token`, `mfa`, `device`, `session`, `account_binding`
- `user_id`, `account_id`, `target_user_id`, `phone`, `email`, `device_id`, `purpose`, `channel`
- `code`, `token`, `verified`, `used`, `expired`, `attempt_count`, `step`

## Trust-boundary and replay keywords
- `expires_at`, `used_at`, `attempts`, `nonce`, `challenge_id`, `session_id`
- `verifyCode`, `validateToken`, `markVerified`, `issueToken`, `bindPhone`, `bindEmail`, `resetPassword`
- `RateLimiter`, `lock`, `one_time`, `purpose`, `target`

## Graph search recipes
```text
Route::post + code/token + verifyOtp
$request->input phone/email + bindPhone/bindEmail
resetPassword + token + user_id/target_user_id
sendOtp + target phone/email + RateLimiter
issueToken + challenge_id/session_id + markVerified
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send code, invite, export, search, report, AI/LLM, notification, upload, or batch routes
- queue workers and scheduled commands that trigger repeated or expensive work
- GraphQL mutations or RPC-style endpoints that fan out downstream work

## Business source candidates
- limiter keys from user ID, tenant ID, IP, device, target, operation, route, or payload
- request fields that define workload size, target count, export type, model, or provider
- headers or context values used as rate dimensions

## Object, quota, and cost keywords
- `rate_limit`, `quota`, `bucket`, `window`, `usage`, `credit`, `limit`, `counter`
- `user_id`, `tenant_id`, `ip`, `device_id`, `target`, `operation`, `payload_hash`
- `batch_size`, `page_count`, `target_count`, `model`, `provider`, `cost`, `credits`

## Trust-boundary and replay keywords
- `RateLimiter`, `throttle`, `quotaService`, `consume`, `remaining`, `cooldown`, `dedupe`
- `request_id`, `job_id`, `retry_count`, `window`, `ttl`

## Graph search recipes
```text
Route::post + target phone/email + sendOtp + RateLimiter
export/report route + tenant_id/user_id + quota
$request->input ids/batch_size + dispatch
AI route + model/prompt + consumeCredits
job + retry_count/job_id + handle
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approve, reject, publish, cancel, archive, restore, finalize, assign, moderate, disable, or enable routes
- GraphQL mutations and command handlers for state transitions
- queue events, admin actions, and scheduled jobs that transition workflow objects

## Business source candidates
- route parameters and request fields for object IDs, action, status, and target state
- actor, approver, assignee, role, reason, tenant context, and current state values

## Object, state, and permission keywords
- `workflow`, `approval`, `ticket`, `order`, `document`, `content`, `lifecycle`, `state_machine`
- `object_id`, `order_id`, `ticket_id`, `document_id`, `approver_id`, `assignee_id`, `owner_id`
- `action`, `status`, `state`, `target_state`, `current_state`, `terminal`, `reason`

## Trust-boundary and replay keywords
- `allowedTransition`, `stateMachine`, `canApprove`, `canPublish`, `lockForUpdate`, `version`, `terminal_state`
- `approve`, `reject`, `publish`, `cancel`, `finalize`, `assign`, `archive`

## Graph search recipes
```text
Route::post + action/target_state + transition
PATCH route + status + updateStatus
route id + approve/reject/publish
event listener + state + finalize
current_state + target_state + stateMachine
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
- `coupon`, `promotion`, `campaign`, `reward`, `referral`, `invite`, `discount`, `credit`, `point`, `entitlement`
- `coupon_code`, `campaign_id`, `referrer_id`, `invitee_id`, `beneficiary_id`, `claimant_id`
- `discount_amount`, `reward_value`, `points`, `credits`, `eligible`, `first_use`, `usage_count`

## Trust-boundary and replay keywords
- `redeem`, `claim`, `applyDiscount`, `issueReward`, `grantPoints`, `eligibility`, `usageLimit`
- `idempotency_key`, `referralBinding`, `beneficiaryBinding`, `already_claimed`, `stacking`

## Graph search recipes
```text
Route::post + coupon_code + redeem/applyDiscount
referral route + referrer_id/invitee_id + issueReward
reward job + event_id + grantPoints
checkout + discount_amount + calculatePayable
campaign_id + beneficiary_id + claim
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- export, report, OCR, AI/LLM, translation, notification, search, import, or batch routes
- queue workers and scheduled commands that trigger downstream providers
- admin replay or operational tools that rerun expensive tasks

## Business source candidates
- file ID, report type, export type, model, prompt, provider, target list, batch IDs, workload size, and quota key fields
- actor, tenant, requester, target, and recipient values

## Object, quota, and downstream cost keywords
- `export`, `report`, `ocr`, `ai`, `llm`, `translation`, `notification`, `search`, `batch_job`, `file`, `document`
- `file_id`, `report_id`, `batch_ids`, `recipient_ids`, `model`, `prompt`, `provider`, `page_count`, `token_budget`
- `cost`, `quota`, `credits`, `tenant_id`, `user_id`

## Trust-boundary and replay keywords
- `dispatch`, `generate`, `export`, `send`, `fanout`, `consumeCredits`, `RateLimiter`, `quota`
- `job_id`, `request_id`, `retry_count`, `dedupe`, `cost_estimate`

## Graph search recipes
```text
route + file_id/report_type + dispatch
AI route + prompt/model + consumeCredits
export route + batch_ids + generateExport
notification route + recipient_ids + fanout
job + job_id + retry_count
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- webhook routes, callback routes, partner sync jobs, provider event consumers, reconciliation jobs, manual replay tools, import jobs
- queue consumers handling provider or partner events
- GraphQL/RPC admin tools that import external state

## Business source candidates
- event ID, provider status, external order ID, external payment ID, integration account, tenant, amount, currency, signature, timestamp, and replay fields
- imported external state mapped to local records

## Object, integration, and state keywords
- `webhook`, `callback`, `provider_event`, `integration`, `partner`, `sync`, `reconciliation`, `external_order`, `external_payment`
- `event_id`, `external_id`, `provider_event_id`, `provider_status`, `integration_account_id`, `tenant_id`
- `amount`, `currency`, `status`, `state`, `mapping_id`

## Trust-boundary and replay keywords
- `verifySignature`, `hash_hmac`, `timestamp`, `nonce`, `processed_event`, `dedupe`, `replay`, `validateCallback`
- `sync`, `reconcile`, `updateStatus`, `createLocalRecord`, `mapExternalId`

## Graph search recipes
```text
webhook route + event_id/status + updateStatus
callback + signature/timestamp + validateCallback
job + provider_event_id + processed_event
replay tool + external_id + reconcile
integration_account_id + tenant_id + sync
```
