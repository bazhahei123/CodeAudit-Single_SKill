# C# / .NET Access Control Cases

## Purpose

This file contains C# and .NET-specific access control sink candidates, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in C# or .NET, especially:
- ASP.NET Core MVC
- ASP.NET Core Web API
- Razor Pages
- Minimal APIs
- SignalR
- gRPC services
- Blazor Server endpoints
- background jobs or admin tools exposed through .NET services
- Entity Framework or Dapper data-access layers

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the candidates below. Always verify the real control path in the target code.

---

# 1. C# / .NET Authorization Control Points

## 1.1 Route and endpoint sink candidates
Look for:
- `[ApiController]`
- `[Controller]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- MVC controller actions
- Razor Page handlers such as `OnGet`, `OnPost`, `OnPut`, `OnDelete`
- Minimal API `MapGet`, `MapPost`, `MapPut`, `MapPatch`, `MapDelete`
- `MapControllers`
- `MapControllerRoute`
- `MapRazorPages`
- `MapHub`
- `MapGrpcService`
- endpoint groups
- conventional routing

Questions:
- Which endpoints expose sensitive resources or actions?
- Are admin/internal endpoints separated and protected?
- Are route parameters such as `id`, `userId`, `tenantId`, `orgId`, `fileId` scoped?

## 1.2 Authentication enforcement candidates
Look for:
- `AddAuthentication`
- `UseAuthentication`
- `UseAuthorization`
- authentication schemes
- JWT bearer configuration
- cookie authentication
- OpenID Connect configuration
- `RequireAuthenticatedUser`
- fallback policies
- endpoint metadata
- `[AllowAnonymous]`
- global filters
- custom middleware

Questions:
- Is authentication middleware correctly ordered before authorization?
- Does `[AllowAnonymous]` expose sensitive actions?
- Are minimal APIs or hubs covered by authorization policies?

## 1.3 Function-level authorization candidates
Look for:
- `[Authorize]`
- `[Authorize(Roles = "...")]`
- `[Authorize(Policy = "...")]`
- `[AllowAnonymous]`
- `RequireAuthorization`
- `RequireAuthorization("Policy")`
- `IAuthorizationService.AuthorizeAsync`
- `AuthorizationHandler`
- `AuthorizationRequirement`
- custom filters such as `IAsyncAuthorizationFilter`
- `User.IsInRole`
- `User.HasClaim`
- `ClaimsPrincipal`
- policy names such as `AdminOnly`, `CanManageUsers`, `CanExport`

Questions:
- Is the action restricted to the intended role, permission, or policy?
- Are fallback/default policies strong enough?
- Are equivalent actions protected consistently?

## 1.4 Object-level and tenant authorization candidates
Look for:
- Entity Framework `Find`, `FindAsync`
- `FirstOrDefault`, `FirstOrDefaultAsync`
- `SingleOrDefault`, `SingleOrDefaultAsync`
- `Where(x => x.Id == id)` without user/tenant scope
- Dapper queries by ID
- repository `GetById`, `FindById`, `DeleteById`
- object returned before authorization
- `tenantId`, `orgId`, `accountId`, `workspaceId`, `userId` from route/query/body
- claims used as tenant or role without validation against server-side records

Questions:
- Is the object lookup scoped to the authenticated principal or trusted tenant?
- Is authorization checked before return, update, delete, export, or share?
- Is tenant scope trusted or client-selectable?

## 1.5 SignalR, gRPC, jobs, and alternate entry candidates
Look for:
- SignalR `Hub`
- hub methods called by clients
- `[Authorize]` on hubs and methods
- gRPC service classes
- `ServerCallContext`
- background jobs triggered by endpoints
- Hangfire dashboards and job methods
- Quartz jobs triggered through admin UI
- health/admin/debug endpoints
- file download/export endpoints
- GraphQL resolvers in HotChocolate or GraphQL.NET

Questions:
- Are hub or gRPC methods checked per operation?
- Can a job be triggered for another user's object?
- Are dashboards and admin tooling protected?

## 1.6 High-coverage C# / .NET sink candidate inventory

Search candidates:
- `[ApiController]`
- `[Route]`
- `[HttpGet]`
- `[HttpPost]`
- `[HttpPut]`
- `[HttpPatch]`
- `[HttpDelete]`
- `[Authorize]`
- `[AllowAnonymous]`
- `RequireAuthorization`
- `MapGet`
- `MapPost`
- `MapPut`
- `MapPatch`
- `MapDelete`
- `MapGroup`
- `MapHub`
- `MapGrpcService`
- `OnGet`
- `OnPost`
- `UseAuthentication`
- `UseAuthorization`
- `AddAuthorization`
- `AddAuthentication`
- `IAuthorizationService`
- `User.IsInRole`
- `User.HasClaim`
- `ClaimsPrincipal`
- `FindAsync`
- `Find`
- `FirstOrDefaultAsync`
- `SingleOrDefaultAsync`
- `GetById`
- `DeleteById`
- `tenantId`
- `ownerId`
- `userId`
- `export`
- `download`
- `admin`
- `impersonate`

---

# 2. C# / .NET Access Control Anti-Patterns

## A1. Sensitive endpoint has `[AllowAnonymous]`
Why risky:
Anonymous access may reach protected resources or actions.

Verify:
- route sensitivity,
- upstream gateway controls,
- fallback authorization policy,
- endpoint-specific authz.

## A2. Minimal API endpoint lacks `RequireAuthorization`
Why risky:
Minimal APIs can bypass controller-level conventions.

Verify:
- group-level authorization,
- fallback policies,
- explicit `RequireAuthorization`.

## A3. Entity loaded by ID without owner or tenant scope
Why risky:
Object-level authorization may fail when a route ID can be changed.

Verify:
- query predicates,
- `IAuthorizationService` checks,
- trusted tenant context.

## A4. SignalR or gRPC method lacks per-operation authorization
Why risky:
Class-level authentication may not prove the user can invoke every hub or service method.

Verify:
- method-level policy,
- object scope,
- tenant/account binding.

---

# 3. Quick C# / .NET Audit Checklist

- Are controllers, Razor Pages, minimal APIs, hubs, and gRPC services authenticated?
- Are admin/internal actions protected by roles or policies?
- Are `[AllowAnonymous]` endpoints intentionally public?
- Are object lookups scoped by user, owner, tenant, or organization?
- Are Minimal API groups and endpoints protected consistently?
- Are SignalR hub methods checked per action?
- Are export/download/bulk operations scoped like normal reads?
- Is middleware order `UseAuthentication()` before `UseAuthorization()`?
