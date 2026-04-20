# Access Control Common Cases

## Purpose

This file defines shared access control concepts, audit logic, anti-patterns, false-positive controls, and finding standards that apply across languages and frameworks.

Use this file as the base reference for access control review before loading any language-specific reference.

This file explains:
- what authentication and authorization mean,
- how to distinguish function-level and object-level authorization issues,
- how to reason about horizontal and vertical privilege escalation,
- how business-context restrictions affect authorization,
- when to report `Confirmed`, `Suspected`, or `Not enough evidence`.

This reference is guidance, not proof. Do not report a vulnerability only because code resembles a pattern described here. Always verify the real control path in the target code.

---

# 1. Core Concepts

## 1.1 Authentication vs authorization

### Authentication
Authentication answers:
**Who is the user?**

Examples:
- session validation
- token validation
- login state
- identity extraction from trusted server-side context

### Authorization
Authorization answers:
**What is this authenticated user allowed to do?**

Examples:
- role checks
- permission checks
- ownership checks
- tenant scoping
- workflow / state restrictions

A system can be correctly authenticated but still vulnerable if authorization is weak or missing.

## 1.2 Protected resource, protected function, protected object

### Protected resource
A sensitive route, page, API, file, or action that should not be freely reachable.

### Protected function
A privileged capability such as:
- create user
- delete record
- export data
- publish content
- reset password
- approve transaction

### Protected object
A specific record or resource instance such as:
- one order
- one invoice
- one ticket
- one file
- one user profile
- one project

Authorization bugs often happen when the system protects the route in general, but not the **specific object** being accessed.

## 1.3 Function-level vs object-level authorization

### Function-level authorization
Checks whether the user may access a capability at all.

Examples:
- only admins can disable users
- only finance can void invoices
- only support staff can access support admin pages

### Object-level authorization
Checks whether the user may access this specific object.

Examples:
- user A should not read user B's order
- tenant A should not read tenant B's project
- one employee should not download another employee's payroll file

Function-level and object-level authorization are separate. Passing one does not prove the other.

---

# 2. Shared Access Control Categories

## 2.1 Authentication boundary failure
Sensitive resources or actions are reachable without a valid authenticated identity.

Examples:
- no-login access to admin pages
- unauthenticated file download
- export endpoint reachable anonymously

## 2.2 Function-level authorization failure
An authenticated user can invoke functions reserved for stronger roles, permissions, or scopes.

Examples:
- normal user can call admin reset-password route
- ordinary staff can access finance-only actions
- hidden admin feature is callable directly by URL or API

## 2.3 Object-level authorization failure
A user can access, modify, delete, or export objects they do not own or should not control.

Examples:
- changing `/orders/101` to `/orders/102`
- changing `file_id`, `invoice_id`, or `user_id`
- reading another user's resource through a shared API

## 2.4 Horizontal privilege escalation
A user accesses another peer user's resources.

Examples:
- user A reads user B's records
- one employee accesses another employee's ticket
- one customer downloads another customer's invoice

This is often a subtype of object-level authorization failure.

## 2.5 Vertical privilege escalation
A lower-privileged user accesses higher-privileged features.

Examples:
- normal user reaches admin functionality
- support user performs finance-only action
- operator reaches super-admin behavior

This is often a subtype of function-level authorization failure.

## 2.6 Tenant or organization boundary failure
A user crosses organizational scope.

Examples:
- tenant A accesses tenant B's project
- org-scoped route trusts client-supplied `tenant_id`
- query lacks mandatory org scoping

This is often a subtype of object-level authorization failure, but it is important enough to review separately.

## 2.7 Business-context authorization failure
Identity and role may be correct, but action should still be denied due to business state or workflow stage.

Examples:
- already finalized invoice can still be voided
- already published content can still be modified improperly
- archived record can still be deleted
- approved object can still be re-approved or reverted without authorization

## 2.8 Client-side-only authorization
The UI hides or disables actions, but the server does not enforce the restriction.

Examples:
- hidden button but callable backend route
- frontend route guard only
- mobile/web client blocks action but backend accepts direct request

---

# 3. Shared Attack Surfaces

Prioritize these attack surfaces first:

- admin and management endpoints
- export and download functionality
- account settings and identity operations
- billing, finance, approval, publish, delete, disable, and reset actions
- object fetch / update / delete handlers
- file and attachment access
- tenant- or org-scoped resources
- internal, legacy, alternate, or debug routes
- alternate HTTP methods for the same logical feature
- background jobs or indirect actions triggered by user input

---

# 4. Shared Anti-Patterns

These are common danger signals across languages.

## A1. Authentication without authorization
A route requires login, but no role, permission, ownership, or tenant check exists.

Why risky:
Being authenticated does not mean the user may perform the action or access the object.

## A2. Route protection only on some methods
Example idea:
- GET is protected
- POST / DELETE / PATCH is not

Why risky:
An alternate method may bypass the intended restriction.

## A3. Object lookup by raw external identifier
Example idea:
- fetch by `id`
- delete by `file_id`
- update by `order_id`

Why risky:
If the object is looked up directly from attacker-controlled input with no ownership or tenant constraint, object-level authorization may fail.

## A4. Client-supplied tenant or organization scope
Example idea:
- `tenant_id` comes from query parameter or request body
- server trusts it directly

Why risky:
The requester may switch scope and cross boundaries.

## A5. Frontend-only restrictions
Example idea:
- button hidden
- menu hidden
- route hidden in UI
- page guarded in JavaScript only

Why risky:
The backend may still accept direct calls.

## A6. Inconsistent enforcement across layers
Example idea:
- controller checks permission but service does not
- one route checks ownership, another helper route does not
- list route is scoped, export route is not

Why risky:
Attackers will use the weakest equivalent path.

## A7. Business action ignores object state
Example idea:
- cancel, approve, publish, void, archive, or reset action ignores workflow stage

Why risky:
Authorization depends not only on identity, but also on valid state transition rules.

## A8. Implicit trust in framework defaults
Example idea:
- assuming middleware covers all routes
- assuming ORM scoping exists automatically
- assuming annotation presence proves all paths are protected

Why risky:
One alternate path or one unprotected helper may bypass the intended control.

---

# 5. Shared Protection Model

## 5.1 What strong protection usually looks like

Strong protections typically include:
- server-side authentication enforcement
- explicit role or permission checks for privileged functions
- object-level ownership checks
- tenant or organization scoping from trusted server-side context
- policy checks after object fetch when needed
- business-state validation for workflow-sensitive actions
- consistent enforcement across routes, methods, and layers

## 5.2 What does not automatically mean the code is safe

The following do **not** automatically mean the code is safe:
- route requires login
- user is authenticated
- frontend hides the button
- one controller method has a permission annotation
- one code path checks ownership
- tenant ID exists in the request
- ORM or framework is being used
- route name looks internal or admin-only

## 5.3 Object authorization often needs both identity and scope
For many sensitive objects, correct access depends on:
- current authenticated user
- role / permission
- ownership
- tenant / org scope
- business state

One factor alone is often insufficient.

---

# 6. False-Positive Controls

Do not report a vulnerability as `Confirmed` if:
- the resource is intentionally public,
- the object is public by design,
- the missing control is clearly enforced upstream,
- the route is a non-sensitive health or diagnostic endpoint,
- the visible code is incomplete but strong evidence shows the control exists elsewhere,
- the apparent object lookup is followed by a real policy/ownership/tenant check before use.

Use `Suspected` or `Not enough evidence` if:
- the route looks dangerous but the real enforcement point may be hidden in middleware, service layer, policy layer, or gateway logic,
- the object lookup is visible but downstream authorization is not,
- the input is visible but the final protected action is abstracted away,
- the framework configuration may enforce the boundary but the active rule is unclear.

Do not over-claim based only on:
- route naming
- controller naming
- login requirement alone
- object fetch by ID alone
- missing annotation alone
- hidden frontend button alone

---

# 7. Finding Classification

## Confirmed
Use `Confirmed` when there is clear evidence that:
- the protected function or object is reachable,
- the expected boundary is absent, weak, or inconsistent,
- and exploitation is plausibly possible.

## Suspected
Use `Suspected` when:
- there is a strong dangerous pattern,
- the target appears reachable,
- but hidden enforcement may still exist elsewhere.

## Not enough evidence
Use `Not enough evidence` when:
- critical parts of the enforcement path are not visible,
- the sink action is abstracted,
- or framework / middleware behavior cannot be verified.

## Probably safe
Use `Probably safe` when:
- the path is visible,
- the relevant authentication / authorization controls are clearly present,
- and no weaker equivalent route or object path is evident.

---

# 8. What Good Evidence Looks Like

Strong access control findings usually include:
- the exact entry point
- the protected target
- the expected boundary
- the actual boundary found
- the object ID / tenant ID / role input path if relevant
- the reason the user can cross the intended restriction

Good findings usually answer:
1. What can the attacker reach or do?
2. What boundary should stop them?
3. What code shows that the boundary is absent, weak, or inconsistent?
4. Why is exploitation plausible?

---

# 9. Shared Remediation Guidance

Preferred fixes include:
- enforce authentication server-side on protected routes
- enforce role / permission checks on privileged functions
- bind object access to current user, tenant, or policy result
- derive tenant / org scope from trusted server-side identity, not request input
- apply the same control across all equivalent routes and methods
- enforce workflow / state validation for sensitive business actions
- treat frontend restrictions only as UX, never as security

Avoid weak fixes such as:
- hiding UI elements only
- checking only one HTTP method
- checking only role but not object ownership
- trusting client-provided scope values
- assuming framework defaults cover every path

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Is the route properly authenticated?
- Is the function restricted to the correct role or permission scope?
- Is the object restricted to the correct user or tenant?
- Can a peer user access another peer user's object?
- Can a lower-privileged user reach higher-privileged functionality?
- Is tenant or org scope derived from trusted server-side identity?
- Are all equivalent routes and methods protected consistently?
- Does the action also require valid business state or workflow stage?
- Is the restriction enforced on the server, not only in the client?
