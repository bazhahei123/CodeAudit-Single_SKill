# PHP Access Control Cases

## Purpose

This file contains PHP-specific access control patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- ThinkPHP
- Yii
- CodeIgniter
- raw PHP / custom MVC backends
- PHP backends exposing REST APIs, admin panels, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real control path in the target code.

---

# 1. PHP Authorization Control Points

When auditing PHP applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- Laravel routes in `routes/web.php` and `routes/api.php`
- controller methods
- middleware assignments
- custom router definitions
- AJAX endpoints
- admin panels
- API endpoints
- legacy PHP files acting as entry points

Questions:
- Which routes expose sensitive resources or actions?
- Which entry points accept external identifiers such as `id`, `user_id`, `order_id`, `tenant_id`, `org_id`, `file_id`?
- Are admin or internal routes clearly separated from public/user routes?

## 1.2 Authentication enforcement points
Look for:
- Laravel auth middleware
- guard configuration
- route groups with middleware
- Symfony security/firewall rules
- custom session/token checks
- shared base controller auth checks
- raw PHP session validation logic

Questions:
- Is authentication enforced for this route?
- Is the route accidentally exposed as public?
- Are alternate route groups protected consistently?

## 1.3 Function-level authorization controls
Look for:
- Laravel policies and gates
- middleware such as `can:...`
- role/permission package checks
- custom admin/operator checks
- controller/service-level authorization calls
- Symfony voters or access controls
- manual `isAdmin()` / `hasRole()` / `can()` checks

Questions:
- Is a privileged action restricted to the correct role or permission scope?
- Is protection implemented only in some actions but not others?
- Is there a weaker alternate route or alternate HTTP method?

## 1.4 Object-level authorization controls
Look for:
- ownership checks after model lookup
- query scoping by current user
- query scoping by tenant/org
- policy checks such as `$this->authorize(...)`
- service-layer policy checks after loading an object
- access-control checks tied to trusted session/user context

Examples of stronger patterns:
- `Order::where('id', $id)->where('user_id', auth()->id())->firstOrFail()`
- tenant context from trusted session or middleware
- explicit policy checks after fetching a model

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
- Eloquent queries
- query builders
- repository/service methods
- raw SQL
- custom model loaders
- helper functions loading objects

Questions:
- Is the query scoped by `user_id`, `owner_id`, `tenant_id`, or `org_id`?
- Is scope enforced at query time or only assumed elsewhere?
- Is client-supplied `tenant_id` trusted directly?

---

# 2. PHP Access Control Anti-Patterns

These are high-risk signals. They are not automatic proof of a vulnerability, but they should trigger deeper review.

## 2.1 Authentication Boundary Anti-Patterns

### A1. Sensitive route outside auth middleware
```php
Route::get('/admin/export', [ReportController::class, 'exportAll']);
```

Why risky:
A clearly sensitive route is exposed without visible authentication.

What to verify:
- Is auth enforced globally?
- Is this route inside an authenticated middleware group?
- Does this expose admin panels, exports, debug tools, or private APIs?

### A2. Legacy PHP entry point without session validation
```php
<?php
$id = $_GET['id'] ?? null;
echo $reportService->exportAll();
```

Why risky:
Direct entry-point scripts may bypass centralized framework auth controls.

What to verify:
- Is session or token validation performed before sensitive logic?
- Is this file reachable directly from the web root?

### A3. Download or export endpoint with no visible identity requirement
```php
public function exportAll()
{
    return response()->json($this->reportService->exportAll());
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
```php
public function disableUser($id)
{
    $this->userService->disable($id);
    return response()->json(['ok' => true]);
}
```

Why risky:
A privileged administrative operation is exposed without visible server-side authorization.

What to verify:
- Is there middleware- or policy-based authorization elsewhere?
- Is `$this->userService->disable(...)` protected internally?
- Can ordinary authenticated users reach this route?

### A5. UI hides action but backend route remains callable
```php
Route::post('/admin/reset-password', [AdminController::class, 'resetPassword']);
```

Why risky:
If the frontend hides the button but the route itself lacks server-side enforcement, direct API calls may bypass UI restrictions.

What to verify:
- Is there a backend permission check?
- Is access controlled only in the frontend?

### A6. Only some HTTP methods are protected
```php
Route::get('/admin/users', [UserController::class, 'index'])->middleware('can:admin');
Route::post('/admin/users', [UserController::class, 'store']);
```

Why risky:
Protection may exist for one method but not another method on the same logical resource.

What to verify:
- Are GET/POST/PUT/DELETE/PATCH consistently protected?
- Does alternate method access create a privilege bypass?

---

## 2.3 Object-Level Authorization Anti-Patterns

### A7. Direct object fetch from user-controlled ID
```php
public function show($id)
{
    return Order::findOrFail($id);
}
```

Why risky:
The object is fetched directly from a user-controlled identifier with no visible ownership or tenant validation.

What to verify:
- Is the order bound to the current user?
- Is there a later policy or service-layer permission check?
- Would a constrained Eloquent query be the expected pattern here?

### A8. Update/delete by raw object ID only
```php
public function destroy($id)
{
    $this->fileService->deleteById($id);
    return response()->json(['ok' => true]);
}
```

Why risky:
Deletion based only on object ID is a common object-level authorization failure pattern.

What to verify:
- Does `deleteById` internally verify ownership or tenant scope?
- Can users delete another user's file by changing the ID?

### A9. Fetch first, trust later
```php
public function show($id)
{
    $doc = Document::findOrFail($id);
    return response()->json($doc);
}
```

Why risky:
A fetch with no visible post-fetch authorization check often indicates IDOR/BOLA risk.

What to verify:
- Is there any downstream policy or transformer restriction?
- Is the object intentionally public?
- Is ownership checked elsewhere before response generation?

---

## 2.4 Tenant Boundary Anti-Patterns

### A10. Trusting client-supplied tenant identifier
```php
public function show($id, Request $request)
{
    return Project::where('id', $id)
        ->where('tenant_id', $request->tenant_id)
        ->firstOrFail();
}
```

Why risky:
Tenant scope is derived from user input instead of trusted server-side identity.

What to verify:
- Can the requester swap `tenant_id`?
- Should tenant scope come from session/JWT/server context instead?

### A11. Query without tenant/org scoping
```php
public function getInvoice($id)
{
    return Invoice::findOrFail($id);
}
```

Why risky:
In multi-tenant systems, object fetch by raw ID alone may cross tenant boundaries.

What to verify:
- Is tenant boundary enforced earlier?
- Should the ORM query include tenant/org filters?

---

## 2.5 Business-Context Authorization Anti-Patterns

### A12. Sensitive action ignores object state
```php
public function cancel($id)
{
    $this->orderService->cancel($id);
    return response()->json(['ok' => true]);
}
```

Why risky:
Cancellation may require not only role/ownership checks but also state validation, such as "not shipped" or "not locked".

What to verify:
- Does `cancel(...)` validate order state?
- Can users cancel already-finalized objects?
- Is workflow-stage enforcement missing?

### A13. Approval/publish/disable actions gated only by login
```php
public function publish($id)
{
    $this->postService->publish($id);
    return response()->json(['ok' => true]);
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

## Case H-1: Unauthenticated Access

### Vulnerable pattern
```php
Route::get('/admin/export', [ReportController::class, 'exportAll']);
```

### Why risky
The route appears to expose a sensitive export function without visible authentication or authorization enforcement.

### What to verify
- Is the route behind authenticated middleware?
- Is the route protected elsewhere at controller or global config level?
- Is the export action intended for admins only?

### Safer signals
- route explicitly requires authentication
- route is restricted to admin/operator roles
- export action is gated by policy or permission service

---

## Case H-2: Function-Level Authorization

### Vulnerable pattern
```php
public function disableUser($id)
{
    $this->userService->disable($id);
    return response()->json(['ok' => true]);
}
```

### Why risky
A privileged management function is exposed without visible role or permission checks.

### What to verify
- Is admin permission enforced at route, controller, or method level?
- Is there a service-layer guard?
- Is this path reachable by ordinary authenticated users?

### Safer signals
- middleware or policy enforcing admin permission
- centralized gate/policy check
- controller and service both treat this as privileged behavior

---

## Case H-3: Object-Level Authorization / IDOR

### Vulnerable pattern
```php
public function show($id)
{
    return Order::findOrFail($id);
}
```

### Why risky
The object is loaded directly from a user-controlled ID without visible ownership or tenant validation.

### What to verify
- Is the order restricted to the current user?
- Is there a policy check after the fetch?
- Is the Eloquent query expected to include user/tenant constraints?

### Safer pattern
```php
public function show($id)
{
    return Order::where('id', $id)
        ->where('user_id', auth()->id())
        ->firstOrFail();
}
```

---

## Case H-4: Tenant Isolation Failure

### Vulnerable pattern
```php
public function show($id, Request $request)
{
    return Project::where('id', $id)
        ->where('tenant_id', $request->tenant_id)
        ->firstOrFail();
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
- query uses trusted server-side tenant scope
- access policy validates tenant membership before returning data

---

## Case H-5: Business-Context Authorization Failure

### Vulnerable pattern
```php
public function voidInvoice($id)
{
    $this->invoiceService->voidInvoice($id);
    return response()->json(['ok' => true]);
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

# 4. PHP-Specific Audit Heuristics

## 4.1 Laravel heuristics
Pay attention to:
- route middleware groups
- `auth`, `can`, policy usage
- controller helpers such as `$this->authorize(...)`
- Eloquent queries
- form request / service-layer validation
- guards and multiple auth contexts

Audit questions:
- Is the route actually inside the intended middleware group?
- Is authentication present without object authorization?
- Are policies applied consistently for show/update/delete actions?

## 4.2 Symfony heuristics
Pay attention to:
- firewall/access_control config
- voters
- controller annotations/attributes
- service-level checks
- object fetch patterns before authorization checks

High-risk pattern:
```php
public function profile(Request $request)
{
    $userId = $request->query->get('user_id');
    return $this->profileService->getByUserId($userId);
}
```

Why risky:
The server trusts an externally supplied user ID instead of binding access to authenticated identity.

## 4.3 Raw PHP / custom MVC heuristics
Pay attention to:
- direct use of `$_GET`, `$_POST`, `$_REQUEST`
- session checks
- legacy admin scripts
- alternate entry-point PHP files bypassing framework routing
- helper functions loading objects by raw ID

Preferred safer patterns:
- trusted session user identity
- tenant context from server-side session or middleware
- object query constrained by current user or tenant
- policy check after fetch

## 4.4 ORM / Query heuristics
Pay attention to:
- `find($id)`, `findOrFail($id)`
- query builder calls without user/tenant scope
- raw SQL or repository helpers
- service methods that load an entity and immediately return it
- updates/deletes executed after raw object fetch

High-risk pattern:
```php
$order = Order::findOrFail($id);
```

Why risky:
If the object should be user- or tenant-scoped, raw ID lookup may enable object-level authorization bypass.

## 4.5 Layer inconsistency heuristics
Check whether access control is consistent across:
- route vs controller vs service
- web routes vs API routes
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
- the presence of `findOrFail($id)` alone,
- the absence of middleware or annotations alone.

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

# 7. Quick PHP Audit Checklist

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
