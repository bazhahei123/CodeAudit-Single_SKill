# Python Business Logic Candidate Sink Cases

## Purpose

This file defines Python-specific candidate search terms for business logic abuse review.

Use it when the target application is primarily implemented in Python, especially:
- Django
- Django REST Framework
- Flask
- FastAPI
- Starlette
- Tornado
- Celery / RQ / Dramatiq workers
- Graphene, Strawberry, Ariadne, or other GraphQL frameworks
- SQLAlchemy, Django ORM, repository/service layers

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. Python Entry Candidate Baseline

Use these Python entry candidates across all scenarios:
- Django `urlpatterns`
- `path(...)`
- `re_path(...)`
- function-based views
- class-based views
- DRF `APIView`
- DRF `ViewSet`
- DRF `ModelViewSet`
- DRF `@api_view`
- DRF `@action`
- Flask `@app.route`
- Flask blueprint routes
- FastAPI `@app.get`
- FastAPI `@app.post`
- FastAPI `@app.put`
- FastAPI `@app.patch`
- FastAPI `@app.delete`
- `APIRouter`
- `Depends(...)`
- Starlette `Route`
- Tornado `RequestHandler`
- GraphQL mutations and resolvers
- Celery `@shared_task`
- Celery `@app.task`
- RQ jobs
- Dramatiq actors
- webhook views
- admin actions
- management commands reachable through admin or jobs

Baseline write/effect candidates:
- `save`
- `update`
- `delete`
- `create`
- `bulk_create`
- `bulk_update`
- `commit`
- `flush`
- `dispatch`
- `send`
- `notify`
- `enqueue`
- `delay`
- `apply_async`
- `handle`
- `process`
- `execute`

Baseline control candidates:
- `validate`
- `check`
- `verify`
- `authorize`
- `permission`
- `has_perm`
- `is_staff`
- `is_superuser`
- `can`
- `idempotency`
- `dedupe`
- `processed`
- `select_for_update`
- `transaction.atomic`
- `unique`
- `rate_limit`
- `quota`
- `signature`
- `hmac`
- `status`
- `state`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- `@app.post` / `@router.post` on checkout, pay, charge, refund, capture, settlement, callback, or webhook paths
- DRF actions for pay, refund, callback, notify, webhook, wallet, or order state
- Celery tasks processing payment events
- scheduled reconciliation jobs
- admin refund or replay tools

## Business-effect sink candidates
- `create_order`
- `submit_order`
- `pay`
- `charge`
- `capture`
- `confirm_payment`
- `mark_paid`
- `update_payment_status`
- `refund`
- `refund_order`
- `reverse`
- `void_payment`
- `settle`
- `reconcile`
- `update_balance`
- `credit`
- `debit`
- `create_ledger_entry`
- `grant_entitlement`
- `activate_subscription`
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
- `verify_signature`
- `validate_callback`
- `idempotency_key`
- `transaction_id`
- `payment_id`
- `processed`
- `already_paid`
- `already_refunded`
- `recompute_amount`
- `calculate_payable`
- `transaction.atomic`
- `select_for_update`
- `unique`
- `status`
- `beneficiary`
- `user_id`
- `tenant_id`

## Graph search recipes
```text
@router.post + refund
Celery task + mark_paid
webhook + grant_entitlement
update_payment_status + without idempotency_key
create_ledger_entry + amount
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- `@app.post` / `@router.post` on login, OTP, verify, reset, bind, recovery, MFA, or step-up paths
- DRF actions named send_code, verify, reset_password, bind_phone, bind_email
- Celery tasks sending OTP or verification messages
- account recovery callbacks

## Business-effect sink candidates
- `send_code`
- `send_otp`
- `verify_code`
- `verify_otp`
- `verify_email`
- `verify_phone`
- `reset_password`
- `change_password`
- `bind_account`
- `bind_phone`
- `bind_email`
- `unbind`
- `issue_token`
- `create_session`
- `mark_verified`
- `enable_mfa`
- `disable_mfa`
- `recover_account`

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
- `rate_limit`

## Graph search recipes
```text
@router.post + reset_password
verify_otp + without used_at
bind_phone + target
issue_token + verify_code
send_otp + rate_limit
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- `@app.post` / `@router.post` on send, verify, invite, export, report, search, notification, or task trigger paths
- Celery / RQ / Dramatiq jobs consuming user-triggered work
- scheduled jobs processing user-created tasks
- admin tools replaying jobs

## Business-effect sink candidates
- `send`
- `send_sms`
- `send_email`
- `verify`
- `invite`
- `export`
- `generate_report`
- `create_job`
- `delay`
- `apply_async`
- `notify`
- `increment_usage`
- `consume_quota`
- `create_attempt`
- `process_batch`

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
- `IP`
- `Device`
- `Tenant`
- `User`
- `Job`
- `Report`
- `Notification`
- `Cost`

## Required-control keywords
- `rate_limit`
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
@router.post + send_sms
@router.post + export
delay + without quota
send_email + target + cooldown
Celery task + process_batch + limit
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- `@app.post`, `@router.post`, or `@router.patch` on approve, reject, submit, publish, archive, cancel, close, reopen, assign, or resolve paths
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
- `update_status`
- `set_status`
- `set_state`
- `change_state`

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
- `allowed_transition`
- `state_machine`
- `terminal`
- `role`
- `permission`
- `approver`
- `owner`
- `prerequisite`
- `sequence`
- `select_for_update`
- `idempotency`

## Graph search recipes
```text
@router.patch + update_status
approve + without current_state
publish + approval
set_state + terminal
Celery task + transition
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- `@app.post` / `@router.post` on claim, redeem, coupon, reward, referral, invite, campaign, or points paths
- checkout discount application endpoints
- event listeners or tasks issuing rewards
- scheduled reward settlement jobs

## Business-effect sink candidates
- `claim`
- `redeem`
- `apply_coupon`
- `calculate_discount`
- `issue_coupon`
- `grant_reward`
- `add_points`
- `credit_reward`
- `create_referral`
- `bind_invite`
- `mark_eligible`
- `consume_benefit`
- `settle_reward`

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
- `select_for_update`

## Graph search recipes
```text
@router.post + redeem
apply_coupon + discount
grant_reward + beneficiary
claim + unique
create_referral + self_referral
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- `@app.post` / `@router.post` on export, report, OCR, AI, LLM, translation, notification, preview, or batch paths
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
- `delay`
- `apply_async`
- `fanout`
- `notify`
- `call_model`
- `call_vendor`
- `create_task`
- `start_job`

## Object, quota, and downstream cost keywords
- `Report`
- `Export`
- `File`
- `Document`
- `Image`
- `OCR`
- `LLM`
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
- `rate_limit`
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
@router.post + generate_report
delay + call_model
export + without quota
notify + fanout
Celery task + call_vendor + retry
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- `@app.post` / `@router.post` under webhook, callback, provider, partner, sync, import, or reconcile paths
- Celery / RQ / Dramatiq jobs for provider events
- admin replay endpoints
- scheduled sync and reconciliation jobs

## Business-effect sink candidates
- `handle_webhook`
- `process_callback`
- `process_event`
- `sync`
- `import`
- `reconcile`
- `update_external_status`
- `apply_provider_state`
- `mark_delivered`
- `settle`
- `fulfill`
- `grant_reward`
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
@router.post + handle_webhook
process_event + without event_id
replay + apply_provider_state
Celery task + fulfill
callback + signature
```
