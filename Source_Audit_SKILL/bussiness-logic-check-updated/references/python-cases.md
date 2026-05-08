# Python Business Logic Candidate Source Cases

## Purpose

This file defines Python-specific candidate search terms for business logic source discovery.

Use it when the target application is primarily implemented in Python, especially:
- Django / Django REST Framework
- Flask
- FastAPI / Starlette
- Tornado
- Graphene / Strawberry / Ariadne
- Celery / RQ / Dramatiq / Huey
- SQLAlchemy, Django ORM, or repository/service layers

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. Python Entry Candidate Baseline

Use these Python entry candidates across all scenarios:
- Django `path(...)`, `re_path(...)`, view functions, class-based views
- DRF `APIView`, `ViewSet`, `GenericViewSet`, `ModelViewSet`, `@api_view`, `@action`
- Flask `@app.route`, `@blueprint.route`, `MethodView`
- FastAPI `@app.get`, `@app.post`, `@app.put`, `@app.patch`, `@app.delete`, `APIRouter`
- Starlette routes and WebSocket handlers
- GraphQL resolvers and mutations
- gRPC servicer methods
- Celery tasks, RQ jobs, Dramatiq actors, Huey tasks
- webhook handlers, import jobs, admin actions, management commands

Baseline source extraction candidates:
- Django `request.GET`, `request.POST`, `request.body`, `request.headers`, `request.COOKIES`, `request.FILES`
- DRF `request.data`, `request.query_params`, serializer `validated_data`
- Flask `request.args`, `request.form`, `request.json`, `request.get_json()`, `request.headers`
- FastAPI path/query/body parameters, Pydantic model fields, `Header`, `Cookie`, `Form`, `File`
- `request.user`, `current_user`, `Depends(get_current_user)`, `request.state`
- task payloads, message bodies, GraphQL arguments, protobuf request fields

Baseline downstream mapping candidates:
- `save`, `update`, `delete`, `bulk_update`, `bulk_create`
- `process`, `handle`, `enqueue`, `delay`, `apply_async`
- `transition`, `calculate`, `recompute`, `verify`, `validate`
- `authorize`, `has_perm`, `check_permission`, `transaction.atomic`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- checkout, pay, charge, refund, capture, settlement, wallet, or callback routes
- payment provider webhooks and reconciliation jobs
- Celery/RQ tasks handling order paid, refund finished, or settlement events
- GraphQL mutations for payment, refund, wallet, or order state

## Business source candidates
- request or serializer fields for amount, currency, coupon, order ID, payment method, payer, payee, merchant, or beneficiary
- provider callback payload fields such as status, event ID, transaction ID, paid amount, refund amount, and signature
- idempotency headers, request IDs, authenticated user, tenant, merchant, or wallet context

## Object, state, amount, identity, and beneficiary keywords
- `Order`, `Payment`, `Transaction`, `Invoice`, `Refund`, `Settlement`, `Wallet`, `Balance`, `Ledger`, `Entitlement`, `Subscription`
- `order_id`, `payment_id`, `transaction_id`, `refund_id`, `invoice_id`, `merchant_id`, `payer_id`, `payee_id`, `beneficiary_id`
- `amount`, `payable_amount`, `total_amount`, `refund_amount`, `currency`, `discount`, `quantity`, `status`

## Trust-boundary and replay keywords
- `idempotency_key`, `request_id`, `event_id`, `provider_event_id`, `transaction_id`
- `verify_signature`, `validate_callback`, `hmac`, `timestamp`, `nonce`
- `processed`, `already_paid`, `already_refunded`, `recompute_amount`, `calculate_payable`, `select_for_update`, `atomic`

## Graph search recipes
```text
route/@action + request.data amount + create_order
webhook + event_id/status/amount + update_payment_status
header idempotency_key + refund
Celery task + provider_event_id + mark_paid
request.user + merchant_id/order_id + refund
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- login, OTP, verify, reset, bind, recovery, MFA, or step-up routes
- GraphQL mutations for password reset, contact binding, token issue, or MFA
- queue/task paths that send or validate verification messages

## Business source candidates
- request/Pydantic/serializer fields for OTP, token, phone, email, password, target user, device ID, purpose, or channel
- session, current user, JWT claim, challenge record, or verification event values

## Object, state, identity, and permission keywords
- `otp`, `verification_code`, `challenge`, `reset_token`, `mfa`, `device`, `session`, `account_binding`
- `user_id`, `account_id`, `target_user_id`, `phone`, `email`, `device_id`, `purpose`, `channel`
- `code`, `token`, `verified`, `used`, `expired`, `attempt_count`, `step`

## Trust-boundary and replay keywords
- `expires_at`, `used_at`, `attempts`, `nonce`, `challenge_id`, `session_id`
- `verify_code`, `validate_token`, `mark_verified`, `issue_token`, `bind_phone`, `bind_email`, `reset_password`
- `rate_limit`, `lock`, `one_time`, `purpose`, `target`

## Graph search recipes
```text
route + code/token + verify_otp
request.data + phone/email + bind_phone/bind_email
reset_password + token + user_id/target_user_id
send_otp + target phone/email + rate_limit
issue_token + challenge_id/session_id + mark_verified
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send code, invite, export, search, report, AI/LLM, notification, upload, or batch routes
- Celery/RQ tasks and scheduled jobs that trigger repeated or expensive work
- GraphQL mutations or RPC methods that fan out downstream work

## Business source candidates
- limiter keys from user ID, tenant ID, IP, device, target, operation, route, or payload
- request fields that define workload size, target count, export type, model, or provider
- headers or context values used as rate dimensions

## Object, quota, and cost keywords
- `rate_limit`, `quota`, `bucket`, `window`, `usage`, `credit`, `limit`, `counter`
- `user_id`, `tenant_id`, `ip`, `device_id`, `target`, `operation`, `payload_hash`
- `batch_size`, `page_count`, `target_count`, `model`, `provider`, `cost`, `credits`

## Trust-boundary and replay keywords
- `rate_limiter`, `throttle`, `quota_service`, `consume`, `remaining`, `cooldown`, `dedupe`
- `request_id`, `job_id`, `retry_count`, `window`, `ttl`

## Graph search recipes
```text
route + target phone/email + send_otp + rate_limiter
export/report route + tenant_id/user_id + quota
request.data + batch_size/ids + enqueue
AI route + model/prompt + consume_credits
Celery task + retry_count/job_id + dispatch
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approve, reject, publish, cancel, archive, restore, finalize, assign, moderate, disable, or enable routes
- GraphQL mutations and RPC methods for state transitions
- queue events, admin actions, and scheduled jobs that transition workflow objects

## Business source candidates
- path or request body object IDs and action/status/target state fields
- actor, approver, assignee, role, reason, tenant context, and current state values

## Object, state, and permission keywords
- `workflow`, `approval`, `ticket`, `order`, `document`, `content`, `lifecycle`, `state_machine`
- `object_id`, `order_id`, `ticket_id`, `document_id`, `approver_id`, `assignee_id`, `owner_id`
- `action`, `status`, `state`, `target_state`, `current_state`, `terminal`, `reason`

## Trust-boundary and replay keywords
- `allowed_transition`, `state_machine`, `can_approve`, `can_publish`, `lock`, `version`, `terminal_state`
- `approve`, `reject`, `publish`, `cancel`, `finalize`, `assign`, `archive`

## Graph search recipes
```text
route/@action + action/target_state + transition
PATCH route + status + update_status
path id + approve/reject/publish
event listener + state + finalize
current_state + target_state + state_machine
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- coupon claim, redeem, referral, invite, reward, checkout discount, campaign, or loyalty routes
- queue/task paths issuing rewards or credits
- GraphQL mutations for coupon, reward, referral, or invite actions

## Business source candidates
- coupon code, campaign ID, referral code, referrer, invitee, claimant, beneficiary, reward amount, and discount request fields
- first-use, eligibility, usage count, stacking, and tenant values

## Object, reward, and entitlement keywords
- `coupon`, `promotion`, `campaign`, `reward`, `referral`, `invite`, `discount`, `credit`, `point`, `entitlement`
- `coupon_code`, `campaign_id`, `referrer_id`, `invitee_id`, `beneficiary_id`, `claimant_id`
- `discount_amount`, `reward_value`, `points`, `credits`, `eligible`, `first_use`, `usage_count`

## Trust-boundary and replay keywords
- `redeem`, `claim`, `apply_discount`, `issue_reward`, `grant_points`, `eligibility`, `usage_limit`
- `idempotency_key`, `referral_binding`, `beneficiary_binding`, `already_claimed`, `stacking`

## Graph search recipes
```text
route + coupon_code + redeem/apply_discount
referral route + referrer_id/invitee_id + issue_reward
reward task + event_id + grant_points
checkout + discount_amount + calculate_payable
campaign_id + beneficiary_id + claim
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
- `export`, `report`, `ocr`, `ai`, `llm`, `translation`, `notification`, `search`, `batch_job`, `file`, `document`
- `file_id`, `report_id`, `batch_ids`, `recipient_ids`, `model`, `prompt`, `provider`, `page_count`, `token_budget`
- `cost`, `quota`, `credits`, `tenant_id`, `user_id`

## Trust-boundary and replay keywords
- `enqueue`, `dispatch`, `generate`, `export`, `send`, `fanout`, `consume_credits`, `rate_limit`, `quota`
- `job_id`, `request_id`, `retry_count`, `dedupe`, `cost_estimate`

## Graph search recipes
```text
route + file_id/report_type + enqueue
AI route + prompt/model + consume_credits
export route + batch_ids + generate_export
notification route + recipient_ids + fanout
task + job_id + retry_count
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
- `verify_signature`, `hmac`, `timestamp`, `nonce`, `processed_event`, `dedupe`, `replay`, `validate_callback`
- `sync`, `reconcile`, `update_status`, `create_local_record`, `map_external_id`

## Graph search recipes
```text
webhook route + event_id/status + update_status
callback + signature/timestamp + validate_callback
task + provider_event_id + processed_event
replay tool + external_id + reconcile
integration_account_id + tenant_id + sync
```
