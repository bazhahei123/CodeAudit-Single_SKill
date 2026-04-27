# Java Access Control Source Cases

## Purpose

This file contains Java-specific source point patterns and audit cases for access-control source discovery.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- Spring Security
- MyBatis
- JPA / Hibernate
- Servlet-based web applications
- Java backends exposing REST APIs, admin panels, GraphQL, RPC, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real source origin and downstream use in the target code.

---

# 1. Java Source Discovery Points

When auditing Java applications, prioritize these source points.

## 1.1 Route and entry-point sources
Look for:
- `@RestController`
- `@Controller`
- `@RequestMapping`
- `@GetMapping`
- `@PostMapping`
- `@PutMapping`
- `@DeleteMapping`
- `@PatchMapping`
- servlet mappings
- GraphQL resolvers
- RPC-style handlers
- scheduled or async job triggers reachable from user input

Source questions:
- Which routes accept external identifiers such as `id`, `userId`, `orderId`, `tenantId`, `orgId`, `fileId`?
- Which routes expose sensitive actions such as export, delete, approve, publish, reset, or disable?
- Are equivalent capabilities exposed through multiple controller methods or HTTP methods?

## 1.2 Request parameter sources
Look for:
- `@PathVariable`
- `@RequestParam`
- `@RequestBody`
- `@RequestHeader`
- `@CookieValue`
- `HttpServletRequest`
- DTO fields bound from request bodies
- GraphQL arguments

Source questions:
- Is the value client-controlled?
- Does the value represent identity, role, permission, object ID, tenant scope, or business action?
- Is the value later used in a repository, service, policy, or workflow call?

## 1.3 Authentication identity sources
Look for:
- `Principal`
- `Authentication`
- `SecurityContextHolder`
- `@AuthenticationPrincipal`
- JWT claim parsing
- session attributes
- custom current-user helpers
- servlet filters setting request attributes

Source questions:
- Is the identity derived from a verified authentication context?
- Is any user ID taken from request input instead of the authenticated principal?
- Is identity copied into a request attribute by trusted middleware?

## 1.4 Function authorization sources
Look for:
- `@PreAuthorize`
- `@PostAuthorize`
- `@Secured`
- `@RolesAllowed`
- Spring Security expressions
- role or permission checks in controller/service code
- policy service calls such as `permissionService.canXxx(...)`
- custom admin/operator checks

Source questions:
- Where do role and permission values come from?
- Is authorization based on server-side user context or request-supplied role/scope?
- Are policy inputs also source points, such as object ID, tenant ID, and action name?

## 1.5 Object and tenant source points
Look for:
- repository method arguments
- service method arguments
- JPA `findById(...)`
- custom `findByIdAndUserId(...)`
- custom `findByIdAndTenantId(...)`
- MyBatis mapper parameters
- query DTOs and filter objects

Source questions:
- Which identifier selects the protected object?
- Is tenant/org scope passed from request or trusted context?
- Are batch ID lists checked per object downstream?

## 1.6 Business action and state sources
Look for:
- action names in route paths
- command DTOs
- status or state fields in request bodies
- workflow service calls
- approval, publish, cancel, void, refund, delete, disable, reset, lock, or unlock methods

Source questions:
- Is the action selected by the route, request body, enum, or service method?
- Is state loaded from database or supplied by client?
- Does the action combine object ID, current user, role, tenant, and state?

---

# 2. Java Source Patterns and Blind Spots

These are high-priority source signals. They are not automatic proof of a vulnerability.

## 2.1 Authentication Identity Source Patterns

### S1. Trusted principal identity
```java
@GetMapping("/account/profile")
public Profile getProfile(Authentication authentication) {
    Long userId = ((CustomUser) authentication.getPrincipal()).getId();
    return profileService.getProfile(userId);
}
```

Why relevant:
The current identity is sourced from Spring Security and flows into a protected profile lookup.

What to record:
- `authentication.getPrincipal()` as server-trusted identity source
- derived `userId`
- downstream service call using that identity

### S2. User ID from request input
```java
@GetMapping("/account/profile")
public Profile getProfile(@RequestParam Long userId) {
    return profileService.getProfile(userId);
}
```

Why relevant:
`userId` is client-controlled and influences account profile access.

What to verify next:
- whether service binds the request to the authenticated user
- whether this is intended admin behavior
- whether role or ownership checks exist downstream

### S3. Identity copied by custom filter
```java
public class AuthFilter extends OncePerRequestFilter {
    protected void doFilterInternal(HttpServletRequest req, HttpServletResponse res, FilterChain chain) {
        req.setAttribute("currentUserId", tokenService.verify(req.getHeader("Authorization")).userId());
    }
}
```

Why relevant:
A request attribute becomes a server-side identity source only if token verification is real and active.

What to verify next:
- whether the filter is registered for the target route
- whether token verification rejects tampered tokens
- whether later code uses the request attribute instead of raw headers

---

## 2.2 Function Authorization Source Patterns

### S4. Annotation-based permission source
```java
@PreAuthorize("hasRole('ADMIN')")
@PostMapping("/admin/user/{id}/disable")
public Result disableUser(@PathVariable Long id) {
    userService.disable(id);
    return Result.ok();
}
```

Why relevant:
The function authorization source is the Spring Security role expression, while `id` is an object source.

What to record:
- role requirement from annotation
- object identifier from path
- privileged action from route and service call

### S5. Request-supplied role value
```java
@PostMapping("/reports/export")
public Report export(@RequestParam String role) {
    return reportService.exportForRole(role);
}
```

Why relevant:
`role` is client-controlled and may influence privileged export behavior.

What to verify next:
- whether `role` is only a filter label or a real authorization input
- whether server-side permission checks override it
- whether ordinary users can request stronger scopes

### S6. Permission helper with mixed inputs
```java
if (permissionService.canAccess(userId, projectId, action)) {
    return projectService.runAction(projectId, action);
}
```

Why relevant:
The policy result depends on multiple sources: identity, object ID, and action.

What to record:
- each input origin
- whether `userId` is trusted
- whether `projectId` and `action` are client-controlled

---

## 2.3 Object and Tenant Source Patterns

### S7. Path variable object ID
```java
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderService.getOrder(id);
}
```

Why relevant:
`id` is a client-controlled object identifier used to retrieve a protected order.

What to verify next:
- whether the service scopes by current user or tenant
- whether the order is public
- whether object authorization occurs after lookup

### S8. Client-supplied tenant ID
```java
@GetMapping("/projects/{id}")
public Project getProject(@PathVariable Long id, @RequestParam Long tenantId) {
    return projectRepository.findByIdAndTenantId(id, tenantId);
}
```

Why relevant:
`tenantId` is client-controlled but used as tenant scope.

What to verify next:
- whether tenant membership is verified
- whether tenant scope should come from authenticated context
- whether the repository call is the real enforcement point

### S9. Batch ID source
```java
@PostMapping("/files/delete")
public Result deleteFiles(@RequestBody DeleteFilesRequest req) {
    fileService.deleteAll(req.getFileIds());
    return Result.ok();
}
```

Why relevant:
`fileIds` is a client-controlled batch source for protected delete operations.

What to verify next:
- whether each file ID is checked for ownership or tenant scope
- whether partial failures are handled safely
- whether batch logic bypasses per-object policy checks

---

## 2.4 Business Action and State Source Patterns

### S10. Action selected by route
```java
@PostMapping("/invoice/{id}/void")
public Result voidInvoice(@PathVariable Long id) {
    invoiceService.voidInvoice(id);
    return Result.ok();
}
```

Why relevant:
The route itself supplies the business action `void`, and `id` selects the protected invoice.

What to verify next:
- whether current user has finance/admin permission
- whether invoice state allows voiding
- whether tenant or owner scope is enforced

### S11. Status supplied by request body
```java
@PatchMapping("/posts/{id}")
public Result updatePost(@PathVariable Long id, @RequestBody UpdatePostRequest req) {
    postService.updateStatus(id, req.getStatus());
    return Result.ok();
}
```

Why relevant:
`status` is a client-controlled business-state source that may trigger publish/archive behavior.

What to verify next:
- whether users can set privileged statuses directly
- whether allowed state transitions are checked
- whether status changes require stronger permissions

---

# 3. Case Templates

Use these as source reasoning patterns, not as direct proof.

## Case J-S-1: Identity Source

### Source pattern
```java
Long userId = ((CustomUser) SecurityContextHolder.getContext().getAuthentication().getPrincipal()).getId();
```

### Why relevant
The code derives current identity from Spring Security context.

### What to record
- source type: identity
- trust boundary: server-trusted if authentication setup is active
- downstream protected object or action using `userId`

### Follow-up
- confirm the route is covered by authentication
- confirm the principal cannot be attacker-controlled

---

## Case J-S-2: Object Identifier Source

### Source pattern
```java
public Order getOrder(@PathVariable Long orderId) {
    return orderRepository.findById(orderId).orElseThrow();
}
```

### Why relevant
`orderId` is client-controlled and selects a protected object.

### What to record
- source type: object-id
- trust boundary: client-controlled
- downstream use: repository lookup

### Follow-up
- verify ownership or tenant scoping in controller, service, repository, or policy layer

---

## Case J-S-3: Tenant Scope Source

### Source pattern
```java
return reportService.export(tenantId, reportType);
```

### Why relevant
`tenantId` may determine cross-tenant data access.

### What to record
- whether `tenantId` is from request or trusted context
- downstream export or query use
- any membership validation inputs

### Follow-up
- verify tenant scope is not raw client authority

---

## Case J-S-4: Function Authorization Source

### Source pattern
```java
@PreAuthorize("@policy.can(authentication, #projectId, 'DELETE')")
```

### Why relevant
The authorization decision depends on trusted identity, client-controlled object ID, and action.

### What to record
- identity source
- object source
- action source
- policy helper location

### Follow-up
- inspect the policy helper if access-control validation is in scope

---

## Case J-S-5: Business State Source

### Source pattern
```java
workflowService.transition(orderId, req.getTargetState());
```

### Why relevant
The requested state transition may affect authorization-sensitive workflow.

### What to record
- target state source
- object ID source
- current user source if present

### Follow-up
- verify allowed transition, permission, owner, and tenant checks

---

# 4. Java-Specific Audit Heuristics

## 4.1 Spring MVC heuristics
Pay attention to:
- all route annotations on the class and method
- inherited base controller mappings
- route groups split across multiple controllers
- DTO fields bound by `@RequestBody`
- validation annotations that do not prove authorization

Audit questions:
- Which request fields become authorization-relevant sources?
- Are sensitive actions exposed through both REST and web controllers?
- Are batch operations using arrays or collection fields?

## 4.2 Spring Security heuristics
Pay attention to:
- `SecurityFilterChain`
- `requestMatchers(...)`
- `permitAll()`
- `authenticated()`
- role expressions
- method security annotations
- custom permission evaluators

Audit questions:
- Which source values are used by security expressions?
- Are path variables referenced in `@PreAuthorize`?
- Does authorization depend on server-trusted identity or request-supplied values?

## 4.3 `Principal` / `Authentication` / `SecurityContextHolder`
Pay attention to:
- where current identity is read
- whether identity is copied into local variables, DTOs, or service arguments
- whether request-supplied `userId` competes with authenticated user ID

High-priority pattern:
```java
public Profile getProfile(@RequestParam Long userId, Authentication auth) {
    return profileService.getProfile(userId);
}
```

The presence of `Authentication` does not prove it is used.

## 4.4 JPA / Hibernate heuristics
Pay attention to:
- `findById(...)`
- repository method names
- `@Query`
- specifications and criteria builders
- entity relationships used for tenant or owner scope

Audit questions:
- Which method argument is the object source?
- Which method argument is identity or tenant scope?
- Are query filters sourced from trusted context or request input?

## 4.5 MyBatis heuristics
Pay attention to:
- mapper interface parameters
- XML mapper placeholders
- annotation SQL
- parameter maps and DTOs
- dynamic SQL filters

Audit questions:
- Does SQL scope by `user_id`, `owner_id`, `tenant_id`, or `org_id`?
- Where do those values come from?
- Are tenant filters omitted or client-supplied?

## 4.6 Layer inconsistency heuristics
Pay attention to:
- controller passes client source directly to service
- service passes source directly to repository
- one route derives user from principal while another accepts `userId`
- list route scopes by tenant but export route accepts tenant input

Audit questions:
- Are equivalent capabilities using different source origins?
- Does a weaker alternate path exist?

---

# 5. False-Positive Controls

Do not mark a Java source as high-priority if:
- the value does not reach access-control-relevant logic,
- the endpoint is intentionally public and unrelated to protected resources,
- the request value is overwritten by trusted identity or tenant context before sensitive use,
- the route is not reachable or is test-only outside production wiring,
- the source is only used for pagination, sorting, or display formatting.

Use `Suspected source` or `Not enough evidence` if:
- the value appears in a DTO but downstream use is unclear,
- the active security configuration is not visible,
- the service or repository implementation is missing,
- custom filters may rewrite identity or tenant context,
- annotations reference helper logic that has not been inspected.

---

# 6. What Good Evidence Looks Like

Good Java source evidence includes:
- controller method and route annotation,
- source annotation such as `@PathVariable`, `@RequestParam`, or `@RequestBody`,
- DTO field name when relevant,
- authentication context access when relevant,
- service or repository call receiving the source,
- query or policy call showing access-control relevance.

Good source evidence answers:
1. Which Java entry point receives the value?
2. Is the value client-controlled or server-trusted?
3. Which protected function, object, tenant, or workflow does it influence?
4. What follow-up access-control check is needed?

---

# 7. Quick Java Source Checklist

- Are object IDs read from `@PathVariable`, `@RequestParam`, or request DTOs?
- Are `userId`, `tenantId`, `orgId`, or `role` accepted from request input?
- Is current user read from `Authentication`, `Principal`, or `SecurityContextHolder`?
- Are policy helpers passed both trusted and client-controlled values?
- Are repository queries scoped by trusted user or tenant values?
- Are batch ID arrays used for delete, export, approve, or share actions?
- Are workflow states supplied by the client?
- Are equivalent REST, web, GraphQL, or RPC paths using different source origins?
