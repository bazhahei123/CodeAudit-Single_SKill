# Python Access Control Cases

## Purpose

This file contains Python-specific access control patterns, anti-patterns, and audit cases.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Django REST Framework
- Flask
- FastAPI
- Starlette
- Tornado
- Python backends exposing REST APIs, admin panels, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real control path in the target code.

---

# 1. Python Authorization Control Points

When auditing Python applications, prioritize these control points.

## 1.1 Route and entry-point controls
Look for:
- Django `views.py`
- DRF `APIView`, `ViewSet`, `GenericViewSet`
- Flask `@app.route`
- FastAPI `@app.get`, `@app.post`, `APIRouter`
- GraphQL resolvers
- RPC-style handlers
- async task entry points triggered by user input

Questions:
- Which routes expose sensitive resources or actions?
- Which entry points accept external identifiers such as `id`, `user_id`, `order_id`, `tenant_id`, `org_id`, `file_id`?
- Are admin or internal routes clearly separated from public/user routes?

## 1.2 Authentication enforcement points
Look for:
- Django authentication middleware
- DRF authentication classes
- DRF permission classes
- Flask login/session/token decorators
- FastAPI dependency-based auth
- middleware validating JWT/session/API key
- custom authentication helpers
- route groups with shared auth dependencies

Questions:
- Is authentication enforced for this route?
- Is the route accidentally exposed as public?
- Are alternate route groups protected consistently?

## 1.3 Function-level authorization controls
Look for:
- DRF permission classes
- Django decorators such as `@login_required`, `@permission_required`
- custom role checks in views/services
- Flask decorators for admin/operator checks
- FastAPI `Depends(...)` enforcing roles/permissions
- policy service calls such as `can_xxx(...)`, `has_perm(...)`, `is_admin(...)`

Questions:
- Is a privileged action restricted to the correct role or permission scope?
- Is protection implemented only in some actions but not others?
- Is there a weaker alternate route or alternate HTTP method?

## 1.4 Object-level authorization controls
Look for:
- queryset scoping by current user
- queryset scoping by tenant/org
- explicit ownership checks after object lookup
- DRF `get_queryset()` / `get_object()` restrictions
- service-layer policy checks after loading an object
- access-control checks tied to `request.user`, trusted token claims, or server-side tenant context

Examples of stronger patterns:
- `Model.objects.get(id=id, user=request.user)`
- queryset filtered by tenant from trusted context
- explicit `can_access(user, obj)` checks
- service methods verifying ownership before returning/updating data

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
- Django ORM filters
- DRF queryset definitions
- raw SQL
- repository/service methods
- helper functions loading objects
- SQLAlchemy-style filters in hybrid codebases

Questions:
- Is the query scoped by `user_id`, `owner_id`, `tenant_id`, or `org_id`?
- Is scope enforced at query time or only assumed elsewhere?
- Is client-supplied `tenant_id` trusted directly?

---

# 2. Python Access Control Anti-Patterns

These are high-risk signals. They are not automatic proof of a vulnerability, but they should trigger deeper review.

## 2.1 Authentication Boundary Anti-Patterns

### A1. Sensitive route missing login/auth enforcement
```python
@app.get("/admin/export")
def export_all():
    return report_service.export_all()
```

Why risky:
A clearly sensitive route is exposed without visible authentication.

What to verify:
- Is auth enforced globally in middleware?
- Is there a router-level dependency or decorator?
- Does this expose admin panels, exports, debug tools, or private APIs?

### A2. DRF view with weak or missing permissions
```python
class AccountView(APIView):
    def get(self, request):
        return Response(profile_service.get_current_profile())
```

Why risky:
If authentication/permission classes are absent or too weak, the route may allow anonymous access.

What to verify:
- Does the class use `permission_classes`?
- Is authentication enforced globally?
- Are there overrides that weaken default protections?

### A3. Download or export endpoint with no visible identity requirement
```python
@router.get("/export/all")
def export_all():
    return report_service.export_all()
```

Why risky:
Exports often expose large volumes of sensitive data and should normally be authenticated and authorized.

What to verify:
- Is authentication enforced before this route?
- Is there role or permission gating for export behavior?

---

## 2.2 Function-Level Authorization Anti-Patterns

### A4. Admin action without role or permission check
```python
@app.post("/admin/user/<int:user_id>/disable")
def disable_user(user_id):
    user_service.disable(user_id)
    return {"ok": True}
```

Why risky:
A privileged administrative operation is exposed without visible server-side authorization.

What to verify:
- Is there decorator- or middleware-based authorization elsewhere?
- Is `user_service.disable(...)` protected internally?
- Can ordinary authenticated users reach this route?

### A5. UI hides action but backend route remains callable
```python
@router.post("/admin/reset-password")
def reset_password(user_id: int):
    admin_service.reset_password(user_id)
    return {"ok": True}
```

Why risky:
If the frontend hides the button but the route itself lacks server-side enforcement, direct API calls may bypass UI restrictions.

What to verify:
- Is there a backend permission check?
- Is access controlled only in the frontend?

### A6. Only some HTTP methods are protected
```python
@router.get("/admin/users")
@require_admin
def list_users():
    ...

@router.post("/admin/users")
def create_user():
    ...
```

Why risky:
Protection may exist for one method but not another method on the same logical resource.

What to verify:
- Are GET/POST/PUT/DELETE/PATCH consistently protected?
- Does alternate method access create a privilege bypass?

---

## 2.3 Object-Level Authorization Anti-Patterns

### A7. Direct object fetch from user-controlled ID
```python
@router.get("/orders/{order_id}")
def get_order(order_id: int):
    return Order.objects.get(id=order_id)
```

Why risky:
The object is fetched directly from a user-controlled identifier with no visible ownership or tenant validation.

What to verify:
- Is the order bound to the current user?
- Is there a later service-layer permission check?
- Would a filtered query by user/tenant be the expected pattern here?

### A8. Update/delete by raw object ID only
```python
@router.delete("/files/{file_id}")
def delete_file(file_id: int):
    file_service.delete_by_id(file_id)
    return {"ok": True}
```

Why risky:
Deletion based only on object ID is a common object-level authorization failure pattern.

What to verify:
- Does `delete_by_id` internally verify ownership or tenant scope?
- Can users delete another user's file by changing the ID?

### A9. Fetch first, trust later
```python
def get_document(request, doc_id):
    doc = Document.objects.get(id=doc_id)
    return JsonResponse({"title": doc.title, "content": doc.content})
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
```python
@router.get("/projects/{project_id}")
def get_project(project_id: int, tenant_id: int):
    return project_repo.get_by_id_and_tenant(project_id, tenant_id)
```

Why risky:
Tenant scope is derived from user input instead of trusted server-side identity.

What to verify:
- Can the requester swap `tenant_id`?
- Should tenant scope come from session/JWT/server context instead?

### A11. Query without tenant/org scoping
```python
def get_invoice(invoice_id: int):
    return Invoice.objects.get(id=invoice_id)
```

Why risky:
In multi-tenant systems, object fetch by raw ID alone may cross tenant boundaries.

What to verify:
- Is tenant boundary enforced earlier?
- Should the ORM query include tenant/org filters?

---

## 2.5 Business-Context Authorization Anti-Patterns

### A12. Sensitive action ignores object state
```python
@router.post("/orders/{order_id}/cancel")
def cancel_order(order_id: int):
    order_service.cancel(order_id)
    return {"ok": True}
```

Why risky:
Cancellation may require not only role/ownership checks but also state validation, such as "not shipped" or "not locked".

What to verify:
- Does `cancel(...)` validate order state?
- Can users cancel already-finalized objects?
- Is workflow-stage enforcement missing?

### A13. Approval/publish/disable actions gated only by login
```python
@router.post("/post/{post_id}/publish")
@login_required
def publish_post(post_id: int):
    post_service.publish(post_id)
    return {"ok": True}
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

## Case P-1: Unauthenticated Access

### Vulnerable pattern
```python
@app.get("/admin/export")
def export_all():
    return report_service.export_all()
```

### Why risky
The route appears to expose a sensitive export function without visible authentication or authorization enforcement.

### What to verify
- Is the route behind authenticated security configuration?
- Is the route protected elsewhere by middleware or dependency injection?
- Is the export action intended for admins only?

### Safer signals
- route explicitly requires authentication
- route is restricted to admin/operator roles
- export action is gated by policy or permission service

---

## Case P-2: Function-Level Authorization

### Vulnerable pattern
```python
@app.post("/admin/user/<int:user_id>/disable")
def disable_user(user_id):
    user_service.disable(user_id)
    return {"ok": True}
```

### Why risky
A privileged management function is exposed without visible role or permission checks.

### What to verify
- Is admin permission enforced at route, class, or method level?
- Is there a service-layer guard?
- Is this path reachable by ordinary authenticated users?

### Safer signals
- explicit role/permission decorator
- centralized policy check
- route and service both treat this as privileged behavior

---

## Case P-3: Object-Level Authorization / IDOR

### Vulnerable pattern
```python
@router.get("/orders/{order_id}")
def get_order(order_id: int):
    return Order.objects.get(id=order_id)
```

### Why risky
The object is loaded directly from a user-controlled ID without visible ownership or tenant validation.

### What to verify
- Is the order restricted to the current user?
- Is there a policy check after the fetch?
- Is the ORM query expected to include user/tenant constraints?

### Safer pattern
```python
@router.get("/orders/{order_id}")
def get_order(order_id: int, request: Request):
    return Order.objects.get(id=order_id, user=request.user)
```

---

## Case P-4: Tenant Isolation Failure

### Vulnerable pattern
```python
@router.get("/projects/{project_id}")
def get_project(project_id: int, tenant_id: int):
    return project_repo.get_by_id_and_tenant(project_id, tenant_id)
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

## Case P-5: Business-Context Authorization Failure

### Vulnerable pattern
```python
@router.post("/invoice/{invoice_id}/void")
def void_invoice(invoice_id: int):
    invoice_service.void_invoice(invoice_id)
    return {"ok": True}
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

# 4. Python-Specific Audit Heuristics

## 4.1 Django / DRF heuristics
Pay attention to:
- `permission_classes`
- `authentication_classes`
- `get_queryset()`
- `get_object()`
- `perform_update()`
- `perform_destroy()`
- Django decorators such as `@login_required`, `@permission_required`
- custom `has_permission` / `has_object_permission`

Audit questions:
- Is the view relying only on authentication but not object permission?
- Is the queryset filtered by the current user or tenant?
- Are object permissions enforced consistently for retrieve/update/delete?

## 4.2 Flask heuristics
Pay attention to:
- route decorators
- custom auth decorators
- `g.user`, `session`, token parsing
- blueprint-level protection
- service methods using request parameters instead of trusted identity

High-risk pattern:
```python
@app.get("/profile")
def get_profile():
    user_id = request.args.get("user_id")
    return profile_service.get_by_user_id(user_id)
```

Why risky:
The server trusts an externally supplied user ID instead of binding access to authenticated identity.

## 4.3 FastAPI heuristics
Pay attention to:
- `Depends(...)`
- shared router dependencies
- dependency injection for current user / tenant
- whether sensitive routes omit required dependencies
- whether object queries are scoped using trusted dependency output

Preferred safer patterns:
- `current_user = Depends(get_current_user)`
- tenant context from trusted dependency
- object query constrained by current user or tenant
- policy check after fetch

## 4.4 ORM / Query heuristics
Pay attention to:
- `.get(id=...)`
- `.filter(id=...)` without user/tenant scope
- raw SQL or repository helpers
- service methods that load an entity and immediately return it
- updates/deletes executed after raw object fetch

High-risk pattern:
```python
order = Order.objects.get(id=order_id)
```

Why risky:
If the object should be user- or tenant-scoped, raw ID lookup may enable object-level authorization bypass.

## 4.5 Layer inconsistency heuristics
Check whether access control is consistent across:
- route vs service
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
- the presence of `.get(id=...)` alone,
- the absence of decorators alone.

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

# 7. Quick Python Audit Checklist

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
