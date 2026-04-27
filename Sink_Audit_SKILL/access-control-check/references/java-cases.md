# Java Access Control Cases

## Purpose

This file contains Java-specific access control patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Java, especially in:
- Spring / Spring Boot
- Spring MVC
- Spring Security
- MyBatis
- JPA / Hibernate
- Servlet-based web applications
- Java backends exposing REST APIs, admin panels, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real control path in the target code.

---

# 1. Java Authorization Control Points

When auditing Java applications, prioritize these control points.

## 1.1 Route and entry-point controls
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

Questions:
- Which routes expose sensitive resources or actions?
- Which entry points accept external identifiers such as `id`, `userId`, `orderId`, `tenantId`, `orgId`, `fileId`?
- Are admin or internal routes clearly separated from public/user routes?

## 1.2 Authentication enforcement points
Look for:
- `SecurityFilterChain`
- Spring Security configuration
- `requestMatchers(...)`
- `antMatchers(...)`
- `permitAll()`
- `authenticated()`
- custom filters
- `OncePerRequestFilter`
- `HandlerInterceptor`
- servlet filters
- token parsing middleware
- session validation logic

Questions:
- Is authentication enforced for this route?
- Is the route accidentally placed under `permitAll()`?
- Are alternate route groups protected consistently?

## 1.3 Function-level authorization controls
Look for:
- `@PreAuthorize`
- `@PostAuthorize`
- `@Secured`
- `@RolesAllowed`
- role or permission checks in controller/service code
- policy service calls such as `permissionService.canXxx(...)`
- custom admin/operator checks
- internal-only route gating

Questions:
- Is a privileged action restricted to the correct role or permission scope?
- Is protection implemented only in some actions but not others?
- Is there a weaker alternate route or alternate HTTP method?

## 1.4 Object-level authorization controls
Look for:
- ownership checks after object lookup
- repository methods scoped by user ID
- repository methods scoped by tenant/org ID
- service-level policy checks after loading an object
- access-control checks tied to `Principal`, `Authentication`, or `SecurityContextHolder`

Examples of stronger patterns:
- `findByIdAndUserId(...)`
- `findByIdAndTenantId(...)`
- explicit `canAccess(user, object)` checks
- service methods that verify ownership before returning/updating data

Questions:
- Is the object fetched directly from a user-controlled identifier?
- Is ownership or tenant scope enforced before returning or mutating the object?
- Is the check server-side and tied to trusted identity context?

## 1.5 Business-context authorization controls
Look for:
- workflow or state checks before action execution
- order/payment/approval/lock/archive/publish state validation
- delete/disable/reset actions tied to business conditions
- actions allowed only during certain lifecycle phases

Questions:
- Is role check alone enough, or should object state also restrict the action?
- Can a user perform a sensitive action after the object reaches a forbidden state?

## 1.6 Data-access scoping controls
Look for:
- repository methods
- JPA queries
- `@Query`
- MyBatis XML
- MyBatis annotations
- SQL fragments in DAOs/services
- specification builders / criteria queries

Questions:
- Is the query scoped by `user_id`, `owner_id`, `tenant_id`, or `org_id`?
- Is scope enforced in query time or only assumed elsewhere?
- Is client-supplied `tenantId` trusted directly?

---

# 2. Java Access Control Anti-Patterns

These are high-risk signals. They are not automatic proof of a vulnerability, but they should trigger deeper review.

## 2.1 Authentication Boundary Anti-Patterns

### A1. Sensitive route exposed under `permitAll()`
```java
http
    .authorizeHttpRequests(auth -> auth
        .requestMatchers("/admin/**").permitAll()
        .anyRequest().authenticated()
    );
```

Why risky:
A clearly sensitive path is exposed without authentication.

What to verify:
- Is `/admin/**` really intended to be public?
- Is there another upstream gateway enforcing auth?
- Does this expose admin panels, exports, debug tools, or private APIs?

### A2. Missing auth guard on sensitive controller
```java
@GetMapping("/account/profile")
public Profile getProfile() {
    return profileService.getCurrentProfile();
}
```

Why risky:
If this route is not placed behind authenticated middleware or route configuration, it may allow anonymous access.

What to verify:
- Is authentication enforced globally?
- Is this controller inside a protected route group?
- Does framework configuration explicitly secure this path?

### A3. Download or export endpoint with no visible identity requirement
```java
@GetMapping("/export/all")
public ResponseEntity<byte[]> exportAll() {
    return ResponseEntity.ok(reportService.exportAll());
}
```

Why risky:
Exports often expose large volumes of sensitive data and should normally be authenticated and authorized.

What to verify:
- Is authentication enforced before this route?
- Is there role or permission gating for export behavior?

---

## 2.2 Function-Level Authorization Anti-Patterns

### A4. Admin action without role or permission check
```java
@PostMapping("/admin/user/{id}/disable")
public Result disableUser(@PathVariable Long id) {
    userService.disable(id);
    return Result.ok();
}
```

Why risky:
A privileged administrative operation is exposed without visible server-side authorization.

What to verify:
- Is there method/class-level security elsewhere?
- Is `userService.disable(...)` protected internally?
- Can ordinary authenticated users reach this route?

### A5. UI hides action but backend route remains callable
```java
@PostMapping("/admin/reset-password")
public Result resetPassword(@RequestParam Long userId) {
    adminService.resetPassword(userId);
    return Result.ok();
}
```

Why risky:
If the frontend hides the button but the route itself lacks server-side enforcement, direct API calls may bypass UI restrictions.

What to verify:
- Is there a backend permission check?
- Is access controlled only in the frontend?

### A6. Only some HTTP methods are protected
```java
@GetMapping("/admin/users")
@PreAuthorize("hasRole('ADMIN')")
public List<User> listUsers() { ... }

@PostMapping("/admin/users")
public Result createUser(@RequestBody CreateUserRequest req) { ... }
```

Why risky:
Protection may exist for one method but not another method on the same logical resource.

What to verify:
- Are GET/POST/PUT/DELETE/PATCH consistently protected?
- Does alternate method access create a privilege bypass?

---

## 2.3 Object-Level Authorization Anti-Patterns

### A7. Direct object fetch from user-controlled ID
```java
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderRepository.findById(id).orElseThrow();
}
```

Why risky:
The object is fetched directly from a user-controlled identifier with no visible ownership or tenant validation.

What to verify:
- Is the order bound to the current user?
- Is there a later service-layer permission check?
- Would `findByIdAndUserId(...)` be the expected pattern here?

### A8. Update/delete by raw object ID only
```java
@DeleteMapping("/files/{id}")
public Result deleteFile(@PathVariable Long id) {
    fileService.deleteById(id);
    return Result.ok();
}
```

Why risky:
Deletion based only on object ID is a common object-level authorization failure pattern.

What to verify:
- Does `deleteById` internally verify ownership or tenant scope?
- Can users delete another user's file by changing the ID?

### A9. Fetch first, trust later
```java
@GetMapping("/documents/{id}")
public Document getDocument(@PathVariable Long id) {
    Document doc = documentRepository.findById(id).orElseThrow();
    return doc;
}
```

Why risky:
A fetch with no visible post-fetch authorization check often indicates IDOR/BOLA risk.

What to verify:
- Is there any downstream policy or serializer restriction?
- Is the object intentionally public?
- Is ownership checked elsewhere before response generation?

---

## 2.4 Tenant Boundary Anti-Patterns

### A10. Trusting client-supplied tenant identifier
```java
@GetMapping("/projects/{id}")
public Project getProject(@PathVariable Long id, @RequestParam Long tenantId) {
    return projectRepository.findByIdAndTenantId(id, tenantId);
}
```

Why risky:
Tenant scope is derived from user input instead of trusted server-side identity.

What to verify:
- Can the requester swap `tenantId`?
- Should tenant scope come from session/JWT/server context instead?

### A11. Query without tenant/org scoping
```java
@Select("select * from invoice where id = #{id}")
Invoice getInvoiceById(Long id);
```

Why risky:
In multi-tenant systems, object fetch by raw ID alone may cross tenant boundaries.

What to verify:
- Is tenant boundary enforced earlier?
- Should the SQL include tenant/org filters?

---

## 2.5 Business-Context Authorization Anti-Patterns

### A12. Sensitive action ignores object state
```java
@PostMapping("/orders/{id}/cancel")
public Result cancel(@PathVariable Long id) {
    orderService.cancel(id);
    return Result.ok();
}
```

Why risky:
Cancellation may require not only role/ownership checks but also state validation, such as "not shipped" or "not locked".

What to verify:
- Does `cancel(...)` validate order state?
- Can users cancel already-finalized objects?
- Is workflow-stage enforcement missing?

### A13. Approval/publish/disable actions gated only by login
```java
@PostMapping("/post/{id}/publish")
public Result publish(@PathVariable Long id) {
    postService.publish(id);
    return Result.ok();
}
```

Why risky:
Publishing or approval actions are often business-sensitive and should require role, ownership, or workflow checks.

What to verify:
- Who is allowed to publish?
- Is draft/review/approval state respected?
- Can ordinary users trigger privileged lifecycle changes?

---

# 3. Case Templates

Use these as reasoning patterns, not as direct proof.

## Case J-1: Unauthenticated Access

### Vulnerable pattern
```java
@GetMapping("/admin/export")
public ResponseEntity<byte[]> exportAll() {
    return ResponseEntity.ok(reportService.exportAll());
}
```

### Why risky
The route appears to expose a sensitive export function without visible authentication or authorization enforcement.

### What to verify
- Is the route behind authenticated security configuration?
- Is the route class or method protected elsewhere?
- Is the export action intended for admins only?

### Safer signals
- route explicitly requires authentication
- route is restricted to admin/operator roles
- export action is gated by policy or permission service

---

## Case J-2: Function-Level Authorization

### Vulnerable pattern
```java
@PostMapping("/admin/user/{id}/disable")
public Result disableUser(@PathVariable Long id) {
    userService.disable(id);
    return Result.ok();
}
```

### Why risky
A privileged management function is exposed without visible role or permission checks.

### What to verify
- Is admin permission enforced at route, class, or method level?
- Is there a service-layer guard?
- Is this path reachable by ordinary authenticated users?

### Safer signals
- `@PreAuthorize("hasRole('ADMIN')")`
- centralized policy check
- controller and service both treat this as privileged behavior

---

## Case J-3: Object-Level Authorization / IDOR

### Vulnerable pattern
```java
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id) {
    return orderRepository.findById(id).orElseThrow();
}
```

### Why risky
The object is loaded directly from a user-controlled ID without visible ownership or tenant validation.

### What to verify
- Is the order restricted to the current user?
- Is there a policy check after the fetch?
- Is the repository expected to use `findByIdAndUserId(...)` or equivalent?

### Safer pattern
```java
@GetMapping("/orders/{id}")
public Order getOrder(@PathVariable Long id, Authentication auth) {
    Long userId = ((CustomUser) auth.getPrincipal()).getId();
    return orderRepository.findByIdAndUserId(id, userId)
        .orElseThrow(() -> new AccessDeniedException("Forbidden"));
}
```

---

## Case J-4: Tenant Isolation Failure

### Vulnerable pattern
```java
@GetMapping("/projects/{id}")
public Project getProject(@PathVariable Long id, @RequestParam Long tenantId) {
    return projectRepository.findByIdAndTenantId(id, tenantId);
}
```

### Why risky
Tenant scope is derived from client input and may allow cross-tenant access if the value can be changed.

### What to verify
- Does trusted tenant identity come from JWT/session/server context?
- Can the requester swap tenant ID and access another tenant's resource?
- Is this query the true enforcement point?

### Safer signals
- tenant context derived from authenticated identity
- repository query uses trusted server-side tenant scope
- access policy validates tenant membership before returning data

---

## Case J-5: Business-Context Authorization Failure

### Vulnerable pattern
```java
@PostMapping("/invoice/{id}/void")
public Result voidInvoice(@PathVariable Long id) {
    invoiceService.voidInvoice(id);
    return Result.ok();
}
```

### Why risky
Voiding an invoice may require role checks and workflow/state checks. If only identity is checked, the route may allow invalid or unsafe state transitions.

### What to verify
- Is only finance/admin allowed?
- Is the invoice state validated before voiding?
- Are already-finalized or locked invoices protected?

### Safer signals
- explicit role/permission checks
- explicit workflow/state checks
- denial on invalid lifecycle transitions

---

# 4. Java-Specific Audit Heuristics

## 4.1 Spring Security heuristics
Pay attention to:
- `SecurityFilterChain`
- `requestMatchers(...)`
- `permitAll()`
- `anyRequest().authenticated()`
- class-level vs method-level security
- security config split across multiple config classes

Audit questions:
- Is the route actually matched by the intended rule?
- Is an earlier matcher making the route more permissive than expected?
- Is the path protected in web routes but not in API routes?

## 4.2 `Principal` / `Authentication` / `SecurityContextHolder`
Pay attention to:
- where current identity is read
- whether object ownership is tied to trusted principal data
- whether user ID is taken from request instead of security context

High-risk pattern:
```java
@GetMapping("/profile")
public Profile getProfile(@RequestParam Long userId) {
    return profileService.getByUserId(userId);
}
```

Why risky:
The server trusts an externally supplied user ID instead of binding access to authenticated identity.

## 4.3 JPA / Hibernate heuristics
Pay attention to:
- `findById(...)`
- repository methods missing user/tenant scoping
- `@Query` without ownership or tenant filters
- service methods that load an entity and immediately return it
- updates/deletes executed after raw object fetch

Preferred safer patterns:
- `findByIdAndUserId(...)`
- `findByIdAndTenantId(...)`
- policy check after fetch
- service methods that validate ownership before mutation

## 4.4 MyBatis heuristics
Pay attention to:
- XML mapper SQL
- `@Select`, `@Update`, `@Delete`
- dynamic SQL fragments
- whether SQL includes `user_id`, `owner_id`, `tenant_id`, or `org_id`

High-risk pattern:
```xml
<select id="getOrder" resultType="Order">
  select * from orders where id = #{id}
</select>
```

Why risky:
If the object should be user- or tenant-scoped, raw ID lookup may enable object-level authorization bypass.

## 4.5 Layer inconsistency heuristics
Check whether access control is consistent across:
- controller vs service
- web routes vs REST APIs
- GET vs POST/PUT/DELETE/PATCH
- primary route vs admin/internal/legacy route
- single-object route vs export/bulk route

Common failure:
A list route is protected, but export or delete routes for the same resource are not protected equivalently.

---

# 5. False-Positive Controls

Do not report a vulnerability as confirmed if:
- the resource is intentionally public,
- the missing control is clearly enforced upstream,
- the route is a non-sensitive health/diagnostic endpoint,
- the object is public by design,
- the true enforcement point exists in middleware, service logic, policy logic, or gateway logic and evidence supports that.

If enforcement may exist elsewhere but cannot be confirmed:
- mark the result as `Suspected`, or
- mark the result as `Not enough evidence`.

Do not over-claim based only on:
- route naming,
- method naming,
- the presence of `findById(...)` alone,
- the absence of annotations alone.

---

# 6. What Good Evidence Looks Like

Strong evidence usually includes:
- the exact entry point
- the object identifier source
- the object fetch or action sink
- the current user / tenant identity source
- the missing or weak auth/authz control
- the reason an attacker could change route, object ID, tenant ID, or action path to cross the intended boundary

Good findings usually answer:
1. What can the attacker reach?
2. What boundary should stop them?
3. What code shows that the boundary is absent or weak?
4. Why is exploitation plausible?

---

# 7. Quick Java Audit Checklist

Use this as a reminder, not as a substitute for reasoning.

- Are sensitive routes authenticated?
- Are admin/internal routes function-authorized?
- Are object fetches bound to current user or tenant?
- Are deletes/updates checked as strictly as reads?
- Are alternate HTTP methods protected consistently?
- Are exports/downloads protected like primary views?
- Is tenant scope derived from trusted identity, not request input?
- Are workflow-sensitive actions gated by state as well as role?
- Are frontend-only restrictions backed by server-side enforcement?
