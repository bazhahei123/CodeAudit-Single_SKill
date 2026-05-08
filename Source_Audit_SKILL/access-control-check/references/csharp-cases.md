# C# and .NET Access Control Source Cases

## Purpose

This file contains C#/.NET-specific source point patterns and audit cases for access-control source discovery.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC and Web API
- Razor Pages
- minimal APIs
- SignalR hubs
- gRPC services
- WCF services
- Azure Functions and worker services
- .NET services exposing REST APIs, admin panels, GraphQL, RPC, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real source origin and downstream use in the target code.

---

# 1. High-Coverage C#/.NET Source Candidate Inventory

## 1.1 HTTP, controller, and route entry candidates

Look for:
- `[ApiController]`
- `[Controller]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpDelete]`
- `[HttpPatch]`
- `[AcceptVerbs]`
- `ControllerBase`
- `Controller`
- `IActionResult`
- `ActionResult<T>`
- Razor Pages `PageModel`
- `OnGet`, `OnPost`, `OnPut`, `OnDelete`
- minimal APIs: `MapGet`, `MapPost`, `MapPut`, `MapDelete`, `MapPatch`, `MapMethods`, `MapGroup`
- endpoint filters and route groups
- legacy ASP.NET MVC/WebForms handlers

Source questions:
- Which routes accept external identifiers such as `id`, `userId`, `orderId`, `tenantId`, `orgId`, `fileId`?
- Which routes expose sensitive actions such as export, delete, approve, publish, reset, refund, or disable?
- Are equivalent capabilities exposed through MVC, Razor, API, SignalR, Functions, or background jobs?

## 1.2 Request binding and client-controlled source candidates

Look for:
- `[FromRoute]`
- `[FromQuery]`
- `[FromBody]`
- `[FromForm]`
- `[FromHeader]`
- `[FromServices]`
- model binding properties
- `HttpRequest.Query`
- `HttpRequest.Form`
- `HttpRequest.Headers`
- `HttpRequest.Cookies`
- `Request.Body`
- `IFormFile`
- `RouteData.Values`
- DTO/record properties
- JSON body models
- GraphQL arguments
- gRPC request message fields

High-priority fields:
- `id`, `ids`, `userId`, `accountId`, `ownerId`
- `tenantId`, `orgId`, `organizationId`, `companyId`, `workspaceId`
- `role`, `roles`, `permission`, `scope`, `isAdmin`, `accessLevel`
- `orderId`, `invoiceId`, `fileId`, `projectId`, `documentId`, `paymentId`
- `action`, `operation`, `status`, `state`, `targetState`

## 1.3 Authentication identity source candidates

Look for:
- `HttpContext.User`
- `ClaimsPrincipal`
- `User.Identity`
- `User.FindFirst(...)`
- `ClaimTypes.NameIdentifier`
- JWT claims such as `sub`, `user_id`, `uid`, `tenant_id`, `scope`, `roles`
- `IHttpContextAccessor`
- ASP.NET Identity `UserManager<TUser>`
- `SignInManager<TUser>`
- session values `HttpContext.Session`
- custom helpers such as `CurrentUser`, `CurrentUserId`, `UserContext`, `TenantContext`, `GetCurrentUser`, `GetTenantId`
- middleware-populated `HttpContext.Items`

## 1.4 Role, permission, policy, and authority candidates

Look for:
- `[Authorize]`
- `[AllowAnonymous]`
- `[Authorize(Roles = ...)]`
- `[Authorize(Policy = ...)]`
- `IAuthorizationService.AuthorizeAsync`
- `AuthorizationHandler`
- `AuthorizationRequirement`
- `RequireAuthorization`
- `RequireRole`
- `RequireClaim`
- `RequirePolicy`
- `User.IsInRole(...)`
- `ClaimsPrincipal.HasClaim(...)`
- custom calls such as `CanAccess`, `CanRead`, `CanWrite`, `CanDelete`, `CheckPermission`, `RequireRole`, `IsAdmin`, `IsOwner`, `IsTenantMember`

## 1.5 Object, tenant, ORM, and repository source candidates

Look for:
- object IDs: `id`, `userId`, `ownerId`, `accountId`, `orderId`, `invoiceId`, `fileId`, `projectId`, `documentId`, `resourceId`, `paymentId`
- tenant scopes: `tenantId`, `orgId`, `organizationId`, `companyId`, `workspaceId`, `teamId`, `departmentId`, `accountId`
- Entity Framework Core: `Find`, `FindAsync`, `First`, `FirstOrDefault`, `Single`, `SingleOrDefault`, `Where`, `Include`, `ExecuteDelete`, `ExecuteUpdate`
- repository methods: `GetById`, `FindById`, `DeleteById`, `GetForUser`, `GetForTenant`, `ExportForOrg`
- Dapper or raw SQL parameters carrying identity, object, tenant, role, action, or status values
- batch operations using `IEnumerable<Guid>`, `List<int>`, arrays, or request DTO collections

## 1.6 SignalR, GraphQL, gRPC, Functions, and async entry candidates

Look for:
- SignalR `Hub`, `Hub<T>`, hub methods, `Context.User`, `Context.UserIdentifier`, `Clients.User`, `Groups`
- GraphQL.NET and HotChocolate resolvers, mutations, inputs, `IResolverContext`
- gRPC service methods and protobuf request fields
- WCF `[ServiceContract]`, `[OperationContract]`
- Azure Functions `[HttpTrigger]`, `[QueueTrigger]`, `[ServiceBusTrigger]`, `[EventGridTrigger]`, `[TimerTrigger]` when payloads originate from user-triggered actions
- MassTransit, NServiceBus, MediatR handlers, hosted services, queue consumers, webhook handlers

## 1.7 Business action and workflow candidates

Look for method, route, DTO, or command names containing:
- `Approve`
- `Reject`
- `Publish`
- `Archive`
- `Delete`
- `Disable`
- `Enable`
- `Lock`
- `Unlock`
- `Reset`
- `Refund`
- `Void`
- `Cancel`
- `Transfer`
- `Assign`
- `Share`
- `Export`
- `Download`
- `Invite`
- `Promote`
- `Demote`
- `Grant`
- `Revoke`

Also inspect fields:
- `Action`
- `Operation`
- `Status`
- `State`
- `Stage`
- `Transition`
- `TargetState`
- `TargetStatus`

---

# 2. C#/.NET Source Patterns and Blind Spots

## 2.1 Trusted claims identity source

```csharp
[HttpGet("profile")]
public Task<Profile> Profile()
{
    var userId = User.FindFirst(ClaimTypes.NameIdentifier)?.Value;
    return profileService.GetProfile(userId);
}
```

Why relevant:
The current identity is sourced from the framework claims principal and flows into a protected lookup.

What to verify next:
- whether authentication is required for the endpoint
- whether the claim is issued by a trusted identity provider
- whether request `userId` can override the trusted claim

## 2.2 Client-supplied object and tenant source

```csharp
[HttpGet("projects/{id}")]
public Task<Project> GetProject(Guid id, [FromQuery] Guid tenantId)
{
    return repository.GetProject(id, tenantId);
}
```

Why relevant:
`id` and `tenantId` are client-controlled values that may influence protected object and tenant access.

What to verify next:
- whether tenant membership is verified
- whether repository scoping uses trusted tenant context
- whether this route is intended for admin users only

## 2.3 SignalR action source

```csharp
public Task RunAction(Guid projectId, string action)
{
    return projectService.RunAction(projectId, action);
}
```

Why relevant:
Hub method arguments are client-controlled source points for object and business-action review.

What to verify next:
- whether the hub or method requires authorization
- whether `Context.User` is used in the service or policy check
- whether group membership is trusted or client-selected

---

# 3. C#/.NET Graph Search Recipes

```text
[HttpGet]/[HttpPost]/MapPost + [FromRoute]/[FromQuery]/[FromBody] + id/tenantId/action/status
[Authorize]/AuthorizeAsync/RequireAuthorization + object id/tenant id/action + protected operation
HttpContext.User/ClaimsPrincipal/UserManager + repository/service/policy call
SignalR Hub/gRPC/GraphQL resolver + request argument id/tenantId/action + service method
Azure Function/queue handler + payload userId/tenantId/objectId + protected action
```

---

# 4. False-Positive Controls

Do not mark a C#/.NET source as high-priority if:
- the value does not reach protected object access, tenant scope, policy logic, or privileged workflow,
- the request value is overwritten by trusted claims, membership, or server-side tenant context before sensitive use,
- the endpoint is intentionally public and code evidence supports that,
- the value only affects pagination, sorting, display, or non-sensitive filtering.

Use `Suspected source` or `Not enough evidence` if:
- authorization policy implementation is hidden,
- middleware may populate or rewrite context values,
- service/repository code is missing,
- route groups or endpoint filters may apply controls outside the visible method.

---

# 5. Quick C#/.NET Source Checklist

- Are object IDs bound from route, query, body, form, GraphQL, gRPC, hub, or function payloads?
- Are `userId`, `tenantId`, `orgId`, `role`, `permission`, or `scope` accepted from request input?
- Is current identity read from `HttpContext.User`, `ClaimsPrincipal`, or ASP.NET Identity?
- Are policy helpers passed both trusted and client-controlled values?
- Are EF/Dapper/repository queries scoped by trusted user or tenant values?
- Are SignalR, Azure Function, gRPC, GraphQL, or worker paths exposing equivalent protected actions?
