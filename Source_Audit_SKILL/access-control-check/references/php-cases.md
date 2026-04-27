# PHP Access Control Source Cases

## Purpose

This file contains PHP-specific source point patterns and audit cases for access-control source discovery.

Use it when the target application is primarily implemented in PHP, especially in:
- Laravel
- Symfony
- ThinkPHP
- Yii
- CodeIgniter
- raw PHP / custom MVC backends
- PHP backends exposing REST APIs, admin panels, GraphQL, RPC, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real source origin and downstream use in the target code.

---

# 1. PHP Source Discovery Points

When auditing PHP applications, prioritize these source points.

## 1.1 Route and entry-point sources
Look for:
- Laravel routes in `routes/web.php` and `routes/api.php`
- controller methods
- Symfony route attributes or YAML/XML route config
- middleware assignments
- custom router definitions
- AJAX endpoints
- admin panels
- API endpoints
- legacy PHP files acting as entry points

Source questions:
- Which routes accept external identifiers such as `id`, `user_id`, `order_id`, `tenant_id`, `org_id`, `file_id`?
- Which routes expose sensitive actions such as export, delete, approve, publish, reset, or disable?
- Are equivalent capabilities exposed through multiple route files or HTTP methods?

## 1.2 Request input sources
Look for:
- route parameters
- `$request->input(...)`
- `$request->query(...)`
- `$request->post(...)`
- `$request->get(...)`
- `$request->header(...)`
- `$_GET`
- `$_POST`
- `$_REQUEST`
- `$_COOKIE`
- decoded JSON body fields
- GraphQL or RPC arguments

Source questions:
- Is the value client-controlled?
- Does the value represent identity, role, permission, object ID, tenant scope, or business action?
- Is the value later used in a model, service, policy, query, or workflow call?

## 1.3 Authentication identity sources
Look for:
- `auth()->user()`
- `Auth::user()`
- `$request->user()`
- session values
- guard-specific user objects
- Symfony security user
- JWT claim parsing
- custom current-user helpers

Source questions:
- Is the identity derived from a verified authentication context?
- Is any user ID taken from request input instead of the authenticated user?
- Is tenant or organization context derived from authenticated membership?

## 1.4 Function authorization sources
Look for:
- Laravel policies and gates
- `$this->authorize(...)`
- `Gate::allows(...)`
- middleware such as `can:...`
- role or permission package checks
- Symfony voters or access controls
- manual `isAdmin()`, `hasRole()`, `can()`, or `hasPermission()` checks

Source questions:
- Where do role and permission values come from?
- Is authorization based on server-side user context or request-supplied role/scope?
- Are policy inputs also source points, such as object ID, tenant ID, and action name?

## 1.5 Object and tenant source points
Look for:
- Eloquent model lookups
- route model binding
- query builder filters
- repository/service arguments
- raw SQL parameters
- helper functions loading objects
- tenant middleware or tenant resolver classes

Source questions:
- Which identifier selects the protected object?
- Is tenant/org scope passed from request or trusted context?
- Are batch ID lists checked per object downstream?

## 1.6 Business action and state sources
Look for:
- action names in route paths
- request fields such as `action`, `status`, `state`, `role`, `scope`
- service calls for approval, publish, cancel, void, refund, delete, disable, reset, lock, or unlock
- workflow transition helpers

Source questions:
- Is the action selected by the route, request body, enum, or service method?
- Is state loaded from database or supplied by client?
- Does the action combine object ID, current user, role, tenant, and state?

---

# 2. PHP Source Patterns and Blind Spots

These are high-priority source signals. They are not automatic proof of a vulnerability.

## 2.1 Authentication Identity Source Patterns

### S1. Trusted authenticated user source
```php
public function profile(Request $request)
{
    $userId = $request->user()->id;
    return $this->profileService->getProfile($userId);
}
```

Why relevant:
The current identity is sourced from framework authentication context and flows into a protected profile lookup.

What to record:
- `$request->user()` as server-trusted identity source if authentication is enforced
- derived `$userId`
- downstream service call using that identity

### S2. User ID from request input
```php
public function profile(Request $request)
{
    $userId = $request->input('user_id');
    return $this->profileService->getProfile($userId);
}
```

Why relevant:
`user_id` is client-controlled and influences account profile access.

What to verify next:
- whether service binds the request to the authenticated user
- whether this is intended admin behavior
- whether role or ownership checks exist downstream

### S3. Raw session identity
```php
$userId = $_SESSION['user_id'] ?? null;
```

Why relevant:
The session value may be a server-trusted identity source if session handling is active and protected.

What to verify next:
- whether session initialization is required before sensitive use
- whether the value can be set by client input
- whether authentication middleware covers the entry point

---

## 2.2 Function Authorization Source Patterns

### S4. Laravel policy source
```php
public function destroy(Project $project)
{
    $this->authorize('delete', $project);
    $project->delete();
}
```

Why relevant:
The function authorization source is the policy check, while `$project` is an object source through route model binding.

What to record:
- policy action `delete`
- object model source
- privileged function reached after policy check

### S5. Request-supplied role value
```php
public function export(Request $request)
{
    return $this->reportService->exportForRole($request->input('role'));
}
```

Why relevant:
`role` is client-controlled and may influence privileged export behavior.

What to verify next:
- whether `role` is only a filter label or a real authorization input
- whether server-side permission checks override it
- whether ordinary users can request stronger scopes

### S6. Permission helper with mixed inputs
```php
if ($this->permissionService->canAccess($userId, $projectId, $action)) {
    return $this->projectService->runAction($projectId, $action);
}
```

Why relevant:
The policy result depends on multiple sources: identity, object ID, and action.

What to record:
- each input origin
- whether `$userId` is trusted
- whether `$projectId` and `$action` are client-controlled

---

## 2.3 Object and Tenant Source Patterns

### S7. Route parameter object ID
```php
public function show($id)
{
    return Order::findOrFail($id);
}
```

Why relevant:
`$id` is a client-controlled object identifier used to retrieve a protected order.

What to verify next:
- whether the query scopes by current user or tenant
- whether the order is public
- whether object authorization occurs after lookup

### S8. Client-supplied tenant ID
```php
public function show(Request $request, $id)
{
    return Project::where('id', $id)
        ->where('tenant_id', $request->input('tenant_id'))
        ->firstOrFail();
}
```

Why relevant:
`tenant_id` is client-controlled but used as tenant scope.

What to verify next:
- whether tenant membership is verified
- whether tenant scope should come from authenticated context
- whether the query is the real enforcement point

### S9. Batch ID source
```php
public function deleteFiles(Request $request)
{
    $this->fileService->deleteAll($request->input('file_ids', []));
    return response()->json(['ok' => true]);
}
```

Why relevant:
`file_ids` is a client-controlled batch source for protected delete operations.

What to verify next:
- whether each file ID is checked for ownership or tenant scope
- whether partial failures are handled safely
- whether batch logic bypasses per-object policy checks

---

## 2.4 Business Action and State Source Patterns

### S10. Action selected by route
```php
public function voidInvoice($id)
{
    $this->invoiceService->voidInvoice($id);
    return response()->json(['ok' => true]);
}
```

Why relevant:
The route/controller action supplies the business action `void`, and `$id` selects the protected invoice.

What to verify next:
- whether current user has finance/admin permission
- whether invoice state allows voiding
- whether tenant or owner scope is enforced

### S11. Status supplied by request body
```php
public function updatePost(Request $request, $id)
{
    $this->postService->updateStatus($id, $request->input('status'));
    return response()->json(['ok' => true]);
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

## Case H-S-1: Identity Source

### Source pattern
```php
$userId = auth()->id();
```

### Why relevant
The code derives current identity from framework authentication context.

### What to record
- source type: identity
- trust boundary: server-trusted if authentication setup is active
- downstream protected object or action using `$userId`

### Follow-up
- confirm the route is covered by authentication
- confirm the identity cannot be supplied by the client

---

## Case H-S-2: Object Identifier Source

### Source pattern
```php
public function show($orderId)
{
    return Order::findOrFail($orderId);
}
```

### Why relevant
`$orderId` is client-controlled and selects a protected object.

### What to record
- source type: object-id
- trust boundary: client-controlled
- downstream use: model lookup

### Follow-up
- verify ownership or tenant scoping in controller, service, model, policy, or query layer

---

## Case H-S-3: Tenant Scope Source

### Source pattern
```php
return $this->reportService->export($request->input('tenant_id'), $request->input('type'));
```

### Why relevant
`tenant_id` may determine cross-tenant data access.

### What to record
- whether tenant scope is from request or trusted context
- downstream export or query use
- any membership validation inputs

### Follow-up
- verify tenant scope is not raw client authority

---

## Case H-S-4: Function Authorization Source

### Source pattern
```php
Gate::allows('delete-project', [$projectId, $action]);
```

### Why relevant
The authorization decision may depend on trusted identity, client-controlled object ID, and action.

### What to record
- identity source used by gate
- object source
- action source
- policy or gate location

### Follow-up
- inspect the gate/policy if access-control validation is in scope

---

## Case H-S-5: Business State Source

### Source pattern
```php
$this->workflowService->transition($orderId, $request->input('target_state'));
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

# 4. PHP-Specific Audit Heuristics

## 4.1 Laravel heuristics
Pay attention to:
- route definitions in `routes/web.php` and `routes/api.php`
- route groups and middleware stacks
- route model binding
- request validation classes
- policy and gate calls
- Eloquent query scopes

Audit questions:
- Which request fields become authorization-relevant sources?
- Are route-bound models scoped before binding or authorized after binding?
- Are sensitive actions exposed through both web and API routes?

## 4.2 Symfony heuristics
Pay attention to:
- controller route attributes
- firewall and access control configuration
- voters
- `$this->getUser()`
- request attributes
- ParamConverter or entity resolution

Audit questions:
- Which request attributes become object identifiers?
- Are voters passed client-controlled object IDs or resolved entities?
- Is tenant context derived from authenticated membership?

## 4.3 Raw PHP / custom MVC heuristics
Pay attention to:
- files directly under web root
- manual includes and router switches
- `$_GET`, `$_POST`, `$_REQUEST`, `$_COOKIE`
- manual session checks
- helper functions that load current user or tenant

Audit questions:
- Does the file act as an entry point?
- Are request values passed directly into database or service operations?
- Is authentication context initialized before source use?

## 4.4 ORM / Query heuristics
Pay attention to:
- `find(...)`
- `findOrFail(...)`
- `where(...)`
- query scopes
- repository methods
- raw SQL and bound parameters

Audit questions:
- Which method argument is the object source?
- Which method argument is identity or tenant scope?
- Are query filters sourced from trusted context or request input?

## 4.5 Layer inconsistency heuristics
Pay attention to:
- controller passes client source directly to service
- service passes source directly to model/query
- one route derives user from `auth()` while another accepts `user_id`
- list route scopes by tenant but export route accepts tenant input

Audit questions:
- Are equivalent capabilities using different source origins?
- Does a weaker alternate path exist?

---

# 5. False-Positive Controls

Do not mark a PHP source as high-priority if:
- the value does not reach access-control-relevant logic,
- the endpoint is intentionally public and unrelated to protected resources,
- the request value is overwritten by trusted identity or tenant context before sensitive use,
- the route is not reachable or is test-only outside production routing,
- the source is only used for pagination, sorting, or display formatting.

Use `Suspected source` or `Not enough evidence` if:
- the value appears in a request object but downstream use is unclear,
- the active middleware stack is not visible,
- the service or model implementation is missing,
- route model binding scope is unclear,
- custom helpers may rewrite identity or tenant context.

---

# 6. What Good Evidence Looks Like

Good PHP source evidence includes:
- route definition and controller method,
- request input API or route parameter,
- model binding or DTO/form request field when relevant,
- authentication context access when relevant,
- service, model, query, policy, or gate call receiving the source,
- query or workflow call showing access-control relevance.

Good source evidence answers:
1. Which PHP entry point receives the value?
2. Is the value client-controlled or server-trusted?
3. Which protected function, object, tenant, or workflow does it influence?
4. What follow-up access-control check is needed?

---

# 7. Quick PHP Source Checklist

- Are object IDs read from route parameters or request input?
- Are `user_id`, `tenant_id`, `org_id`, or `role` accepted from request input?
- Is current user read from `auth()->user()`, `Auth::user()`, or `$request->user()`?
- Are policy or gate helpers passed both trusted and client-controlled values?
- Are Eloquent queries scoped by trusted user or tenant values?
- Are batch ID arrays used for delete, export, approve, or share actions?
- Are workflow states supplied by the client?
- Are equivalent web, API, AJAX, GraphQL, or legacy paths using different source origins?
