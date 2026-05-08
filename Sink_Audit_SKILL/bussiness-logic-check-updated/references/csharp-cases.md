# C# / .NET Business Logic Candidate Sink Cases

## Purpose

This file defines C# and .NET-specific candidate search terms for business logic abuse review.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC / Web API
- Minimal APIs
- Razor Pages
- SignalR
- gRPC services
- Hangfire / Quartz / background services
- MassTransit / Azure Service Bus / RabbitMQ consumers
- Entity Framework, Dapper, repository, or service layers

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. C# / .NET Entry Candidate Baseline

Use these C# / .NET entry candidates across all scenarios:
- `[ApiController]`
- `[Controller]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- Razor Page `OnGet`, `OnPost`, `OnPut`, `OnDelete`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- `MapGroup`
- `MapHub`
- `MapGrpcService`
- SignalR `Hub` methods
- gRPC service methods
- GraphQL resolvers and mutations
- `BackgroundService`
- `IHostedService`
- Hangfire jobs
- Quartz jobs
- MassTransit consumers
- Azure Function HTTP triggers
- queue or topic message handlers

Baseline write/effect candidates:
- `SaveChanges`
- `SaveChangesAsync`
- `Add`
- `Update`
- `Remove`
- `ExecuteAsync`
- `Publish`
- `Send`
- `Enqueue`
- `Dispatch`
- `Process`
- `Handle`
- `Commit`
- `Submit`

Baseline control candidates:
- `Validate`
- `Check`
- `Verify`
- `AuthorizeAsync`
- `RequireAuthorization`
- `User.IsInRole`
- `User.HasClaim`
- `Can`
- `Policy`
- `Idempotency`
- `Dedupe`
- `Processed`
- `TransactionScope`
- `BeginTransaction`
- `RowVersion`
- `ConcurrencyStamp`
- `Unique`
- `RateLimit`
- `Quota`
- `Signature`
- `Hmac`
- `Status`
- `State`

---

# 2. Payment and Settlement Candidates

## Entry candidates
- `[HttpPost]` or `MapPost` on checkout, pay, charge, refund, capture, settlement, callback, or webhook paths
- SignalR or gRPC methods processing payment results
- MassTransit / queue consumers for payment events
- Hangfire or Quartz reconciliation jobs
- admin refund or replay endpoints

## Business-effect sink candidates
- `CreateOrder`
- `SubmitOrder`
- `Pay`
- `Charge`
- `Capture`
- `ConfirmPayment`
- `MarkPaid`
- `UpdatePaymentStatus`
- `Refund`
- `RefundOrder`
- `Reverse`
- `VoidPayment`
- `Settle`
- `Reconcile`
- `UpdateBalance`
- `Credit`
- `Debit`
- `CreateLedgerEntry`
- `GrantEntitlement`
- `ActivateSubscription`
- `Fulfill`
- `Ship`
- `Deliver`

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
- `Amount`
- `PayableAmount`
- `TotalAmount`
- `Currency`
- `Discount`
- `Status`
- `Paid`
- `Pending`
- `Refunded`
- `Settled`
- `Cancelled`

## Required-control keywords
- `VerifySignature`
- `ValidateCallback`
- `IdempotencyKey`
- `TransactionId`
- `PaymentId`
- `Processed`
- `AlreadyPaid`
- `AlreadyRefunded`
- `RecomputeAmount`
- `CalculatePayable`
- `TransactionScope`
- `BeginTransaction`
- `RowVersion`
- `Unique`
- `Beneficiary`
- `UserId`
- `TenantId`

## Graph search recipes
```text
[HttpPost] + Refund
MapPost + Charge
consumer + MarkPaid
UpdatePaymentStatus + without IdempotencyKey
CreateLedgerEntry + Amount
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- `[HttpPost]` or `MapPost` on login, OTP, verify, reset, bind, recovery, MFA, or step-up paths
- Razor Page handlers for reset or bind actions
- SignalR/gRPC auth-related methods
- background jobs sending OTP or verification messages

## Business-effect sink candidates
- `SendCode`
- `SendOtp`
- `VerifyCode`
- `VerifyOtp`
- `VerifyEmail`
- `VerifyPhone`
- `ResetPassword`
- `ChangePassword`
- `BindAccount`
- `BindPhone`
- `BindEmail`
- `Unbind`
- `IssueToken`
- `CreateSession`
- `MarkVerified`
- `EnableMfa`
- `DisableMfa`
- `RecoverAccount`

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
- `Purpose`
- `Target`
- `Verified`
- `Used`
- `Expired`
- `Attempt`

## Required-control keywords
- `Issued`
- `Verify`
- `Purpose`
- `Target`
- `Session`
- `ExpiresAt`
- `UsedAt`
- `Consume`
- `AttemptLimit`
- `Cooldown`
- `Lockout`
- `Nonce`
- `OneTime`
- `RateLimit`

## Graph search recipes
```text
MapPost + ResetPassword
VerifyOtp + without UsedAt
BindPhone + Target
IssueToken + VerifyCode
SendOtp + RateLimit
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- `[HttpPost]` or `MapPost` on send, verify, invite, export, report, search, notification, or task trigger paths
- Hangfire/Quartz jobs consuming user-triggered work
- queue consumers processing expensive tasks
- admin tools replaying or bulk-triggering jobs

## Business-effect sink candidates
- `Send`
- `SendSms`
- `SendEmail`
- `Verify`
- `Invite`
- `Export`
- `GenerateReport`
- `CreateJob`
- `Enqueue`
- `Dispatch`
- `Notify`
- `IncrementUsage`
- `ConsumeQuota`
- `CreateAttempt`
- `ProcessBatch`

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
- `Quota`
- `Cooldown`
- `Attempt`
- `Lockout`
- `Dedupe`
- `InFlight`
- `Unique`
- `Window`
- `Daily`
- `Hourly`
- `Target`
- `Tenant`
- `PayloadHash`
- `Size`

## Graph search recipes
```text
MapPost + SendSms
[HttpPost] + Export
Enqueue + without Quota
SendEmail + Target + Cooldown
consumer + ProcessBatch + Limit
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- `[HttpPost]`, `[HttpPatch]`, `MapPost`, or `MapPatch` on approve, reject, submit, publish, archive, cancel, close, reopen, assign, or resolve paths
- GraphQL mutations for lifecycle transitions
- SignalR/gRPC methods changing business state
- queue consumers and admin batch endpoints

## Business-effect sink candidates
- `Approve`
- `Reject`
- `Submit`
- `Resubmit`
- `Publish`
- `Unpublish`
- `Archive`
- `Restore`
- `Cancel`
- `Close`
- `Reopen`
- `Finalize`
- `Assign`
- `Resolve`
- `Transition`
- `UpdateStatus`
- `SetStatus`
- `SetState`
- `ChangeState`

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
- `CurrentState`
- `ExpectedState`
- `AllowedTransition`
- `StateMachine`
- `Terminal`
- `Role`
- `Permission`
- `Approver`
- `Owner`
- `Prerequisite`
- `Sequence`
- `RowVersion`
- `Idempotency`

## Graph search recipes
```text
MapPatch + UpdateStatus
Approve + without CurrentState
Publish + Approval
SetState + Terminal
consumer + Transition
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- `[HttpPost]` or `MapPost` on claim, redeem, coupon, reward, referral, invite, campaign, or points paths
- checkout discount application endpoints
- provider/event consumers issuing rewards
- scheduled reward settlement jobs

## Business-effect sink candidates
- `Claim`
- `Redeem`
- `ApplyCoupon`
- `CalculateDiscount`
- `IssueCoupon`
- `GrantReward`
- `AddPoints`
- `CreditReward`
- `CreateReferral`
- `BindInvite`
- `MarkEligible`
- `ConsumeBenefit`
- `SettleReward`

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
- `Eligibility`
- `FirstUse`
- `FirstOrder`
- `NewUser`
- `Limit`
- `Unique`
- `Redeemed`
- `Claimed`
- `Consumed`
- `Stacking`
- `Policy`
- `Beneficiary`
- `SelfReferral`
- `Idempotency`
- `RowVersion`

## Graph search recipes
```text
MapPost + Redeem
ApplyCoupon + Discount
GrantReward + Beneficiary
Claim + Unique
CreateReferral + SelfReferral
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- `[HttpPost]` or `MapPost` on export, report, OCR, AI, LLM, translation, notification, preview, or batch paths
- queue producers and consumers for expensive tasks
- scheduled retries for downstream jobs
- admin endpoints triggering replay or batch work

## Business-effect sink candidates
- `Generate`
- `Export`
- `Process`
- `Analyze`
- `Translate`
- `Transcribe`
- `Classify`
- `Summarize`
- `Enqueue`
- `Dispatch`
- `Fanout`
- `Notify`
- `CallModel`
- `CallVendor`
- `CreateTask`
- `StartJob`

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
- `Quota`
- `RateLimit`
- `Limit`
- `Dedupe`
- `Hash`
- `InFlight`
- `Size`
- `Cost`
- `Tenant`
- `Daily`
- `Monthly`
- `Budget`
- `Retry`
- `Idempotency`
- `Admission`

## Graph search recipes
```text
MapPost + GenerateReport
Enqueue + CallModel
Export + without Quota
Notify + Fanout
consumer + CallVendor + Retry
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- `[HttpPost]` or `MapPost` under webhook, callback, provider, partner, sync, import, or reconcile paths
- queue consumers for provider events
- admin replay endpoints
- scheduled sync and reconciliation jobs

## Business-effect sink candidates
- `HandleWebhook`
- `ProcessCallback`
- `ProcessEvent`
- `Sync`
- `Import`
- `Reconcile`
- `UpdateExternalStatus`
- `ApplyProviderState`
- `MarkDelivered`
- `Settle`
- `Fulfill`
- `GrantReward`
- `Retry`
- `Replay`

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
- `Signature`
- `Hmac`
- `Timestamp`
- `Nonce`
- `Replay`
- `Processed`
- `EventId`
- `Idempotency`
- `Tenant`
- `Merchant`
- `Account`
- `State`
- `Reconcile`
- `Freshness`
- `Source`

## Graph search recipes
```text
MapPost + HandleWebhook
ProcessEvent + without EventId
Replay + ApplyProviderState
consumer + Fulfill
Callback + Signature
```
