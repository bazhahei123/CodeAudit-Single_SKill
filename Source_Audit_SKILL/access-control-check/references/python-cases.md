# Python Access Control Source Cases

## Purpose

This file contains Python-specific source point patterns and audit cases for access-control source discovery.

Use it when the target application is primarily implemented in Python, especially in:
- Django
- Django REST Framework
- Flask
- FastAPI
- Starlette
- Tornado
- Python backends exposing REST APIs, admin panels, GraphQL, RPC, or tenant-scoped business systems

This reference is guidance, not proof. Do not report a vulnerability only because a code pattern resembles one of the cases below. Always verify the real source origin and downstream use in the target code.

---

# 1. Python Source Discovery Points

When auditing Python applications, prioritize these source points.

## 1.1 Route and entry-point sources
Look for:
- Django `views.py`
- Django URL patterns
- DRF `APIView`, `ViewSet`, `GenericViewSet`
- Flask `@app.route` and blueprints
- FastAPI `@app.get`, `@app.post`, and `APIRouter`
- GraphQL resolvers
- RPC-style handlers
- async task entry points triggered by user input

Source questions:
- Which routes accept external identifiers such as `id`, `user_id`, `order_id`, `tenant_id`, `org_id`, `file_id`?
- Which routes expose sensitive actions such as export, delete, approve, publish, reset, or disable?
- Are equivalent capabilities exposed through multiple viewsets, routers, or HTTP methods?

## 1.2 Request input sources
Look for:
- path parameters
- `request.GET`
- `request.POST`
- `request.data`
- `request.query_params`
- `request.json`
- `request.args`
- `request.form`
- headers and cookies
- Pydantic request models
- GraphQL arguments
- RPC arguments

Source questions:
- Is the value client-controlled?
- Does the value represent identity, role, permission, object ID, tenant scope, or business action?
- Is the value later used in a model, service, permission, query, or workflow call?

## 1.3 Authentication identity sources
Look for:
- Django `request.user`
- DRF `request.user`
- Flask-Login `current_user`
- FastAPI dependencies returning current user
- session values
- JWT claim parsing
- middleware setting request state
- custom current-user helpers

Source questions:
- Is the identity derived from a verified authentication context?
- Is any user ID taken from request input instead of the authenticated user?
- Is tenant or organization context derived from authenticated membership?

## 1.4 Function authorization sources
Look for:
- DRF permission classes
- Django decorators such as `@login_required` and `@permission_required`
- custom role checks in views or services
- Flask decorators for admin/operator checks
- FastAPI `Depends(...)` enforcing roles or permissions
- policy service calls such as `can_xxx(...)`, `has_perm(...)`, `is_admin(...)`

Source questions:
- Where do role and permission values come from?
- Is authorization based on server-side user context or request-supplied role/scope?
- Are policy inputs also source points, such as object ID, tenant ID, and action name?

## 1.5 Object and tenant source points
Look for:
- Django ORM lookups
- DRF `get_queryset()` and `get_object()`
- SQLAlchemy filters
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
- DRF `@action` methods
- request fields such as `action`, `status`, `state`, `role`, `scope`
- service calls for approval, publish, cancel, void, refund, delete, disable, reset, lock, or unlock
- workflow transition helpers

Source questions:
- Is the action selected by the route, request body, enum, or service method?
- Is state loaded from database or supplied by client?
- Does the action combine object ID, current user, role, tenant, and state?

---

# 2. Python Source Patterns and Blind Spots

These are high-priority source signals. They are not automatic proof of a vulnerability.

## 2.1 Authentication Identity Source Patterns

### S1. Trusted authenticated user source
```python
def profile(request):
    user_id = request.user.id
    return profile_service.get_profile(user_id)
```

Why relevant:
The current identity is sourced from framework authentication context and flows into a protected profile lookup.

What to record:
- `request.user` as server-trusted identity source if authentication is enforced
- derived `user_id`
- downstream service call using that identity

### S2. User ID from request input
```python
def profile(request):
    user_id = request.GET.get("user_id")
    return profile_service.get_profile(user_id)
```

Why relevant:
`user_id` is client-controlled and influences account profile access.

What to verify next:
- whether service binds the request to the authenticated user
- whether this is intended admin behavior
- whether role or ownership checks exist downstream

### S3. FastAPI dependency identity source
```python
@router.get("/account/profile")
def profile(current_user: User = Depends(get_current_user)):
    return profile_service.get_profile(current_user.id)
```

Why relevant:
`current_user` is server-trusted only if `get_current_user` verifies the token/session and rejects invalid credentials.

What to verify next:
- whether the dependency is attached to the route or router
- whether it verifies tokens instead of only decoding them
- whether later code uses `current_user.id` instead of request `user_id`

---

## 2.2 Function Authorization Source Patterns

### S4. DRF permission source
```python
class UserAdminView(APIView):
    permission_classes = [IsAdminUser]

    def post(self, request, user_id):
        user_service.disable(user_id)
        return Response({"ok": True})
```

Why relevant:
The function authorization source is the DRF permission class, while `user_id` is an object source.

What to record:
- permission class
- object identifier from route
- privileged action from method and service call

### S5. Request-supplied role value
```python
@router.get("/reports/export")
def export(role: str):
    return report_service.export_for_role(role)
```

Why relevant:
`role` is client-controlled and may influence privileged export behavior.

What to verify next:
- whether `role` is only a filter label or a real authorization input
- whether server-side permission checks override it
- whether ordinary users can request stronger scopes

### S6. Permission helper with mixed inputs
```python
if permission_service.can_access(user_id, project_id, action):
    return project_service.run_action(project_id, action)
```

Why relevant:
The policy result depends on multiple sources: identity, object ID, and action.

What to record:
- each input origin
- whether `user_id` is trusted
- whether `project_id` and `action` are client-controlled

---

## 2.3 Object and Tenant Source Patterns

### S7. Path parameter object ID
```python
@router.get("/orders/{order_id}")
def get_order(order_id: int):
    return order_service.get_order(order_id)
```

Why relevant:
`order_id` is a client-controlled object identifier used to retrieve a protected order.

What to verify next:
- whether the service scopes by current user or tenant
- whether the order is public
- whether object authorization occurs after lookup

### S8. Client-supplied tenant ID
```python
@router.get("/projects/{project_id}")
def get_project(project_id: int, tenant_id: int):
    return project_repo.get_by_id_and_tenant(project_id, tenant_id)
```

Why relevant:
`tenant_id` is client-controlled but used as tenant scope.

What to verify next:
- whether tenant membership is verified
- whether tenant scope should come from authenticated context
- whether the repository call is the real enforcement point

### S9. Batch ID source
```python
@router.post("/files/delete")
def delete_files(req: DeleteFilesRequest):
    file_service.delete_all(req.file_ids)
    return {"ok": True}
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
```python
@router.post("/invoice/{invoice_id}/void")
def void_invoice(invoice_id: int):
    invoice_service.void_invoice(invoice_id)
    return {"ok": True}
```

Why relevant:
The route itself supplies the business action `void`, and `invoice_id` selects the protected invoice.

What to verify next:
- whether current user has finance/admin permission
- whether invoice state allows voiding
- whether tenant or owner scope is enforced

### S11. Status supplied by request body
```python
@router.patch("/posts/{post_id}")
def update_post(post_id: int, req: UpdatePostRequest):
    post_service.update_status(post_id, req.status)
    return {"ok": True}
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

## Case P-S-1: Identity Source

### Source pattern
```python
user_id = request.user.id
```

### Why relevant
The code derives current identity from framework authentication context.

### What to record
- source type: identity
- trust boundary: server-trusted if authentication setup is active
- downstream protected object or action using `user_id`

### Follow-up
- confirm the route is covered by authentication
- confirm the identity cannot be supplied by the client

---

## Case P-S-2: Object Identifier Source

### Source pattern
```python
def get_order(request, order_id):
    return Order.objects.get(id=order_id)
```

### Why relevant
`order_id` is client-controlled and selects a protected object.

### What to record
- source type: object-id
- trust boundary: client-controlled
- downstream use: model lookup

### Follow-up
- verify ownership or tenant scoping in view, service, queryset, permission, or repository layer

---

## Case P-S-3: Tenant Scope Source

### Source pattern
```python
return report_service.export(request.GET.get("tenant_id"), request.GET.get("type"))
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

## Case P-S-4: Function Authorization Source

### Source pattern
```python
if policy.can(request.user, project_id, "delete"):
    project_service.delete(project_id)
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

## Case P-S-5: Business State Source

### Source pattern
```python
workflow_service.transition(order_id, request.data.get("target_state"))
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

# 4. Python-Specific Audit Heuristics

## 4.1 Django / DRF heuristics
Pay attention to:
- URL patterns and view methods
- `request.user`
- `request.GET`, `request.POST`, and `request.data`
- DRF serializers and validated data
- `permission_classes`
- `get_queryset()` and `get_object()`
- custom object permissions

Audit questions:
- Which request fields become authorization-relevant sources?
- Are object IDs pulled from URL kwargs or request data?
- Does `get_queryset()` use trusted user or tenant context?

## 4.2 Flask heuristics
Pay attention to:
- route decorators and blueprints
- `request.args`, `request.form`, `request.json`
- `current_user`
- custom auth decorators
- service calls from route functions
- SQLAlchemy filters

Audit questions:
- Which request values flow into protected service or query calls?
- Is current identity from Flask-Login or manually supplied input?
- Are sensitive admin routes protected by decorators and server-side checks?

## 4.3 FastAPI heuristics
Pay attention to:
- route function parameters
- Pydantic models
- `Depends(...)` dependencies
- router-level dependencies
- JWT helper functions
- service and repository calls

Audit questions:
- Which parameters are path/query/body sources?
- Which dependencies provide trusted identity or role context?
- Are dependency outputs mixed with client-controlled IDs?

## 4.4 ORM / Query heuristics
Pay attention to:
- Django ORM `get(...)` and `filter(...)`
- DRF queryset definitions
- SQLAlchemy filters
- repository methods
- raw SQL parameters

Audit questions:
- Which method argument is the object source?
- Which method argument is identity or tenant scope?
- Are query filters sourced from trusted context or request input?

## 4.5 Layer inconsistency heuristics
Pay attention to:
- view passes client source directly to service
- service passes source directly to model/query
- one route derives user from `request.user` while another accepts `user_id`
- list route scopes by tenant but export route accepts tenant input

Audit questions:
- Are equivalent capabilities using different source origins?
- Does a weaker alternate path exist?

---

# 5. False-Positive Controls

Do not mark a Python source as high-priority if:
- the value does not reach access-control-relevant logic,
- the endpoint is intentionally public and unrelated to protected resources,
- the request value is overwritten by trusted identity or tenant context before sensitive use,
- the route is not reachable or is test-only outside production routing,
- the source is only used for pagination, sorting, or display formatting.

Use `Suspected source` or `Not enough evidence` if:
- the value appears in a serializer or Pydantic model but downstream use is unclear,
- the active middleware/dependency stack is not visible,
- the service or repository implementation is missing,
- object permission behavior is configured elsewhere,
- custom helpers may rewrite identity or tenant context.

---

# 6. What Good Evidence Looks Like

Good Python source evidence includes:
- URL pattern, route decorator, viewset action, or resolver,
- request input API or function parameter,
- serializer/Pydantic field when relevant,
- authentication context access when relevant,
- service, model, query, policy, or permission call receiving the source,
- query or workflow call showing access-control relevance.

Good source evidence answers:
1. Which Python entry point receives the value?
2. Is the value client-controlled or server-trusted?
3. Which protected function, object, tenant, or workflow does it influence?
4. What follow-up access-control check is needed?

---

# 7. Quick Python Source Checklist

- Are object IDs read from path, query, body, serializer, or Pydantic inputs?
- Are `user_id`, `tenant_id`, `org_id`, or `role` accepted from request input?
- Is current user read from `request.user`, `current_user`, or a FastAPI dependency?
- Are policy or permission helpers passed both trusted and client-controlled values?
- Are ORM queries scoped by trusted user or tenant values?
- Are batch ID arrays used for delete, export, approve, or share actions?
- Are workflow states supplied by the client?
- Are equivalent web, API, GraphQL, RPC, or task paths using different source origins?
