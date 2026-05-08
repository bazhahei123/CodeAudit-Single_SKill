# C# / .NET Business Logic Candidate Source Cases

## Purpose

This file defines C#/.NET-specific candidate search terms for business logic source discovery.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC / Web API
- Razor Pages
- minimal APIs
- SignalR hubs
- gRPC services
- WCF services
- Azure Functions
- Hangfire, Quartz.NET, hosted services, queue consumers
- EF Core, Dapper, repository/service layers

This reference is guidance, not proof. Candidate keywords only identify review targets.

---

# 1. C# / .NET Entry Candidate Baseline

Use these C#/.NET entry candidates across all scenarios:
- `[ApiController]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- Razor Pages `PageModel`, `OnGet`, `OnPost`
- minimal APIs `MapGet`, `MapPost`, `MapPut`, `MapPatch`, `MapDelete`, `MapGroup`
- SignalR `Hub` methods
- gRPC service methods
- WCF `[ServiceContract]`, `[OperationContract]`
- Azure Functions `[HttpTrigger]`, `[QueueTrigger]`, `[ServiceBusTrigger]`, `[EventGridTrigger]`, `[TimerTrigger]`
- MediatR handlers, MassTransit consumers, NServiceBus handlers, hosted services, Hangfire jobs

Baseline source extraction candidates:
- `[FromRoute]`, `[FromQuery]`, `[FromBody]`, `[FromForm]`, `[FromHeader]`
- `HttpRequest.Query`, `HttpRequest.Form`, `HttpRequest.Headers`, `HttpRequest.Cookies`, `Request.Body`
- model binding properties, DTO/record fields, `IFormFile`
- `HttpContext.User`, `ClaimsPrincipal`, `User.FindFirst`, `IHttpContextAccessor`
- SignalR `Context.User`, `Context.ConnectionId`, hub method arguments
- protobuf request fields, function trigger payloads, queue messages

Baseline downstream mapping candidates:
- `SaveChanges`, `SaveChangesAsync`, `Update`, `Remove`, `ExecuteUpdate`, `ExecuteDelete`
- `Publish`, `Send`, `Enqueue`, `Dispatch`, `Handle`, `Process`
- `Transition`, `Calculate`, `Recompute`, `Verify`, `Validate`, `AuthorizeAsync`
- transactions, locks, unique constraints, idempotency services

---

# 2. Payment and Settlement Candidates

## Entry candidates
- checkout, pay, charge, refund, capture, settlement, wallet, or callback controllers/minimal APIs
- payment provider webhooks, Azure Functions, queue consumers, and reconciliation jobs
- SignalR/gRPC methods that update payment or order state

## Business source candidates
- body/query/route fields for amount, currency, coupon, order ID, payment method, payer, payee, merchant, or beneficiary
- provider callback payload fields such as status, event ID, transaction ID, paid amount, refund amount, and signature
- idempotency headers, request IDs, claims, tenant, merchant, or wallet context

## Object, state, amount, identity, and beneficiary keywords
- `Order`, `Payment`, `Transaction`, `Invoice`, `Refund`, `Settlement`, `Wallet`, `Balance`, `Ledger`, `Entitlement`, `Subscription`
- `OrderId`, `PaymentId`, `TransactionId`, `RefundId`, `InvoiceId`, `MerchantId`, `PayerId`, `PayeeId`, `BeneficiaryId`
- `Amount`, `PayableAmount`, `TotalAmount`, `RefundAmount`, `Currency`, `Discount`, `Quantity`, `Status`

## Trust-boundary and replay keywords
- `IdempotencyKey`, `RequestId`, `EventId`, `ProviderEventId`, `TransactionId`
- `VerifySignature`, `ValidateCallback`, `Hmac`, `Timestamp`, `Nonce`
- `Processed`, `AlreadyPaid`, `AlreadyRefunded`, `RecomputeAmount`, `CalculatePayable`, `RowVersion`, `TransactionScope`

## Graph search recipes
```text
[HttpPost]/MapPost + FromBody Amount + CreateOrder
webhook Function + EventId/Status/Amount + UpdatePaymentStatus
header IdempotencyKey + Refund
queue consumer + ProviderEventId + MarkPaid
ClaimsPrincipal + MerchantId/OrderId + Refund
```

---

# 3. Authentication, Verification, and Account Binding Candidates

## Entry candidates
- login, OTP, verify, reset, bind, recovery, MFA, or step-up controllers/minimal APIs
- Azure Functions or queue consumers that send or validate verification messages
- SignalR/gRPC methods for auth or account binding

## Business source candidates
- body/query fields for OTP, token, phone, email, password, target user, device ID, purpose, or channel
- claims, session, challenge records, JWT values, and verification event payloads

## Object, state, identity, and permission keywords
- `Otp`, `VerificationCode`, `Challenge`, `ResetToken`, `Mfa`, `Device`, `Session`, `AccountBinding`
- `UserId`, `AccountId`, `TargetUserId`, `Phone`, `Email`, `DeviceId`, `Purpose`, `Channel`
- `Code`, `Token`, `Verified`, `Used`, `Expired`, `AttemptCount`, `Step`

## Trust-boundary and replay keywords
- `ExpiresAt`, `UsedAt`, `Attempts`, `Nonce`, `ChallengeId`, `SessionId`
- `VerifyCode`, `ValidateToken`, `MarkVerified`, `IssueToken`, `BindPhone`, `BindEmail`, `ResetPassword`
- `RateLimiter`, `Lockout`, `OneTime`, `Purpose`, `Target`

## Graph search recipes
```text
[HttpPost]/MapPost + Code/Token + VerifyOtp
FromBody Phone/Email + BindPhone/BindEmail
ResetPassword + Token + UserId/TargetUserId
SendOtp + target phone/email + RateLimiter
IssueToken + ChallengeId/SessionId + MarkVerified
```

---

# 4. Rate, Quota, and Anti-Abuse Candidates

## Entry candidates
- send code, invite, export, search, report, AI/LLM, notification, upload, or batch endpoints
- hosted services, queue consumers, scheduled jobs, and Azure Functions triggering repeated work
- SignalR/gRPC methods that fan out downstream work

## Business source candidates
- limiter keys from user ID, tenant ID, IP, device, target, operation, route, claims, or payload
- request fields defining workload size, target count, export type, model, provider, or retry count

## Object, quota, and cost keywords
- `RateLimit`, `Quota`, `Bucket`, `Window`, `Usage`, `Credit`, `Limit`, `Counter`
- `UserId`, `TenantId`, `Ip`, `DeviceId`, `Target`, `Operation`, `PayloadHash`
- `BatchSize`, `PageCount`, `TargetCount`, `Model`, `Provider`, `Cost`, `Credits`

## Trust-boundary and replay keywords
- `RateLimiter`, `Throttle`, `QuotaService`, `Consume`, `Remaining`, `Cooldown`, `Dedupe`
- `RequestId`, `JobId`, `RetryCount`, `Window`, `Ttl`

## Graph search recipes
```text
[HttpPost]/MapPost + target phone/email + SendOtp + RateLimiter
export/report endpoint + TenantId/UserId + Quota
FromBody BatchSize/Ids + Enqueue
AI endpoint + Model/Prompt + ConsumeCredits
queue consumer + RetryCount/JobId + Dispatch
```

---

# 5. Workflow, Approval, and Lifecycle Candidates

## Entry candidates
- approve, reject, publish, cancel, archive, restore, finalize, assign, moderate, disable, or enable endpoints
- SignalR/gRPC methods and MediatR commands for state transitions
- queue events, admin actions, and scheduled jobs that transition workflow objects

## Business source candidates
- route/body object IDs and action/status/target state fields
- actor, approver, assignee, role, reason, tenant context, claims, and current state values

## Object, state, and permission keywords
- `Workflow`, `Approval`, `Ticket`, `Order`, `Document`, `Content`, `Lifecycle`, `StateMachine`
- `ObjectId`, `OrderId`, `TicketId`, `DocumentId`, `ApproverId`, `AssigneeId`, `OwnerId`
- `Action`, `Status`, `State`, `TargetState`, `CurrentState`, `Terminal`, `Reason`

## Trust-boundary and replay keywords
- `AllowedTransition`, `StateMachine`, `CanApprove`, `CanPublish`, `RowVersion`, `TerminalState`
- `Approve`, `Reject`, `Publish`, `Cancel`, `Finalize`, `Assign`, `Archive`

## Graph search recipes
```text
[HttpPost]/MapPost + Action/TargetState + Transition
PATCH endpoint + Status + UpdateStatus
route id + Approve/Reject/Publish
MediatR command + State + Finalize
CurrentState + TargetState + StateMachine
```

---

# 6. Promotion, Coupon, Reward, and Referral Candidates

## Entry candidates
- coupon claim, redeem, referral, invite, reward, checkout discount, campaign, or loyalty endpoints
- queue consumers issuing rewards or credits
- SignalR/gRPC methods for coupon, reward, referral, or invite actions

## Business source candidates
- coupon code, campaign ID, referral code, referrer, invitee, claimant, beneficiary, reward amount, and discount request fields
- first-use, eligibility, usage count, stacking, and tenant values

## Object, reward, and entitlement keywords
- `Coupon`, `Promotion`, `Campaign`, `Reward`, `Referral`, `Invite`, `Discount`, `Credit`, `Point`, `Entitlement`
- `CouponCode`, `CampaignId`, `ReferrerId`, `InviteeId`, `BeneficiaryId`, `ClaimantId`
- `DiscountAmount`, `RewardValue`, `Points`, `Credits`, `Eligible`, `FirstUse`, `UsageCount`

## Trust-boundary and replay keywords
- `Redeem`, `Claim`, `ApplyDiscount`, `IssueReward`, `GrantPoints`, `Eligibility`, `UsageLimit`
- `IdempotencyKey`, `ReferralBinding`, `BeneficiaryBinding`, `AlreadyClaimed`, `Stacking`

## Graph search recipes
```text
[HttpPost]/MapPost + CouponCode + Redeem/ApplyDiscount
referral endpoint + ReferrerId/InviteeId + IssueReward
reward consumer + EventId + GrantPoints
checkout + DiscountAmount + CalculatePayable
CampaignId + BeneficiaryId + Claim
```

---

# 7. Resource Consumption and Downstream Tool Usage Candidates

## Entry candidates
- export, report, OCR, AI/LLM, translation, notification, search, import, or batch endpoints
- workers, hosted services, Azure Functions, and queue consumers that trigger downstream providers
- admin replay or operational tools that rerun expensive tasks

## Business source candidates
- file ID, report type, export type, model, prompt, provider, target list, batch IDs, workload size, and quota key fields
- actor, tenant, requester, target, and recipient values

## Object, quota, and downstream cost keywords
- `Export`, `Report`, `Ocr`, `Ai`, `Llm`, `Translation`, `Notification`, `Search`, `BatchJob`, `File`, `Document`
- `FileId`, `ReportId`, `BatchIds`, `RecipientIds`, `Model`, `Prompt`, `Provider`, `PageCount`, `TokenBudget`
- `Cost`, `Quota`, `Credits`, `TenantId`, `UserId`

## Trust-boundary and replay keywords
- `Enqueue`, `Dispatch`, `Generate`, `Export`, `Send`, `Fanout`, `ConsumeCredits`, `RateLimit`, `Quota`
- `JobId`, `RequestId`, `RetryCount`, `Dedupe`, `CostEstimate`

## Graph search recipes
```text
endpoint + FileId/ReportType + Enqueue
AI endpoint + Prompt/Model + ConsumeCredits
export endpoint + BatchIds + GenerateExport
notification endpoint + RecipientIds + Fanout
Function/consumer + JobId/RetryCount + provider call
```

---

# 8. Third-Party Callback and Integration Candidates

## Entry candidates
- webhook controllers, callback minimal APIs, partner sync jobs, provider event consumers, reconciliation jobs, manual replay tools, import jobs
- queue consumers handling provider or partner events
- Azure Functions or admin APIs that import external state

## Business source candidates
- event ID, provider status, external order ID, external payment ID, integration account, tenant, amount, currency, signature, timestamp, and replay fields
- imported external state mapped to local records

## Object, integration, and state keywords
- `Webhook`, `Callback`, `ProviderEvent`, `Integration`, `Partner`, `Sync`, `Reconciliation`, `ExternalOrder`, `ExternalPayment`
- `EventId`, `ExternalId`, `ProviderEventId`, `ProviderStatus`, `IntegrationAccountId`, `TenantId`
- `Amount`, `Currency`, `Status`, `State`, `MappingId`

## Trust-boundary and replay keywords
- `VerifySignature`, `Hmac`, `Timestamp`, `Nonce`, `ProcessedEvent`, `Dedupe`, `Replay`, `ValidateCallback`
- `Sync`, `Reconcile`, `UpdateStatus`, `CreateLocalRecord`, `MapExternalId`

## Graph search recipes
```text
webhook endpoint + EventId/Status + UpdateStatus
callback + Signature/Timestamp + ValidateCallback
queue consumer + ProviderEventId + ProcessedEvent
replay tool + ExternalId + Reconcile
IntegrationAccountId + TenantId + Sync
```
