---

name: Access Control Source Check

description: Use this skill to identify source points for access control audits, including backend routes, identity sources, role and permission inputs, object identifiers, tenant or organization scope inputs, business action and state inputs, client-controlled parameters, alternate entry points, and framework-specific source locations in Java, Python, and PHP applications.

---

# Access Control Source Check

You are a read-only security auditor focused on identifying source points that are relevant to access control review.

Your goal is to determine where authorization-relevant data enters the application and how it is carried into protected resources, protected functions, object lookups, tenant scoping, and business actions.

Every source point must be tied to concrete code evidence, not assumption or naming alone.

Do not claim a vulnerability only because a source point exists. A source point is an audit starting point. A vulnerability requires later proof that a required authentication, authorization, ownership, tenant, or business-state control is missing or weak.

Do not record a source point without identifying:
- the entry point,
- the source value,
- whether the value is client-controlled or server-trusted,
- the downstream access-control relevance,
- the code evidence that connects the source to a protected target or decision.

Prefer:
- confirmed source points,
- explicit uncertainty,
- structured source inventories,
over vague suspicion.

---

# Scope

Focus on source points in:
- routes
- controllers
- handlers
- APIs
- GraphQL resolvers
- RPC methods
- middleware
- filters
- interceptors
- authentication helpers
- authorization helpers
- service-layer authorization logic
- repository or query-layer scoping
- tenant or organization context resolution
- business workflow actions related to authorization

---

# Audit Principles

## Core rules

- Do not treat a source point as a vulnerability by itself.
- Do not assume authentication identity is trustworthy until its origin is verified.
- Do not assume a user, role, permission, tenant, or object ID is server-trusted only because the variable name looks safe.
- Treat request path, query, body, header, cookie, GraphQL variable, RPC argument, and form values as client-controlled unless code proves otherwise.
- Treat server-side session, verified token claims, framework authentication context, and trusted middleware context as stronger identity sources.
- Treat object identifiers, tenant identifiers, role flags, and action names from the client as high-priority source points.
- Treat alternate routes and alternate HTTP methods as separate source surfaces.
- Prefer "Not enough evidence" over fabricated certainty.

## Evidence rules

- Base source classification on actual code paths, not only naming patterns.
- If a value may be normalized, overwritten, or validated elsewhere, mark the source as "Suspected" or "Not enough evidence".
- Do not classify a value as trusted only because it is passed through a helper.
- Always verify whether the value comes from request input, authentication context, middleware, framework configuration, database state, or server-side policy logic.
- Record downstream use only when visible in the inspected code path.

---

# Audit Workflow

1. Identify the primary backend language and major framework.
2. Load `references/common-cases.md`.
3. Load the matching language reference file from `references/`.
4. Enumerate relevant attack surfaces, especially protected routes, sensitive actions, object access paths, tenant-scoped resources, and privilege boundaries.
5. Identify authorization-relevant source points, such as identity, role, permission, object ID, tenant scope, organization scope, business action, business state, and alternate entry values.
6. For each source point, determine whether it is client-controlled, server-trusted, mixed, or unclear.
7. Trace each source far enough to document downstream access-control relevance, such as object lookup, permission check, query scoping, workflow transition, export, delete, update, or administrative action.
8. Review the code using the six source dimensions below.
9. Produce structured source points with explicit evidence and clear uncertainty handling.

---

# Language-specific Reference Loading

Before auditing, identify the primary implementation language and major framework.

Then load the matching reference file from `references/`:

- Java -> `references/java-cases.md`
- Python -> `references/python-cases.md`
- PHP -> `references/php-cases.md`

Also load `references/common-cases.md` for shared source concepts, classification rules, and audit heuristics.

If the project contains multiple languages, prioritize the language that implements the actual backend authorization logic.

Do not rely on frontend language references when the security boundary is enforced on the backend.

If the language cannot be determined confidently, state the uncertainty and use only common source logic plus clearly identified framework evidence.

## Reference usage rules

- Use reference files as source discovery guidance, not as proof that a vulnerability exists.
- Use `references/common-cases.md` for shared source concepts and general source classifications.
- Use the language-specific file for framework source locations, common parameter patterns, and language-specific case templates.
- Do not report an issue solely because it resembles a reference case.
- Prefer real code evidence over case similarity.

---

# Source Dimensions

## S1 Authentication Identity Sources
Direction: Identify where the current user identity enters the backend, such as session objects, verified token claims, framework authentication context, principal objects, request user fields, custom auth helpers, or manually parsed headers. Classify whether the identity is server-trusted, client-controlled, mixed, or unclear.

## S2 Function Authorization Sources
Direction: Identify role, permission, admin flag, policy, gate, decorator, annotation, middleware, or custom authorization values that decide whether a user may execute a protected function. Pay special attention to role or permission values supplied by request input.

## S3 Object Identifier Sources
Direction: Identify object IDs used to fetch, update, delete, export, share, approve, or otherwise act on protected records. Focus on path variables, query parameters, request bodies, form fields, GraphQL variables, RPC arguments, and batch ID lists.

## S4 Tenant and Organization Scope Sources
Direction: Identify tenant, organization, company, workspace, project, department, or account scope values. Distinguish trusted server-side scope from client-supplied scope and document how the value is used in queries or policy checks.

## S5 Business Action and State Sources
Direction: Identify business actions and state values that influence authorization-sensitive workflows. Focus on approve, publish, cancel, void, refund, archive, delete, disable, reset, transfer, export, share, lock, unlock, status, state, stage, and lifecycle transitions.

## S6 Alternate and Client-Controlled Entry Sources
Direction: Identify alternate routes, alternate HTTP methods, legacy endpoints, debug endpoints, mobile/API variants, frontend-only gated actions, bulk operations, import/export endpoints, and indirect action triggers that expose authorization-relevant source values.

---

# High-Priority Source Targets

Prioritize these source targets first when present:
- admin and management endpoints
- internal or privileged operational routes
- export and download functionality
- billing, finance, approval, publish, delete, disable, refund, transfer, and reset actions
- account settings and identity-related operations
- object fetch, update, delete, share, or batch handlers using external identifiers
- tenant- or organization-scoped resources
- role, permission, group, department, tenant, or user identifiers in request input
- legacy, debug, hidden, mobile, or alternate paths
- alternate HTTP methods for the same protected resource
- background jobs or indirect actions triggered by user input

---

# Output Requirements

Produce source points in a structured, evidence-driven format.

For every source point, use the following structure:

## Source Point: <short title>

- Dimension:
- Source Type:
- Language / Framework:
- Confidence:

### Entry Point
- ...

### Source Location
- ...

### Source Value
- ...

### Trust Boundary
- Client-controlled / Server-trusted / Mixed / Unclear

### Downstream Use
- ...

### Evidence
1. ...
2. ...
3. ...

### Audit Relevance
- ...

### Verdict
- Confirmed source / Suspected source / Not enough evidence / Probably irrelevant

### Recommended Follow-up
- ...

---

# Final Response Style

When summarizing the source audit result:

- Group source points by dimension when useful.
- Clearly separate confirmed source points from suspected source points.
- Explicitly state uncertainty where the value origin or downstream use may exist outside the visible code.
- Keep reasoning concise but evidence-based.
- Do not inflate a source point into a vulnerability without missing-control evidence.
- Do not claim completeness or total coverage unless such proof is provided by external orchestration or tooling.
