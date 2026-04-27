# Access Control Source Common Cases

## Purpose

This file defines shared access-control source concepts, source classification logic, false-positive controls, and evidence standards that apply across languages and frameworks.

Use this file as the base reference for access-control source review before loading any language-specific reference.

This file explains:
- what a source point is,
- how to distinguish source points from controls and sinks,
- how to classify trust boundaries,
- which source categories matter most for access-control review,
- when to record `Confirmed source`, `Suspected source`, `Not enough evidence`, or `Probably irrelevant`.

This reference is guidance, not proof. Do not report a vulnerability only because code contains a source described here. Always verify the real code path in the target application.

---

# 1. Core Concepts

## 1.1 Source point

A source point is where authorization-relevant data enters or becomes available to the backend code path.

Examples:
- current user identity
- role or permission values
- object identifiers
- tenant or organization identifiers
- requested action names
- workflow status or state values
- route, method, GraphQL, RPC, or job inputs

A source point is an audit starting point, not proof of a vulnerability.

## 1.2 Source, control, and sink

### Source
The origin of a value used in access-control-sensitive code.

Examples:
- `user_id` from a path parameter
- `tenant_id` from a request body
- current user from session
- role from verified authentication context
- action name from route dispatch

### Control
The logic intended to enforce access rules.

Examples:
- authentication middleware
- role checks
- permission checks
- policy or gate checks
- ownership checks
- tenant scoping
- workflow state validation

### Sink
The protected operation or data access reached by the source.

Examples:
- object lookup
- export
- update
- delete
- publish
- approve
- reset password
- cross-tenant query

Source discovery should document enough downstream use to explain why the source matters.

## 1.3 Trust boundary

Classify every source by trust boundary:

### Client-controlled
The value comes from request path, query string, request body, form data, headers, cookies, GraphQL variables, RPC arguments, or uploaded data.

### Server-trusted
The value comes from a verified server-side source, such as authenticated session, validated token claims, framework authentication context, trusted middleware context, database state, or server-side configuration.

### Mixed
The value combines client-controlled and server-trusted data, or a client value is validated against server-side context before use.

### Unclear
The origin cannot be determined from visible code.

---

# 2. Shared Source Categories

## 2.1 Authentication identity sources
Values identifying the requester.

Examples:
- current user object
- user ID from session
- JWT subject claim after verification
- framework principal
- request-scoped user context

## 2.2 Function authorization sources
Values or checks that influence whether a function can be executed.

Examples:
- role
- permission
- admin flag
- group membership
- policy result
- feature entitlement
- operation scope

## 2.3 Object identifier sources
Values identifying a protected record.

Examples:
- `id`
- `user_id`
- `order_id`
- `invoice_id`
- `file_id`
- `project_id`
- batch ID arrays

## 2.4 Tenant or organization scope sources
Values identifying a tenant, organization, company, workspace, department, project, or account boundary.

Examples:
- `tenant_id`
- `org_id`
- `company_id`
- `workspace_id`
- `department_id`
- tenant slug or domain

## 2.5 Business action and state sources
Values identifying requested actions or workflow state.

Examples:
- `approve`
- `publish`
- `cancel`
- `void`
- `refund`
- `archive`
- `status`
- `state`
- `stage`

## 2.6 Alternate entry sources
Equivalent or indirect entry paths that expose the same protected capability.

Examples:
- API route parallel to web route
- mobile route parallel to admin route
- alternate HTTP method
- GraphQL mutation parallel to REST endpoint
- background job trigger
- legacy script
- debug endpoint

---

# 3. Shared Source Surfaces

Prioritize these source surfaces first:

- route path parameters
- query parameters
- request body fields
- form fields
- headers and cookies
- GraphQL variables and resolver arguments
- RPC arguments
- batch operation arrays
- file metadata and import rows
- session and authentication context
- token claims after verification
- tenant context middleware
- policy or permission helper inputs
- repository query arguments
- workflow action names and status fields

---

# 4. Shared Source Patterns

These patterns are common source signals across languages. They are not automatic proof of a vulnerability.

## S1. Current identity source
Example idea:
- current user read from session, request context, principal, or token claims

Audit relevance:
Access-control review often starts by confirming whether later controls use trusted current identity rather than client-supplied identity.

## S2. Client-supplied user identifier
Example idea:
- `user_id` comes from query, body, or path

Audit relevance:
Client-supplied user IDs may become object or identity-confusion sources if used for profile access, account changes, export, or administration.

## S3. Client-supplied object identifier
Example idea:
- `order_id`, `invoice_id`, `file_id`, or `project_id` comes from path or body

Audit relevance:
Object IDs are high-priority source points for object-level authorization review.

## S4. Client-supplied tenant scope
Example idea:
- `tenant_id`, `org_id`, or workspace slug comes from request input

Audit relevance:
Tenant scope should usually come from trusted server-side context or verified membership, not raw request input.

## S5. Role or permission input from request
Example idea:
- `role`, `is_admin`, `permission`, or `scope` comes from request body or query

Audit relevance:
Authorization logic should not trust client-supplied role or permission values without server-side verification.

## S6. Action or status input from request
Example idea:
- `action=approve`, `status=published`, or `state=locked` comes from request input

Audit relevance:
Business actions and state transitions often need role, ownership, tenant, and workflow checks.

## S7. Alternate entry with equivalent capability
Example idea:
- REST and GraphQL expose the same action through different inputs

Audit relevance:
Equivalent entry points may apply different controls and should be traced separately.

## S8. Batch source
Example idea:
- a list of IDs is accepted for bulk delete, export, approve, or share

Audit relevance:
Batch operations can hide per-object authorization gaps if every object is not checked individually.

---

# 5. Shared Source Protection Model

## 5.1 Stronger source handling usually looks like

Stronger source handling typically includes:
- identity derived from verified server-side authentication context
- role and permission derived from server-side policy or database state
- tenant or organization scope derived from trusted context
- object access bound to current user, tenant, or policy result
- client-supplied IDs treated as selectors, not authority
- workflow actions validated against server-side state
- equivalent entry points using the same source interpretation and controls

## 5.2 What does not automatically make a source trusted

The following do not automatically make a source trusted:
- variable is named `currentUser`
- parameter is named `tenantId`
- value is passed through a helper
- value is present in a JWT before verification is shown
- frontend hides the input
- route name looks internal
- framework is used
- route requires login

## 5.3 Source points often combine multiple values

Important access-control review often depends on combinations such as:
- current user + object ID
- current user + tenant ID
- role + object owner
- permission + workflow state
- route action + object state
- batch ID list + tenant scope

Record the combination when the code shows values being used together.

---

# 6. False-Positive Controls

Do not record a source point as high-priority if:
- the value is not connected to a protected resource, function, object, tenant boundary, or business action,
- the value is static server configuration unrelated to access control,
- the route is a public health or static asset endpoint,
- the apparent input is overwritten by trusted server-side context before sensitive use,
- the downstream operation is intentionally public and code evidence supports that.

Use `Suspected source` or `Not enough evidence` if:
- the source origin is visible but downstream use is hidden,
- the downstream sensitive operation is visible but the origin is hidden,
- middleware or framework behavior may rewrite the value,
- the value appears in a helper but the helper implementation is not visible,
- the value may be validated elsewhere but the active control path is unclear.

Do not over-claim based only on:
- parameter naming,
- route naming,
- controller naming,
- a single object ID,
- a missing annotation,
- a hidden frontend field,
- a familiar anti-pattern.

---

# 7. Source Classification

## Confirmed source
Use `Confirmed source` when there is clear evidence that:
- the value origin is visible,
- the value reaches access-control-relevant code,
- and the trust boundary can be classified.

## Suspected source
Use `Suspected source` when:
- a value appears authorization-relevant,
- the source or downstream use is partially visible,
- but hidden framework, middleware, service, or policy behavior may change the classification.

## Not enough evidence
Use `Not enough evidence` when:
- the value origin cannot be determined,
- the downstream use cannot be determined,
- or critical framework behavior is not visible.

## Probably irrelevant
Use `Probably irrelevant` when:
- the value is visible,
- the downstream code is visible,
- and the value does not influence protected resources, protected functions, object access, tenant scope, or business actions.

---

# 8. What Good Evidence Looks Like

Strong source findings usually include:
- the exact entry point
- the source value name
- the source origin
- the trust boundary
- the first downstream access-control-relevant use
- whether the value is combined with identity, role, tenant, object, or workflow state

Good source points usually answer:
1. Where does the value enter?
2. Who controls the value?
3. What protected target or decision can the value influence?
4. What code proves that connection?
5. What should the next access-control audit verify?

---

# 9. Shared Follow-up Guidance

After source discovery, the access-control audit should verify:
- whether authentication is enforced before sensitive use,
- whether privileged functions check role or permission,
- whether object IDs are scoped to current user or tenant,
- whether tenant scope is derived from trusted context,
- whether business actions validate workflow state,
- whether equivalent routes and methods enforce the same controls,
- whether frontend-only restrictions have backend enforcement.

Avoid weak conclusions such as:
- source exists, therefore vulnerability exists,
- login exists, therefore source is safe,
- helper exists, therefore source is trusted,
- route is internal, therefore source is unreachable.

---

# 10. Shared Quick Checklist

Use this as a reminder, not as a substitute for reasoning.

- Where is the current user identity sourced?
- Are any user IDs supplied by the client?
- Are object IDs supplied by path, query, body, GraphQL, RPC, or batch input?
- Is tenant or organization scope client-controlled or server-trusted?
- Are role, permission, or admin flags ever accepted from request input?
- Are business actions or state transitions driven by request fields?
- Are there equivalent web, API, mobile, GraphQL, or job entry points?
- Is downstream use visible enough to classify the source?
- What access-control check should review this source next?
